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
<img width="1406" height="807" alt="image" src="https://github.com/user-attachments/assets/3379d85e-ee01-4fa8-aaa6-230636c43554" /><br>
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
# 注意：下面的内容老师的视频没有讲，我参考老师的GitHub page来完成
## 6.2>然后我们回到reviews/service.py,添加获取一条评论，获取所有评论和删除一条评论的函数
```
from datetime import datetime
from sqlmodel.ext.asyncio.session import AsyncSession
from sqlmodel import select, desc
from src.books.schemas import BookCreateModel, BookUpdateModel
from src.db.models import Review
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
            new_review = Review(**review_data_dict)
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

    # 获取一条评论
    async def get_review(self,review_id:str,session:AsyncSession):
        stmt = select(Review).where(Review.uid == review_id)
        result = await session.exec(stmt)
        return result.first()

    # 获取所有评论
    async def get_reviews(self,session:AsyncSession):
        stmt = select(Review).order_by(desc(Review.created_at))
        result = await session.exec(stmt)

        return result.all()

    # 删除一条评论
    async def delete_review_to_from_book(self, review_uid: str, email: str, session: AsyncSession):
        user = await user_service.get_user_by_email(email,session)
        review = await self.get_review(review_uid,session)
        if not review or (review.user is not user):
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail="Can't delete this review..."
            )
        session.delete(review)
        await session.commit()
        return review
```
## 6.3>然后我们来完成reviews/routes.py里面的所有路由
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
admin_checker = Depends(RoleChecker(allowed_roles=['admin']))
user_checker = Depends(RoleChecker(allowed_roles=['user', 'admin']))


@review_router.post("/book/{book_id}", dependencies=[user_checker])
async def add_review_to_book(book_id: str, review_data: ReviewCreateModel,
                             current_user: User = Depends(get_current_user),
                             session: AsyncSession = Depends(get_session)):
    new_review = await review_service.add_review_to_book(current_user.email, book_id, review_data, session)
    return new_review


@review_router.get("/", dependencies=[admin_checker])
async def get_reviews(session: AsyncSession = Depends(get_session)):
    reviews = await review_service.get_all_reviews(session)

    return reviews


@review_router.get("/{review_id}", dependencies=[user_checker])
async def get_review(review_id: str, session: AsyncSession = Depends(get_session)):
    review = await review_service.get_review(review_id, session)
    if not review:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Review not found"
        )
    return review


@review_router.delete("/{review_id}", dependencies=[user_checker])
async def delete_review(review_id:str,current_user:User=Depends(get_current_user),session: AsyncSession = Depends(get_session)):
    await review_service.delete_review_to_from_book(review_id,current_user.email,session)
    return {
        "message":"review deleted..."
    }

```
## 6.4>在测试中，我们发现无法创建管理员账户，我们来到auth/schemas.py，给UserModel和UserCreateModel添加role字段
```
import uuid
from datetime import datetime
from typing import List
from pydantic import BaseModel
from sqlmodel import Field
from src.books.schemas import Book
from src.reviews.schemas import ReviewModel


class UserCreateModel(BaseModel):
    username: str = Field(max_length=50)  # 注意：用户名不一定是first_name + last_name,他们可以不一样的。
    first_name: str = Field(max_length=25)
    last_name: str = Field(max_length=25)
    role:str 
    email: str = Field(max_length=40)
    password: str = Field(min_length=6)


class UserModel(BaseModel):
    uid: uuid.UUID
    username: str
    first_name: str
    last_name: str
    role:str
    is_verified: bool
    email: str
    password_hash: str = Field(exclude=True)
    created_at: datetime
    update_at: datetime


class UserBooksModel(UserModel):
    books: List[Book]
    reviews: List[ReviewModel]


class UserLoginModel(BaseModel):
    email: str = Field(max_length=40)
    password: str = Field(min_length=6)

```
## 6.5> 然后我们需要修改auth/sercice.py里面的UserService类的创建用户代码
```
from src.db.models import User
from sqlmodel.ext.asyncio.session import AsyncSession
from sqlmodel import select

from .schemas import UserCreateModel
from .utils import generate_passwd_hash


class UserService:
    async def get_user_by_email(self, email: str, session: AsyncSession):
        stmt = select(User).where(User.email == email)
        result = await session.exec(stmt)
        return result.first()

    async def user_exists(self, email, session: AsyncSession):
        user = await self.get_user_by_email(email, session)
        return True if user is not None else False

    async def create_user(self, user_data: UserCreateModel, sesson: AsyncSession):
        user_data_dict = user_data.model_dump()
        new_user = User(
            **user_data_dict
        )
        # print(user_data_dict["role"])
        new_user.password_hash = generate_passwd_hash(user_data_dict["password"])
        # 添加设置用户角色的代码
        role = user_data_dict.get("role")
        new_user.role = role
        sesson.add(new_user)
        await sesson.commit()
        return new_user

```
## 用postman测试通过，注意：只有创建评论的人才能够删除评论，哪怕是管理员也不能删除别人的评论<br>
<img width="1436" height="859" alt="image" src="https://github.com/user-attachments/assets/0ecc0657-203a-4b94-873a-dc97f451b5cf" /><br>
<img width="1335" height="749" alt="image" src="https://github.com/user-attachments/assets/b9b968db-bd02-4f1e-9610-487b08061e83" /><br>
<img width="1517" height="761" alt="image" src="https://github.com/user-attachments/assets/7be77611-3b1a-4634-9fbe-34f290929035" /><br>
# 7.然后我们来把书籍和标签关联起来，他们的关系如图,所谓的tag其实就是类别category，我们可以通过tag来给书籍分类<br>
<img width="878" height="542" alt="image" src="https://github.com/user-attachments/assets/fb094047-5337-4a24-9461-1af422790074" /><br>
## 这里我们创建另外2张表格tags和book_tags,book_tags起到一个桥梁的作用，可以实现books和tags的多对多关系<br>
## 7.1.我们需要在src/db/models.py里面创建2个类Tag和BookTag<br>
<img width="1160" height="615" alt="image" src="https://github.com/user-attachments/assets/dd66a810-4a6e-4d7c-bab5-663743ce1cc1" /><br>
## 7.2>然后用alembic创建一个修订，命令如下：
```
alembic revision --autogenerate -m "add book tags table"
```
## 7.3>然后执行upgrade命令迁移到数据库
```
alembic upgrade head
```
<img width="1472" height="527" alt="image" src="https://github.com/user-attachments/assets/46293aee-b421-47f8-9ca2-c8ae9c5f5d1b" /><br>
## 7.4>到数据库查看，发现tag表和booktag表格创建成功了<br>
<img width="308" height="459" alt="image" src="https://github.com/user-attachments/assets/77c4b0d8-57b1-417c-8a54-4ea98c7fd60c" /><br>
# 8.创建tag相关的路由，先在src包里面新建一个tags包，然后在里面新建一个schemas.py,内容如下
```
import uuid
from datetime import datetime
from typing import List

from pydantic import BaseModel


class TagModel(BaseModel):
    uid: uuid.UUID
    name: str
    created_at: datetime


class TagCreateModel(BaseModel):
    name: str


class TagAddModel(BaseModel):
    tags: List[TagCreateModel]
```
## 8.1>然后我们需要给db/models.py里面的Book类添加一个tags关系属性<br>
<img width="1272" height="713" alt="image" src="https://github.com/user-attachments/assets/d0a25f38-3f2e-41d1-8866-268e57cfd207" /><br>
### 有一点需要注意，我们的模型之间存在依赖，所以需要根据需要调整这些类的定义顺序，调整后models.py的内容如下<br>
```
from datetime import datetime, date
from typing import List, Optional
from sqlmodel import SQLModel, Field, Column, Relationship
import sqlalchemy.dialects.postgresql as pg
import uuid


class BookTag(SQLModel, table=True):
    book_id: uuid.UUID = Field(default=None, foreign_key="books.uid", primary_key=True)
    tag_id: uuid.UUID = Field(default=None, foreign_key="tags.uid", primary_key=True)


class Tag(SQLModel, table=True):
    __tablename__ = "tags"
    uid: uuid.UUID = Field(
        sa_column=Column(pg.UUID, nullable=False, primary_key=True, default=uuid.uuid4)
    )
    name: str = Field(sa_column=Column(pg.VARCHAR, nullable=False))
    created_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now))
    books: List["Book"] = Relationship(
        link_model=BookTag,
        back_populates="tags",
        sa_relationship_kwargs={"lazy": "selectin"},
    )

    def __repr__(self) -> str:
        return f"<Tag {self.name}>"


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
    role: str = Field(sa_column=Column(pg.VARCHAR, nullable=False, server_default="user"))
    is_verified: bool = False
    email: str
    password_hash: str = Field(exclude=True)  # 这个字段需要隐藏，不要让别人知道你的密码
    # add books
    books: List["Book"] = Relationship(back_populates="user", sa_relationship_kwargs={'lazy': 'selectin'})
    # add reviews
    reviews: List["Review"] = Relationship(back_populates="user", sa_relationship_kwargs={'lazy': 'selectin'})
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
    # add reviews
    reviews: List["Review"] = Relationship(back_populates="book", sa_relationship_kwargs={'lazy': 'selectin'})
    # add tags
    tags: List["Tag"] = Relationship(
        link_model=BookTag,
        back_populates="books",
        sa_relationship_kwargs={"lazy": "selectin"},
    )
    created_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now))
    update_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now))

    def __repr__(self):
        return f"<Book: {self.title}>"


#  review
class Review(SQLModel, table=True):
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
    # add user
    user: Optional["User"] = Relationship(back_populates="reviews")
    # add book
    book: Optional["Book"] = Relationship(back_populates="reviews")
    created_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now))
    update_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now))

    def __repr__(self):
        return f"<Review for: book {self.book_id}> by user {self.user_id}"

```
# 8.2 用alembic创建一个修订，命令如下
```
alembic revision --autogenerate -m "add tags field to Book model"
```
<img width="1456" height="328" alt="image" src="https://github.com/user-attachments/assets/56b02e8c-b377-45ce-a4be-1c244820fd82" /><br>

### 然后用下面的命令更新到数据库
```
alembic upgrade head
```
<img width="1239" height="144" alt="image" src="https://github.com/user-attachments/assets/992ce123-e700-4709-98be-2cee2e107b0f" /><br>

## 8.3>然后，我们需要修改books/schemas.py,也给他添加一个tags属性
<img width="1049" height="729" alt="image" src="https://github.com/user-attachments/assets/58bba414-d38d-4a8d-b2c9-de8a02c95e12" />


## 8.4>然后我们需要在tags包里面创建一个services.py，在里面创建一个TagService类，代码如下
```
from fastapi import status
from fastapi.exceptions import HTTPException
from sqlmodel import desc, select
from sqlmodel.ext.asyncio.session import AsyncSession

from src.books.service import BookService
from src.db.models import Tag

from .schemas import TagAddModel, TagCreateModel

book_service = BookService()


server_error = HTTPException(
    status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail="Something went wrong"
)


class TagService:
    async def get_all_tags(self,session:AsyncSession):
        stmt = select(Tag).order_by(desc(Tag.created_at))
        result = await session.exec(stmt)
        return result.all()

    async def add_tags_to_book(self,book_id:str,tag_data:TagAddModel,session:AsyncSession):
        book = await book_service.get_book(book_id,session)
        if not book:
            raise HTTPException(status_code=404, detail="Book not found")
        # 防止重复创建
        for tag_item in tag_data.tags:
            result = await session.exec(
                select(Tag).where(Tag.name == tag_item.name)
            )

            tag = result.one_or_none()
            if not tag:
                tag = Tag(name=tag_item.name)
            book.tags.append(tag)
        session.add(book)
        await session.commit()
        await session.refresh(book)
        return book

    async def get_tag_by_uid(self, tag_uid: str, session: AsyncSession):
        stmt = select(Tag).where(Tag.uid==tag_uid)
        result = await session.exec(stmt)

        return result.first()

    async def add_tag(self, tag_data: TagCreateModel, session: AsyncSession):
        stmt = select(Tag).where(Tag.name==tag_data.name)
        result = await session.exec(stmt)
        tag = result.first()
        if tag:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN, detail="Tag exists"
            )
        new_tag = Tag(name=tag_data.name)
        session.add(new_tag)
        await session.commit()
        return new_tag

    async def update_tag(
            self, tag_uid, tag_update_data: TagCreateModel, session: AsyncSession
    ):
        """Update a tag"""

        tag = await self.get_tag_by_uid(tag_uid, session)

        update_data_dict = tag_update_data.model_dump()

        for k, v in update_data_dict.items():
            setattr(tag, k, v)

            await session.commit()

            await session.refresh(tag)

        return tag

    async def delete_tag(self, tag_uid: str, session: AsyncSession):
        """Delete a tag"""

        tag = await self.get_tag_by_uid(tag_uid, session)

        if not tag:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND, detail="Tag does not exist"
            )

        await session.delete(tag)

        await session.commit()
```
# 9.创建tag相关的路由，
## 9.1我们在tags包下面新建一个routes.py,内容如下
```
from fastapi import APIRouter, status, Depends
from sqlmodel.ext.asyncio.session import AsyncSession
from fastapi.exceptions import HTTPException
from typing import List
from .schemas import TagCreateModel, TagModel
from .service import TagService
from ..auth.dependencies import RoleChecker
from ..books.schemas import Book
from ..db.main import get_session

tag_router = APIRouter()
tag_service = TagService()
user_ckecker = Depends(RoleChecker(allowed_roles=["user", "admin"]))
admin_checker = Depends(RoleChecker(allowed_roles=["admin"]))


@tag_router.get("/", response_model=List[TagModel], dependencies=[user_ckecker])
async def get_all_tags(session: AsyncSession = Depends(get_session)):
    tags = await tag_service.get_all_tags(session)
    return tags


@tag_router.get("/{tag_id}")
async def get_a_tag(tag_id: str, session: AsyncSession = Depends(get_session)):
    tag = await tag_service.get_tag_by_uid(tag_id, session)
    if not tag:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Tag not found..."
        )
    return tag


@tag_router.post("/", response_model=TagModel, dependencies=[user_ckecker])
async def create_a_tag(tag_data: TagCreateModel, session: AsyncSession = Depends(get_session)):
    tag = await tag_service.add_tag(tag_data, session)

    return tag


@tag_router.post("/book/{book_id}/tags", response_model=Book, dependencies=[user_ckecker])
async def add_tags_to_a_book(book_id: str, tag_data: TagCreateModel, session: AsyncSession = Depends(get_session)):
    book_with_tag = await tag_service.add_tags_to_book(book_id, tag_data, session)

    return book_with_tag


@tag_router.put("/{tag_id}", dependencies=[user_ckecker])
async def update_a_tag(tag_id: str, tag_update_data: TagCreateModel, session: AsyncSession = Depends(get_session)):
    tag_updated = await tag_service.update_tag(tag_id, tag_update_data, session)

    return tag_updated


@tag_router.delete("/{tag_id}", dependencies=[user_ckecker])
async def delete_a_tag(tag_id: str, session: AsyncSession = Depends(get_session)):
    deleted_tag = await tag_service.delete_tag(tag_id, session)

    return {
        "deleted" : deleted_tag
    }

```
## 9.2然后我们需要在src/__init__.py里面注册我们的tag相关路由<br>
<img width="1208" height="611" alt="image" src="https://github.com/user-attachments/assets/9a206317-f0c1-4cb3-bc44-57eaa34a67ca" /><br>
### 然后，我们就可以在postman里面测试路由了<br>
### 添加标签成功<br>
<img width="1429" height="732" alt="image" src="https://github.com/user-attachments/assets/71c6c5b0-1ab4-425d-b8fe-54fab7055c11" /><br>
### 查询所有标签成功<br>
<img width="1421" height="863" alt="image" src="https://github.com/user-attachments/assets/240a1cd4-a5c7-49e1-be98-22cd016d259c" /><br>
### 工具id来查找tag成功<br>
<img width="1490" height="795" alt="image" src="https://github.com/user-attachments/assets/fe1c0ffc-e284-4369-ba7b-80eda0093bfb" /><br>
### 更新tag成功<br>
<img width="1456" height="837" alt="image" src="https://github.com/user-attachments/assets/8d9f4fe6-4fd5-4445-a6b4-cfe6105b0615" /><br>
### 删除tag也成功了<br>
<img width="1434" height="796" alt="image" src="https://github.com/user-attachments/assets/fa01d576-0516-4273-a925-2bb85b61a96d" /><br>
# 至此，第15节课学习完毕





