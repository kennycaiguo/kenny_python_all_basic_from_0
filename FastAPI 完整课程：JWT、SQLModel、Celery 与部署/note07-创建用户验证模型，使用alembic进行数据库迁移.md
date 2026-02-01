# 1.我们要做的的一个功能就是创建用户账户<br>
## 1.1 把chapter6项目复制一份，改名chapter7-user-auth-alembic-migrate,然后用pycharm打开<br>
<img width="499" height="529" alt="image" src="https://github.com/user-attachments/assets/1673deaf-7d93-4eae-ae24-ef96175dde4f" /><br>
## 1.2 在src包里面创建一个auth包<br>
<img width="499" height="521" alt="image" src="https://github.com/user-attachments/assets/00b62c5e-f5ec-426d-a25f-e4789c705d99" /><br>
## 1.3 然后在auth包里面创建一个models.py文件，在里面定义一个User类，它也是SQLModel的子类<br>
```
from datetime import datetime

from sqlmodel import SQLModel, Field, Column
import sqlalchemy.dialects.postgresql as pg
import uuid


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
    is_verified: bool = False
    email: str
    password_hash: str
    created_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now))
    update_at: datetime = Field(sa_column=Column(pg.TIMESTAMP, default=datetime.now))

    def __repr__(self) -> str:
        return f"<User {self.username}>"
```
# 2.我们需要做数据库迁移（所谓的迁移，就在在不影响数据库的原有数据的情况下堆数据库进行更改。）
## 2.1这里需要安装一个库叫做alembic<br>
```
pip install alembic
```
### 安装成功如图：<br>
<img width="1225" height="506" alt="image" src="https://github.com/user-attachments/assets/9e4859b3-c211-47b8-ba47-7e8e3414d9ca" /><br>
## 2.2创建一个迁移环境，打开终端定位到项目的根目录，然后输入下面的命令<br>
```
 alembic init -t async migrations
```
### 按下回车，就创建迁移文件夹，他和src是在同一级目录下面的，还会创建一个alembic.ini文件<br>
<img width="1452" height="932" alt="image" src="https://github.com/user-attachments/assets/19576d45-1664-4d3f-ada4-ac5b6bdb871c" /><br>
## 2.3打开migrations文件夹里面的env.py,我们需要在这里设置数据库以及我们的模型,做3件事情<br>。
```
import asyncio
from logging.config import fileConfig

from sqlalchemy import pool
from sqlalchemy.engine import Connection
from sqlalchemy.ext.asyncio import async_engine_from_config

from alembic import context
from src.auth.models import User
from src.books.models import Book
from sqlmodel import SQLModel
from src.config import Config

# step 1
db_url = Config.DATABASE_URL
# this is the Alembic Config object, which provides
# access to the values within the .ini file in use.
config = context.config
# step2
config.set_main_option('sqlalchemy.url',db_url)

# Interpret the config file for Python logging.
# This line sets up loggers basically.
if config.config_file_name is not None:
    fileConfig(config.config_file_name)

# add your model's MetaData object here
# for 'autogenerate' support
# from myapp import mymodel
# target_metadata = mymodel.Base.metadata
# step3
# target_metadata = None
target_metadata = SQLModel.metadata

# other values from the config, defined by the needs of env.py,
# can be acquired:
# my_important_option = config.get_main_option("my_important_option")
# ... etc.


def run_migrations_offline() -> None:
    """Run migrations in 'offline' mode.

    This configures the context with just a URL
    and not an Engine, though an Engine is acceptable
    here as well.  By skipping the Engine creation
    we don't even need a DBAPI to be available.

    Calls to context.execute() here emit the given string to the
    script output.

    """
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )

    with context.begin_transaction():
        context.run_migrations()


def do_run_migrations(connection: Connection) -> None:
    context.configure(connection=connection, target_metadata=target_metadata)

    with context.begin_transaction():
        context.run_migrations()


async def run_async_migrations() -> None:
    """In this scenario we need to create an Engine
    and associate a connection with the context.

    """

    connectable = async_engine_from_config(
        config.get_section(config.config_ini_section, {}),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)

    await connectable.dispose()


def run_migrations_online() -> None:
    """Run migrations in 'online' mode."""

    asyncio.run(run_async_migrations())


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()


```
## 2.4打开script.py.mako，做如下配置，其实就一句话：import sqlmodel<br>
```
"""${message}

Revision ID: ${up_revision}
Revises: ${down_revision | comma,n}
Create Date: ${create_date}

"""
from typing import Sequence, Union

from alembic import op
import sqlalchemy as sa
import sqlmodel  # only import this
${imports if imports else ""}

# revision identifiers, used by Alembic.
revision: str = ${repr(up_revision)}
down_revision: Union[str, Sequence[str], None] = ${repr(down_revision)}
branch_labels: Union[str, Sequence[str], None] = ${repr(branch_labels)}
depends_on: Union[str, Sequence[str], None] = ${repr(depends_on)}


def upgrade() -> None:
    """Upgrade schema."""
    ${upgrades if upgrades else "pass"}


def downgrade() -> None:
    """Downgrade schema."""
    ${downgrades if downgrades else "pass"}

```
## 2.5然后打开终端定位到项目根目录，输入下面的命令
```
 alembic revision --autogenerate -m "init"
```
### 然后就会在version文件夹里面生成一个xxxx_init.py文件，这个就是我们的迁移文件。
<img width="1253" height="898" alt="image" src="https://github.com/user-attachments/assets/1e27d347-d353-46be-9883-4c215c398d3e" /><br>
### 这个文件的内容如下
```
"""init

Revision ID: 8fb6ad863694
Revises: 
Create Date: 2026-01-31 19:53:38.610233

"""
from typing import Sequence, Union

from alembic import op
import sqlalchemy as sa
import sqlmodel
from sqlalchemy.dialects import postgresql

# revision identifiers, used by Alembic.
revision: str = '8fb6ad863694'
down_revision: Union[str, Sequence[str], None] = None
branch_labels: Union[str, Sequence[str], None] = None
depends_on: Union[str, Sequence[str], None] = None


def upgrade() -> None:
    """Upgrade schema."""
    # ### commands auto generated by Alembic - please adjust! ###
    op.create_table('users',
    sa.Column('uid', sa.UUID(), nullable=False),
    sa.Column('username', sqlmodel.sql.sqltypes.AutoString(), nullable=False),
    sa.Column('first_name', sqlmodel.sql.sqltypes.AutoString(), nullable=True),
    sa.Column('last_name', sqlmodel.sql.sqltypes.AutoString(), nullable=True),
    sa.Column('is_verified', sa.Boolean(), nullable=False),
    sa.Column('email', sqlmodel.sql.sqltypes.AutoString(), nullable=False),
    sa.Column('password_hash', sqlmodel.sql.sqltypes.AutoString(), nullable=False),
    sa.Column('created_at', postgresql.TIMESTAMP(), nullable=True),
    sa.Column('update_at', postgresql.TIMESTAMP(), nullable=True),
    sa.PrimaryKeyConstraint('uid'),
    sa.UniqueConstraint('uid')
    )
    op.create_unique_constraint(None, 'books', ['uid'])
    # ### end Alembic commands ###


def downgrade() -> None:
    """Downgrade schema."""
    # ### commands auto generated by Alembic - please adjust! ###
    op.drop_constraint(None, 'books', type_='unique')
    op.drop_table('users')
    # ### end Alembic commands ###

```
### 然后会在数据库里面生成一个alembic_version文件夹，暂时还没有生成数据表<br>
<img width="840" height="600" alt="image" src="https://github.com/user-attachments/assets/f3374740-3b50-4232-9d68-b4d11430255e" /><br>

### 注意：env.py,script.py.mako和alembic.ini里面一个中文字都不能够写，否则会报编码错误，命令执行失败！！！<br>

## 2.6接下来，我们就可以利用这个迁移文件来创建users数据表，命令如下<br>
```
alembic upgrade head
```
## 效果：<br>
<img width="969" height="159" alt="image" src="https://github.com/user-attachments/assets/5fb430b4-bbb1-4df0-94fb-dd1fa268822a" /><br>
### 我们到数据库里面看看，发现users数据表被创建了。<br>
<img width="583" height="541" alt="image" src="https://github.com/user-attachments/assets/18b9a4de-2ab1-4123-a0d9-b852d31497a0" /><br>
### 我们可以使用postgres控制台工具来查看<br>
```
Server [localhost]:
Database [postgres]:
Port [5432]:
Username [postgres]:
用户 postgres 的口令：

psql (17.2)
输入 "help" 来获取帮助信息.

postgres=# \c booklydb
您现在已经连接到数据库 "booklydb",用户 "postgres".
booklydb=# \dt
                    关联列表
 架构模式 |      名称       |  类型  |  拥有者
----------+-----------------+--------+----------
 public   | alembic_version | 数据表 | postgres
 public   | books           | 数据表 | postgres
 public   | users           | 数据表 | postgres
(3 行记录)


booklydb=# select * from alembic_version
booklydb-# ;
 version_num
--------------
 8fb6ad863694
(1 行记录)


booklydb=# \d users;
                          数据表 "public.users"
     栏位      |            类型             | 校对规则 |  可空的  | 预设
---------------+-----------------------------+----------+----------+------
 uid           | uuid                        |          | not null |
 username      | character varying           |          | not null |
 first_name    | character varying           |          |          |
 last_name     | character varying           |          |          |
 is_verified   | boolean                     |          | not null |
 email         | character varying           |          | not null |
 password_hash | character varying           |          | not null |
 created_at    | timestamp without time zone |          |          |
 update_at     | timestamp without time zone |          |          |
索引：
    "users_pkey" PRIMARY KEY, btree (uid)


booklydb=#
```
## 这一节的学习到此为止







