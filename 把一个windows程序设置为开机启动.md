# 1.以postgres数据库服务为例，创建一个startposgres.cmd，内容如下<br>
```
sc start postgresql-x64-17
```
# 2.然后打开注册表编辑器，找到： 计算机\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Run，新建一个字符串值，内容如下
<img width="1115" height="511" alt="image" src="https://github.com/user-attachments/assets/c34d6bb5-a915-4417-a7c2-2ed4778a7022" />
# 3.然后保存退出，重新启动电脑。
