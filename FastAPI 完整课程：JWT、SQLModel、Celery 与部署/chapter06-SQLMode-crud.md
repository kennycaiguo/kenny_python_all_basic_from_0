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
### 4.1然后创建一个BookCreateModel类，把这个Book类注释掉，否则后面你可能因为导入错误的类而经常出错。代码如下<br>
```
from pydantic import BaseModel
from datetime import datetime
import uuid


# class Book(BaseModel): # 这个类其实是不需要的。
#     uid: uuid.UUID
#     title: str
#     author: str
#     publisher: str
#     published_date: str
#     page_count: int
#     language: str
#     created_at: datetime
#     update_at: datetime


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


# class Book(BaseModel):
#     uid: uuid.UUID
#     title: str
#     author: str
#     publisher: str
#     published_date: str
#     page_count: int
#     language: str
#     created_at: datetime
#     update_at: datetime


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

### 4.2然后我们需要在models.py里面把Book类的出版日期的类型修改一下，注意Book数据类型类(继承自SQLModel)比BookCreateModel类多了3个字段，但是他们都有默认值，不用管。<br>
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
from datetime import datetime
from sqlmodel.ext.asyncio.session import AsyncSession
from sqlmodel import select, desc
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
        new_book.published_date = datetime.strptime(book_data_dict['published_date'], "%Y-%m-%d")
        session.add(new_book)  # 添加数据使用session.add方法
        await session.commit()  # 添加数据需要调用提交方法

        return new_book

    async def update_book(self, book_uid: str, update_data: BookUpdateModel, session: AsyncSession):
        book_to_update = await self.get_book(book_uid, session)
        if book_to_update is not None:
            update_data_dict = update_data.model_dump()
            for k, v in update_data_dict.items():
                setattr(book_to_update, k, v)
            await session.commit()
            return book_to_update
        else:
            return None

    async def delete_book(self, book_uid: str, session: AsyncSession):
        book_to_delete = await self.get_book(book_uid, session)
        if book_to_delete is not None:
            await session.delete(book_to_delete)
            await session.commit()
            return book_to_delete
        else:
            return None


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
### 7.然后我们进入books/routes.py文件，在这里使用这个session，我们这里学习一种叫做依赖注入的方法，然后我们在路由函数里面调用BookService的功能，代码如下
```
from datetime import datetime

from fastapi import FastAPI, status, APIRouter, Depends
from sqlmodel.ext.asyncio.session import AsyncSession
from fastapi.exceptions import HTTPException

from src.books.models import Book
from src.db.main import get_session
from src.books.service import BookService
from typing import List
from src.books.schemas import BookUpdateModel, BookCreateModel

book_router = APIRouter()
book_service = BookService()


@book_router.get("/",response_model=List[Book])
async def get_all_books(session: AsyncSession = Depends(get_session)):
    return await book_service.get_all_books(session)


@book_router.get("/{book_id}")
async def get_book(book_id: str, session: AsyncSession = Depends(get_session)):
    book = await book_service.get_book(book_id, session)
    if book is not None:
        return book
    else:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND, detail="Book not found"
        )


# 创建一本书,有防止重复创建相同的书的功能
@book_router.post("/",status_code=status.HTTP_201_CREATED,response_model=Book)
async def create_book(book_data: BookCreateModel, session: AsyncSession = Depends(get_session)):
    new_book = await book_service.create_book(book_data, session)
    return new_book


# 部分更新书本
@book_router.patch("/{book_id}")
async def path_book(book_id: str, update_data: BookUpdateModel, session: AsyncSession = Depends(get_session)):
    book_to_update = await book_service.update_book(book_id,update_data,session)
    if book_to_update is not None:
        return book_to_update
    else:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Book not found")


# 根据id来删除书本,good
@book_router.delete("/{bid}")
async def delete_book(bid: str, session: AsyncSession = Depends(get_session)):
    book_to_delete = await book_service.delete_book(bid,session)
    if book_to_delete is not None:
        return {"Book deleted":book_to_delete}
    else:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Book not found")
```
## 测试1，新增<br>
<img width="1437" height="817" alt="image" src="https://github.com/user-attachments/assets/3645dbd9-528a-476e-8897-2adf350736ab" /><br>
## 查看数据库，发现数据成功添加了<br>
<img width="1525" height="365" alt="image" src="https://github.com/user-attachments/assets/a2c18dcf-eab3-4db7-b072-b2b6425c78f9" /><br>
## 测试2，查看所有<br>
<img width="1437" height="771" alt="image" src="https://github.com/user-attachments/assets/e6d2d70d-c7de-493c-bf3b-ccda55c00b62" /><br>
## 测试3.查找一本书，如果数的id存在，就可以找到书<br>
<img width="1483" height="783" alt="image" src="https://github.com/user-attachments/assets/2596e50d-1b49-4e72-ad00-79db0418c8ff" /><br>
### 如果数的id不存在，就找不到书<br>
<img width="1486" height="665" alt="image" src="https://github.com/user-attachments/assets/26fb9d87-bca0-4238-8a33-63ca3f3f404c" /><br>
## 测试4.更新书籍，先找到一本书，他的数据是这样子的<br>
<img width="1425" height="744" alt="image" src="https://github.com/user-attachments/assets/2a723d9c-d228-4411-8deb-f4ccabf2b12b" /><br>
## 然后我们把他的数据修改一下<br>
<img width="1426" height="774" alt="image" src="https://github.com/user-attachments/assets/fc6d23ed-2e8e-4a93-a83e-14dcf03a9136" /><br>
## 然后我们再用他的id查找一下，发现数据修改成功了<br>
<img width="1442" height="734" alt="image" src="https://github.com/user-attachments/assets/270b5d0f-5c94-498f-93df-9567c109aca7" /><br>
## 测试5，删除书籍，我们打开删除路由，把刚才那本书的id添加到路由中，然后点击发送，返回信息说书本被删除了<br>
<img width="1424" height="778" alt="image" src="https://github.com/user-attachments/assets/1bc3fce0-9006-4b61-b1f2-c41b9a142a10" /><br>
## 然后我们再来查找这不是，就找不到了，说明删除成功 <br>
<img width="1441" height="570" alt="image" src="https://github.com/user-attachments/assets/7d5bf3d7-9da0-4bd0-993c-ef87fa6ff58a" /><br>
### 这一节的学习到此为止。需要注意，不要把数据类型搞错了，否则会有很多奇怪的错误<br>


