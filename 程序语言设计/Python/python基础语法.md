## Abstruct
python：脚本语言。
除了基础语法外，对numpy，机器学习等库也需要着重学习。
## 标识符

## 注释
单行：#开头
多行：（'''）三个单引号或者（"""）三个双引号
```python
#
"""
"""
```
## 基本函数
### input输入&等待用户输入
接受输入，传入一个==字符串==
```python
a= int(input())
```

### print基本输出函数
将变量输出（或者说，打印）到控制台，**默认输出==换行**==，如果要实现不换行需要在变量末尾加上 ==end=""==

### import与from...import导入相应模块
- 将整个模块 (somemodule) 导入，格式为：​ `import somemodule`
- 从某个模块中导入某个函数,格式为：​ `from somemodule import somefunction`
- 从某个模块中导入多个函数,格式为：​ `from somemodule import firstfunc, secondfunc, thirdfunc`
- 将某个模块中的全部函数导入，格式为：​ `from somemodule import *`
```python
from sys import argv,path #导入特定的成员
```
导入模块，扩展的“插件库”模块，来源：Python标准库（安装Python时自带的但没预装在内存中），第三方库（别人写好分享的），自定义模块；而上面的input、print、len、type、max等则是自带的

## 基本数据类型
在python中，变量就是变量，它没有类型，我们所说的"类型"是变量所指的==内存中对象的类型==

| 类型名称 |                                 | 核心特点          | 是否可变 | 有序性    |
| ---- | ------------------------------- | ------------- | ---- | ------ |
| 数字   | Numbers(int,float.bool.complex) | 存储数值，不可变      | 不可变  | -      |
| 字符串  | String'   ' /"  "               | 文本，字符，不可变，可索引 | 不可变  | 有序     |
| 元组   | Tuple(  )                       | 内容不可修改，安全稳定   | 不可变  | 有序     |
| 列表   | List[  ]                        | 最常用，可增删改查     | 可变   | 有序     |
| 集合   | Set{  }                         | 自动去重，无索引，无重复  | 可变   | 无序     |
| 字典   | Dict{key:value}(key键value值)     | 键值对，键唯一，查找快   | 可变   | 3.7+有序 |


### Numbers数字
#### 定义

- Python 3 支持 int（整型）、float（浮点型）、bool（布尔型）、complex（复数）
- 当你指定一个值时，Number 对象就会被创建：var1 = 1
- 可以使用 del 语句删除一些对象引用 del var
- 数值运算
```python
5 + 4 # 加法 9
4.3 - 2 # 减法 2.3
3 * 7 # 乘法 21
2 / 4 # 除法，得到一个浮点数 0.5 
2 // 4 # 除法，得到一个整数 0 
17 % 3 # 取余 2
2 ** 5 # 乘方 32
a,b=b,a+b先计算右边的再同时赋值左边
```
#### 注意
1. Python 可以同时为多个变量赋值，如 a, b = 1, 2
2. 一个变量可以通过赋值指向不同类型的对象
3. 数值的除法（/）总是返回一个浮点数，要获取整数使用​`//`​操作符
4. 在混合计算时，Python 会把整型转换成为浮点数
### String字符串
#### 定义

1. Python 中单引号和双引号使用完全相同，但单引号和双引号不能匹配
2. 使用三对引号('''或""")可以囊括一个多行字符串
3. 与其他语言相似，python也使用 '\'作为转义字符
4. 自然字符串， 通过在字符串前加 r 或 R。 如 r"this is a line with \n" 则\n会显示，并不是换行
5. Python 允许处理 unicode 字符串，加前缀 u 或 U
6. 索引从左往右0开始，从右往左以 -1 开始
7. Python中的字符串不能改变
8. Python 没有单独的字符类型，一个字符就是长度为 1 的字符串
9. 字符串的截取的语法格式如下：变量 **[头下标: 尾下标: 步长]**
10. 与 C 字符串不同的是，Python 字符串不能被改变。**向一个索引位置赋值，比如 word[0] = m' 会导致错误**
#### 实例
```python
str='W3Cschool' print(str)  #输出字符串 
print(str[0:-1])   #输出第一个到倒数第二个的所有字符 
print(str[0])  #输出字符串第一个字符 
print(str[2:5])  #输出从第三个开始到第五个的字符 
print(str[2:])  #输出从第三个开始后的所有字符 
print(str[1:5:2])  #输出从第二个开始到第五个且每隔两个的字符 
print(str * 2)  #输出字符串两次 
print(str + '你好') #连接字符串 
print('hello\nW3Cschool') #使用反斜杠+n转义特殊字符（相当于换行）
print(r'hello\nW3Cschool') #在字符串前面添加一个 r，表示原始字符串，不会发生转义
```
#### 注意
1. 反斜杠可以用来转义，使用 r 可以让反斜杠不发生转义
2. 字符串可以用 + 运算符连接在一起，用 * 运算符重复
3. Python 中的字符串有两种索引方式，从左往右以 0 开始，从右往左以 -1 开始
4. Python 中的字符串不能改变

### 元组

### 列表

### 集合

### 字典


### 类型查询判断
#### type(x)->查询类型
直接返回变量的精确类型，不考虑继承关系（不认为子类是一种父类类型）
```python
x=1
print(type(x))  #输出int
x=1.2
print(type(x))  #输出float
x="1"
print(type(x))  #输出str 
```
#### isinstance(x，要判断的类型)->判断类型
isinstance(x，要判断的类型)
会考虑继承关系，更适合做类型判断，在开发中更常用
```python
a=111
print(isinstance(a,int))  #判断单一类型 #输出True表示判断正确
print(isinstance(a,(int,float))) #同时判断多种类型 #类型需要用元组包起来 #只要是整数或者浮点数都返回Ture
```

## 删除函数
| 函数       | 作用                      | 用法                                                     | 是否返回值  | 适用对象     | 如何使用      | 核心结果            |
| -------- | ----------------------- | ------------------------------------------------------ | ------ | -------- | --------- | --------------- |
| del      | 按**索引/键**删除，可删整个变量      | del lst[0]<br><br>del d["name"]<br><br>del lst<br><br> | 不      | 列表，字典，变量 | 想按位置删     | **删元素 / 删整个变量** |
| remove() | 按**值**删除（第一个匹配值）        | lst.remove(5)                                          | 不      | 列表       | 想按值删      | **删掉对应的值**      |
| pop()    | 按**索引/键**删除，并**返回被删的值** | lst.pop(0)<br><br>d.pop("name")                        | 返回被删元素 | 列表，字典    | 想删了还要用这个值 | **删元素并拿回值**     |
| clear()  | **清空所有内容**，变量还在         | lst.clear()<br><br>d.clear()                           | 不      | 列表，字典，集合 | 想清空内容     | **内容变空，变量还在**   |


## 运算符


## 条件判断
### if-elif-else
```python
if 条件为True: 
	statement_block_1 
elif 条件为True: 
	statement_block_2 
else: 
	statement_block_3
```

### match-case(在python3.10之后版本用)
```python
def http_error(status): 
	match status: 
		case 400: 
			return "Bad request" 
		case 404: 
			return "Not found" 
		case 418: 
			return "I'm a teapot" 
		case _ : 
			return "Something's wrong with the Internet"
```



## 循环
### while
```python
while <条件为True>:
	statements
```
### for
```python
for <循环变量> in <可迭代对象>: 
	<statements> 
else:  # 只有循环正常结束完后才执行，中途break跳出循环就不执行
	<statements>
```
### range()遍历数字时常用

```python
for <循环变量> in range(<start>, <stop>, <step>):
	<statements>
```
#### 特别注意
1. 循环变量从start开始，不包含stop
2. 在省略的情况下默认start=0, step=1
3. 不能只省略 start，必须按照 start→stop→step 的顺序传参
4. 无论哪种写法，stop 都是必填项，且循环值永远不包含stop
####  range(len())遍历一个序列的索引
```python
a = ['Mary', 'had', 'a', 'little', 'lamb'] 
for i in range(len(a)):
	print(i, a[i])
```
### break与continue及循环中的 else 子句
break直接跳出循环
continue跳出当前循环
循环中的else子句，只有循环正常结束完后才执行，中途break跳出循环就不执行
### pass语句
语法上必须有代码块但程序无需执行任何逻辑时使用，仅作为占位符补全语法框架
```python
def a(val):#通过占位，告诉后来的开发者这里我们之后再写
	pass
i = 10
while i:
	pass
a=2
```
## 迭代器与生成器
### 迭代器iter()与next

| 特性   | 普通迭代器（`iter(list)`） | 生成器迭代器（`yield` 函数） |
| ---- | ------------------- | ------------------ |
| 元素来源 | 提前存在的（列表 / 字符串）     | 实时计算生成（无提前存储）      |
| 内存占用 | 等于原数据大小（因为元素已存在）    | 极低（只存计算状态，不存元素）    |
| 核心逻辑 | 单纯 “遍历返回”           | 自定义 “计算 + 返回”      |
| 创建方式 | iter(可迭代对象)         | 生成器函数（def + yield） |

#### 迭代器
- 迭代器是访问集合元素的一种方法
- 是一个可以记住遍历的位置的对象
- 从集合的第一个元素开始访问，直到所有的元素被访问完结束。迭代器只能往前不会后退
#### 基本方法：iter()与next
条件：字符串，列表或元组对象
iter()相当于“转换工具”，将列表/字符串/元组转换成只能按照顺序访问的迭代器
next()相当于“按照iter的顺序，拿一个少一个，不可逆”

```python
import sys # 引入 sys 模块 
list=[1,2,3,4] 
it = iter(list) # 创建迭代器对象 
while True: #无限循环
	try: 
		print (next(it)) # 尝试获取迭代器的下一个元素并打印
	except StopIteration: # 捕获"迭代器没有下一个元素"的异常
		sys.exit()# 捕获到异常后，退出程序
#`try...except` “尝试执行一段代码，如果这段代码出错了，就执行我们提前准备好的‘补救代码’，而不是让程序直接崩溃”**。
```
注意
1. 与普通列表的区别：迭代器是==一次性、顺序访问==，普通列表中的元素不会因为用过消失，可随便拿任意位置的元素
2. 取完最后一个后，再调用next()就会触发`StopIteration` 异常
### 生成器迭代器
在调用生成器运行的过程中，每次遇到 yield 时函数会暂停并保存当前所有的运行信息，返回 yield 的值。并在下一次执行 next() 方法时从当前位置继续运行
#### 实例：（利用yield实现斐波那契数列）
```python
import sys

def fibonacci(n):
	a, b, counter = 0, 1, 0
	while True:
		if (counter > n): 
			return
		yield a
		a, b = b, a + b
		counter += 1
f = fibonacci(10) #f是一个迭代器，由生成器返回生成
while True:
	try:
		print (next(f), end=" ")
	except StopIteration:
		sys.exit()
#输出结果
#0 1 1 2 3 5 8 13 21 34 55
```
## 函数
### def定义函数
格式：
```python
def 函数名（参数列表）： 
	函数体
```
实例：
```python
def area(width, height): 
	return width * height
print(" area =", area(4, 5))
#输出结果： area =20
```
### 函数变量作用域
局部作用域：定义在函数内部的变量拥有局部作用域，只能在函数内部用
全局作用域：定义在函数外的拥有全局作用域，全局可用
```python
a=10#全局作用域
def c(a):
	a=13#局部作用域
	return a
print(c(a))
#输出结果：13
```

