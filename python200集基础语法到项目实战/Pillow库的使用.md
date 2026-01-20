# 1.安装
```
pip install pillow
```
# 2.使用
## 案例1，加载网络图片并且保存
```
import io
'''
加载网络图片并且保存
'''
from PIL import Image
import requests
import numpy as np

# 使用request获取图片，然后用Image类打开并且保存
img_url = "https://i-blog.csdnimg.cn/blog_migrate/00cdaa37834b947c87b02d3307cacea8.png"
pic_data = requests.get(img_url)
img = Image.open(io.BytesIO(pic_data.content))

# 显示图片
img.show()
# 保存图片
img.save("lucky.png",dpi=(300,300))
```
## 案例2.修改图片
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw

'''
加载本地文件并且修改
参考： https://www.geeksforgeeks.org/python/python-pillow-tutorial/
'''

img = Image.open(r"D:\pics\png\mj3.png")
# 输出图片的相关信息
print(img)
# img.show()
# img.save("scene.png")
# 图片旋转
img1 = img.rotate(90)
# img1.show()
# 修改图片大小
img2 = img.resize((img.size[0]*2,img.size[1]*2))
img2.save("sc_big.png")
# img2.show()

# 图片转置
t_img = img.transpose(method=Image.FLIP_TOP_BOTTOM) # 上下翻转
# t_img.show()
t_img2 = img.transpose(method=Image.FLIP_LEFT_RIGHT) # 左右翻转
# t_img2.show()

# 分割通道
r,g,b = img.split()
img_m = Image.merge("RGB",(b,g,r))  # 改变通道的顺序然后合并
# img_m.show()

# 创建缩略图,注意会修改原图，需要先创建一个副本
imgcpy = img.copy()
imgcpy.thumbnail((100,100))
# imgcpy.show()

# 图片模糊效果
blur = img.filter(ImageFilter.BLUR)
# blur.show()

# 高斯模糊
g_blur = img.filter(ImageFilter.GaussianBlur(radius=2))
# g_blur.show()

# 均值模糊
b_blur = img.filter(ImageFilter.BoxBlur(4))
# b_blur.show()

# 给图片添加文本
txt_img = img.copy()
font = ImageFont.truetype("arial",50)
draw = ImageDraw.Draw(txt_img)
draw.text((50,50),"Hello Pillow",(255,0,255),font=font)
txt_img.show()

```
## 案例3，绘制图形
### 1>画线
```
def drawLine():
    w, h = 220, 190
    shape = [(40, 40), (w - 10, h - 10)]
    im = Image.new("RGB", (w, h))
    img = ImageDraw.Draw(im)
    img.line(shape, fill=None, width=0)
    im.show()
```
### 效果：<br>
<img width="271" height="238" alt="{9CD78798-2210-4187-8F98-4CF0DBFC2A12}" src="https://github.com/user-attachments/assets/631cca51-a299-4e1e-977c-a72e5dcb2d20" /><br>
### 2>画矩形<br>
```
from PIL import Image,ImageDraw,ImagePath
def drawRect():
    w, h = 220, 190
    shape = [(40, 40), (w - 10, h - 10)]
    im = Image.new("RGB", (w, h))
    img = ImageDraw.Draw(im)
    img.rectangle(shape, fill="#ffff33", outline='green')
    im.show()
```
### 效果：<br>
<img width="340" height="276" alt="{69A9DA70-DC75-45E7-BD32-5521A4701247}" src="https://github.com/user-attachments/assets/60c273ae-bfce-493a-a147-3dfc12cabcf5" /><br>
### 3>绘制8边形<br>
```
import math
from PIL import Image,ImageDraw,ImagePath

def drawPolygon():
    side = 8
    xy = [
        ((math.cos(th)+1)*90,(math.sin(th)+1)*60) for th in [i*2*math.pi/side for i in range(side)]
    ]
    image = ImagePath.Path(xy).getbbox()
    size = list(map(int,map(math.ceil,image[2:])))
    img = Image.new('RGB',size,"#f9f9f9")
    img_draw = ImageDraw.Draw(img)
    img_draw.polygon(xy,fill="deeppink",outline='gold')
    img.show()
```
### 效果：<br>
<img width="285" height="242" alt="{9002215C-48D0-4422-B779-441C82AD3456}" src="https://github.com/user-attachments/assets/1051de2e-096e-44d2-a46c-ba15c6c8001d" /><br>


## 案例4，图片增强
### 1>颜色增强，代码如下
```
from PIL import Image, ImageEnhance


# 图片增强，颜色
def enhance_color():
    img = Image.open(r"D:\pics\png\mj3.png")
    img2 = ImageEnhance.Color(img)
    img2.enhance(5.0).show()

```
### 效果：<br>
<img width="641" height="643" alt="{8B536870-ABF6-4024-9DFF-140D1B87BE5A}" src="https://github.com/user-attachments/assets/41503d5d-1971-43ca-b36a-d568e586167d" /><br>
### 2>.对比度增强，代码如下
```
from PIL import Image, ImageEnhance

def enhance_contrast():
    img = Image.open(r"D:\pics\png\mj3.png")
    img2 = ImageEnhance.Contrast(img)
    img2.enhance(5.0).show()
```
### 效果：<br>
<img width="640" height="635" alt="{365B30D1-EF49-4C87-AAA3-C52DE4B58DE9}" src="https://github.com/user-attachments/assets/8ce56431-9b09-4238-8924-e6d78af41676" /><br>
### 3>亮度增强，代码如下
```
from PIL import Image, ImageEnhance


def enhance_brightness():
    img = Image.open(r"D:\pics\png\mj3.png")
    img2 = ImageEnhance.Brightness(img)
    img2.enhance(1.5).show()
```
### 效果：<br>
<img width="640" height="640" alt="{CF85CCA0-77B5-4326-986C-F8661F5452F1}" src="https://github.com/user-attachments/assets/a6014553-e47e-4ffe-b7dd-243a2219d494" /><br>
### 4>清晰度增强，代码如下
```
from PIL import Image, ImageEnhance


def enhance_sharpness():
    img = Image.open(r"D:\pics\png\mj3.png")
    img2 = ImageEnhance.Sharpness(img)
    img2.enhance(5.0).show()
```
### 效果：<br>
<img width="639" height="636" alt="{07101ACA-F0A9-45BB-9FDF-D5F0B8143B72}" src="https://github.com/user-attachments/assets/23f09837-86d3-4080-a6ab-d74e84630fb0" /><br>









