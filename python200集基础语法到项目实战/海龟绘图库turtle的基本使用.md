# 1.turtle海龟绘图<br>
## 1.画板设置p14<br>
```
import turtle
# 设置画板大小
# 宽度 高度 画板的背景色(默认是白色)
# 写法1
turtle.screensize(200,400,'pink')
# 写法2 宽度 高度  startx,starty
# turtle.setup(100,100,0,0) # 很小
# 控制关闭画板界面
turtle.done()
```
## 2.画笔的操作p15<br>
### 画笔的形态<br>
```
import turtle
# 画笔的操作
# 画笔的形态，默认是尖角形态
'''
turtle 海龟心态
circle 圆点
'''
turtle.shape('turtle')
# 画笔的颜色,turtle.pencolor('颜色英文名称')或者turtle.pencolor(r,g,b) r,g,b可以0-255或者0-1
# turtle.pencolor('purple')
# turtle.pencolor(0.3,0.7,0.9) # 0-1
turtle.colormode(255)
turtle.pencolor(200,70,150)
# 设置画笔的宽度
turtle.pensize(4)
# 上涨绘画速度0-10
turtle.speed(10)
# 绘制一条直线
turtle.forward(100)

# 控制关闭画板界面
turtle.done()
```
## 3.画板的坐标系p16<br>
### 坐标分为2种：<br>
### 海龟画板的原点(0,0)在画板的中心位置
#### 1>绝对坐标使用`goto(x,y)`
#### 2>相对指标如`turtle.forword(x)`,`turtle.backword(x)`
### 运动方向
#### 1.left(angle)逆时针或者right(angle)顺时针
#### 2.setheading(angle),正数是顺时针旋转，负数是逆时针旋转
### 小案例，用turtle绘制一个矩形
```
import turtle

# 设置画板大小
# 宽度 高度 画板的背景色(默认是白色)
# 写法1
turtle.screensize(200,400,'pink')
# 写法2 宽度 高度  startx,starty
# turtle.setup(100,100,0,0) # 很小
# 画笔的操作
# 画笔的形态，默认是尖角形态
'''
turtle 海龟心态
circle 圆点
'''
turtle.shape('turtle')
# 画笔的颜色,turtle.pencolor('颜色英文名称')或者turtle.pencolor(r,g,b) r,g,b可以0-255或者0-1
# turtle.pencolor('purple')
# turtle.pencolor(0.3,0.7,0.9) # 0-1
turtle.colormode(255)
turtle.pencolor(200,70,150)
# 设置画笔的宽度
turtle.pensize(4)
# 上涨绘画速度0-10
turtle.speed(10)
# 3.坐标
# 绘制矩形
turtle.forward(100)
# 绝对坐标
turtle.goto(100,100)
turtle.goto(0,100)
turtle.goto(0,0)
# 旋转乌龟头
# turtle.seth(90) # 正数是顺时针旋转，负数是逆时针旋转
# left(angle) 逆时针
# turtle.left(30)
# right(angle) 顺时针
turtle.right(45)

# 控制关闭画板界面
turtle.done()
```
### 效果：<br>
<img width="955" height="851" alt="{ED3F4FFC-D529-4CD5-A136-6C3F37FADE2A}" src="https://github.com/user-attachments/assets/ac0ee30a-4eeb-49a9-adaf-36de101af0fd" /><br>
### 还可以这么写<br>
```
import turtle

# 设置画板大小
# 宽度 高度 画板的背景色(默认是白色)
# 写法1
turtle.screensize(200,400,'pink')
# 写法2 宽度 高度  startx,starty
# turtle.setup(100,100,0,0) # 很小
# 画笔的操作
# 画笔的形态，默认是尖角形态
'''
turtle 海龟心态
circle 圆点
'''
turtle.shape('turtle')
# 画笔的颜色,turtle.pencolor('颜色英文名称')或者turtle.pencolor(r,g,b) r,g,b可以0-255或者0-1
# turtle.pencolor('purple')
# turtle.pencolor(0.3,0.7,0.9) # 0-1
turtle.colormode(255)
turtle.pencolor(200,70,150)
# 设置画笔的宽度
turtle.pensize(4)
# 上涨绘画速度0-10
turtle.speed(10)
# 3.坐标.用相对坐标绘制矩形
turtle.forward(100)

turtle.left(90)
turtle.forward(100)

turtle.left(90)
turtle.forward(100)

turtle.left(90)
turtle.forward(100)
# 控制关闭画板界面
turtle.done()
```
### 效果是一样的，使用相对坐标就需要注意海龟头的方向<br>
<img width="960" height="848" alt="{EE8B802A-B25B-4BEA-BE1D-4F359E62B9F9}" src="https://github.com/user-attachments/assets/ab6579c4-cffa-47fa-a26f-17d2792ccba8" />
# 图形的绘制p17<br>
## 矩形，就是旋转海龟头和控制海龟的向前或者向后移动就可以绘制出来，参考说明的案例
## 圆形，`turtle.circle(radius,extent,step)` 也就是半径，弧度，步数，步数越大越圆滑，这个函数可以绘制圆，圆弧和多边形
## 绘制点，`turtle.dot(直径，颜色)`
```
import turtle

# 设置画板大小
# 宽度 高度 画板的背景色(默认是白色)
turtle.screensize(400.400)
# 画笔的操作
# 设置画笔的宽度
turtle.pensize(4)
# 画笔的形态，默认是尖角形态
turtle.shape('turtle')
# 设置画笔的颜色
turtle.pencolor("deeppink")
# 设置绘画速度0-10
turtle.speed(10)
# 绘制圆形
# turtle.circle(100,360,300)
turtle.circle(100) # 后面两个参数可省
# 绘圆弧
turtle.pencolor("purple")
turtle.right(90)
turtle.circle(30,180)
turtle.pencolor("lime")
# 绘制等边多边形，比如这里绘制三角形
turtle.circle(40,steps=3) # step是边数
turtle.pencolor("orange")
# 绘制6边形
turtle.circle(50,steps=6)
# 抬笔
turtle.penup()
# 移动位置
turtle.goto(100,100)
# 落笔
turtle.pendown()
# 绘制圆点turtle.dot(直接,颜色)
turtle.dot(50,"yellow")
turtle.done()
```
### 效果<br>
<img width="953" height="844" alt="{6041E696-C9CC-46EB-9282-3DAC926A7ED3}" src="https://github.com/user-attachments/assets/af9cff8a-0b7d-4afa-aea3-9137b9e98136" /><br>


