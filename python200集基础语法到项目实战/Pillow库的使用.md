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

```
