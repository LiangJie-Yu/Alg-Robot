## 定义
USART（Universal Synchronous/Asynchronous Receiver/Transmitter）通用同步/UART异步收发器

### USART分为发送和接收两部分：
发送：将数据寄存器的一个字节数据自动转换为协议规定的波形，然后从TX引脚发送出去
接收：自动接受RX引脚波形，按照协议规定解码为一个字节数据，存放在DR数据寄存器里

### 发送与接收数据，都存放在DR数据寄存器中，互不干扰
TDR发送缓存区：存要发出去的数据，对应Tx引脚
RDR接收缓存区：存放要接收的数据，对应Rx引脚
 
### 标志位
==RXND==接收标志位
- 硬件动作：RX 引脚收到数据→自动存到 DR 的**接收右格**→硬件自动把 RXNE 标志位置 **==1==**（灯亮）
- 代码动作：读 DR 寄存器（拿数据→**硬件自动清 RXNE 标志位（灯灭）**；
==TXE==发送标志位
- 硬件动作：DR 的**发送左格**数据发完了→硬件自动把 TXE 标志位置 **==1==**（灯亮）
- 代码动作：写 DR 寄存器（放新数据→**硬件自动清 TXE 标志位（灯灭）**；

自带波特率发生器，最高达4.5Mbits/s
可配置数据位长度（8/9）、停止位长度（0.5/1/1.5/2）
可选校验位（无校验/奇校验/偶校验）
支持同步模式、硬件流控制、DMA、智能卡、IrDA、LIN

STM32F103C8T6 USART资源： USART1、 USART2、 USART3
## USART框图
![[Pasted image 20260315121414.png]]
## 外设的复用引脚
看芯片的引脚定义，需提前规划好引脚，不要重复了
![[Pasted image 20260315131543.png]]
## USART基本结构简图
![[Pasted image 20260315131704.png]]
 “>>” 表示右移，低位先行

## 数据帧
![[Pasted image 20260315132244.png]]
最好选择**9位有校验**或者**8位无校验**，因为这样的话，每个有效载荷都是1字节

空闲帧和断开帧是局域网协议用的
![[Pasted image 20260315132526.png]]
STM32可配置的停止位长度为0.5，1，1.5，2这四种
### 输入电路
信号的输出比信号接收考虑的事情少，信号接收需要保证
是在中间位接收的，不然会导致接收错误，
配置波特率，频率一致，不然会出现重复接收或者少接收
来不及传输的时候最好有标志位提醒
减少噪声
![[Pasted image 20260315132714.png]]
![[Pasted image 20260315133503.png]]
## 波特率发生器
发送器和接收器的波特率由波特率寄存器BRR里的DIV确定
计算公式：![](file:///C:\Users\YU\AppData\Local\Temp\ksohtml8760\wps1.jpg)
![[Pasted image 20260315133424.png]]
比如我现在的波特率是9600，STM32的fPCLK2(APB2总线时钟一般是72MHz)计算得知DIV为468.75，二进制为111010100.11
所以最后15-14位多出来补0，13-4位依次为111010100，3-0位为1100
## 寄存器经典分类
- 状态寄存器SR(Status Register)：存放各种标志位
- 数据寄存器DR(Date Register)：存放关键数据
- 配置寄存器CR(Config Register)：存放各种配置参数
## 串口发送
### 初始化流程
1. 开启时钟，把需要用到的USART与GPIO时钟打开
2. GPIO初始化，TX配置复用输出，RX输入
3. 配置USART，使用结构体配置其余参数
4. (如果只需要发送的功能)直接开启USART
5. (如果还需要接收的功能)再配置中断，开启USART之前加上ITConfig和NVIC的代码
### 使用思路
发送数据调用发送函数
获取数据调用获取函数
获取发送和接收的状态，就调用获取**标志位**的函数
### 常用函数
stm32f103x_usart.h
```c
void USART_DeInit(USART_TypeDef* USARTx);
void USART_Init(USART_TypeDef* USARTx, USART_InitTypeDef* USART_InitStruct);
void USART_StructInit(USART_InitTypeDef* USART_InitStruct);

//以下两个是时钟输出用函数
void USART_ClockInit(USART_TypeDef* USARTx, USART_ClockInitTypeDef* USART_ClockInitStruct);
void USART_ClockStructInit(USART_ClockInitTypeDef* USART_ClockInitStruct);

void USART_Cmd(USART_TypeDef* USARTx, FunctionalState NewState);
void USART_ITConfig(USART_TypeDef* USARTx, uint16_t USART_IT, FunctionalState NewState);

void USART_DMACmd(USART_TypeDef* USARTx, uint16_t USART_DMAReq, FunctionalState NewState);//开启USART到DMA的触发通道

void USART_SendData(USART_TypeDef* USARTx, uint16_t Data);//发送数据//写DR寄存器
uint16_t USART_ReceiveData(USART_TypeDef* USARTx);//接收数据//读DR寄存器

//以下四个是标志位相关函数
USART_GetFlagStatus(USART_TypeDef* USARTx, uint16_t USART_FLAG);//获取标志位
void USART_ClearFlag(USART_TypeDef* USARTx, uint16_t USART_FLAG);//清除标志位
ITStatus USART_GetITStatus(USART_TypeDef* USARTx, uint16_t USART_IT);//获取中断状态
void USART_ClearITPendingBit(USART_TypeDef* USARTx, uint16_t USART_IT);//清除中断位数
```

### Serial.c
```c
#include "stm32f10x.h"                  // Device header
void Serial_Init(void)
{
	//开启时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1,ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	
	//初始化GPIO引脚
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Mode=GPIO_Mode_AF_PP;//复用推挽输出
	GPIO_InitStruct.GPIO_Pin=GPIO_Pin_9;//根据引脚定义图，可得9为TX发送数据
	GPIO_InitStruct.GPIO_Speed=GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStruct);
	
	//初始化USART
	USART_InitTypeDef USART_InitStruct;
	USART_InitStruct.USART_BaudRate=9600;
	//USART_Init函数会自动计算好对应的分频系数，放到BRR寄存器
	USART_InitStruct.USART_HardwareFlowControl=USART_HardwareFlowControl_None;
	//USART_HardwareFlowControl硬件流控制
	USART_InitStruct.USART_Mode=USART_Mode_Tx;
	USART_InitStruct.USART_Parity=USART_Parity_No;
	//校验位：NO不校验Odd奇校验Even偶校验
	USART_InitStruct.USART_StopBits=USART_StopBits_1;//停止位
	USART_InitStruct.USART_WordLength=USART_WordLength_8b;//字长
	//目前配置数据为9600波特率，8位字长，无校验，1位停止位，无流控，只有发送模式
	USART_Init(USART1,&USART_InitStruct);
	
	//开启USART外设使能
	USART_Cmd(USART1,ENABLE);
}
void Serial_SendByte(uint8_t Byte)//发送数据的函数//从TX引脚发送一个字节数据
{
	USART_SendData(USART1, Byte);
	while(USART_GetFlagStatus(USART1,USART_FLAG_TXE)==RESET);
}
```
### main.c
```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "OLED.h"
#include "Serial.h"
int main(void)
{
	OLED_Init();
	Serial_Init();
	Serial_SendByte(0x41);
	while(1)
	{
	}
}
```

### 看串口
“此电脑”右键-“属性”-“设备管理器”-“端口”-查看接入的USB-SERIAL CH340是什么串口号
![[Pasted image 20260315220936.png]]
如图所示串口号就是COM15
根据写的Serial.c中USART_Init写的参数“目前配置数据为9600波特率，8位字长，无校验，1位停止位，无流控，只有发送模式”选择信息一致，接收数据，按板子上的复位按钮，再串口助手中信息如下：
![[Pasted image 20260315220955.png]]
A是接收区配置-接收模式的”文本模式“
41是接收区配置-接收模式的”HEX模式“
### 数据模式
HEX模式/十六进制模式/二进制模式：以原始数据的形式显示
文本模式/字符模式：以原始数据编码后的形式显示
下图为ASCII码字符集
![[Pasted image 20260315221538.png]]
下图为字符和数据在发送和接收的转换关系
![[Pasted image 20260315221844.png]]

```c
#一样的，传输还是0x41
Serial_SendByte('A');
Serial_SendByte(0x41);
//第一个会先对A进行编码，转换成0x41进行传输
```

### Serial.c增加函数以及main.c用法
#### 输出多个数字，使用数组
```c
Serial.c
void Serial_SendArray(uint8_t *Array,uint16_t Length)
{
	uint16_t  i;
	for(i=0;i<Length;i++)
	{
		Serial_SendByte(Array[i]);
	}
}
---------
main.c
uint8_t MyArray[]={0x41,0x42,0x43,0x44};
Serial_SendArray(MyArray,4);
```
#### 输出字符
```c
Serial.c
void Serial_SendString(char *String)
{
	uint8_t i;
	for(i=0;String[i]!=0;i++)
	{
		Serial_SendByte(String[i]);
	}
}
-----------
main.c
Serial_SendString("HelloWord!\r\n");
```
- 因为写完字符串后，编译器会自动补上结束标志位，所以字符串的存储空间会比字符个数大一个
- 正因为这个特性所以我们在写函数时，只要检查是否到最后一个结束标志位即可
#### 输入数字，输出字符类型的数字



```c
Serial.c
uint32_t Serial_Pow(uint32_t X,uint32_t Y)
{
	uint32_t Result=1;
	while (Y--)
	{
		Result *=X;
	}
	return Result;
}
void Serial_SendNumber(uint32_t Number,uint8_t Length)
{
	uint8_t i;
	for(i=0;i<Length;i++)
	{
		Serial_SendByte(Number/Serial_Pow(10,Length-i-1)%10+'0');
	}
}
-------
main.c
Serial_SendNumber(123456,6);//发送字符形式的数字


```
### printf函数移植
#### printf函数单个串口重定向
```c
Serial.c
#include <stdio.h>
int fputc(int ch,FILE *f)
{
	Serial_SendByte(ch);
	return ch;
}
------------
Serial.h
#include <stdio.h>
------------
main.c
printf("Num=%d\r\n",666);
```
- 原本的printf函数调用的时候就是用fputc来一个一个打印的
- 这边重定义到串口，就可以把printf函数输出到串口了
- 但是用这个把printf定义到串口1了，串口2就没有了
- 注意需要**添加头文件#include <stdio.h>**
#### sprintf函数多个串口用
sprintf可以把格式化字符输出到一个字符串里
```c
main.c
	char String[100];
	sprintf(String,"Num=%d\r\n",666);
	Serial_SendString(String);
```
- sprintf是把格式化文字**打印到数组里**
- 是把Num=666\r\n整个字符串存到数组String中，再用串口函数输出String就是输出整句话
- sprintf可以指定打印位置，不涉及重定向，所以每个串口都可以用sprintf进行格式化打印
#### 封装sprintf
```c
Serial.c
#include <stdarg.h>
void Serial_Printf(char *format,...)
{
	char String[100];
	va_list arg;
	//va_list类型名
	//arg变量名
	va_start(arg,format);
	vsprintf(String,format,arg);
	//打印位置String
	//格式化字符串format
	//参数表arg
	//sprintf只能接收直接写的参数
	//对于封装格式要用vsprintf
	va_end(arg);//释放参数表
	Serial_SendString(String);
}
------------
main.c
Serial_Printf("Num=%d\r\n",666);
```
涉及到C语言可变参数，可去看一下

## C语言可变参数
声明头文件 **#include <stdarg.h>** 该文件提供了实现可变参数功能的函数和宏
核心：...就是**可变参数占位符**
常用的宏：
```c
va_list 变量名;//变量名=指向可变参数的指针

va_start(va_list变量, 最后一个固定参数);
//初始化 `va_list` 指针，让它指向第一个可变参数的起始地址

变量 = va_arg(va_list变量, 参数类型);
//读取当前指向的可变参数，并自动把指针移动到下一个可变参数

va_end(va_list变量);
//清理 `va_list` 指针，结束可变参数遍历（必须和 `va_start` 配对）
```

## 串口发送与接收
### 查询
#### Serial.c
```c
#include "stm32f10x.h"                  // Device header
#include <stdio.h>
#include <stdarg.h>

void Serial_Init(void)
{
	//开启时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1,ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	
	//初始化GPIO9引脚(发送TX)
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Mode=GPIO_Mode_AF_PP;//复用推挽输出
	GPIO_InitStruct.GPIO_Pin=GPIO_Pin_9;//根据引脚定义图，可得9为TX发送数据
	GPIO_InitStruct.GPIO_Speed=GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStruct);
	//初始化GPIO10引脚(接收RX)
	GPIO_InitStruct.GPIO_Mode=GPIO_Mode_IPU;//上拉输入模式
	GPIO_InitStruct.GPIO_Pin=GPIO_Pin_10;//根据引脚定义图，可得10为TX发送数据
	GPIO_InitStruct.GPIO_Speed=GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStruct);
	
	
	//初始化USART
	USART_InitTypeDef USART_InitStruct;
	USART_InitStruct.USART_BaudRate=9600;
	//USART_Init函数会自动计算好对应的分频系数，放到BRR寄存器
	USART_InitStruct.USART_HardwareFlowControl=USART_HardwareFlowControl_None;
	//USART_HardwareFlowControl硬件流控制
	USART_InitStruct.USART_Mode=USART_Mode_Tx|USART_Mode_Rx;//只改这个(发送和接收区别)
	USART_InitStruct.USART_Parity=USART_Parity_No;
	//校验位：NO不校验Odd奇校验Even偶校验
	USART_InitStruct.USART_StopBits=USART_StopBits_1;//停止位
	USART_InitStruct.USART_WordLength=USART_WordLength_8b;//字长
	//目前配置数据为9600波特率，8位字长，无校验，1位停止位，无流控，只有发送模式
	USART_Init(USART1,&USART_InitStruct);
	
	
	//开启USART外设使能
	USART_Cmd(USART1,ENABLE);
}

```
增加部分：
在Serial_Init()函数中：
1. 初始化GPIO引脚中，增加接收引脚(如果串口配置不一样需要重新写，但是一样的话直接用|即可)
2. 初始化USART的模式加上接收模式
#### main.c
在此不封装演示：
```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "OLED.h"
#include "Serial.h"
int main(void)
{
	uint8_t RxDate;
	OLED_Init();
	Serial_Init();
	//Serial_SendByte(0x41);
	//uint8_t MyArray[]={0x41,0x42,0x43,0x44};
	//Serial_SendArray(MyArray,4);
	while(1)
	{
		if((USART_GetFlagStatus(USART1,USART_FLAG_RXNE)==SET))
		{
			RxDate=USART_ReceiveData(USART1);//目前的一个字节数据就已经存放在RxDate中
			OLED_ShowHexNum(1,1,RxDate,2);
		}
	}
}
```
查询的流程：
1. 在主函数里不断识别RXNE标志位，置1为收到数据
2. 调用ReceiveDate，读取DR寄存器
3. 关于清除标志位
![[Pasted image 20260318190549.png]]
由此可见读完DR不需要我们清除标志位了
### 中断
#### Serial.c
```c
void Serial_Init(void)
{
	//开启时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1,ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	
	//初始化GPIO9引脚(发送TX)
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Mode=GPIO_Mode_AF_PP;//复用推挽输出
	GPIO_InitStruct.GPIO_Pin=GPIO_Pin_9;//根据引脚定义图，可得9为TX发送数据
	GPIO_InitStruct.GPIO_Speed=GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStruct);
	//初始化GPIO10引脚(接收RX)
	GPIO_InitStruct.GPIO_Mode=GPIO_Mode_IPU;//上拉输入模式
	GPIO_InitStruct.GPIO_Pin=GPIO_Pin_10;//根据引脚定义图，可得10为TX发送数据
	GPIO_InitStruct.GPIO_Speed=GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStruct);
	
	
	//初始化USART
	USART_InitTypeDef USART_InitStruct;
	USART_InitStruct.USART_BaudRate=9600;
	//USART_Init函数会自动计算好对应的分频系数，放到BRR寄存器
	USART_InitStruct.USART_HardwareFlowControl=USART_HardwareFlowControl_None;
	//USART_HardwareFlowControl硬件流控制
	USART_InitStruct.USART_Mode=USART_Mode_Tx|USART_Mode_Rx;//只改这个(发送和接收区别)
	USART_InitStruct.USART_Parity=USART_Parity_No;
	//校验位：NO不校验Odd奇校验Even偶校验
	USART_InitStruct.USART_StopBits=USART_StopBits_1;//停止位
	USART_InitStruct.USART_WordLength=USART_WordLength_8b;//字长
	//目前配置数据为9600波特率，8位字长，无校验，1位停止位，无流控，只有发送模式
	USART_Init(USART1,&USART_InitStruct);
	
	
	
	//开启RXNE标志位到NVIC的输出
	c(USART1,USART_IT_RXNE,ENABLE);
	
	//初始化NVIC
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
	NVIC_InitTypeDef NVIC_InitStruct;
	NVIC_InitStruct.NVIC_IRQChannel=USART1_IRQn;
	NVIC_InitStruct.NVIC_IRQChannelCmd=ENABLE;
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority=1;
	NVIC_InitStruct.NVIC_IRQChannelSubPriority=1;
	NVIC_Init(&NVIC_InitStruct);
	
	
	
	
	//开启USART外设使能
	USART_Cmd(USART1,ENABLE);
}
```
步骤：
1. 开启中断输出口(USART_ITConfig)，开启RXNE标志位传到NVIC
2. 初始化NVIC(NVIC_Init)
RXNE标志位一旦置1，就会向NVIC申请中断，然后在中断函数中接收数据
```c

//查有没有新数据 + 自动清标志」二合一
uint8_t Serial_GetRxFlag(void)
{
	if(Serial_RxFlag==1)
	{
		Serial_RxFlag=0;
		return 1;
	}
	return 0;
}
//安全拿数据，不让你乱改全局变量
uint8_t Serial_GetRxDate(void)
{
	return Serial_RxDate;
}

void USART1_IRQHandler(void)
{
	if(USART_GetFlagStatus(USART1,USART_FLAG_RXNE)==SET)
	{
		Serial_RxDate=USART_ReceiveData(USART1);//读
		Serial_RxFlag=1;//置标志位
		USART_ClearITPendingBit(USART1,USART_IT_RXNE);
	}
}
```

#### main.c
```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "OLED.h"
#include "Serial.h"
uint8_t RxDate;
int main(void)
{
	OLED_Init();
	OLED_ShowString(1,1,"RxDate:");
	Serial_Init();
	while(1)
	{
		if(Serial_GetRxFlag()==1)
		{
			RxDate=Serial_GetRxDate();
			Serial_SendByte(RxDate);//发送到电脑上
			OLED_ShowHexNum(1,8,RxDate,2);//接收
		}
	}
}
```
## HEX数据收发包
### 数据包格式的定义
#### 数据包作用
把一个个单独的数据打包起来，方便我们进行多字节的数据通信
#### 数据包任务
把同一批的数据进行打包和分割，方便接收方进行识别
#### 常用方法
串口数据包通常采用的是额外添加包头包尾的方式
![[Pasted image 20260319133111.png]]
以上为HEX数据包
数据都以原始的字节数据本身呈现
优点：传输最简单，解析数据非常简单，比较适合一些模块发送原始的数据，比如一些使用串口通信的陀螺仪、温湿度传感器
缺点：灵活性不足，载荷容易和包头包尾重复
![[Pasted image 20260319133121.png|331]]
以上为文本数据包
每个字节经过了一层编码和译码，最后表现出来的就是文本格式，但是实际上每个文本字节的背后都是HEX数据
优点：数据直观易理解，非常灵活，适合一些输入指令进行人机交互的场合，比如蓝牙模块常用的AT指令，CNC和3D打印机常用的G代码
缺点：解析效率低
#### 讲解
定义0xFF包头，0xFE包尾，当接收到0xFF知道一个数据包来了，接着接收到的字节就当作是数据包的第1，2，3，个数据，存在一个数组里，最后跟一个包尾，当收到0xFE后，置一个标志位，告诉程序我收到了一个数据包，然后不断重复
问题：数据包中如果有数据为0xFF或0xFE，与包头包尾重复的话，可能会引起误判
解决方法
1. 限制载荷数据的范围，在发送的时候对数据进行限幅
2. 使用固定长度的数据包
3. 增加包头包尾数量，并且尽量避免出现数据与包头包尾重复的情况
4. 包头包尾可以删掉一个
问题：各种数据转换为字节流
### 数据包的收发流程
#### 发送
HEX数据包中，发送数据包就定义数组，填充数据，然后用上面的SendArray
文本数据包中，写字符串，调用SendString

![[Pasted image 20260319141505.png]]
固定包长HEX数据包接收方法

![[Pasted image 20260319141530.png]]
可变包长文本数据包接收方法
### 代码部分
#### 固定包长HEX数据包
##### 发送
Serial.c
```c
//以下两个数组只存储发送或者接收的载荷数据
uint8_t Serial_TxPacket[4];
uint8_t Serial_RxPacket[4];
//作为标志//如果收到Rx就置RxFlag为1(SET)
uint8_t Serial_RxFlag;

//调用后，TxPacket的4个数据，会自动加上包头包尾发送出去
void Serial_SendPacket(void)
{
	Serial_SendByte(0xFE);
	Serial_SendArray(Serial_TxPacket,4);
	Serial_SendByte(0xFF);
}
```
 Serial.h
因为我们需要在别的文件中(main.c)中使用Serial_TxPacket这个变量，需要在.h文件中进行一个数组声明
```c
extern uint8_t Serial_TxPacket[];
extern uint8_t Serial_RxPacket[];
```
 main.c
 ```c
Serial_TxPacket[0]=0x00;
Serial_TxPacket[1]=0x01;
Serial_TxPacket[2]=0x02;
Serial_TxPacket[3]=0x03;
Serial_SendPacket();
 ```
串口接收显示：
![[Pasted image 20260320204548.png]]
##### 接收
Serial.c
```c
void USART1_IRQHandler(void)
{
	static uint8_t RxState=0;
	static uint8_t pRxPacket=0;
	if(USART_GetFlagStatus(USART1,USART_FLAG_RXNE)==SET)
	{
		uint8_t RxDate=USART_ReceiveData(USART1);
		if(RxState==0)
		{
			if(RxDate==0xFE)
			{
				RxState=1;
				pRxPacket=0;
			}
		}
		else if(RxState==1)//依次接收四个数据，存在RxPacket数组里
		{
			Serial_RxPacket[pRxPacket]=RxDate;
			pRxPacket++;
			if(pRxPacket>=4)
			{
				RxState=2;
				
			}
		}
		else if(RxState==2)
		{
			if(RxDate==0xFE)
			{
				RxState=0;
				Serial_RxFlag=1;
			}
		}
		
		
		
		USART_ClearITPendingBit(USART1,USART_IT_RXNE);
	}
}

```
增加状态机状态判断，根据上面的状态转移图来写
main.c
```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "OLED.h"
#include "Serial.h"
#include "Key.h"
uint8_t RxDate;
uint8_t KeyNum;
int main(void)
{
	OLED_Init();
	Serial_Init();
	Key_Init();
	OLED_ShowString(1,1,"TxPacket");
	OLED_ShowString(3,1,"RxPacket");
	while(1)
	{
		KeyNum=Key_GetNum();
		if(KeyNum==1)
		{
			Serial_TxPacket[0]++;
			Serial_TxPacket[1]++;
			Serial_TxPacket[2]++;
			Serial_TxPacket[3]++;
			Serial_SendPacket();
			OLED_ShowHexNum(2,1,Serial_TxPacket[0],2);
			OLED_ShowHexNum(2,4,Serial_TxPacket[1],2);
			OLED_ShowHexNum(2,7,Serial_TxPacket[2],2);
			OLED_ShowHexNum(2,10,Serial_TxPacket[3],2);
			
		}
		
		if(Serial_GetRxFlag()==1)
		{
			OLED_ShowHexNum(4,1,Serial_RxPacket[0],2);
			OLED_ShowHexNum(4,4,Serial_RxPacket[1],2);
			OLED_ShowHexNum(4,7,Serial_RxPacket[2],2);
			OLED_ShowHexNum(4,10,Serial_RxPacket[3],2);
		}
	}
}
```
#### 可变包长文本数据包
##### 发送
因为是可变包长，载荷数不固定，所以不方便修改
直接在主函数里**SendString**或者**printf**就可以了
#### 接收
Serial.c
```c
void USART1_IRQHandler(void)
{
	static uint8_t RxState=0;
	static uint8_t pRxPacket=0;
	if(USART_GetFlagStatus(USART1,USART_FLAG_RXNE)==SET)
	{
		uint8_t RxDate=USART_ReceiveData(USART1);
		if(RxState==0)
		{
			if(RxDate=='@'&&Serial_RxFlag==0)
			{
				RxState=1;
				pRxPacket=0;
			}
		}
		else if(RxState==1)//依次接收四个数据，存在RxPacket数组里
		{
			if(RxDate=='\r')
			{
				RxState=2;
			}
			else
			{
				Serial_RxPacket[pRxPacket]=RxDate;
				pRxPacket++;
			}
		}
		else if(RxState==2)
		{
			if(RxDate=='\n')
			{
				RxState=0;
				Serial_RxPacket[pRxPacket]='\0';
				Serial_RxFlag=1;
			}
		}
		
		
		
		USART_ClearITPendingBit(USART1,USART_IT_RXNE);
	}
}
```
main.c
```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "LED.h"
#include "OLED.h"
#include "Serial.h"
#include <string.h>


int main(void)
{
	LED_Init();
	OLED_Init();
	Serial_Init();
	OLED_ShowString(1,1,"TxPacket");
	OLED_ShowString(3,1,"RxPacket");
	while(1)
	{
		if(Serial_RxFlag==1)
		{
			OLED_ShowString(4,1,"               ");
			OLED_ShowString(4,1,Serial_RxPacket);
			
			if(strcmp(Serial_RxPacket,"LED_ON")==0)
			{
				LED1_OFF();
				Serial_SendString("LED_ON\r\n");
				OLED_ShowString(2,1,"                ");
				OLED_ShowString(2,1,"LED_ON_OK");
			}
			else if(strcmp(Serial_RxPacket,"LED_OFF")==0)
			{
				LED1_ON();
				Serial_SendString("LED_OFF\r\n");
				OLED_ShowString(2,1,"                ");
				OLED_ShowString(2,1,"LED_OFF_OK");
			}
			else
			{
				Serial_SendString("ERROR_COMMAND");
				OLED_ShowString(2,1,"                ");
				OLED_ShowString(2,1,"ERROR_COMMAND");
			}
			
			Serial_RxFlag=0;
			
		}
	}
}

```


## FlyMcu串口下载
通过串口给STM32下载程序
没有STLINK可用这个软件通过串口下载程序
必须连接串口USART1






## STLINK Utility STLINK下载
配合STLINK使用的工具
通过STLINK给STM32下载程序
必须连接STLINK
跳线帽恢复成两个都在最左边，复位
![[Pasted image 20260320235053.png]]
打开选项字节的配置
Target-Option Bytes
![[Pasted image 20260320235909.png]]
STLINK更新固件功能
STLINK-Firmware update-Connect
提示重启后，拔掉重新插Connect
## 状态机













