# 1.把chapter8的代码复制一份改名chapter9-JWT-authentication<br>
<img width="1226" height="918" alt="image" src="https://github.com/user-attachments/assets/83df7e19-7782-43b3-ab9b-573b1fc18f57" /><br>
# 1.2JWT示例<br>
<img width="1378" height="719" alt="image" src="https://github.com/user-attachments/assets/79f9f259-aea3-4dbd-b36a-62cbb63bc742" /><br>
# 2.1打开终端定位到项目根目录，然后我们来安装pyjwt<br>
```
pip install pyjwt
```
## 然后就安装成功了<br>
<img width="792" height="307" alt="image" src="https://github.com/user-attachments/assets/f266540a-8e59-443f-b2af-389c7f049fb6" /><br>
# 2.2我们需要在.env文件里面设置一个密钥和加密算法，用来供jwt使用（python有一个内置的secrets模块可以调用他的token_hex(16)函数来输出密钥）<br>
<img width="1140" height="745" alt="image" src="https://github.com/user-attachments/assets/aafc7b2b-d7a6-473c-88c5-0a98e73429d1" /><br>
# 2.3>然后，我们需要找到config.py文件，在里面配置我们的密钥和算法,注意：名称必须和你在.env文件里面配置的一致。
```
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    DATABASE_URL: str
    JWT_SECRET:str  # 密钥
    JWT_ALGORITHM:str # 算法
    model_config = SettingsConfigDict(
        env_file=".env",
        extra="ignore"
    )


Config = Settings()
```
# 2.4> 转到我们的utils.py中，在里面添加2个函数create_access_token和decode_token<br>
```
from datetime import timedelta, datetime
from passlib.context import CryptContext
import jwt
from src.config import Config
import uuid
import logging

ACCESS_TOKEN_EXPIRY = 3600

passwd_context = CryptContext(
    schemes=['bcrypt']
)


def generate_passwd_hash(passwd: str) -> str:
    hash = passwd_context.hash(passwd)

    return hash


def verify_passwd(passwd: str, hash: str) -> bool:
    return passwd_context.verify(passwd, hash)


def create_access_token(user_data: dict, expiry: timedelta = None, refresh: bool = False):
    exp =  datetime.now() + (expiry if expiry is not None else timedelta(seconds=ACCESS_TOKEN_EXPIRY)) # 这里需要括号，否则会报错
    payload = {
        'user': user_data,
        'exp':exp.timestamp(),  # 大坑，这里需要timestamp如果你不是传递timestamp，后续使用jwt.decode(token,key,algo)就会报错。
        'jti': str(uuid.uuid4()),
        'refresh': refresh
    }

    token = jwt.encode(
        payload=payload,
        key=Config.JWT_SECRET,
        algorithm=Config.JWT_ALGORITHM
    )

    return token


def decode_token(token: str) -> dict:
    try:
        token_data = jwt.decode(
            jwt=token,
            key=Config.JWT_SECRET,
            algorithms=[Config.JWT_ALGORITHM]
        )
        return token_data
    except jwt.PyJWTError as e:
        logging.exception(e)
        return None

```
# 2.4>回到auth/routes.py，
## 2.4.1>先导入我们需要的东西<br>
<img width="1233" height="392" alt="image" src="https://github.com/user-attachments/assets/af438872-d5d2-453a-b9b1-ddce717b4796" /><br>
## 2.4.2>然后我们创建一个/login路由函数先定义，不写代码<br>
<img width="855" height="493" alt="image" src="https://github.com/user-attachments/assets/b9d0723e-aaae-497f-8e6c-3e69263914a5" /><br>
## 2.4.3>转到auth/schemas.py,在这里创建一个UserLoginModel类
```
import uuid
from datetime import datetime

from pydantic import BaseModel
from sqlmodel import Field


class UserCreateModel(BaseModel):
    username: str = Field(max_length=50)  # 注意：用户名不一定是first_name + last_name,他们可以不一样的。
    first_name: str = Field(max_length=25)
    last_name: str = Field(max_length=25)
    email: str = Field(max_length=40)
    password: str = Field(min_length=6)


class UserModel(BaseModel):
    uid: uuid.UUID
    username: str
    first_name: str
    last_name: str
    is_verified: bool
    email: str
    password_hash: str = Field(exclude=True)
    created_at: datetime
    update_at: datetime


class UserLoginModel(BaseModel):
    email: str = Field(max_length=40)
    password: str = Field(min_length=6)

```
## 2.5>再次回到auth/routes.py，先导入这个UserLoginModel<br>
<img width="1085" height="425" alt="image" src="https://github.com/user-attachments/assets/c1ce2084-7c79-401b-a2b0-eedb34389bc6" /><br>
## 2.6然后我们来完成这个/login路由函数,参考下面的login函数代码
```
from datetime import timedelta
from fastapi.responses import JSONResponse
from fastapi import APIRouter, Depends, status
from .schemas import UserCreateModel, UserModel, UserLoginModel
from .service import UserService
from src.db.main import get_session
from sqlmodel.ext.asyncio.session import AsyncSession
from fastapi.exceptions import HTTPException
from .utils import create_access_token, decode_token, verify_passwd

auth_router = APIRouter()
user_service = UserService()

REFRESH_TOKEN_EXPIRY = 2


@auth_router.post("/signup", response_model=UserModel, status_code=status.HTTP_201_CREATED)
async def create_user_account(user_data: UserCreateModel, session: AsyncSession = Depends(get_session)):
    email = user_data.email
    # 先判断用户是否存在，只有不存在才创建
    user_exists = await user_service.user_exists(email, session)
    if user_exists:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="User with email already exists"
        )

    new_user = await user_service.create_user(user_data, session)
    return new_user


@auth_router.post("/login")
async def login(login_data: UserLoginModel, session: AsyncSession = Depends(get_session)):
    email = login_data.email
    password = login_data.password
    user = await user_service.get_user_by_email(email, session)
    if user is not None:
        password_valid = verify_passwd(password, user.password_hash)
        if password_valid:
            access_token = create_access_token(
                user_data={
                    'email': user.email,
                    'user uid': str(user.uid),

                }
            )

            refresh_token = create_access_token(
                user_data={
                    'email': user.email,
                    'user uid': str(user.uid),

                },
                refresh=True,
                expiry=timedelta(days=REFRESH_TOKEN_EXPIRY)
            )

            return JSONResponse(
                content={
                    "message": "Login Successful",
                    "access_token": access_token,
                    "refresh_token": refresh_token,
                    "user":{
                        "email":user.email,
                        "uid": str(user.uid)
                    }
                }
            )
    raise HTTPException(status_code=status.HTTP_403_FORBIDDEN,detail="Invalid Email or Password...")
```
## 测试一下，成功<br>
<img width="1468" height="823" alt="image" src="https://github.com/user-attachments/assets/b8e76534-f0a6-4af2-b565-a461b4f47da9" /><br>







