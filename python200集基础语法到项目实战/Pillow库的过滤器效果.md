## 1. 转化为灰度图
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 转化为灰度图
def to_gray():
    image = Image.open("./lucky.png")

    gray = image.convert("L")
    gray.show()


if __name__ == '__main__':
    to_gray()
```
## 效果<br>
<img width="785" height="788" alt="{31B18776-0A91-4089-B55F-EB58D1167441}" src="https://github.com/user-attachments/assets/01bbaff7-9501-4203-9d33-b62fc917a4ee" /><br>

## 浮雕效果<br>
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 图片过滤,浮雕效果
def filter_emboss():
    image = Image.open("./lucky.png")
    filtered = image.filter(ImageFilter.EMBOSS)
    filtered.show()


if __name__ == '__main__':
    filter_emboss()
```
## 效果<br>
<img width="779" height="780" alt="{1A35C281-824A-4B96-BBF9-00BD608D5FB8}" src="https://github.com/user-attachments/assets/93fa0834-e156-468d-9932-0b0cc76aab43" /><br>

## 铅笔画效果<br>
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 图片过滤,铅笔画效果
def filter_contour():
    image = Image.open("./lucky.png")
    filtered = image.filter(ImageFilter.CONTOUR)
    filtered.show()


if __name__ == '__main__':
    filter_contour()
```
## 效果<br>
<img width="779" height="756" alt="{A43D3B72-F1D1-4694-BEB6-C937172B3459}" src="https://github.com/user-attachments/assets/03eea27c-ef46-49da-8e15-08245d37b103" /><br>

## 细节增强滤波<br>
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 细节增强滤波
def filter_detail():
    image = Image.open("./cats.png")
    filtered = image.filter(ImageFilter.DETAIL)
    filtered.show()


if __name__ == '__main__':
    filter_detail()
```
## 效果<br>
<img width="1040" height="767" alt="{FDF79751-7098-4935-A625-8BD1C27B24CD}" src="https://github.com/user-attachments/assets/a83b8432-3acc-4583-9baf-d278b75d49d8" /><br>


## 边缘增强<br>
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 边缘增强
def filter_edge_enhance():
    image = Image.open("./cats.png")
    filtered = image.filter(ImageFilter.EDGE_ENHANCE)
    filtered.show()


if __name__ == '__main__':
    filter_edge_enhance()
```
## 效果<br>
<img width="1045" height="767" alt="{91990FA8-1E00-4865-AE32-0379087DCF99}" src="https://github.com/user-attachments/assets/c31a491d-0985-44da-aa32-d58cf47db101" /><br>


## 更多边缘增强<br>
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 更多边缘增强
def filter_edge_enhance_more():
    image = Image.open("./cats.png")
    filtered = image.filter(ImageFilter.EDGE_ENHANCE_MORE)
    filtered.show()


if __name__ == '__main__':
    filter_edge_enhance_more()
```
## 效果<br>
<img width="1044" height="764" alt="{29499944-DE25-4184-894C-FB4B44770878}" src="https://github.com/user-attachments/assets/530b3960-d093-4d1d-92c7-ddbc7d7e8428" /><br>

## 查找边缘<br>
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 查找边缘
def filter_find_edges():
    image = Image.open("./cats.png")
    filtered = image.filter(ImageFilter.FIND_EDGES)
    filtered.show()


if __name__ == '__main__':
    filter_find_edges()
```
## 效果<br>
<img width="1042" height="780" alt="{991987C9-43A7-4668-9D50-A42124415C5F}" src="https://github.com/user-attachments/assets/91bae09b-a9b6-45dd-b73a-455f49f0652f" /><br>


## 平滑滤波<br>
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 平滑滤波
def filter_smooth():
    image = Image.open("./cats.png")
    filtered = image.filter(ImageFilter.SMOOTH)
    filtered.show()


if __name__ == '__main__':
    filter_smooth()

```
## 效果<br>
<img width="1040" height="765" alt="{CEA77ED7-885E-4D11-B097-233AF6ED39DC}" src="https://github.com/user-attachments/assets/420ffdf6-3f9b-4f40-be55-ef2ca3a470a9" /><br>


## 加强平滑滤波<br>
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 加强平滑滤波
def filter_smooth_more():
    image = Image.open("./cats.png")
    filtered = image.filter(ImageFilter.SMOOTH_MORE)
    filtered.show()


if __name__ == '__main__':
    filter_smooth_more()

```
## 效果<br>
<img width="1041" height="761" alt="{D9815407-9BCB-4351-B664-5098B34D63F2}" src="https://github.com/user-attachments/assets/c765219e-f34b-4254-a51d-3703c93cfc2c" /><br>


##  锐化滤波<br>
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 锐化滤波
def filter_sharpen():
    image = Image.open("./cats.png")
    filtered = image.filter(ImageFilter.SHARPEN)
    filtered.show()


if __name__ == '__main__':
    filter_sharpen()

```
## 效果<br>
<img width="1042" height="765" alt="{88DC14FB-36C3-4DDE-9A4D-741AE01B4AC2}" src="https://github.com/user-attachments/assets/c9c2afe4-29df-45a5-bd1c-24b49f983553" /><br>


## 卷积核处理Kernel<br>
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 卷积核处理Kernel
def filter_kernel_demo():
    image = Image.open("./cats.png")
    result = image.filter(ImageFilter.Kernel((3, 3), (1, 1, 1, 0, 0, 0, 2, 0, 2)))
    result.show()


if __name__ == '__main__':
   filter_kernel_demo()

```
## 效果<br>
<img width="1037" height="763" alt="{E29430C6-BFEB-4DB2-902D-A055201A9A8E}" src="https://github.com/user-attachments/assets/ef831002-6aed-4ee0-8686-d5e937fdc454" /><br>


## 等级滤波器<br>
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 等级滤波器
def filter_rankfilter():
    image = Image.open("./cats.png")
    filtered = image.filter(ImageFilter.RankFilter(5, 24))
    filtered.show()


if __name__ == '__main__':
    filter_rankfilter()

```
## 效果<br>
<img width="1043" height="764" alt="{B35DF886-6E04-4B11-B522-E85183C4BC3B}" src="https://github.com/user-attachments/assets/02ff5af3-89fd-4272-a9d0-4a55a9e891a2" /><br>


## 最小滤波器<br>
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 最小滤波器
def filter_minfilter():
    image = Image.open("./cats.png")
    filtered = image.filter(ImageFilter.MinFilter(5))
    filtered.show()


if __name__ == '__main__':
    filter_minfilter()

```
## 效果<br>
<img width="1040" height="760" alt="{B422A91B-0EE0-47A0-9C38-36333994477C}" src="https://github.com/user-attachments/assets/1046c120-1221-4229-9143-df59ce541ea4" /><br>


## 中值滤波<br>
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 中值滤波
def filter_medianfilter():
    image = Image.open("./cats.png")
    filtered = image.filter(ImageFilter.MedianFilter(5))
    filtered.show()


if __name__ == '__main__':
    filter_medianfilter()
```
## 效果<br>
<img width="1040" height="764" alt="{C9B34DD3-BF34-4F7D-8898-C1651CC2FB49}" src="https://github.com/user-attachments/assets/3739fe36-c6d4-4f20-a318-23a8d7bfec71" /><br>


## 最大滤波器<br>
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 最大滤波器
def filter_maxfilter():
    image = Image.open("./cats.png")
    filtered = image.filter(ImageFilter.MaxFilter(5))
    filtered.show()


if __name__ == '__main__':
    filter_maxfilter()
```
## 效果<br>
<img width="1036" height="762" alt="{679C3B01-AC65-4426-88A6-E7AACE69A8EA}" src="https://github.com/user-attachments/assets/9fea5421-f627-4c4d-9512-06e42df8e1ba" /><br>


## 模式滤波器<br>
```
from PIL import Image, ImageFilter, ImageFont, ImageDraw


# 模式滤波器
def filter_modefilter():
    image = Image.open("./cats.png")
    filtered = image.filter(ImageFilter.ModeFilter(5))
    filtered.show()


if __name__ == '__main__':
    filter_modefilter()

```
## 效果<br>
<img width="1042" height="762" alt="{AC86A5EA-2C7B-470A-B9C1-517F5A6EA8CF}" src="https://github.com/user-attachments/assets/26eab199-f465-49e3-a2e9-c1287dfde535" /><br>
## 效果


## <br>
```


```
## 效果<br>


## <br>
```


```
## 效果<br>


## <br>
```


```
## 效果<br>


## <br>
```


```
## 效果<br>







