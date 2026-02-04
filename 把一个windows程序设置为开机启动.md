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
## 2>点击右边的创建基本任务，就会弹出下面的界面，我们填写好任务名称和描述<br>
<img width="876" height="684" alt="image" src="https://github.com/user-attachments/assets/fce2b490-915d-42b3-8603-c9a6f9b95689" /><br>
## 3>点击下一页，在触发器哪里点击当计算机启动时<br>
<img width="872" height="684" alt="image" src="https://github.com/user-attachments/assets/4c8fc19b-680a-46dd-8225-a44020511d24" /><br>
## 4>点击下一页，保持默认<br>
<img width="874" height="681" alt="image" src="https://github.com/user-attachments/assets/d4fb514c-5e85-428c-a46d-ba3969de3046" /><br>
## 5>点击下一页，最后，选择我们的脚本（就是方法一里面创建的那个基本），然后点击下一页完成，重启电脑<br>
<img width="859" height="693" alt="image" src="https://github.com/user-attachments/assets/475b2b31-68d5-4d98-8874-52ae7ee87970" /><br>
## 6>点击下一页,然后点击下一页完成，重启电脑<br>




