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
### 图片旋转
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw

# 图片旋转
def rotate_img():
    img = Image.open(r"D:\pics\png\mj3.png")
    img1 = img.rotate(90) # 注意：可以使用center参数来设置旋转点如： img1 = img.rotate(90,center=(0,0))
    img1.show()

if __name__ == '__main__':
    rotate_img()

```
### 效果：<br>
<img width="634" height="633" alt="{C84DBE58-AF4F-4E46-A33F-88312C42588D}" src="https://github.com/user-attachments/assets/0a3f2ea5-48f1-47c3-956e-fc3e634d85c5" /><br>

### 修改图片大小
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


def resize_pic():
    img = Image.open(r"D:\pics\png\mj3.png")
    # 修改图片大小
    img2 = img.resize((img.size[0] * 2, img.size[1] * 2))
    # img2.save("sc_big.png")
    img2.show()


if __name__ == '__main__':
    resize_pic()
```
### 效果：<br>
<img width="778" height="776" alt="{3AFD4550-D47C-4B33-BF98-71CBF05495C9}" src="https://github.com/user-attachments/assets/68245d33-0b9f-4208-a939-ef78883d912a" /><br>
### 图片上下翻转
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


def transpos_pic1():
    img = Image.open(r"D:\pics\png\mj3.png")
    # 图片转置
    t_img = img.transpose(method=Image.FLIP_TOP_BOTTOM)  # 上下翻转
    t_img.show()


if __name__ == '__main__':
    transpos_pic1()
```
### 效果：<br>
<img width="640" height="640" alt="{C5454AD6-7BB3-448C-A07C-C5A70F148AA0}" src="https://github.com/user-attachments/assets/3862a5a6-4aeb-4a6c-97af-df8b996fafb6" /> <br>


### 图片左右翻转
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


def transpos_pic2():
    img = Image.open(r"D:\pics\png\mj3.png")
    # 图片转置
    t_img2 = img.transpose(method=Image.FLIP_LEFT_RIGHT)  # 左右翻转
    t_img2.show()


if __name__ == '__main__':
    transpos_pic2()
```
### 效果：<br>
<img width="638" height="634" alt="{666FA1C4-4880-4392-9BF1-01E808399922}" src="https://github.com/user-attachments/assets/cdbbae01-8186-4c9b-9f2f-352dcfc7dd5b" /><br>

### 图片通道分割和合并
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


def split_merge():
    img = Image.open(r"D:\pics\png\mj3.png")

    # 分割通道，然后合并，改变图片通道顺序就可以改变图片颜色
    r, g, b = img.split()
    img_m = Image.merge("RGB", (b, g, r))  # 改变通道的顺序然后合并
    img_m.show()


if __name__ == '__main__':
    split_merge()
```
### 效果：<br>
<img width="639" height="640" alt="{F737DEF2-D670-4FB3-9D35-3BB9BED8E737}" src="https://github.com/user-attachments/assets/f41c472a-98af-4f73-8a26-98c72382ae59" /><br>

### 创建图片的缩略图,注意：会改变原图，需要用他的副本来做
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


def create_thumnail():
    img = Image.open(r"D:\pics\png\mj3.png")
    # 创建缩略图,注意会修改原图，需要先创建一个副本
    imgcpy = img.copy()
    imgcpy.thumbnail((100, 100))
    imgcpy.show()


if __name__ == '__main__':
    create_thumnail()
```
### 效果：<br>
<img width="124" height="125" alt="{0459EE61-1232-4E39-9C73-B944ABB6EBC9}" src="https://github.com/user-attachments/assets/1d2ab168-414d-4ba5-981d-b7c28d85c22d" /><br>

### 图片普通模糊
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


def blur_demo():
    img = Image.open(r"D:\pics\png\mj3.png")
    # 图片模糊效果
    blur = img.filter(ImageFilter.BLUR)
    blur.show()


if __name__ == '__main__':
    blur_demo()
```
### 效果：<br>
<img width="640" height="638" alt="{8F508E76-808F-450C-A969-0D35C591CF9D}" src="https://github.com/user-attachments/assets/dc785ba4-6443-4976-91fc-007c190e8bfc" /><br>

### 图片高斯模糊
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


def gaussianblur_demo():
    img = Image.open(r"D:\pics\png\mj3.png")
    # 高斯模糊
    g_blur = img.filter(ImageFilter.GaussianBlur(radius=2))
    g_blur.show()


if __name__ == '__main__':
    gaussianblur_demo()
```
### 效果：<br>
<img width="639" height="641" alt="{66828281-68B0-4E60-82B1-C9CBDC206370}" src="https://github.com/user-attachments/assets/eabf824b-6abd-41a7-be53-1fc0f08e0704" /><br>

### 图片均值模糊
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


def boxblur_demo():
    img = Image.open(r"D:\pics\png\mj3.png")
    # 均值模糊
    b_blur = img.filter(ImageFilter.BoxBlur(4))
    b_blur.show()


if __name__ == '__main__':
    boxblur_demo()
```
### 效果：<br>
<img width="638" height="639" alt="{08790150-8277-48DD-9F39-F64FD6F9ECD8}" src="https://github.com/user-attachments/assets/2ece0289-a20a-4deb-b415-0153755d3f1a" />

### 给图片添加文本
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


def add_text_to_pic():
    img = Image.open(r"D:\pics\png\mj3.png")
    # 给图片添加文本
    txt_img = img.copy()
    font = ImageFont.truetype("arial", 50)
    draw = ImageDraw.Draw(txt_img)
    draw.text((50, 50), "Hello Pillow", (255, 0, 255), font=font)
    txt_img.show()


if __name__ == '__main__':
    add_text_to_pic()
```
### 效果：<br>
<img width="637" height="638" alt="{1DB6147D-9725-4FBD-8466-C53CE5D09604}" src="https://github.com/user-attachments/assets/c5d12311-43a6-4885-8782-88606393275b" /><br>

## 裁剪图片<br>
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 图片裁剪
def crop_pic():
    img = Image.open("./lucky.png")
    cropped = img.crop(box=(220,80,785,600))
    cropped.show()


if __name__ == '__main__':
    crop_pic()
```
### 效果：<br>
<img width="710" height="649" alt="{B05F2A57-40E5-4ADD-A85F-AAEE2971F518}" src="https://github.com/user-attachments/assets/84cca821-8151-4a13-bea4-371f405fafc2" /><br>

## 图片马赛克
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 图片马赛克
def pic_mosaic():
    # 原图片
    img = Image.open("./sexy.png")
    # 作为马赛克的图片
    mosaic_src = Image.open("./lucky.png")
    # 图片太大，把他变小
    mosaic_pic = mosaic_src.copy()
    mosaic_pic.thumbnail(size=(50, 50))
    # mosaic_pic.show()
    img.paste(mosaic_pic, box=(135, 180))  # 这个方法没有返回值
    img.paste(mosaic_pic, box=(215, 180))
    img.paste(mosaic_pic, box=(180, 350))
    img.show()


if __name__ == '__main__':
    pic_mosaic()
```
### 效果：<br>
<img width="503" height="624" alt="{5B5CF6BE-D004-4D2B-81CF-8805A62382E1}" src="https://github.com/user-attachments/assets/8ed7c802-f030-4bcc-ab01-29892fa8318a" /><br>

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









