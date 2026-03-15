USART（Universal Synchronous/Asynchronous Receiver/Transmitter）通用同步/UART异步收发器

USART分为发送和接收两部分：
发送：将数据寄存器的一个字节数据自动转换为协议规定的波形，然后从TX引脚发送出去
接收：自动接受RX引脚波形，按照协议规定解码为一个字节数据，存放在数据寄存器里

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
获取发送和接收的状态，就调用获取标志位的函数
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

