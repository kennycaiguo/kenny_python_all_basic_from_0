# kenny_python_all_basic_from_0
python基础大全

# CSDN未登录禁止复制解决办法

不知道何时开始CSDN的内容需要登录后才能复制，这对我这种面相搜索引擎编程的人十分不友好。不过网页上根本禁止不了，懂的都懂，基本上都是靠JS来实现的，只需禁用JS即可破解，下面是网友提供的两种解决CSDN未登录禁止复制的办法。

方法一

打开一个CSDN网站，按下F12进入开发者模式。注意此模式在IE内核中无效。然后在开发者模式中选择Console面板，复制如下命令粘贴并回车执行。

javascript:document.body.contentEditable=true&;document.designMode="on"; <br>

<img width="598" height="193" alt="image" src="https://github.com/user-attachments/assets/651f3814-4deb-4b77-973d-f099923fdda7" /> <br>



方法二

打开一个CSDN网站，按下F12进入开发者模式。注意此模式在IE内核中无效。然后在开发者模式中选择Elements面板，找到 id=&#34;content_views&#​34; 这句，将其中的 content_views 删除或者重命名即可。双击即可修改。 <br>

<img width="599" height="286" alt="image" src="https://github.com/user-attachments/assets/9ea13845-4b0c-4913-894a-c9881bfb8d38" /> <br>

# 浏览器的console不让粘贴？<br>
<img width="1252" height="627" alt="{77B3D8F1-6809-428E-8E31-F59DAF8E424C}" src="https://github.com/user-attachments/assets/aeffe7a4-caaa-4d30-b174-7dd9746e579a" /> <br>
<img width="1261" height="254" alt="{06ED010F-3373-4AB2-9E7F-DFDC948F989F}" src="https://github.com/user-attachments/assets/78d89b93-2d12-4c0d-a2f5-bab38c6f5f96" /><br>
<img width="1258" height="466" alt="{66C9C28B-E964-417F-BEDC-3E88CE345C23}" src="https://github.com/user-attachments/assets/e76fdc58-20a3-4b36-a07b-0e87398b5c03" /><br>
<img width="1236" height="553" alt="{74B8B9EE-713C-4688-BD76-3B35E775DECB}" src="https://github.com/user-attachments/assets/938d1abe-9cc3-442c-aec9-34c9c53135e4" /><br>


