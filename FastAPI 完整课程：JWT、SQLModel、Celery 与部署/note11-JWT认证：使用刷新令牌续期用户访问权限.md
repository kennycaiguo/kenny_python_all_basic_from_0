# 1.把chapter10复制一份。改名：chapter11-pyjwt-token-renewal-refresh_token，然后用pycharm打开<br>
<img width="404" height="638" alt="image" src="https://github.com/user-attachments/assets/c0aac9a9-b8bd-4d4c-af15-157c78a5b2c1" /><br>
# 2.这一节课我们主要来学习如何使用刷新令牌来给访问令牌续期<br>
## 2.1>打开auth/depencies.py文件，先把我们的AccessTokenBearer改名为TokenBearer作为基类<br>
<img width="1384" height="610" alt="image" src="https://github.com/user-attachments/assets/38b0cfe3-21a3-4bd9-a2b1-84f99c635b4f" /><br>
## 2.2>然后我们创建一个AccessTokenBearer从TokenBearer派生<br>
<img width="834" height="343" alt="image" src="https://github.com/user-attachments/assets/26487527-f254-46ec-9bc8-b07cf86a6a8d" /><br>
## 2.3>然后我们创建一个RefreshTokenBearer类，也是从TokenBearer派生，修改后的dependencies.py代码如下,这里我们在父类创建一个有子类实现的方法，相当于java的接口或者c++的纯虚函数<br>
```
from fastapi.security import HTTPBearer
from fastapi.security.http import HTTPAuthorizationCredentials
from fastapi import Request, HTTPException, status
from .utils import decode_token


class TokenBearer(HTTPBearer):
    def __init__(self, auto_error=True):
        super().__init__(auto_error=auto_error)

    async def __call__(self, request: Request) -> HTTPAuthorizationCredentials | None:
        creds = await super().__call__(request)
        # print(creds.scheme)
        # print(creds.credentials)
        token = creds.credentials
        # print(token)
        token_data = decode_token(token)
        # 验证token是否是有效
        if not self.token_valid(token):
            raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="Invalid Or Expired Token...")
        self.verify_token_data(token_data)
        return token_data

    def token_valid(self, token: str) -> bool:
        token_data = decode_token(token)
        print(token_data)
        return token_data is not None

    def verify_token_data(self, token_data: dict) -> None: # 这个方法相当于java中的接口，具体逻辑有子类实现
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
### 测试一下，一切正常<br>
<img width="1502" height="904" alt="image" src="https://github.com/user-attachments/assets/cb438e11-ede9-43fc-9c62-e54e52723843" /><br>
<img width="1419" height="827" alt="image" src="https://github.com/user-attachments/assets/7a41ee25-0bdf-4ff7-a714-b671091d698a" /><br>
<img width="1322" height="769" alt="image" src="https://github.com/user-attachments/assets/f2934d83-4eb5-475c-8d9b-e38b9cf310c0" /><br>
## 2.4>然后进入auth/routes.py,我们来创建一个获取刷新令牌的路由,先导入我们的RefreshTokenBearer类<br>
<img width="948" height="366" alt="image" src="https://github.com/user-attachments/assets/32c8673d-1917-4786-8b47-1a21db71cc88" /><br>
### 我们继续来完成这个路由,我们先写一个框架，加入依赖注入，简单返回一个{}<br>
<img width="1189" height="667" alt="image" src="https://github.com/user-attachments/assets/cb9e2b1a-f25c-41f0-a1cb-1651cf1ec0ef" /><br>
### 然后我们在postman里面测试，先不在请求头里面携带然后token，此时会报错说没有授权<br>
<img width="1457" height="593" alt="image" src="https://github.com/user-attachments/assets/648f4bb6-d57c-4284-9cb2-bb8327d3b756" /><br>
### 然后我们在请求头里面添加access_token令牌，然后再发送请求此时会报错说需要刷新令牌<br>
<img width="1438" height="650" alt="image" src="https://github.com/user-attachments/assets/46cde503-18ca-4d5c-9165-debc2a238a42" /><br>
### 然后我们在请求头里面添加refresh_token令牌，然后再发送请求此时验证通过<br>
<img width="1544" height="774" alt="image" src="https://github.com/user-attachments/assets/4cd0d661-e288-4724-aec2-47053f22af9f" /><br>
## 2.5>然后我们来完成这个路由函数的编写，代码如下,参考@auth_router.get("/refresh_token")部分
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
from .dependencies import RefreshTokenBearer

auth_router = APIRouter()
user_service = UserService()
refresh_token_bearer = RefreshTokenBearer()
print("refresh:",refresh_token_bearer)

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

```
### 然后测试一下，就会生成新的access_token<br>
<img width="1408" height="677" alt="image" src="https://github.com/user-attachments/assets/26b34cd5-24e4-4947-a1c4-5957ceaf5886" /><br>
### 进入我们在header里面传递access_token,就会抛异常<br>
<img width="1429" height="703" alt="image" src="https://github.com/user-attachments/assets/1eb3c454-6cc5-4320-958a-8b0595a81893" /><br>



