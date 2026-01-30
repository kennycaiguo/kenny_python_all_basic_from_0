# 把chapter5复制一份，改名chapter6-SQLModel-crud,然后用pycharm打开，然后在终端运行：fastapi dev src/ ，此时项目是可以正常运行，并且可以连接到数据库的。<br>
<img width="750" height="889" alt="image" src="https://github.com/user-attachments/assets/7ee481b3-ff91-45e2-b875-6243300dc833" /><br>
# 这一节我们需要做的就是实现堆数据库的增删改查<br>
## 1.首先我们需要修改一些错误
```
from sqlmodel import SQLModel, Field, Column
import sqlalchemy.dialects.postgresql as pg
from datetime import datetime
import uuid


class Book(SQLModel, table=True):
    __tablename__ = "books"

    uid: uuid.UUID = Field(
        sa_column=Column(
            pg.UUID,
            nullable=False,
            primary_key=True,
            default=uuid.uuid4
        )
    )
    title: str
    author: str
    publisher: str
    published_date: str
    page_count: int
    language: str
    created_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now()))
    updated_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now()))

    def __repr__(self):
        return f"<Book: {self.title}>"

# class BookUpdateModel():
#     title: str
#     author: str
#     publisher: str
#     page_count: int
#     language: str

```
## 2.然后我们需要进入postgres数据库控制台或者管理工具，把我们的books表格删除<br>
<img width="1202" height="688" alt="image" src="https://github.com/user-attachments/assets/cc896256-f434-4bea-b5a4-41a2f0791679" /><br>
## 3.然后重新启动应用程序，又会利用我们修改后的模型创建一个books表格<br>
<img width="1379" height="834" alt="image" src="https://github.com/user-attachments/assets/ba21ff59-466a-433a-bbd4-07218a7279b7" /><br>
## 4.然后我们需要创建一个服务类，在这个类里面实现数据库的增删改查<br>
### 4.0在books包里面的schemas.py里面把Book类修改一下，代码如下<br>
```
from pydantic import BaseModel
from datetime import datetime
import uuid


class Book(BaseModel):
    uid: uuid.UUID
    title: str
    author: str
    publisher: str
    published_date: str
    page_count: int
    language: str
    created_at: datetime
    update_at: datetime
```
### 4.1然后创建一个BookCreateModel类，代码如下<br>
```
from pydantic import BaseModel
from datetime import datetime
import uuid


class Book(BaseModel):
    uid: uuid.UUID
    title: str
    author: str
    publisher: str
    published_date: str
    page_count: int
    language: str
    created_at: datetime
    update_at: datetime


class BookCreateModel(BaseModel):
    title: str
    author: str
    publisher: str
    published_date: str
    page_count: int
    language: str
```
### 4.1.2 然后我们把BookUpdateModel类型添加上
```
from pydantic import BaseModel
from datetime import datetime
import uuid


class Book(BaseModel):
    uid: uuid.UUID
    title: str
    author: str
    publisher: str
    published_date: str
    page_count: int
    language: str
    created_at: datetime
    update_at: datetime


class BookCreateModel(BaseModel):
    title: str
    author: str
    publisher: str
    published_date: str
    page_count: int
    language: str


class BookUpdateModel(BaseModel):
    title: str
    author: str
    publisher: str
    page_count: int
    language: str

```

### 4.2然后我们需要在models.py里面把Book类的出版日期的类型修改一下(注意，我们有2个Book类，一个是数据库模型的Book类，另外一个是Python的Book类，他们的一些数据类型是不一样的，但是可以转化)<br>
```
from sqlmodel import SQLModel, Field, Column
import sqlalchemy.dialects.postgresql as pg
from datetime import datetime, date
import uuid


class Book(SQLModel, table=True):
    __tablename__ = "books"

    uid: uuid.UUID = Field(
        sa_column=Column(
            pg.UUID,
            nullable=False,
            primary_key=True,
            default=uuid.uuid4
        )
    )
    title: str
    author: str
    publisher: str
    published_date: date
    page_count: int
    language: str
    created_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now()))
    updated_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now()))

    def __repr__(self):
        return f"<Book: {self.title}>"


```
#### 注意：每一次修改了模型，我们需要把数据表删除，然后重新启动服务器来生成新的表。
### 5.在books包里面新建一个service.py文件，然后在这里新建一个BookService类，代码如下<br>
```
from fastapi import status, HTTPException
from sqlmodel.ext.asyncio.session import AsyncSession
from sqlmodel import select, desc, update
from src.books.schemas import BookCreateModel, BookUpdateModel
from .models import Book


class BookService:
    async def get_all_books(self, session: AsyncSession):
        stmt = select(Book).order_by(desc(Book.created_at))
        ret = await session.exec(stmt)
        return ret.all()  # 返回所有数据

    async def get_book(self, book_uid: str, session: AsyncSession):
        stmt = select(Book).where(Book.uid == book_uid)
        ret = await session.exec(stmt)
        book = ret.first()  # 查找一个的方法
        return book if book is not None else None

    async def create_book(self, book_data: BookCreateModel, session: AsyncSession):
        book_data_dict = book_data.model_dump()
        new_book = Book(
            **book_data_dict
        )
        session.add(new_book)  # 添加数据使用session.add方法
        await session.commit()  # 添加数据需要调用提交方法

        return new_book

    async def update_book(self, book_uid: str, update_data: BookUpdateModel, session: AsyncSession):
        book_to_update = self.get_book(book_uid, session)
        if book_to_update is not None:
            update_data_dict = update_data.model_dump()
            for k, v in update_data_dict.items():
                setattr(book_to_update, k, v)
            await session.commit()
            return book_to_update
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Book not found")

    async def delete_book(self, book_uid: str, session: AsyncSession):
        book_to_delete = self.get_book(book_uid, session)
        if book_to_delete is not None:
            await session.delete(book_to_delete)
            await session.commit()
            return {
                "Message": "Book Deleted..."
            }
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Book not found")

```
### 6.然后我们需要在路由文件里面使用session，我们需要进入db包里面的main.py,在里面创建一个函数get_session,代码如下
```
from sqlmodel import create_engine, SQLModel
from sqlalchemy.ext.asyncio import AsyncEngine
from sqlmodel.ext.asyncio.session import AsyncSession
from sqlalchemy.orm import sessionmaker
from src.config import Config

'''
fastapi是异步的所以我们需要使用异步数据库引擎
'''

async_engine = AsyncEngine(
    create_engine(
        url=Config.DATABASE_URL,
        echo=True
    )
)


async def init_db():
    async with async_engine.begin() as conn:
        from src.books.models import Book
        await conn.run_sync(SQLModel.metadata.create_all)


async def get_session() -> AsyncSession:
    Session = sessionmaker(
       bind=async_engine,
       class_=AsyncSession,
       expire_on_commit=False
    )

    async with Session() as session:
        yield session

```
### 7.然后我们进入books/routes.py文件，在这里使用这个session
```

```
