# 1.把chapter12项目复制一份，改名：然后用pycharm打开<br>
<img width="599" height="615" alt="image" src="https://github.com/user-attachments/assets/9b81b51b-a9b7-4cdd-a45d-87449d5c766d" /><br>
# 2.这一节课我们来学习基于角色的访问控制，在我们的例子中我们有2种用户角色，管理员和普通用户，他们的权限如图，当然，你可以根据你的需要设置更多的角色<br>
<img width="1027" height="395" alt="image" src="https://github.com/user-attachments/assets/33fc93ad-4a76-4587-98b6-4b0132062048" /><br>
## 2.1>进入auth/dependencies.py,我们创建一个获取当前用户的函数，这个函数需要一些依赖项我们先导入相关的依赖，注意这是一个异步函数<br>
<img width="1105" height="407" alt="image" src="https://github.com/user-attachments/assets/36b7708e-982d-4725-a57e-12a39da459ae" /><br>
## 2.2>函数的代码如下：<br>
```
from fastapi.security import HTTPBearer
from fastapi.security.http import HTTPAuthorizationCredentials
from fastapi import Request, HTTPException, status, Depends
from sqlmodel.ext.asyncio.session import AsyncSession

from .service import UserService
from .utils import decode_token
from src.db.redis import jti_in_blocklist
from src.db.main import get_session

user_service = UserService()


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
                                detail={"error": "Token is not valid or expired...",
                                        "resolution": "Get new access_token"})
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

# 获取当前用户的函数
async def get_current_user(token_details: dict = Depends(AccessTokenBearer()), session: AsyncSession = Depends(get_session)):
    user_email = token_details['user']['email']
    user = await user_service.get_user_by_email(user_email,session)
    return user


```

## 2.3>然后我们需要创建一个路由来获取当前用户<br>
<img width="1139" height="672" alt="image" src="https://github.com/user-attachments/assets/bccc76aa-edcf-419b-9525-093fdb5d2d28" /><br>
## 2.4>然后在postman中测试一下，请求头里面先不携带任何东西，发送请求，就会报没有授权的错误。<br>
<img width="1435" height="703" alt="image" src="https://github.com/user-attachments/assets/d136e344-dfbf-45e1-b281-4262e0a308d5" /><br>
## 2.5>然后我们登录，并且把访问令牌添加到请求头中，再发送请求，就可以获取到当前用户的信息<br>
<img width="1409" height="773" alt="image" src="https://github.com/user-attachments/assets/18bf8194-3fcf-4e30-9bb1-eba45697f7bd" /><br>
# 3.给用户创建角色<br>
## 3.1>打开auth/models.py给User类添加一个字段：role<br>
<img width="1193" height="717" alt="image" src="https://github.com/user-attachments/assets/42697d8c-1c05-48bb-9bc5-8359277464cb" /><br>
## 3.2>我们打开src文件夹里面的__init__.py,把数据库生命周期事件移除，因为我们现在使用alembic来帮助我们修改数据库<br>
<img width="1183" height="700" alt="image" src="https://github.com/user-attachments/assets/53e57b88-0df7-4f00-acf1-a615a5f8378a" /><br>
## 3.3需要使用alembic来生成迁移脚本并且做数据迁移,需要打开一个终端并且定位到项目的根目录，然后输入下面的命令<br>
```
$ alembic revision --autogenerate -m "add role to users table"
```
<img width="1405" height="484" alt="image" src="https://github.com/user-attachments/assets/763dbc04-9a1d-4684-ba04-c2879d662fa7" /><br>
### 3.3.2>生成修订版后，我们需要更新到数据库，使用下面的命令
```
alembic upgrade head
```
<img width="1131" height="179" alt="image" src="https://github.com/user-attachments/assets/bc371cf4-be8e-4057-8a22-ebe9ea5c4407" /><br>
## 3.4执行命令后，我们可以用navicat17查看一下数据库，可以看到我们以前创建的用户默认角色都是user<br>
<img width="1556" height="648" alt="image" src="https://github.com/user-attachments/assets/9412d521-1b4d-46bb-9793-3dab10e70005" /><br>





