# 1.安装fastapi<br>
```
安装 FastAPI 很简单，这里我们使用 pip 命令来安装。

pip install fastapi
也可以使用以下命令直接安装 FastAPI 及所有可选依赖:

pip install "fastapi[all]" 或者pip install "fastapi[standard]"
这会安装：

fastapi - FastAPI 框架
uvicorn[standard] - ASGI 服务器
python-multipart - 表单和文件上传支持
jinja2 - 模板引擎
python-jose - JWT 令牌支持
passlib - 密码哈希
bcrypt - 密码加密
python-dotenv - 环境变量支持
另外我们还需要一个 ASGI 服务器，生产环境可以使用 Uvicorn 或者 Hypercorn：

pip install "uvicorn[standard]"
这样我们就安装完成了。
```
## 我们这里使用pip install "fastapi[all]"的安装方式因为我们需要安装fastapi-cli,用最简单的方式是不会安装这个组件的。<br>
<img width="1217" height="637" alt="image" src="https://github.com/user-attachments/assets/fd1e1351-4ac8-4eec-beb6-7f6894a0b3fa" />

# 2.验证安装
## 打开一个cmd窗口，输入fastapi --version，有输出说明cli安装成功<br>
<img width="654" height="118" alt="image" src="https://github.com/user-attachments/assets/b7c4570e-1073-4228-851c-fb5dbfe36feb" /><br>
## 打开python控制台，输入from fastapi import FastAPI，没有报错，说明fastapi安装成功<br>
<img width="360" height="61" alt="{40E5A2A9-4323-4CB6-B4C7-2CDA2FE6E59C}" src="https://github.com/user-attachments/assets/04316618-9d35-45c0-9b31-13aa1b811507" /><br>


