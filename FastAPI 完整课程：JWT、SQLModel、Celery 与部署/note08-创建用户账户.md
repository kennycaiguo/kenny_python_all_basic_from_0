# 1.把chpater7项目复制一份，改名chapter8-user-account-creation<br>
<img width="481" height="590" alt="image" src="https://github.com/user-attachments/assets/6424aec6-2a99-497b-9c0c-57edecd89279" /><br>
# 2.然后我们来进行用户账户相关的路由操作<br>
## 2.1>在auth包里面新建一个routes.py,先创建一个APIRouter对象，然后定义一个创建用户账户的路由函数
```
from fastapi import APIRouter

auth_router = APIRouter()


@auth_router.post("/signup")
async def create_user_account():
    pass
```
## 2.2>然后我们回到auth/models.py,给密码添加一个隐藏设置，也就是序列号给前端时把他排除，避免让用户看到明文密码。<br>
<img width="1181" height="647" alt="image" src="https://github.com/user-attachments/assets/2c844a30-858c-45ba-9def-77634330fd68" /><br>
## 2.3>然后我们需要打开终端，输入下面的命令<br>
```
alembic revision --autogenerate -m "add password exclusion"
```
### 效果如下<br>
<img width="1362" height="465" alt="image" src="https://github.com/user-attachments/assets/c6ab99fa-c0ec-4f57-bbcd-4c273cf23ee4" /><br>
## 2.4>然后还需要执行下面的命令
```
alembic upgrade head
```
### 效果如下<br>
<img width="1094" height="170" alt="image" src="https://github.com/user-attachments/assets/d1b901d2-80cf-4ddd-b47b-602f0503643e" /><br>
### 用navicat17检查数据表的设计器，发现这个字段现在是not null<br>
<img width="913" height="430" alt="image" src="https://github.com/user-attachments/assets/b0991e60-493b-4d3e-9139-f9b25818bda3" /><br>
## 2.5>在auth包中创建一个schemas.py,<br>
<img width="522" height="718" alt="image" src="https://github.com/user-attachments/assets/366dccdb-8938-4b48-acea-8a9cf73152bd" /><br>
### 内容如下<br>
```
from pydantic import BaseModel
from sqlmodel import Field


class UserCreateModel(BaseModel):
    username: str = Field(max_length=8)
    email: str = Field(max_length=40)
    password: str = Field(min_length=6)
```
## 2.6>回到auth/routes.py,我们继续来完成用户创建的路由<br>
```
from fastapi import APIRouter
from .schemas import UserCreateModel

auth_router = APIRouter()


@auth_router.post("/signup")
async def create_user_account(user_data:UserCreateModel):
    pass
```
## 2.7>然后我们需要创建一个service.py文件<br>
<img width="518" height="704" alt="image" src="https://github.com/user-attachments/assets/f1e5f911-7a5d-443c-97fc-8d9ea70ba1ea" /><br>
## 2.7.2我们需要安装passlib用来给密码生成哈希值，命令如下
```
pip install passlib
```
<img width="970" height="193" alt="image" src="https://github.com/user-attachments/assets/3472e747-1dcd-4934-a7bd-eec919ad92ed" /><br>
### 然后我们需要在auth包里面新建一个utils.py文件，把passlib的方法调用写在里面
```
from passlib.context import CryptContext

passwd_context = CryptContext(
    schemes=['bcrypt']
)


def generate_passwd_hash(passwd: str) -> str:
    hash = passwd_context.hash(passwd)

    return hash


def verify_passwd(passwd: str, hash: str) -> bool:
    return passwd_context.verify(passwd, hash)

```
### 然后，我们来创建一个UserService类，添加查找用户，判断用户是否存在和创建用户方法，代码如下<br>
```
from .models import User
from sqlmodel.ext.asyncio.session import AsyncSession
from sqlmodel import select

from .schemas import UserCreateModel
from .utils import generate_passwd_hash


class UserService:
    async def get_user_by_email(self, email: str, session: AsyncSession):
        stmt = select(User).where(User.email == email)
        result = await session.exec(stmt)
        return result.first()

    async def user_exists(self,email,session:AsyncSession):
        user = await self.get_user_by_email(email,session)
        return True if user is not None else False

    async def create_user(self,user_data:UserCreateModel,sesson:AsyncSession):
        user_data_dict = user_data.model_dump()
        new_user = User(
            ** user_data_dict
        )
        new_user.password_hash = generate_passwd_hash(user_data["password"])
        sesson.add(new_user)
        await sesson.commit()
        return new_user

```
## 2.8>回到auth/routes.py里面，继续完成我们的路由函数
```

```






