## 数据类型
| 关键字                | 位数  | 表示范围                     | stdint关键字 | ST关键字 |
| ------------------ | --- | ------------------------ | --------- | ----- |
| char               | 8   | -128 ~ 127               | int8_t    | s8    |
| unsigned char      | 8   | 0 ~ 255                  | uint8_t   | u8    |
| short              | 16  | -32768 ~ 32767           | int16_t   | s16   |
| unsigned short     | 16  | 0 ~ 65535                | uint16_t  | u16   |
| int                | 32  | -2147483648 ~ 2147483647 | int32_t   | s32   |
| unsigned int       | 32  | 0 ~ 4294967295           | uint32_t  | u32   |
| long               | 32  | -2147483648 ~ 2147483647 |           |       |
| unsigned long      | 32  | 0 ~ 4294967295           |           |       |
| long long          | 64  | -(2^64)/2 ~ (2^64)/2-1   | int64_t   |       |
| unsigned long long | 64  | 0 ~ (2^64)-1             | uint64_t  |       |
| float              | 32  | -3.4e38 ~ 3.4e38         |           |       |
| double             | 64  | -1.7e308 ~ 1.7e308       |           |       |
在51单片机中，int是16位的，但在STM32中，int是32位的，如果要用16位的数据，要用short来表示
C语言官方提供的stdint.h头文件，使用了新的名字
ST是老版本的，我们现在都使用stdint关键字
## # define 宏定义
关键字：#define
用途：用一个字符串代替一个数字，便于理解，防止出错；提取程序中经常出现的参数，便于快速修改

```c
//定义宏定义
#define ABC 12345
//用ABC这个字符串代替12345这个参数

//引用define
int a = ABC;//等效于int a = 12345;

```
## typedef
关键字：typedef
用途：将一个比较长的**变量类型名**换个名字，便于使用
```c
//定义typedef
typedef unsigned char uint8_t;

//引用typedef
uint8_t a;//等于unsigned char a;
```

## Struct结构体
关键字：Struct
用途：数据打包，不同类型变量的集合
定义结构体变量：
```c
struct Student{//Student是结构体名字

}s1,s2,s[30]
//s1,s2是变量，s[30]是数组，创建了30个元素，每个元素类型都是struct Student(结构体)
```

### Struct结构体与typedef
```c
#include <stdio.h>
int main(void)
{
	struct{
	char x;
	int y; 
	float z;
	}c//c 是结构体变量名字
	
	//引用结构体成员
	//方法一：
	//结构体变量名.结构体成员名
	c.x='A';
	c.y=66;
	c.z=1.23;
	//方法二：
	//结构体首地址(即结构体指针)->结构体成员名
	pc->x='A';//其中pc为结构体的地址
	pc->y=66;
	pc->z=1.23;
	
}
```
定义好了一个结构体后，如果我们要在后面加新的变量：
```c
//方法1：
struct{char x;int y; float z;}d;
//在工程中食用太麻烦

//方法2：
//引用typedef来进行命名
typedef struct{
	char x;
	int y; 
	float z;
} StructName_t;

```

## enum枚举
用途：定义一个取值受限制的整形变量，用于限制变量取值范围；宏定义的集合
```c
enum {MONDAY=1,TUESDAY=2,WEDNESDAY=3}week;
//week是变量名字
//如果数是按顺序累加的，那后面的数可以省略
```

### enum枚举与typedef
```c
typedef enum {
	MONDAY=1,
	TUESDAY=2,
	WEDNESDAY=3
}week_t;

int main(void)
{
	week_t;
	//引用枚举
	week=MONDAY;//week=1;
	week=TUESDAY;//week=2;
}
```