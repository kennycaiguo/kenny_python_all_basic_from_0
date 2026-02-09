# 1.把chapter14项目复制一份，改名：chapter15-add_review-and-more-relation,然后用pycharm打开<br>
<img width="512" height="674" alt="image" src="https://github.com/user-attachments/assets/47e4226c-9cef-48a1-9c34-6c4d6d74135a" /><br>
# 2.这一节我们来学习更多的关系，并且我们会创建书籍的评论功能<br>
## 2.1>在我们继续之前，我们需要修复一个错误：就是现在当我们使用/signup路由创建一个新用户，在数据库里面新增用户成功，都是服务器会出错，错误是缺少greenlet。解决办法，我们在创建用户的时候，
## 不应该返回一个有书籍列表的模型，所以我们需要撤销对UserModel类的更改，然后新建一个UserBooksModel从UserModel里面继承<br>
<img width="1043" height="521" alt="image" src="https://github.com/user-attachments/assets/0a8fe1ec-d03f-4994-8b12-f4f2fdea25be" /><br>

## 2.2>然后我们修改auth/routes.py，把/me路由对应的路由函数的返回类型改为UserBooksModel<br>
<img width="1229" height="390" alt="image" src="https://github.com/user-attachments/assets/f54dee04-e660-4405-922c-485be04f1124" /><br>
### 然后我们重新发送请求，发现用户已经创建成功<br>
<img width="1437" height="856" alt="image" src="https://github.com/user-attachments/assets/4434e0be-6c78-4c4f-9da7-da36d2cb56c2" /><br>
## 2.3>然后我们用这个账户来登录，没有问题<br>
<img width="1459" height="824" alt="image" src="https://github.com/user-attachments/assets/16d14b25-b7f3-4db1-b239-40f898504a01" /><br>
## 2.4>然后我们用这个账户来提交2本书，创建成功<br>
<img width="1445" height="809" alt="image" src="https://github.com/user-attachments/assets/b529ff46-13ad-452c-a019-b7b6e2dfc3dc" /><br>
<img width="1540" height="948" alt="image" src="https://github.com/user-attachments/assets/f3c2bbd3-42b9-4d8f-9c4b-fee55388ad6e" /><br>
## 2.5>然后我们来查看当前用户信息，可以得到用户的信息包括它提交的书籍列表<br>
<img width="1481" height="936" alt="image" src="https://github.com/user-attachments/assets/b089b0a9-9d22-4300-a883-4a1e4c1f768e" /><br>
# 3.今天，我们需要修改一下数据库的关系模型，添加评论功能，关系模型如图<br>
<img width="926" height="638" alt="image" src="https://github.com/user-attachments/assets/c7b33ebf-6699-4a7c-a691-dbb76c298508" /><br>
## 从这个模型我们可以看出： 一个用户可以创建多本书籍，一本书籍也可以有多个用户发表评论，一个用户也可以对多本书发表评论，什么他们都是一对多的关系，书籍和评论，用户和评论多少一对多的关系<br>
## 3.1>首先，我们需要把所有的模型放到一个任何模块都能够访问到的地方，这样子可以减少循环导入的问题。我们在db包里面新建一个models.py模块，然后把所有模块的代码都写在这里，代码如下
```
from datetime import datetime, date
from typing import List,Optional
from sqlmodel import SQLModel, Field, Column, Relationship
import sqlalchemy.dialects.postgresql as pg
import uuid
from src.db import models


# 转移自auth/models.py
class User(SQLModel, table=True):
    __tablename__ = "users"

    uid: uuid.UUID = Field(
        sa_column=Column(
            pg.UUID,
            primary_key=True,
            unique=True,
            nullable=False,
            default=uuid.uuid4,
            info={"description": "Unique identifier for the user account"},
        )
    )

    username: str
    first_name: str = Field(nullable=True)
    last_name: str = Field(nullable=True)
    # let us add this line
    role: str = Field(
        sa_column=Column(pg.VARCHAR, nullable=False, server_default="user")
    )

    is_verified: bool = False
    email: str
    password_hash: str = Field(exclude=True)  # 这个字段需要隐藏，不要让别人知道你的密码
    # add books
    books: List["models.Book"] = Relationship(back_populates="user", sa_relationship_kwargs={
        'lazy': 'selectin'
    })
    created_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now))
    update_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now))

    def __repr__(self) -> str:
        return f"<User {self.username}>"


# 转移自books/models.py
class Book(SQLModel, table=True):
    __tablename__ = "books"

    uid: uuid.UUID = Field(
        sa_column=Column(
            pg.UUID,
            nullable=False,
            primary_key=True,
            unique=True,
            default=uuid.uuid4
        )
    )
    title: str
    author: str
    publisher: str
    published_date: date
    page_count: int
    language: str
    # add foreign key
    user_id: Optional[uuid.UUID] = Field(default=None, foreign_key="users.uid")
    # add user field
    user: Optional["models.User"] = Relationship(back_populates="books")
    created_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now))
    update_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now))

    def __repr__(self):
        return f"<Book: {self.title}>"

```
## 3.2>然后我们把auth/models.py和books/models.py删除，在用到他们的地方把导入路径改为from src.db.models import XX的格式<br>
<img width="999" height="312" alt="image" src="https://github.com/user-attachments/assets/86b024e2-3f62-4a2e-a100-4755d4176c67" /><br>
<img width="1126" height="615" alt="image" src="https://github.com/user-attachments/assets/b3e3eb2d-f568-419e-bb08-a87a3745e6cc" /><br>
<img width="1125" height="564" alt="image" src="https://github.com/user-attachments/assets/29283af6-0124-483e-a3eb-edbdf40ac254" /><br>













