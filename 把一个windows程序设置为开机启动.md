# 方法一
# 1.以postgres数据库服务为例，创建一个startposgres.cmd，内容如下<br>
```
net start postgresql-x64-17
```
# 2.然后打开注册表编辑器，找到： 计算机\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Run，新建一个字符串值，内容如下
<img width="1115" height="511" alt="image" src="https://github.com/user-attachments/assets/c34d6bb5-a915-4417-a7c2-2ed4778a7022" /><br>
# 3.然后保存退出，重新启动电脑。<br>

# 方法二，如果上面的方法没有用，可以使用任务计划具体操作如下：
## 1>在计算机图标上面点击右键-》更多选项-》管理-》任务计划，就会出现下面的界面<br>
<img width="1249" height="889" alt="image" src="https://github.com/user-attachments/assets/e486eb3e-f091-47cf-9016-c7977524f763" /><br>
## 2>点击右边的创建任务（不要创建基本任务，没有用），就会弹出下面的界面，我们填写好任务名称和描述<br>
<img width="838" height="721" alt="image" src="https://github.com/user-attachments/assets/d3ded26e-18d5-4a27-bbfd-b074f12421b3" /><br>
## 3>点触发器选项卡，新建一个触发器，只需要把开始任务那一栏设置为启动时，其他默认，点击确定<br>
<img width="1112" height="716" alt="image" src="https://github.com/user-attachments/assets/f8db7c24-6f97-4c10-869a-069aaccb17bc" /><br>
## 4>点击操作选项卡，然后点击新建按钮，在弹出的对话框中选择我们需要启动的脚本程序，然后点击确定<br>
<img width="1013" height="778" alt="image" src="https://github.com/user-attachments/assets/5f89e59c-0fa0-4281-9c53-95f02f5e4f15" /><br>
## 5>点击条件选项卡，取消勾选所有，然后点击确定，设置选项卡可以不设置<br>
<img width="788" height="664" alt="image" src="https://github.com/user-attachments/assets/4185fb2c-7ada-4809-86da-2b98e20651fb" /><br>
## 6>设置完成，重启电脑<br>




