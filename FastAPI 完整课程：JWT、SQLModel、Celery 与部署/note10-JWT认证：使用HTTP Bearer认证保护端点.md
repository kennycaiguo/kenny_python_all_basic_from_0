# 1.把chapter9项目复制一份改名：chapter10-Role-Based-Access-Control然后用pycharm打开<br>
<img width="728" height="883" alt="image" src="https://github.com/user-attachments/assets/fcf63b9b-b482-4173-9eaf-8a58126473de" /><br>
# 2.既然我们可以给用户颁发令牌了，我们需要做一下设置来限制没有令牌的用户的操作访问权限<br>
## 2.1>进入auth包，在里面新建一个dependencies.py文件，内容暂时如下：
```
from fastapi.security import HTTPBearer


class AccessTokenBearer(HTTPBearer):
    pass

```
## 2.2>进入books/routes.py文件里面，我们在这里添加令牌验证，先把AccessTokenBearer类导入<br>
<img width="1116" height="754" alt="image" src="https://github.com/user-attachments/assets/7905cd6d-23f3-42ad-be41-bb1b0a9452ee" /><br>
## 2.3>然后用这个类实例化一个对象<br>
<img width="1061" height="777" alt="image" src="https://github.com/user-attachments/assets/2e117d93-c7bd-4743-ba92-c46cd2e500fa" /><br>
## 2.4>然后需要给books相关的路由添加一个user_details参数，需要使用依赖注入，我们先给get_all_books函数添加这个参数<br>
<img width="1384" height="761" alt="image" src="https://github.com/user-attachments/assets/e10c73cf-9fd6-45ef-8826-fa9581f1ed0e" /><br>
## 2.5>然后我们就可以使用postman来测试一下，效果如下<br>
<img width="1336" height="689" alt="image" src="https://github.com/user-attachments/assets/94fa1559-0f28-41e3-93db-94313c7b6ea4" /><br>
## 2.6此时如果你先访问这个路由，你需要先到/login路由里面登录一下，如果把access_token添加到请求头里面<br>
<img width="1482" height="838" alt="image" src="https://github.com/user-attachments/assets/b84787a6-f8e2-4906-9d99-70727b83615f" /><br>
<img width="1504" height="847" alt="image" src="https://github.com/user-attachments/assets/1d9ea2a7-c143-4ebc-b55c-574c15f3e2b7" /><br>
### 然后再发送请求，就可以访问了。<br>
<img width="1506" height="808" alt="image" src="https://github.com/user-attachments/assets/6ab70e29-c3f5-4afe-affe-17ad032e1efb" /><br>
### 注意：这只是最基本的保护，如果你需要更多保护，可以修改这个AccessTokenBearer类。<br>
## 2.7>我们来完善这个AccessTokenBearer类，代码如下<br>
```
from fastapi.security import HTTPBearer
from fastapi.security.http import HTTPAuthorizationCredentials
from fastapi import Request


class AccessTokenBearer(HTTPBearer):
    def __init__(self, auto_error=True):
        super().__init__(auto_error=auto_error)

    async def __call__(self, request: Request) -> HTTPAuthorizationCredentials | None:
        creds = await super().__call__(request)
        print(creds.scheme)
        print(creds.credentials)

        return creds
```
## 2.8>然后我们再次访问/books，就会在控制台输出我们的access_token,说明内部的确可以获取到令牌<br>
<img width="1920" height="376" alt="image" src="https://github.com/user-attachments/assets/674ee56a-d300-4793-8b2a-b6c544c642bf" /><br>
## 2.9>现在我们需要做的就是验证我们的令牌是否有效，还是回到这个AccessTokenBearer类，我们需要导入utils里面的decode_token函数，<br>
<img width="942" height="573" alt="image" src="https://github.com/user-attachments/assets/4dc2a694-c03e-4865-a618-ae1b42cb461a" /><br>

### 先创建一个token_valid函数，在这里验证token是否有效<br>
<img width="926" height="512" alt="image" src="https://github.com/user-attachments/assets/ea26fb5c-7934-4a78-9cd7-a6dac9460a70" /><br>
### 然后我们继续完善__call__方法的代码，完整代码如下<br>
```
from fastapi.security import HTTPBearer
from fastapi.security.http import HTTPAuthorizationCredentials
from fastapi import Request, HTTPException,status
from .utils import decode_token


class AccessTokenBearer(HTTPBearer):
    def __init__(self, auto_error=True):
        super().__init__(auto_error=auto_error)

    async def __call__(self, request: Request) -> HTTPAuthorizationCredentials | None:
        creds = await super().__call__(request)
        token = creds.credentials
        token_data = decode_token(token)
        # 验证token是否是有效
        if not self.token_valid(token):
            raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="Invalid Or Expired Token...")
        # 验证是否是刷新令牌,如果是，说明令牌类型不正确，需要抛出另外一个异常
        if token_data['refresh']:
            raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="Please Provide Valid Access Token...")
        return token_data

    def token_valid(self, token: str) -> bool:
        token_data = decode_token(token)

        return False if token_data is None else True


```
## 3.然后我们给books/routes.py里面的所有路由都添加这个token验证功能<br>
```
from fastapi import FastAPI, status, APIRouter, Depends
from sqlmodel.ext.asyncio.session import AsyncSession
from fastapi.exceptions import HTTPException
from src.db.main import get_session
from src.books.service import BookService
from typing import List
from src.books.schemas import Book, BookUpdateModel, BookCreateModel
from src.auth.dependencies import AccessTokenBearer

book_router = APIRouter()
book_service = BookService()
access_token_bearer = AccessTokenBearer()


@book_router.get("/", response_model=List[Book])
async def get_all_books(session: AsyncSession = Depends(get_session),user_details=Depends(access_token_bearer)):
    return await book_service.get_all_books(session)


@book_router.get("/{book_id}", response_model=Book)
async def get_book(book_id: str, session: AsyncSession = Depends(get_session),user_details=Depends(access_token_bearer)):
    book = await book_service.get_book(book_id, session)
    if book is not None:
        return book
    else:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND, detail="Book not found"
        )


# 创建一本书,有防止重复创建相同的书的功能
@book_router.post("/", status_code=status.HTTP_201_CREATED, response_model=Book)
async def create_book(book_data: BookCreateModel, session: AsyncSession = Depends(get_session),user_details=Depends(access_token_bearer)):
    new_book = await book_service.create_book(book_data, session)
    return new_book


# 部分更新书本
@book_router.patch("/{book_id}", response_model=Book)
async def path_book(book_id: str, update_data: BookUpdateModel, session: AsyncSession = Depends(get_session),user_details=Depends(access_token_bearer)):
    book_to_update = await book_service.update_book(book_id, update_data, session)
    if book_to_update is not None:
        return book_to_update
    else:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Book not found")


# 根据id来删除书本,good
@book_router.delete("/{bid}")
async def delete_book(bid: str, session: AsyncSession = Depends(get_session),user_details=Depends(access_token_bearer)):
    book_to_delete = await book_service.delete_book(bid, session)
    if book_to_delete is not None:
        return {"Book deleted": book_to_delete}
    else:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Book not found")

```
