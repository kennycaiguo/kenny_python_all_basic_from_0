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
# 4.评论相关路由开发<br>
## 4.1>首先，在db/models.py里面新建一个Reviews类
```
from datetime import datetime, date
from typing import List, Optional
from sqlmodel import SQLModel, Field, Column, Relationship
import sqlalchemy.dialects.postgresql as pg
import uuid


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
    books: List["Book"] = Relationship(back_populates="user", sa_relationship_kwargs={
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
    user: Optional["User"] = Relationship(back_populates="books")
    created_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now))
    update_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now))

    def __repr__(self):
        return f"<Book: {self.title}>"


#  review
class Reviews(SQLModel, table=True):
    __tablename__ = "reviews"

    uid: uuid.UUID = Field(
        sa_column=Column(
            pg.UUID,
            nullable=False,
            primary_key=True,
            unique=True,
            default=uuid.uuid4
        )
    )
    user_id: Optional[uuid.UUID] = Field(default=None, foreign_key="users.uid")
    book_id: Optional[uuid.UUID] = Field(default=None, foreign_key="books.uid")
    review_text: str
    rating: int = Field(
        lt=5
    )
    user: Optional["User"] = Relationship(back_populates="reviews")
    created_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now))
    update_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now))

    def __repr__(self):
        return f"<Review for: book {self.book_id}> by user {self.user_id}"

```
## 4.2>用alembic用这个模型来生成数据表，先创建一个修订，如下命令： 
```
alembic revision --autogenerate -m "create reviews table"
```
<img width="1414" height="335" alt="image" src="https://github.com/user-attachments/assets/8df66875-5813-471f-acac-fe19897b460a" /><br>
### 然后需要用这个修订做数据库迁移，创建一个reviews数据表，使用命令：
```
alembic upgrade head
```
<img width="1094" height="188" alt="image" src="https://github.com/user-attachments/assets/27f6ba8d-acee-495c-b264-20f90aedbd2b" /><br>
### 用navicat17窗口数据库，发现reviews已经创建了<br>
<img width="934" height="273" alt="image" src="https://github.com/user-attachments/assets/023b3728-4da7-4269-867f-b3dbebbb85e9" /><br>
## 4.3>我们已经创建了评论表格并且把它和书籍和用户表用属性关联了，现在我们需要修改Uer类，创建它与评论表的属性关联<br>
<img width="1440" height="803" alt="image" src="https://github.com/user-attachments/assets/04094a8e-f6d6-450b-a5b7-ee16d3afc8a0" /><br>
## 4.4>我们需要创建Book类和Reviews类的属性关联，先给Reviews类添加一个book属性<br>
<img width="1329" height="752" alt="image" src="https://github.com/user-attachments/assets/b84a41a5-a50f-4ccc-ad97-0bac1a365b66" /><br>
## 4.4>现在让我们来修改Books类，创建它与评论表的属性关联<br>
<img width="1406" height="807" alt="image" src="https://github.com/user-attachments/assets/3379d85e-ee01-4fa8-aaa6-230636c43554" />
## 4.5>然后我们需要在src里面创建一个reviws包，然后在里面新建一个service.py文件和一个routes.py文件，先写一些最基本的框架代码<br>
<img width="796" height="419" alt="image" src="https://github.com/user-attachments/assets/f3cc5602-bdca-4db2-a656-46896fd5aa08" /><br>
<img width="1020" height="425" alt="image" src="https://github.com/user-attachments/assets/2f6c146b-902a-475b-b3ae-a24cc055484b" /><br>
## 4.6>然后我们在src/__init__.py里面注册评论路由<br>
<img width="1292" height="787" alt="image" src="https://github.com/user-attachments/assets/2105d7f0-f264-4d9a-be35-092384cd595b" /><br>
## 4.7>回到reviews/service.py,创建添加评论的路由的框架代码，注意：此次需要一个ReviewCreateModel类我们还没有创建<br>
<img width="1362" height="539" alt="image" src="https://github.com/user-attachments/assets/437a04df-95c3-424b-afe6-26df2b32fa2d" /><br>
## 5.0>然后我们需要在reviews包里面创建一个schemas.py，在里面新建ReviewModel和ReviewCreateModel类
```
from typing import Optional
from pydantic import BaseModel
from datetime import datetime, date
import uuid
from sqlmodel import Field


class ReviewModel(BaseModel):
    uid: uuid.UUID
    user_id: Optional[uuid.UUID]
    book_id: Optional[uuid.UUID]
    review_text: str
    rating: int=Field(lt=5)
    created_at: datetime
    update_at: datetime


class ReviewCreateModel(BaseModel):
    review_text: str
    rating: int=Field(lt=5)


```
## 5.1>然后我们回到reviews/services.py,继续完成我们的创建评论函数
```
from datetime import datetime
from sqlmodel.ext.asyncio.session import AsyncSession
from sqlmodel import select, desc
from src.books.schemas import BookCreateModel, BookUpdateModel
from src.db.models import Reviews
from src.auth.service import UserService
from src.books.service import BookService
from .schemas import ReviewCreateModel, ReviewModel
from fastapi.exceptions import HTTPException
from fastapi import status
import logging

user_service = UserService()
book_service = BookService()


class ReviewService:
    async def add_review_to_book(self, email: str, book_id: str, review_data: ReviewCreateModel, session: AsyncSession):
        try:
            book = await book_service.get_book(book_id, session)
            user = await user_service.get_user_by_email(email, session)
            review_data_dict = review_data.model_dump()
            new_review = Reviews(**review_data_dict)
            if not book:
                raise HTTPException(
                    status_code=status.HTTP_404_NOT_FOUND,
                    detail="Book not found"
                )
            if not user:
                raise HTTPException(
                    status_code=status.HTTP_404_NOT_FOUND,
                    detail="User not found"
                )
            new_review.user = user
            new_review.book = book
            # 保存到数据库
            session.add(new_review)
            await session.commit()

            return new_review
        except Exception as e:
            logging.exception(e) # output error log
            raise HTTPException(
                status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
                detail="Server error adding review..."
            )

```
## 5.2>然后我们来到reviews/routes.py,在这里添加创建评论的路由<br>
```
from fastapi import FastAPI, status, APIRouter, Depends
from sqlmodel.ext.asyncio.session import AsyncSession
from fastapi.exceptions import HTTPException
from src.db.main import get_session
from .schemas import ReviewCreateModel
from .service import ReviewService
from typing import List
from src.books.schemas import Book, BookUpdateModel, BookCreateModel
from src.auth.dependencies import AccessTokenBearer, RoleChecker, get_current_user
from ..db.models import User

review_router = APIRouter()
review_service = ReviewService()


@review_router.post("/book/{book_id}")
async def add_review_to_book(book_id: str,review_data: ReviewCreateModel,current_user:User=Depends(get_current_user) ,
                                session: AsyncSession = Depends(get_session)):

    new_review = await review_service.add_review_to_book(current_user.email, book_id, review_data, session)
    return new_review

```
## 然后我们可以在postman中测试一下，成功创建评论（注意：必须保证访问令牌有效）<br>
<img width="1435" height="766" alt="image" src="https://github.com/user-attachments/assets/6bb26e1c-40a9-4ff9-8834-8278557c91f9" /><br>
## 6.然后我们需要保证每一本书返回时都包含它的所有评论或者说每一本书的详细信息中都包含他的所有评论，我们打开books/schemas.py,新建一个BookDetailModel类,从Book继承。<br>
<img width="1216" height="856" alt="image" src="https://github.com/user-attachments/assets/12eb9f10-efe8-433b-8d8e-3d8ec7e5e798" /><br>
## 6.1>然后我们需要修改books/routes.py里面的根据id来查找书籍的路由函数，把他的返回模型改为BookDetailModel,这样了就可以看到评论了
<img width="1206" height="819" alt="image" src="https://github.com/user-attachments/assets/f3f81440-0689-4b56-9908-3b6076cd26c4" /><br>
### 用postman测试一下，成功<br>
<img width="1528" height="918" alt="image" src="https://github.com/user-attachments/assets/b093560a-9592-443b-ad1f-47e80e4ed8b3" /><br>
## 7.我们也可以修改一下auth/schemas.py里面的UserBookModel，添加一个评论集合<br>
<img width="1112" height="691" alt="image" src="https://github.com/user-attachments/assets/e39549eb-ce94-4752-a859-40858796eb24" /><br>
### 用postman测试一下，成功<br>
<img width="1500" height="886" alt="image" src="https://github.com/user-attachments/assets/700efb02-b218-4936-8d1f-19803c3b6f93" /><br>
## 这样，我们获取当前用户的时候，就可以看到他添加了多少本书，并且发表了多少评论<br>










