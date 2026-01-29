# 1.数据库系统的安装
## 我们将会使用postgres数据库，在这里我们使用一个在线股价Neon，网址：https://neon.com/，我们需要先创建一个账号<br>
## 1.进入这个网站，发现可以用Google账户来创建用户<br>
<img width="1038" height="859" alt="image" src="https://github.com/user-attachments/assets/9467623b-5d41-46af-914a-bfa466a9a6a5" /><br>
## 2.点击这个选项，进入选择账户界面，我们使用第一个<br>
<img width="1552" height="572" alt="image" src="https://github.com/user-attachments/assets/8fee5cee-0473-4f12-b974-876ca9bd1526" /><br>
## 3.点击继续<br>
<img width="1447" height="756" alt="image" src="https://github.com/user-attachments/assets/31ad0c11-50c4-4004-8f16-6d49cd39e045" /><br>
## 4.然后就进入创建项目界面<br>
<img width="1670" height="926" alt="image" src="https://github.com/user-attachments/assets/986e8bd3-eef2-499c-ba69-7c3dd452f753" /><br>
## 5.我们在项目名称里面输入bookly，版本选择16<br>
<img width="1459" height="948" alt="image" src="https://github.com/user-attachments/assets/e9fdcfef-f4ab-4d05-bd4c-2c0d2ecb3de8" /><br>
<br>
## 6.点击创建项目，就会进入这个界面<br>
<img width="1836" height="985" alt="image" src="https://github.com/user-attachments/assets/062d3e3e-b6b0-4525-97d6-012b324583ae" /><br>
## 7.点击connect按钮，就会弹出这个界面，它默认帮我们新建一个neondb数据库，我们也可以创建自己的数据库，注意：连接字符串需要复制下来<br>
<img width="1693" height="906" alt="image" src="https://github.com/user-attachments/assets/7a4c3f78-e8e8-4654-86a6-604e00155d9c" /><br>
## 8.这里我们创建一个自己的数据库<br>
<img width="1576" height="955" alt="image" src="https://github.com/user-attachments/assets/1c50d517-d5d4-4555-ba51-fe94911aa406" /><br>
## 9.点击创建按钮，数据库就创建成功了.<br>
<img width="1893" height="994" alt="image" src="https://github.com/user-attachments/assets/fe964645-1230-46e0-b9b0-41181886f655" /><br>
## 参考文档：https://neon.com/docs/connect/connect-from-any-app<br>
## 在实操中不太好用，我们还是使用本地数据库吧。<br>
# 我们手动安装postgress 17，用户名是postgres，密码是{name+year}，为了方便使用，我们把bin目录添加到环境变量<br>
<img width="784" height="773" alt="image" src="https://github.com/user-attachments/assets/db6ba2e8-2ce4-4cdc-b4cc-9c2dd9c05d49" /><br>
# postgres有一个命令行股价psql，他需要使用psql 数据库名的方法来打开数据库，比如我们用我们创建的kenny账户来大量postgres数据库<br>
<img width="865" height="463" alt="image" src="https://github.com/user-attachments/assets/2652fd3c-33a7-47b1-8f10-608f60af0e6a" /><br>
## 我们可以使用pgAdminr4，一个图形界面的管理工具来管理postgres数据库。可能比navicat好。<br>
<img width="1265" height="799" alt="image" src="https://github.com/user-attachments/assets/c9ce0dc4-7a89-44c3-b7b1-4cd249509367" /><br>
# 2.项目配置
## 1>我们需要在我们项目，这里是chapter5的根目录下面创建一个.env文件，把数据库的登录配置保存在里面，注意不要上传到GitHub。<br>
<img width="495" height="407" alt="image" src="https://github.com/user-attachments/assets/76376c3d-a571-4563-87da-592a23c481e4" /><br>
## 2>安装asyncpg和async-timeout
```
pip install asyncpg
pip install async-timeout
```
<img width="1222" height="447" alt="image" src="https://github.com/user-attachments/assets/394a823b-fd17-4bfa-b51c-7fb7551440da" /><br>
## 2>然后你需要在.env里面配置数据库url格式： DATABASE_URL=postgresql+asyncpg://用户名:密码@localhost:5432/数据库名称<br>
## 3>安装pydantic-settings
```
pip install pydantic-settings
```
## 4>然后在src文件夹里面创建一个config.py文件，内容如下
```
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    DATABASE_URL: str
    model_config = SettingsConfigDict(
        env_file=".env",
        extra="ignore"
    )


Config = Settings()

```
## 5>然后我们在pycharm里面打开一个python console，并且测试一下，看看能否获取.env文件里面的配置。
```
>>> from src.config import Settings
>>>
>>> s = Settings()
>>> s.DATABASE_URL
'postgresql+asyncpg://postgres:kenny1975@localhost:5432/booklydb'
>>>
```
### 可以看到它成功获取到了url<br>
### 然后我们可以新建一个.gitignore文件，把.env文件添加到里面
<img width="888" height="471" alt="image" src="https://github.com/user-attachments/assets/228689d4-d38d-4435-ba67-4e2d6d59628e" />
## 6>安装SQLModel,会自动安装SQLAlchemy
```
pip install sqlmodel
```
## 7>然后我们在src里面新建一个db包，然后在里面新建一个main.py,用来编写所有与数据库操作相关的代码，内容如下
```
from sqlmodel import create_engine
from sqlalchemy.ext.asyncio import AsyncEngine
from src.config import Config
'''
fastapi是异步的所以我们需要使用异步数据库引擎
'''

engine = AsyncEngine(
  create_engine(
     url=Config.DATABASE_URL,
     echo=True
  )
)

```
## 8>然后我们需要让这些连接数据库的代码在应用程序启动的时候运行，怎么做的这一点，在fastapi中我们可以使用生命周期事件来实现，我们需要在应用程序的人口出也就是src文件夹里面的__init__.py里面添加代码，
## 需要用到python的内置模块contextlib，代码如下
```
from fastapi import FastAPI
from src.books.routes import book_router
from contextlib import asynccontextmanager  # 这是一个装饰器


# 指定哪些任务需要在应用程序启动时执行
@asynccontextmanager
async def life_span(app: FastAPI):
    print(f"Server is starting...")
    yield  # 在yield之前的代码会在服务器启动的时候运行，在yield之前的代码会在服务器结束的时候运行
    print(f"Server stopped...")


version = 'v1'

app = FastAPI(
    title='chapter4-routing',
    description='A RESTful API for a book review web service',
    version=version,
    lifespan=life_span  # 在应用程序中注册生命周期事件
)

app.include_router(book_router, prefix=f"/api/{version}/books", tags=['books'])
```
### 然后我们可以启动服务器看看效果如何，启动程序，会输出Server is starting...，<br>
<img width="1129" height="421" alt="image" src="https://github.com/user-attachments/assets/cf31d1cb-0ff2-4a1b-b1d5-78485e3830ef" /><br>

### 停止服务器，会输出"Server stopped...<br>
<img width="929" height="379" alt="image" src="https://github.com/user-attachments/assets/86af97b8-99fd-491e-8759-f0a737439bc1" /><br>
## 9>现在我们来添加连接数据库的代码，需要在db包里面的main.py里面添加代码<br>
```
from sqlmodel import create_engine, text
from sqlalchemy.ext.asyncio import AsyncEngine
from src.config import Config

'''
fastapi是异步的所以我们需要使用异步数据库引擎
'''

engine = AsyncEngine(
    create_engine(
        url=Config.DATABASE_URL,
        echo=True
    )
)


async def init_db():
    async with engine.begin() as conn:
        statement = text("SELECT 'Hello';")

        result = await conn.execute(statement)
        print(result.all())
```
## 10>回到src/__init__.py,导入我们的init_db函数并且在生命周期函数里面使用
```
from fastapi import FastAPI
from src.books.routes import book_router
from contextlib import asynccontextmanager  # 这是一个装饰器
from src.db.main import init_db


# 指定哪些任务需要在应用程序启动时执行
@asynccontextmanager
async def life_span(app: FastAPI):
    print(f"Server is starting...")
    await init_db()
    yield  # 在yield之前的代码会在服务器启动的时候运行，在yield之前的代码会在服务器结束的时候运行
    print(f"Server stopped...")

1
version = 'v1'

app = FastAPI(
    title='chapter4-routing',
    description='A RESTful API for a book review web service',
    version=version,
    lifespan=life_span  # 在应用程序中注册生命周期事件
)

app.include_router(book_router, prefix=f"/api/{version}/books", tags=['books'])

```
## 11>然后重新启动应用程序，然后就会连接数据库，执行我们的程序代码并且输出结果<br>
<img width="700" height="292" alt="image" src="https://github.com/user-attachments/assets/437f1338-73c3-49e5-a41c-0aef612d76b6" /><br>
### 说明我们连接数据库成功。<br>
## 12>然后我们需要为我们的书籍创建一个模型，我们在books包里面新建一个model.py文件，内容如下
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
            default=uuid.uuid4()
        )
    )
    title: str
    author: str
    publisher: str
    published_date: str
    page_count: int
    language: str
    created_at: datetime = Field(Column(pg.TIMESTAMP, default=datetime.now()))
    updated_at: datetime = Field(Column(pg.TIMESTAMP, default=datetime.now()))

    def __repr__(self):
        return f"<Book: {self.title}>"

```
## 13>然后我们进入db/main.py里面修改一下代码
```
from sqlmodel import create_engine, SQLModel
from sqlalchemy.ext.asyncio import AsyncEngine
from src.config import Config


'''
fastapi是异步的所以我们需要使用异步数据库引擎
'''

engine = AsyncEngine(
    create_engine(
        url=Config.DATABASE_URL,
        echo=True
    )
)


async def init_db():
    async with engine.begin() as conn:
        from src.books.models import Book
        await conn.run_sync(SQLModel.metadata.create_all)

```
### 保存文件后服务器会重新启动，然后就在控制台输出见表语句<br>
<img width="1335" height="431" alt="image" src="https://github.com/user-attachments/assets/ae733a97-8bc1-4920-b0e5-63cebb83ab49" /><br>
### 然后我们到数据库里面去看一下，发现表创建成功了<br>
<img width="1429" height="876" alt="image" src="https://github.com/user-attachments/assets/308b4903-2297-40b0-b995-9249679ee4a3" /><br>


