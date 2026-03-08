## 定义介绍
通过配置GPIO的端口配置寄存器，端口可以配置成以下8种模式：

| 模式名称   |                            | 性质   | 特征                        |
| ------ | -------------------------- | ---- | ------------------------- |
| 浮空输入   | IN_FLOATING                | 数字输入 | 可读取引脚电平，若引脚悬空，则电平不确定      |
| 上拉输入   | IPU<br>(In Pull Up)        | 数字输入 | 可读取引脚电平，内部连接上拉电阻，悬空时默认高电平 |
| 下拉输入   | IPD<br>(In Put Down)       | 数字输入 | 可读取引脚电平，内部连接下拉电阻，悬空时默认低电平 |
| 模拟输入   | AIN<br>(Analog IN)         | 模拟输入 | GPIO无效，引脚直接接入内部ADC        |
| 开漏输出   | OUT_OD<br>(Out Open Drain) | 数字输出 | 可输出引脚电平，高电平为高阻态，低电平接VSS   |
| 推挽输出   | OUT_PP<br>(Out Push Pull)  | 数字输出 | 可输出引脚电平，高电平接VDD，低电平接VSS   |
| 复用开漏输出 | AF_OD<br>(Atl Open Drain)  | 数字输出 | 由片上外设控制，高电平为高阻态，低电平接VSS   |
| 复用推挽输出 | AF_PP<br>(Atl Push Pull)   | 数字输出 | 由片上外设控制，高电平接VDD，低电平接VSS   |
```c
IPD
(In Put Down)
typedef enum
{ GPIO_Mode_AIN = 0x0,//模拟输入
  GPIO_Mode_IN_FLOATING = 0x04,//浮空输入
  GPIO_Mode_IPD = 0x28,//下拉输入
  GPIO_Mode_IPU = 0x48,//上拉输入
  GPIO_Mode_Out_OD = 0x14,//开漏输出
  GPIO_Mode_Out_PP = 0x10,//推挽输出
  GPIO_Mode_AF_OD = 0x1C,//复用开漏
  GPIO_Mode_AF_PP = 0x18//复用推挽
}GPIOMode_TypeDef;
```
输出模式下可控制端口输出高低电平，用以驱动LED、控制蜂鸣器、模拟通信协议输出时序等
输入模式下可读取端口的高低电平或电压，用于读取按键输入，外接模块电平信号输入、ADC电压采集、模拟通信协议接受数据等
引脚电平：0~3.3V，部分引脚容忍5V(可以在端口输入5V的电压也认为是输入高电平，但是输出只有3.3V，具体查看芯片的引脚定义，带FT的是可以容忍5V)

![[Pasted image 20260307150011.png]]
寄存器内核可以通过APB2总线对寄存器进行读写，以此完成输出电平和读取电平功能
![[Pasted image 20260307150233.png]]



.h文件最下面是所有函数声明
在RCC中，最常用到的是三个函数
```c
void RCC_AHBPeriphClockCmd(uint32_t RCC_AHBPeriph, FunctionalState NewState);//RCC_AHB外设时钟控制
void RCC_APB2PeriphClockCmd(uint32_t RCC_APB2Periph, FunctionalState NewState);//RCC_APB2外设时钟控制
void RCC_APB1PeriphClockCmd(uint32_t RCC_APB1Periph, FunctionalState NewState);//RCC_APB1外设时钟控制
```
在GPIO中，最常用到的是
```C
void GPIO_DeInit(GPIO_TypeDef* GPIOx);//指定的GPIO外设会复位
void GPIO_AFIODeInit(void);//复位AFIO外设

//重要！！
void GPIO_Init(GPIO_TypeDef* GPIOx, GPIO_InitTypeDef* GPIO_InitStruct);
//用结构体的参数来初始化GPIO口
//需要先定义一个结构体变量，然后再给结构体赋值，最后调用这个函数
//函数内部会自动读取结构体的值，自动把外设配置好

void GPIO_StructInit(GPIO_InitTypeDef* GPIO_InitStruct);//把结构体变量赋一个默认值


----------------------------
//下面四个是GPIO的读取函数
uint8_t GPIO_ReadInputDataBit(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);//读取输入数据寄存器某一个端口的输入值的//读取按键用到
uint16_t GPIO_ReadInputData(GPIO_TypeDef* GPIOx);//读取整个输入数据寄存器的
uint8_t GPIO_ReadOutputDataBit(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);//读取输出数据寄存器的某一个位//一般用于输出模式下看自己输出的是什么
uint16_t GPIO_ReadOutputData(GPIO_TypeDef* GPIOx);//读取整个输出寄存器
//下面四个是GPIO的写入函数
void GPIO_SetBits(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);//指定的端口设置为高电平
void GPIO_ResetBits(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);//指定的端口设置为低电平
void GPIO_WriteBit(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin, BitAction BitVal);//BitAction BitVal根据第三个参数的值来设置指定的端口
void GPIO_Write(GPIO_TypeDef* GPIOx, uint16_t PortVal);//同时为16个函数进行写入操作
------------------------------------


```
## GPIO输出
### LED(Light Emitting Diode)
- 就是一个发光二极管，是一种特殊的二极管
- 单向导电性：电流只能从正极(长脚)流向负极(短脚)，反接不会发光
- 导通压降：
- 需要限流：由伏安特性曲线可知，导通后，电阻很小，需串接限流电阻，否则会因电流过大而烧毁

### 点一个呼吸灯
在电灯实验中，利用了LED发光二极管的单向导电性，通过GPIO引脚输出高低电平来进行控制导通截止
LED灯接法：
1. 长脚接面包板正极，短脚接STM32的A0口（低电平点亮）
2. 短脚接面包板负极，长脚接STM32的A0口（高电平点亮）
新建文件夹“System”，加入时间函数文件，加入头#include "Delay.h"，调用时间函数实现呼吸灯
STM32标准库的编程规范：开启外设时钟->配置结构体->初始化外设->读写操作

```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"


int main(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);//开启GPIOA外设时钟
	
	//定义结构体
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode=GPIO_Mode_Out_PP;//推挽输出
	GPIO_InitStructure.GPIO_Pin=GPIO_Pin_0;//选择引脚0
	GPIO_InitStructure.GPIO_Speed=GPIO_Speed_50MHz;//输出速度50Hz
	GPIO_Init(GPIOA,&GPIO_InitStructure);//调用初始化函数
	//GPIOA外设的0号引脚就自动被配置为推挽输出、50Hz的速度了
	
	
	while(1)
	{
		//方法一：用GPIO_WriteBit控制亮灭
		//GPIO_WriteBit(GPIOA,GPIO_Pin_0,Bit_RESET);
		//Delay_ms(500);
		//GPIO_WriteBit(GPIOA,GPIO_Pin_0,Bit_SET);
		//Delay_ms(500);
		
		//方法二：用GPIO_SetBits与GPIO_ResetBits控制亮灭
		GPIO_SetBits(GPIOA,GPIO_Pin_0);
		Delay_ms(500);
		GPIO_ResetBits(GPIOA,GPIO_Pin_0);
		Delay_ms(500);
		
		//方法三：二的基础上改
		//GPIO_SetBits(GPIOA,(BitAction)0);
		//Delay_ms(500);
		//GPIO_ResetBits(GPIOA,(BitAction)1);
		//Delay_ms(500);
		//不能直接写0和1，要加上强制类型转换
		
	}
}
```
在此代码中，不管LED灯是高电平点亮还是低电平点亮，LED都会进行间隔性闪烁，说明在推挽模式下，高低电平都是有驱动能力的
如果将端口模式换成Out_OD开漏输出模式的话，
```c
GPIO_InitStructure.GPIO_Mode=GPIO_Mode_Out_OD;//开漏输出模式
```
LED的高电平没有驱动能力，低电平正常间隔性闪烁
可得，开漏输出高电平相当于高阻态，没有驱动能力，低电平有驱动能力

### 点流水灯

| 特性       | \|是按位或             | \|\|是或          |
| -------- | ------------------ | --------------- |
| **运算对象** | 整数（二进制位）           | 布尔值（真 / 假）      |
| **运算规则** | 逐位进行 “有 1 出 1” 的运算 | 只要有一个为真，结果就为真   |
| **短路特性** | 无，两边都必须计算          | 只要有一个为真，结果就为真   |
| **返回值**  | 一个整数               | 一个布尔值（0 或 1）    |
| **典型用途** | 寄存器位操作、掩码设置        | 条件判断（if/while）  |


```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"


int main(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);//开启GPIOA外设时钟
	//定义结构体
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode=GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin= GPIO_Pin_All ;//或者GPIO_InitStructure.GPIO_Pin=GPIO_Pin_0|GPIO_Pin_1|GPIO_Pin_2······;
	GPIO_InitStructure.GPIO_Speed=GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStructure);
	//GPIOA外设的0号引脚就自动被配置为推挽输出、50Hz的速度了
	while(1)
	{
		GPIO_Write(GPIOA,~0x0001);//0000 0000 0000 0001
		Delay_ms(500);
		GPIO_Write(GPIOA,~0x0002);//0000 0000 0000 0010
		Delay_ms(500);
		GPIO_Write(GPIOA,~0x0004);//0000 0000 0000 0100
		Delay_ms(500);
		GPIO_Write(GPIOA,~0x0008);//0000 0000 0000 1000
		Delay_ms(500);
		GPIO_Write(GPIOA,~0x0010);//0000 0000 0001 0000
		Delay_ms(500);
		GPIO_Write(GPIOA,~0x0020);//0000 0000 0010 0000
		Delay_ms(500);
		GPIO_Write(GPIOA,~0x0040);//0000 0000 0100 0000
		Delay_ms(500);
		GPIO_Write(GPIOA,~0x0080);//0000 0000 1000 0000
		Delay_ms(500);
	}
}

```
在GPIO_InitStructure.GPIO_Pin中，
可以直接用GPIO_Pin_All开启全部的或者一个个按位或GPIO_Pin_0|GPIO_Pin_1|GPIO_Pin_2······;
按位或|的方法不止用在GPIO_InitStructure.GPIO_Pin，同道理可以用在：
同时开启多个外设时钟、对多个引脚进行读写等
### 蜂鸣器
将蜂鸣器的I/O口接到了PB12上
给PB12输出低电平，蜂鸣器响
输出高电平，蜂鸣器不响
```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"


int main(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB,ENABLE);
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode=GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin= GPIO_Pin_12 ;//或者GPIO_InitStructure.GPIO_Pin=GPIO_Pin_0|GPIO_Pin_1|GPIO_Pin_2······;
	GPIO_InitStructure.GPIO_Speed=GPIO_Speed_50MHz;
	GPIO_Init(GPIOB,&GPIO_InitStructure);
	//GPIOA外设的0号引脚就自动被配置为推挽输出、50Hz的速度
	while(1)
	{
		GPIO_ResetBits(GPIOB,GPIO_Pin_12);
		Delay_ms(500);
		GPIO_SetBits(GPIOB,GPIO_Pin_12);
		Delay_ms(500);
	}
}

```
## GPIO输入
### 硬件设备讲解
#### 按键
由于按键内部使用的是机械式弹簧片来进行通断的，所以在按下和松手的瞬间会伴随有一连串的抖动
需要写消抖程序
![[Pasted image 20260308133512.png]]
#### 传感器模块
从左到右分别是：光敏电阻，热敏电阻，对射式红外传感器，反射式红外传感器
利用传感器原件，通过与定值电阻分压即可得到模拟电压输出，再通过电压比较器进行二值化即可得到数字电压输出

![[Pasted image 20260308133645.png]]
#### 按键接法
![[Pasted image 20260308134849.png]]
一般常用左上和右上的接法

|     | 引脚配置模式            | 按键按下 |
| --- | ----------------- | ---- |
| 左上  | **上拉输入**          | 低电平  |
| 右上  | **浮空输入**或**上拉输入** | 低电平  |
| 左下  | **下拉输入**          | 高电平  |
| 右下  | **下拉输入**或**浮空输入** | 高电平  |
左上角的图，按键按下是低电平，不按按键的时候是浮空状态，要求PA0是**上拉输入**模式，否则就会出现引脚电压不确定的错误现象，在上拉模式中，按键不按下，引脚悬空，PA0高电压，按键按下，引脚接地为低电平
右上角，接了一个上拉电阻，可配置为**浮空输入**或**上拉输入**，上拉输入时，内外两个上拉电阻一起作用，高电平更强一点更稳定，但引脚拉低时，损耗大
### 实现按键控制LED
在总文件中新建“Hardware”文件夹，用来存放自己写的函数，注意在此文件夹下，需要同时新建.c与.h文件，在.c文件写好自己定义的代码之后，在.h文件中进行声明，再在main中加入头文件，才能正常使用函数
这是一种工程模块化思维，Hardware中自定义每个硬件的驱动函数，尽量简化main函数，同时记得加注释让其他人也能看懂
LED.c
```c
#include "stm32f10x.h"                  // Device header

void LED_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode=GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin=GPIO_Pin_1|GPIO_Pin_2;
	GPIO_InitStructure.GPIO_Speed=GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStructure);
	
	GPIO_SetBits(GPIOA,GPIO_Pin_1|GPIO_Pin_2);
}
void LED1_ON(void)
{
	GPIO_SetBits(GPIOA,GPIO_Pin_1);
}
void LED1_Turn(void)
{
	if(GPIO_ReadOutputDataBit(GPIOA,GPIO_Pin_1)==0)
	{
		GPIO_SetBits(GPIOA,GPIO_Pin_1);
	}
	else
	{
		GPIO_ResetBits(GPIOA,GPIO_Pin_1);
	}
}
void LED1_OFF(void)
{
	GPIO_ResetBits(GPIOA,GPIO_Pin_1);
}
void LED2_ON(void)
{
	GPIO_SetBits(GPIOA,GPIO_Pin_2);
}
void LED2_Turn(void)
{
	if(GPIO_ReadOutputDataBit(GPIOA,GPIO_Pin_2)==0)
	{
		GPIO_SetBits(GPIOA,GPIO_Pin_2);
	}
	else
	{
		GPIO_ResetBits(GPIOA,GPIO_Pin_2);
	}
}
void LED2_OFF(void)
{
	GPIO_ResetBits(GPIOA,GPIO_Pin_2);
}
```
LED.h
```c
#ifndef __LED_H
#define __LED_H
void LED_Init(void);
void LED1_ON(void);
void LED1_Turn(void);
void LED1_OFF(void);
void LED2_ON(void);
void LED2_Turn(void);
void LED2_OFF(void);
#endif

```
Key.c
```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
void Key_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB,ENABLE);//开启时钟
	GPIO_InitTypeDef GPIO_InitStructre;
	GPIO_InitStructre.GPIO_Mode=GPIO_Mode_IPU;
	GPIO_InitStructre.GPIO_Pin=GPIO_Pin_11|GPIO_Pin_1;
	GPIO_InitStructre.GPIO_Speed=GPIO_Speed_50MHz;
	GPIO_Init(GPIOB,&GPIO_InitStructre);
}	

uint8_t Key_GetNum(void)
{
	uint8_t KeyNum=0;
	if(GPIO_ReadInputDataBit(GPIOB,GPIO_Pin_1)==0)
	{
		Delay_ms(20);
		while(GPIO_ReadInputDataBit(GPIOB,GPIO_Pin_1)==0);
		Delay_ms(20);
		KeyNum=1;
	}
	if(GPIO_ReadInputDataBit(GPIOB,GPIO_Pin_11)==0)
	{
		Delay_ms(20);
		while(GPIO_ReadInputDataBit(GPIOB,GPIO_Pin_11)==0);
		Delay_ms(20);
		KeyNum=2;
	}
	return KeyNum;
}

```
Key.h
```c
#ifndef __KEY_H
#define __KEY_H
void Key_Init(void);
uint8_t Key_GetNum(void);
#endif

```
main.c
```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "LED.H"
#include "Key.h"
uint8_t KeyNum;
int main(void)
{
	LED_Init();
	Key_Init();
	while(1)
	{
		KeyNum=Key_GetNum();
		if (KeyNum==1)
		{
			LED1_Turn();
		}
		if (KeyNum==2)
		{
			LED2_Turn();
		}
	}
}

```
## 光敏传感器控制蜂鸣器
不详细写了，原理类似，就是检测光敏传感器是0(遮住)还是1(没遮住)，跟按键控制类似
蜂鸣器是Buzzer,光敏传感器是LightSensor

GPIO模块常用到此结束