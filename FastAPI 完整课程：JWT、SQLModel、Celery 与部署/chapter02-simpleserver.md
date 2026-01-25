## 0.新建一个SimpleServer项目，然后进入这个文件夹<br>
<img width="499" height="370" alt="{F238B5AA-6BCA-4FFD-8AF8-D06595871456}" src="https://github.com/user-attachments/assets/4250f6be-512a-46e9-a654-9b1fdd90b41b" /><br>


## 1.在我们项目的根目录里面创建一个main.py，内容如下
```
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def home():
    return {"message":"Hello World From FastApi..."}
```
## 2.然后打开一个终端定位到当前文件夹，输入fastapi dev main.py,然后点击程序生成的链接，就可以打开这个页面<br>
<img width="1238" height="564" alt="image" src="https://github.com/user-attachments/assets/f213fa9e-e3e7-42ef-af5a-12d5b2ca8597" /><br>
<img width="603" height="268" alt="image" src="https://github.com/user-attachments/assets/ef362f6e-5012-40c9-a009-2e8db4ded68d" /><br>

## 3.然后我们可以添加一个路由，是一个带参数的路由
```
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def home():
    return {"message": "Hello World From FastApi..."}


@app.get("/greet/{username}") # 带参数的路由
async def greet(username:str) -> dict: # 变量类型和返回值可以省略
    return {"message": f"Hello, {username}"}
```
### 测试一下：<br>
<img width="684" height="206" alt="image" src="https://github.com/user-attachments/assets/499dee7b-1f04-4031-b4c8-e667d4259b33" /><br>
### 3.1为了更好的此时api，我们使用postman web+postman agent来做客户端网址：https://www.postman.com/，当然，还可以使用restfox：https://restfox.dev/，我们还是使用postman<br>

## 4.然后我们添加一个使用querystring的路由
```
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def home():
    return {"message": "Hello World From FastApi..."}


@app.get("/greet/{username}")  # 带参数的路由
async def greet(username: str) -> dict:
    return {"message": f"Hello, {username}"}


# querystring
user_list = [
    "Jerry",
    "Joey",
    "Phil"
]


@app.get("/search")
async def do_search(username: str):
    if username in user_list:
        return {"result": "User exists"}
    else:
        return {"result": "User not exists"}

```
## 效果如下<br>
<img width="908" height="624" alt="image" src="https://github.com/user-attachments/assets/34cc835d-298e-4852-ad38-3c0f1a7125d8" /><br>
<img width="1505" height="657" alt="image" src="https://github.com/user-attachments/assets/235bffd7-ca84-420a-84a5-8c0300ca1330" /><br>

## 5.同时使用路径参数和querystring,非常简单，只要我们没有在路径中提供的函数参数，她就自动错误查询参数
```
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def home():
    return {"message": "Hello World From FastApi..."}


@app.get("/greet/{username}")  # 带参数的路由
async def greet(username: str) -> dict:
    return {"message": f"Hello, {username}"}


# querystring
user_list = [
    "Jerry",
    "Joey",
    "Phil"
]


@app.get("/search")
async def do_search(username: str):
    if username in user_list:
        return {"result": "User exists"}
    else:
        return {"result": "User not exists"}


# 5.同时使用路径参数和查询参数： http://127.0.0.1:8000/info/Jack?age=20&address=kingston&email=Jack123@gmail.com
@app.get("/info/{name}")
async def user_info(name, age, address, email):
    return {"Name": f"{name}", "age": age, "address": address, "email": email}
```
## 效果如下<br>
<img width="1419" height="735" alt="image" src="https://github.com/user-attachments/assets/f017a946-2973-4544-a910-f2cdb34a4df7" /><br>

# 6.可选查询参数
```
from fastapi import FastAPI
from typing import Optional

app = FastAPI()


@app.get("/")
async def home():
    return {"message": "Hello World From FastApi..."}


@app.get("/greet/{username}")  # 带参数的路由
async def greet(username: str) -> dict:
    return {"message": f"Hello, {username}"}


# querystring
user_list = [
    "Jerry",
    "Joey",
    "Phil"
]


@app.get("/search")
async def do_search(username: str):
    if username in user_list:
        return {"result": "User exists"}
    else:
        return {"result": "User not exists"}


# 同时使用路径参数和查询参数： http://127.0.0.1:8000/info/Jack?age=20&address=kingston&email=Jack123@gmail.com
@app.get("/info/{name}")
async def user_info(name, age, address, email):
    return {"Name": f"{name}", "age": age, "address": address, "email": email}


# 可选查询参数
@app.get("/show/")
async def show_data(page: Optional['int'] = 1):
    return {"message": f"current page:{page}"}
```
## 效果如下<br>
<img width="1404" height="637" alt="image" src="https://github.com/user-attachments/assets/2cff0369-f304-4da1-946f-d32892a0eaa6" /><br>
<img width="1434" height="692" alt="image" src="https://github.com/user-attachments/assets/143598c5-fb02-4029-b021-9e10fbed5ef5" /><br>



