# 1.把chapter11项目复制一份，改名：chapter12-kill_access_token_using_redis,然后用pycharm打开<br>
<img width="477" height="680" alt="image" src="https://github.com/user-attachments/assets/e624e12d-1941-4272-abaa-3155b4de0e99" /><br>
# 2.这一节课我们来学习如何使用redis来吊销访问令牌access_token<br>
## 2.0需要先安装redis数据库服务，这里是windows版本的下载网址：：[https://github.com/microsoftarc](https://link.zhihu.com/?target=https%3A//github.com/microsoftarchive/redis/releases/download/win-3.2.100/Redis-x64-3.2.100.zip)  <br>
### 然后解压并且改名字保存到d:\programs <br>
<img width="1341" height="851" alt="image" src="https://github.com/user-attachments/assets/5824519d-0fa2-4cee-a491-a34b59155649" /><br>
### 接着设置环境变量，这样就不需要每次启动redis都要在安装目录下键入命令了：点开环境变量设置，系统环境变量下添加Path路径为安装Redis的解压目录（也就是redis-server.exe文件所在的目录），
#### 我这边是：D:\programs\Redis-x64-3.2.100<br>
<img width="1320" height="961" alt="image" src="https://github.com/user-attachments/assets/715ccf7c-9485-4af5-8703-592321673e6d" /><br>
### 然后启动redis服务器，注意：需要进入redis-server.exe所在的命令里面，命令是：redis-server.exe redis.windows.conf，然后服务就启动了，端口是：6379<br>
<img width="1254" height="660" alt="image" src="https://github.com/user-attachments/assets/7731330d-e4b2-4e2b-a72a-a12ff59b0d02" /><br>
### 不要改变这个窗口，然后我们来打开另外一个窗口，输入： redis-cli.exe -h 127.0.0.1 -p 6379,如果没有错误，应该就连接上了<br>
<img width="778" height="338" alt="image" src="https://github.com/user-attachments/assets/edaeb6f5-b445-44b6-8fc7-8d37ca451741" /><br>
### 此时你在客户端输入ping，服务器会返回pong，说明redis工作正常<br>
<img width="740" height="442" alt="image" src="https://github.com/user-attachments/assets/78dcbd56-b78b-4ec3-8900-cf1a11c5820c" /><br>
## 然后我们把redis安装为服务，此时需要修改配置文件redis.windows.conf，然后打开修改里面的内容，将其中选项daemonize设置成yes，如果没有这行请自行添加，结果是一样的<br>
<img width="1063" height="669" alt="image" src="https://github.com/user-attachments/assets/cdde63b6-8f3b-4e69-a543-106bb4415eac" /><br>
## 接着在安装redis的目录下的cmd中输入：redis-server --service-install redis.windows.conf --service-name redis<br>
<img width="1216" height="208" alt="image" src="https://github.com/user-attachments/assets/785a0002-0b81-41c3-8f6f-030c18405937" /><br>
## 此时服务并没有启动，我们需要启动它，可以点击服务上面的启动按钮<br>
<img width="1063" height="773" alt="image" src="https://github.com/user-attachments/assets/ddc67e54-15e6-4284-abe9-ac2c2108c8cb" /><br>
## 然后服务就启动了<br>
<img width="871" height="708" alt="image" src="https://github.com/user-attachments/assets/3f59c7e6-7de1-409a-97a0-2de1d919c82f" /><br>

## 2.1>我们需要安装一个库：aioredis
```
pip install aioredis
```
<img width="1322" height="345" alt="image" src="https://github.com/user-attachments/assets/ba2da092-7592-4997-8ad6-1edbabc1b331" /><br>
## 2.1.2>为了编程方便，我们可以在.env文件里面配置REDIS_HOST,他的值是localhost，然后设置REDIS_PORT为6379<br>
## 2.1.3>然后需要在src/config.py里面配置获取.env文件里面的配置<br>
```
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    DATABASE_URL: str
    JWT_SECRET: str  # 密钥
    JWT_ALGORITHM: str  # 算法
    REDIS_HOST: str = "localhost"   # redis服务器主机
    REDIS_PORT: int = 6379 # redis服务的端口
    model_config = SettingsConfigDict(
        env_file=".env",
        extra="ignore"
    )


Config = Settings()

```

## 2.2>在db包里面新建一个redis.py文件，他的主要作用是代码如下<br>
```
import aioredis
from src.config import Config

JTI_EXPIRY = 3600
token_block_list = aioredis.StrictRedis(
    host=Config.REDIS_HOST, port=Config.REDIS_PORT, db=0
)


# 添加令牌到阻止列表
async def add_jti_to_blocklist(jti: str) -> None:
    await token_block_list.set(name=jti, value="", ex=JTI_EXPIRY)


# 检查令牌是否在阻止列表里面
async def jti_in_blocklist(jti: str) -> bool:
    jti_exists = await token_block_list.get(jti)
    return jti_exists is not None

```
## 2.3>回到auth/dependencies.py,
### 先从redis.py里面导入jti_in_blocklist函数<br>
<img width="1187" height="797" alt="image" src="https://github.com/user-attachments/assets/d58a4162-a1af-426a-a0c1-a192f85dc811" /><br>
### 我们来修改一下代码
```
from fastapi.security import HTTPBearer
from fastapi.security.http import HTTPAuthorizationCredentials
from fastapi import Request, HTTPException, status
from .utils import decode_token
from src.db.redis import jti_in_blocklist


class TokenBearer(HTTPBearer):
    def __init__(self, auto_error=True):
        super().__init__(auto_error=auto_error)

    async def __call__(self, request: Request) -> HTTPAuthorizationCredentials | None:
        creds = await super().__call__(request)
        token = creds.credentials
        token_data = decode_token(token)
        print("refresh token_data:", token_data)
        # 验证token是否是有效
        if not self.token_valid(token):
            raise HTTPException(status_code=status.HTTP_403_FORBIDDEN,
                                detail={"error": "Token is not valid or expired...", "resolution": "Get new access_token"})
        # 验证令牌是否有效后，还需要验证令牌是否在blocklist里面
        if await jti_in_blocklist(token_data['jti']):
            raise HTTPException(status_code=status.HTTP_403_FORBIDDEN,
                                detail={"error": "Token is blocked...", "resolution": "Get new access_token"})
        self.verify_token_data(token_data)
        return token_data

    def token_valid(self, token: str) -> bool:
        token_data = decode_token(token)
        print(token_data)
        return token_data is not None

    def verify_token_data(self, token_data: dict) -> None:  # 这个方法相当于java中的接口，具体逻辑有子类实现
        raise NotImplementedError("You need to over ride this method...")


class AccessTokenBearer(TokenBearer):
    def verify_token_data(self, token_data: dict) -> None:
        # 验证是否是刷新令牌,如果是，说明令牌类型不正确，需要抛出另外一个异常
        if token_data and token_data['refresh']:
            raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="Please Provide Valid Access Token...")


class RefreshTokenBearer(TokenBearer):
    def verify_token_data(self, token_data: dict) -> None:
        # 验证是否是刷新令牌,如果是，说明令牌类型不正确，需要抛出另外一个异常
        if token_data and not token_data['refresh']:
            raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="Please Provide Valid Refresh Token...")

```
## 2.4>然后我们回到auth/routes.py，在这里添加一个撤销访问令牌的路由/logout<br>
```
from datetime import timedelta, datetime
from fastapi.responses import JSONResponse
from fastapi import APIRouter, Depends, status
from .schemas import UserCreateModel, UserModel, UserLoginModel
from .service import UserService
from src.db.main import get_session
from sqlmodel.ext.asyncio.session import AsyncSession
from fastapi.exceptions import HTTPException
from .utils import create_access_token, decode_token, verify_passwd
from .dependencies import RefreshTokenBearer, AccessTokenBearer
from src.db.redis import add_jti_to_blocklist

auth_router = APIRouter()
user_service = UserService()
refresh_token_bearer = RefreshTokenBearer()
print("refresh:", refresh_token_bearer)

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
                    'user_uid': str(user.uid),
                }
            )
            # print("access_token_data:",jwt.decode(jwt=access_token,key=Config.JWT_SECRET,algorithms=[Config.JWT_ALGORITHM]))
            refresh_token = create_access_token(
                user_data={
                    'email': user.email,
                    'user_uid': str(user.uid),
                },
                refresh=True,
                expiry=timedelta(days=REFRESH_TOKEN_EXPIRY)
            )
            return JSONResponse(
                content={
                    "message": "Login Successful",
                    "access_token": access_token,
                    "refresh_token": refresh_token,
                    "user": {
                        "email": user.email,
                        "uid": str(user.uid)
                    }
                }
            )
    raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="Invalid Email or Password...")


@auth_router.get("/refresh_token")
async def get_new_access_token(token_details: dict = Depends(refresh_token_bearer)):
    expiry_timestamp = token_details["exp"]

    if datetime.fromtimestamp(expiry_timestamp) > datetime.now():
        new_access_token = create_access_token(user_data=token_details["user"])

        return JSONResponse(content={"access_token": new_access_token})

    raise HTTPException(
        status_code=status.HTTP_400_BAD_REQUEST, detail="Invalid Or expired token"
    )


# 撤销访问令牌的路由,也就是登出系统
@auth_router.get("/logout")
async def remove_token(token_details: dict = Depends(AccessTokenBearer())):
    jti = token_details['jti']
    await add_jti_to_blocklist(jti)

    return JSONResponse(
        content={
            "message": "Logout successfully...."
        },
        status_code=status.HTTP_200_OK
    )

```
## 运行程序保存，可能是aioredis出错，我们把它卸载<br>
<img width="931" height="288" alt="image" src="https://github.com/user-attachments/assets/869a08bf-90db-41e5-8133-ac5def1a3445" /><br>
## 然后我们来安装redis-py<br>
```
pip install redis
```
### 然后在异步框架我们需要这么使用，这里导入和老师的是不一样的。<br>
<img width="839" height="416" alt="image" src="https://github.com/user-attachments/assets/32754e3b-7ce6-43ac-80ec-325a2a49a887" /><br>
## 测试，我们在postman的auth文件夹里面新建一个请求：/logout，请求头先不带bearer，就会报一个没有权限的错误<br>
<img width="1441" height="729" alt="image" src="https://github.com/user-attachments/assets/50fb1d69-9c0a-48ad-bdcf-83b91c46f468" /><br>












