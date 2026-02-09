# 1.把chapter14项目复制一份，改名：chapter15-add_review-and-more-relation,然后用pycharm打开<br>
<img width="512" height="674" alt="image" src="https://github.com/user-attachments/assets/47e4226c-9cef-48a1-9c34-6c4d6d74135a" /><br>
# 2.这一节我们来学习更多的关系，并且我们会创建书籍的评论功能<br>
## 2.1>在我们继续之前，我们需要修复一个错误：就是现在当我们使用/signup路由创建一个新用户，在数据库里面新增用户成功，都是服务器会出错，错误是缺少greenlet。解决办法，我们在采集用户的时候，
## 不应该返回一个有书籍列表的模型，所以我们需要撤销对UserModel类的更改，然后新建一个UserBooksModel从UserModel里面继承<br>
## 2.2>然后我们修改auth/routes.py，把/me路由对应的路由函数的返回类型改为UserBooksModel<br>
<img width="1229" height="390" alt="image" src="https://github.com/user-attachments/assets/f54dee04-e660-4405-922c-485be04f1124" /><br>
### 然后我们重新发送请求，发现用户已经创建成功<br>
<img width="1437" height="856" alt="image" src="https://github.com/user-attachments/assets/4434e0be-6c78-4c4f-9da7-da36d2cb56c2" /><br>
## 2.3>然后我们用这个账户来登录，没有问题<br>
<img width="1459" height="824" alt="image" src="https://github.com/user-attachments/assets/16d14b25-b7f3-4db1-b239-40f898504a01" /><br>
## 2.4>然后我们用这个账户来提交2本书，创建成功<br>
<img width="1445" height="809" alt="image" src="https://github.com/user-attachments/assets/b529ff46-13ad-452c-a019-b7b6e2dfc3dc" /><br>
<img width="1540" height="948" alt="image" src="https://github.com/user-attachments/assets/f3c2bbd3-42b9-4d8f-9c4b-fee55388ad6e" /><br>
## 2.5>然后我们来查看当前用户信息，可以得到用户的信息包括它提交的书籍列表<br>
<img width="1481" height="936" alt="image" src="https://github.com/user-attachments/assets/b089b0a9-9d22-4300-a883-4a1e4c1f768e" /><br>
# 3.今天，我们需要修改一下数据库的关系模型，添加评论功能，关系模型如图<br>
<img width="926" height="638" alt="image" src="https://github.com/user-attachments/assets/c7b33ebf-6699-4a7c-a691-dbb76c298508" /><br>













