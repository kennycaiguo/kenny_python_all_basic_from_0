# 1.把chapter13项目复制一份，改名:chapter14-relation-between-model-and-schema,然后用pycharm打开<br>
<img width="631" height="687" alt="image" src="https://github.com/user-attachments/assets/5747ac19-815d-4e21-9296-1ab7bc5d7002" /><br>
# 2.这一节我们来学习数据库模型的一对多，我们想实现的功能是根据用户提交的图书可以找到该用户，也就是建立图书和用户之间的关联，目前他们是没有任何关系的。<br>
## 2.1>要使得他们产生关联，我们可以在books表格里面添加一个user_id字段作为外键，他关联user表格里面的uid<br>
<img width="803" height="441" alt="image" src="https://github.com/user-attachments/assets/7d90a8d2-8e17-414a-aa97-0411051aee9d" /><br>
## 2.2>打开books/models.py,来到Book类的定义，给他添加一个user_id字段<br>
<img width="1240" height="812" alt="image" src="https://github.com/user-attachments/assets/ba16a061-b7a2-4ef1-8249-ea78662f99d8" /><br>
## 2.3>打开终端定位到项目根目录，用alembic来添加一个修订，命令如下<br>
```
alembic revision --autogenerate -m "relate users to books"
```
<img width="1435" height="366" alt="image" src="https://github.com/user-attachments/assets/d62a644c-6abf-4320-bc20-89c9063c765d" /><br>

## 2.4>然后把修订更新到数据库，命令如下<br>
```
alembic upgrade head
```
<img width="1262" height="159" alt="image" src="https://github.com/user-attachments/assets/637fe84f-6223-45e5-95ef-b4c0629ad0a9" /><br>
## 2.4>进入books/routes.py,把所有路由中的user_details参数改名为token_details并且指定类型<br>
<img width="1252" height="922" alt="image" src="https://github.com/user-attachments/assets/4a35971c-b83e-48bb-8683-11cfbf5b8754" /><br>
## 2.5>然后我们修改有效create_book路由函数的代码<br>
<img width="1218" height="309" alt="image" src="https://github.com/user-attachments/assets/99c1b25a-be9a-40cf-9c60-7844049e5c57" /><br>
## 2.6>然后我们需要修改BookServer类里面的create_book方法，给他传递一个user_id,参数，然后修改一下代码<br>
<img width="1305" height="665" alt="image" src="https://github.com/user-attachments/assets/45367175-9d5f-42a8-ae6e-7754be7425ec" /><br>
### 测试，打开postman，登录成功后，用访问令牌创建一本书<br>
<img width="1496" height="837" alt="image" src="https://github.com/user-attachments/assets/eb5e7314-3c96-4b29-a7e5-046f398a8a65" /><br>
### 用navicat17查看数据库，发现新添加的图书已经有user_id了。<br>
<img width="1621" height="581" alt="image" src="https://github.com/user-attachments/assets/e6a6522e-c6e8-448a-be97-d749abbc3756" /><br>
## 2.7>接下来，我们需要创建一个返回一个用户提交的所有书籍的路由<br>
<img width="1313" height="650" alt="image" src="https://github.com/user-attachments/assets/a1a5534c-63c7-4eb1-913f-b3a10de503dc" /><br>
## 2.8>注意：此时BookService类里面还没有这个方法，我们需要在它里面创建一个方法<br>
<img width="1167" height="712" alt="image" src="https://github.com/user-attachments/assets/a9dedc12-9479-4315-86f2-b12c7b523876" /><br>
### 在postman的book文件夹里面新建一个get_books_by_id请求，把当前用户的uid作为参数发送请求，就会得到他提交的所有书籍<br>
<img width="1431" height="831" alt="image" src="https://github.com/user-attachments/assets/7b400913-109b-4057-ac8e-f0302ecdbbee" /><br>
# 3.上面只是基本关联，我们下面来学习关联属性，使得我们无需直接使用id也可以使用关联，我们还可以使用反向关联，我们来实现获取当前用户和他提交的所有数据的功能<br>
## 3.1>打开books/models.py,给Book类添加一个新字段，需要先导入auth包里面的models模块<br>
<img width="961" height="559" alt="image" src="https://github.com/user-attachments/assets/463364c3-8f7e-42e2-8bce-f227bdde829f" /><br>
## 3.2>然后我们给Books类添加一个user字段，它关联User类，需要先导入Relationship<br>
<img width="900" height="560" alt="image" src="https://github.com/user-attachments/assets/ae79d6c1-212b-4cc3-9424-af592c3b7ed8" /><br>
## 3.3>然后我们来创建关联字段<br>
<img width="1142" height="732" alt="image" src="https://github.com/user-attachments/assets/c58c2113-be16-41d5-89e7-e7c0c653f525" /><br>
## 3.4>然后我们进入auth/models,给User类添加一个字段books<br>
<img width="1205" height="687" alt="image" src="https://github.com/user-attachments/assets/199fe9c8-6e1f-446e-8844-ea27b39399fc" /><br>
## 3.5>进入auth/schemas.py,给UserModel添加一个books属性，是typing.List,我们需要先导入它<br>
<img width="930" height="277" alt="image" src="https://github.com/user-attachments/assets/9bc874a1-3ef3-4e57-b084-996ca42a1028" /><br>
### 然后添加books属性，类型需要是src.books.schemas.Book<br>
<img width="1314" height="622" alt="image" src="https://github.com/user-attachments/assets/27b970c3-3ca8-4196-b5c6-3d7d83a7fffa" /><br>
## 3.6>然后我们进入auth/routes.py,我们来给/me路由的返回模型改为UserModel<br>
<img width="1286" height="755" alt="image" src="https://github.com/user-attachments/assets/89d86218-1cd9-4f0f-a6de-8ec3d5238491" /><br>
### 测试一下，我们在postman登录，然后否则token到请求头的Brearer中，然后访问获取当前用户的链接，发现可以返回用户数据，里面有他提交的书籍<br>
<img width="1430" height="831" alt="image" src="https://github.com/user-attachments/assets/31ee46bd-2066-46da-938a-18e518415594" /><br>
### 注意，如果你把返回类型改为数据库模型，你将看不到用户的书籍列表<br>






















