```python
class 类名:
	def __init__(self,形参)#初始化方法
		self.某属性=形参
	def 其他方法(self,形参)
		
对象=类名(实参)
对象.属性
对象.方法()
```
## 代码示例：从字典到类的优化
我们使用dic字典创建三个学生的信息
```python
# 先定义方法 
def listen(): 
	print('听课') 
def write(): 
	print('写作业')
# 用字典创建3个学生（重复代码多，方法复用但属性冗余）
stu1={
	'name':'张三',
	'sex':'男',
	'age':16,
	'height':167,
	
	'listen':listen,
	'write':write
}
stu2={
	'name':'李四',
	'sex':'女',
	'age':15,
	'height':160,
	
	'listen':listen,
	'write':write
}
stu3={
	'name':'王五',
	'sex':'女',
	'age':14,
	'height':163,
	
	'listen':listen,
	'write':write
}
```
引用class类概念
```python
#定义学生类
class Student:
	#__init__ 初始化方法：实例化对象时自动调用，绑定属性
	#self：指代当前创建的对象，a/b/c/d是接收的实参
	def __init__(self,a,b,c,d):
		self.name=a
		self.sex=b
		self.age=c
		self.height=d
	#定义方法：描述学生行为
	def listen(self):
		print('听课')
	def write(self):
		print('写作业')

#用类创建对象：实例化对象
stu1=Student('张三','男',16,167)
stu2=Student('李四','女',15,160)
stu3=Student('王五','女',14,163)

#检验
print(stu1.name)#输出结果:'张三' （访问属性）
stu1.listen()#输出结果：听课 （调用方法）
#在class中已经对listen定义过print了，直接调用就会输出了
stu1.write()#输出结果：写作业
```
## 对象
### 什么是对象？
- 在本次案例中，每个同学(stu1,stu2,stu3)是我们的教学对象，self代表创建的对象
- 对象：操作、管理、提供服务的目标
- 对象是具体的，有具体的数据
- 是一个实际的例子->实例
### 对象的结构组成：属性+方法
- 属性：(name,sex,age,height)具体的信息
- 方法：(listen,write)按照要求完成的动作
## class类
- 类是==描述对象==(我来描述我将来创建的对象有什么内容)
- 类是我们提取的一个共同的模式一个结构
- 类是使用对象的这些方法(属性+方法)
- 类是抽象的，类没有具体的数据，不存储具体的数据
- 通过类的形式去创建对象，用类去创建对象的这个动作->实例化
## init初始化方法
- 实例化对象时(如stu1=Student(...))会==自动调用==
- 作用：给空对象绑定属性，完成对象的初始化
- 注意：self是第一个参数，无需手动传参，python自动传递当前对象
## self的本质
- self表示每一次创建的那个对象，==当前实例化的对象==
- 如，创建stu1时，self就是stu1；创建stu2时，self就是stu2
- self.属性‘我将要新创建的这个对象的属性’
- self.name= a'新创建对象的name属性，用a参数来赋值'
- 对象的属性可以进行动态修改(详见后面的实例)
## 用类创建对象
- 把类的名称当成函数去调用，然后把具体的那个数据值('张三','男',16,167)作为实参传进去
- 实际上，在调用前创建了一个空的对象(会有name,sex,age,height都是空的一个对象)，把空对象传给init初始化方法，传给self
## 对象的属性操作
```python
#1.修改已有参数的值
stu1.age = 20 # 将stu1的age从16改为20 
print(stu1.age) # 输出：20
#2.动态添加新属性
stu1.score = 90 # 给stu1新增score属性，值为90（stu2/stu3无此属性） 
print(stu1.score) # 输出：90
#3.动态删除属性
del stu1.score # 删除stu1的score属性 
#print(stu1.score) # 报错：AttributeError（属性已删除）
```
## 实例化的底层逻辑
1. 调用`Student('张三', '男', 16, 167)`时，Python 先创建一个**无任何属性的空对象**；
2. 自动将空对象传入`__init__`的 self 参数；
3. 执行`__init__`中的代码，给空对象绑定 name/sex/age/height 属性；
4. 返回绑定好属性的对象，赋值给 stu1。
