## Abstruct
python：脚本语言。
除了基础语法外，对numpy，机器学习等库也需要着重学习。

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

#### range()遍历数字时常用
```python
for <循环变量> in range(<start>, <stop>, <step>):
	<statements>
```
##### 特别注意:
- 循环变量从<start>开始，不包含<stop>
- 在省略的情况下默认<start>=0, <step>=1
- 不能只省略 start，必须按照 start→stop→step 的顺序传参
- 无论哪种写法，<stop> 都是必填项，且循环值永远不包含<stop>

#### range() + len() 函数遍历一个序列的索引

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