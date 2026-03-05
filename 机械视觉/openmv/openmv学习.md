## 网站：
- OpenMV官方中文入门教程：[Python背景知识 · OpenMV中文入门教程](https://book.openmv.cc/python-background.html)
- 1




## 软件及基础介绍
### 软件界面介绍

![[Pasted image 20260228133335.png]]
### 更改阈值
- 数字列表项目首先在摄像头中找到目标颜色，在framebuffer中的目标颜色上左击圈出一个矩形
- 在软件右侧framebuffer下面的坐标图中，选择LAB 色彩空间
![[Pasted image 20260228141025.png]]
三个坐标图分别表示出的矩形区域内的颜色LAB值，选取三个坐标图的最大最小值，即（0,100,-54,18,-22,59）
![[Pasted image 20260228141318.png]]
### 文件系统
- 各种文件夹和文件以树形结构排列
- 在代码中可以使用路径来进行读入文件，创建文件等操作
![[Pasted image 20260228141740.png]]
### 脱机运行
##### [https://singtown.com/video](https://singtown.com/video)
### 一键下载
- 在左上角第三个“工具”中，点击“将打开的脚本保存到OpenMV Cam（作为main.py）”
- IDE就会自动将当前文件保存到main.py
![[Pasted image 20260228142230.png]]

## 引脚
### 供电

![[Pasted image 20260228132620.png]]
1. 正面右下角的两个引脚：
- VIN/VCC电源输入；GND接地
- VIN输入为3.6V~5V，推荐5V（板子内部有稳压芯片，会把5V变成稳定3.3V给核心用）
- 特别注意：绝对不能给3.3V引脚供电，3.3V是板子输出给我们用的，用于给外部传感器供电（如串口屏、小模块等）
2. USB输入
### SD卡
![[Pasted image 20260228132316.png]]
SD卡是一个文件系统，上电的时候，如果插入SD卡，那么SD卡的文件系统就会自动取代内置的Flash文件系统，每次上电，就会运行`SD卡中的main.py`
最大支持32G的容量

## 程序
### 感光元件sensor模块
sensor模块设置感光元件的参数

blobs = img.find_blobs([green_threshold])
find_blobs(thresholds, invert=False, roi=Auto),
thresholds为颜色阈值，是一个元组，需要用括号［ ］括起来。
invert=1,反转颜色阈值，invert=False默认不反转。roi设置颜色识别的视野区域，roi是一个元组， roi = (x, y, w, h)，代表
从左上顶点(x,y)开始的宽为w高为h的矩形区域，roi不设置的话默认为整个图像视野。
这个函数返回一个列表，
［0］代表识别到的目标颜色区域左上顶点的x坐标
［1］代表左上顶点y坐标，
［2］代表目标区域的宽，
［3］代表目标区域的高，
［4］代表目标区域像素点的个数，
［5］代表目标区域的中心点==x坐标==，
［6］代表目标区域中心点==y坐标==，
［7］代表目标颜色区域的==旋转角度==（是弧度值，浮点型，列表其他元素是整型），
［8］代表与此目标区域交叉的目标个数，
［9］代表颜色的编号（它可以用来分辨这个区域是用哪个颜色阈值threshold识别出来的）。


- OpenMV 画图的核心是先获取`img = sensor.snapshot()`，所有图形都画在这个`img`上；
- 基础画图函数：
    
    - 点：`draw_point(x,y,color)`
    - 线：`draw_line(x1,y1,x2,y2,color,thickness)`
    - 矩形：`draw_rectangle(x,y,w,h,color,thickness,fill)`
    - 圆：`draw_circle(cx,cy,r,color,thickness,fill)`
    
- 坐标以画面左上角为 (0,0)，QVGA 分辨率下 x 范围 0-319，y 范围 0-239。




导入模块
- `sensor`：控制**摄像头**拍照、设置分辨率
- `image`：**处理图像**、找矩形、画图
- `time`：**时间**相关函数，计时、延时
- `gc`：垃圾回收模块，管理内存，手动释放内存，防止内存溢出


### corners(列表)
- 不用手动维护计数器
- 一看就知道同时获取索引和值
```python
#打印角点坐标
corners = [(10,20), (30,40), (50,60), (70,80)]
for i, point in enumerate(corners):
    print("角点", i+1, ":", point)
#只显示前几个矩形
# 显示前两个最大的矩形
for i, r in enumerate(valid[:2]):
    # i = 0 时显示第一个矩形
    # i = 1 时显示第二个矩形
    img.draw_rectangle(r.rect(), color=150)
```

```python
# 基本用法
for i, item in enumerate(列表):
    print(i, item)

# 可以指定起始数字
for i, item in enumerate(列表, start=1):  # 从1开始数
    print(i, item)
```
### 标定
实际尺寸与屏幕尺寸建立关系
使用场景：摄像头斜着；不同位置像素比例不同
因为摄像头可能倾斜；不是标准矩形
```python
图像中屏幕区域（可能是斜的）：
(x0,y0)───────(x1,y1)
  │              │
  │    ?????     │  ← 要找这个点
  │              │
(x3,y3)───────(x2,y2)
```
使用**双线性插值**(图像坐标)算中点
```python
def screen_to_pixel(self, x_cm, y_cm):
    """把屏幕坐标(25cm,25cm)转成图像坐标"""
    
    # 1. 四个角点的图像坐标
    x0, y0 = self.screen_corners[0]  # 左上 (0cm,0cm) → (40,30)
    x1, y1 = self.screen_corners[1]  # 右上 (50cm,0cm) → (200,35)
    x2, y2 = self.screen_corners[2]  # 右下 (50cm,50cm) → (210,170)
    x3, y3 = self.screen_corners[3]  # 左下 (0cm,50cm) → (30,165)
    
    # 2. 计算相对位置 (0~1)
    tx = x_cm / 50.0  # 25/50 = 0.5
    ty = y_cm / 50.0  # 25/50 = 0.5
    
    # 3. 第一步：水平插值（上边）
    # 上边：从(x0,y0)到(x1,y1)
    top_x = x0 + tx * (x1 - x0)  # 40 + 0.5*(200-40) = 120
    top_y = y0 + tx * (y1 - y0)  # 30 + 0.5*(35-30) = 32.5
    # 得到上边中点 (120,32.5)
    
    # 4. 第一步：水平插值（下边）
    # 下边：从(x3,y3)到(x2,y2)
    bottom_x = x3 + tx * (x2 - x3)  # 30 + 0.5*(210-30) = 120
    bottom_y = y3 + tx * (y2 - y3)  # 165 + 0.5*(170-165) = 167.5
    # 得到下边中点 (120,167.5)
    
    # 5. 第二步：垂直插值
    px = int(top_x + ty * (bottom_x - top_x))  # 120 + 0.5*(120-120) = 120
    py = int(top_y + ty * (bottom_y - top_y))  # 32.5 + 0.5*(167.5-32.5) = 100
    
    return (120, 100)  # 屏幕中心在图像中的位置
```

