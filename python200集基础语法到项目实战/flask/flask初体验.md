# 1.flask安装： 
## 1> pip install flask --项目框架
## 2> pip install flask-script --flask扩展，可以在命令行里面启动项目或者进行数据库操作

# 2.新建一个flask项目,如果你是使用pycharm专业版，你可以直接创建一个flask项目，都是这里为了和老师同步，我们使用原始的方法来创建<br>
<img width="1005" height="751" alt="image" src="https://github.com/user-attachments/assets/e7921599-6e3a-475b-a308-88afa31934c4" /><br>
<img width="851" height="455" alt="image" src="https://github.com/user-attachments/assets/59316ca5-1f78-4149-bfec-af9640641632" /><br>


# 3.这个manage.py就是我们项目的启动文件，我们需要添加下面的内容
```
from flask import Flask
from flask_script import Manager

# 创建应用程序对象
app = Flask(__name__)
# 把项目交给manager对象来管理
manager = Manager(app)

if __name__ == '__main__':  # 标记启动文件

    manager.run()
```
# 4.启动项目，
## 打开一个终端，输入：python manage.py runserver

