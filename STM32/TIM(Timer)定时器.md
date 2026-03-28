## 基本概念
作用：对输入时钟进行计数，并在计数值达到设定值时触发中断
数`72*10^6次是1秒，那数1次就是1/(72*10^6)秒，数72次是1/(72*10^6)*72秒，也就是1us`
主频72MHz ， 一秒可以计数7200万次，72million hz ,1hz一秒计数1次 ，1MHz 一秒计数一百万次，1ms 计数1000次
时基单元：
- **计数器**(执行计数定时的寄存器)
- **预分频器**(对计数器进行分频)
- **自动重装寄存器**(计数的目标值，想要计的多少个时钟申请中断)
- 都是**16**位的
- 2的16次方是65536，如果预分频器设置最大，自动重装也设置最大，72MHz计数时钟下基本可以实现最大59.65s的定时
- 中断频率：72M/65536/65536，中断频率取倒数就是59.65s
- `T=1/f(已分频)，t=n*T`。T是时钟分频后周期，n是重装计数值，t是可计时的时长。
功能：定时中断、内外时钟源选择、输入捕获、输出比较、编码器接口、主从触发模式等
根据复杂度和应用场景分为了高级定时器、通用定时器、基本定时器三种类型
### 定时器类型

| 类型    | 编号                  | 总线   | 功能                                                       |
| ----- | ------------------- | ---- | -------------------------------------------------------- |
| 高级定时器 | TIM1、TIM8           | APB2 | 拥有通用定时器全部功能，*并额外具有重复计数器、死区生成、互补输出、刹车输入等功能*(主要用于三相无刷电机驱动) |
| 通用定时器 | TIM2、TIM3、TIM4、TIM5 | APB1 | 拥有基本定时器全部功能，并额外具有内外时钟源选择、输入捕获、输出比较、编码器接口、主从触发模式等功能       |
| 基本定时器 | TIM6、TIM7           | APB1 | 拥有定时中断、主模式触发DAC的功能                                       |
STM32F103C8T6定时器资源：TIM1、TIM2、TIM3、TIM4

|        | 基本定时器 | 通用定时器 | 高级定时器 |
| ------ | ----- | ----- | ----- |
| 向上计数   | 有     | 有     | 有     |
| 向下计数   | 无     | 有     | 有     |
| 中央对其模式 | 无     | 有     | 有     |

### 原理
#### 基本定时器
![[Pasted image 20260321153904.png]]

##### 时钟选择：只能选择内部72MHz时钟
##### 预分频：
对输入的基准频率提前进行分频，从0开始(输出频率=输入频率)，预分频为1(输出频率=输入频率/2=36MHz)，预分频器的值和实际的分配系数相差1!==实际分频系数=预分频器的值+1==，预分频器是16位的，最大值可以写65535分频
##### 计数器：
根据预分频器的输出频率进行向上计数，从0开始增加到目标值时，产生中断
自动重装寄存器：用来存储目标值，当计数器到达目标值时，产生中断信号，清零计数器
UI带折线的向上箭头：表示这里会产生中断信号
##### 向下的箭头：
更新事件(更新事件不会触发中断，但可以触发内部其他电路的工作)->更新中断：计数值等于自动重装值产生的中断。更新中断后会通往NVIC，我们再配置好NVIC的定时器通道，定时器的更新中断就能得到CPU的响应了
##### 主模式触发DAC：
让内部的硬件在不受程序的控制下实现自动运行，可以极大地减轻CPU的负担
用途：在使用时，用DAC输出一段波形，就需要每隔一段时间来触发一次DAC，让它输出下一个电压点
计数器到了重装载值，产生了一个更新事件，然后内部硬件映射到TRGO，通过TRGO来自动触发DAC转换，不再需要再到定时器中断里再去执行读取DAC数值了。
#### 通用定时器
![[Pasted image 20260321161605.png]]
计数模式：向上计数、向下计数、中央对其模式(先向上自增到重装值申请中断后向下自减到0申请中断)
选择时钟：除了内部时钟72MHz还可以选择外部时钟
TIM_ETR(在引脚定义图看时PA0)接一个外部==方波==时钟，配置内部的极性选择、边沿检测和预分频器电路，再配置输入滤波电路(这些电路可以对信号进行一个滤波)，信号分两路
一路：外部时钟模式2：信号走ETRF进入触发控制器，选择作为时基单元的时钟，场景：想另一路：在ETR外部引脚提供时钟或对ETR时钟进行计数，把时钟当做计数器来用
还有TRGI可以提供时钟，主要用作触发输入来使用，触发输入可以触发定时器的从模式
外部时钟模式1：当TRTI作为外部时钟来使用，通过这一路的外部时钟有**ETR引脚**的信号、**ITR信号**(时钟信号来自其他定时器，可根据以下的表实现级联的功能)、**TI1F_ED**(ED是边沿的意思上升沿和下降沿均有效，连接输入捕获单元的CH1，由CH1提供时钟)、**TI1FP1**(连接CH1引脚的引脚时钟)和**TI2FP2**(连接到CH2引脚的时钟)
![[Pasted image 20260321165317.png]]
编码器接口：读取正交编码器的输出波形
主模式输出：这部分路s可以把内部的一些事件映射到TRGO引脚，用于触发其他定时器、DAC或ADC，比如基本定时器分析的将更新事件映射到TRGO用于触发DAC类似
![[Pasted image 20260321171331.png]]
输出比较电路：有四个通道，分别对应CH1-CH4的引脚，用于输出PWM波形，驱动电机
输入捕获电路：有四个通道，分别对应CH1-CH4的引脚，用于测量输入方波的频率等
捕获/比较寄存器：输入捕获和输出比较电路公用的
#### 高级定时器
![[Pasted image 20260321175217.png]]
申请中断的地方添加了一个重复次数计数器：实现每隔几个计数周期，才发生一次更新事件和更新中断，相当于对输出的更新信号又做了一次分频，由于是8位，原本的59.65s，可以再乘上256
输出PWM引脚前三路驱动无刷电机
死去生成电路：为了防止互补输出的PWM驱动桥臂时，在开关切换的瞬间，由于器件的不理想，造成短暂的直通现象，在开关切换的瞬间产生一定时长的死区，让桥臂的上下管全部关断，防止直通现象
刹车输入：给电机驱动提供安全保障的，外部引脚BKIN产生刹车信号或内部时钟生效，产生故障，控制电路就会自动切断电机的输出，防止意外的发生
## 定时中断基本结构
![[Pasted image 20260321175717.png]]
初始化流程：
1. RCC开启时钟(定时器基准时钟和整个外设的工作时钟就会同时开启)
2. 选择时基单元的是时钟源(对于定时中断，选择内部时钟源)
3. 配置时基单元(包括预分频器、自动重装器、计数模式等，用一个结构体进行配置)
4. 配置输出中断控制，允许更新中断输出到NVIC
5. 配置NVIC，在NVIC中打开定时器中断通道，并分配一个优先级
6. 运行控制，使能一下计数器
7. 定时器中断函数，每隔一段时间自动执行一次
## 时序
### 预分频器时序
![[Pasted image 20260321194934.png]]

CK_PSC预分频器的输入时钟(选内部时钟的话一般是72MHz)
CNT_EN计数器使能(高电平运行，低电平停止)
计数器计数频率：CK_CNT = CK_PSC / (PSC + 1)
### 计数器时序
![[Pasted image 20260321194940.png]]
计数器溢出频率：CK_CNT_OV(定时频率) = CK_CNT / (ARR + 1)= CK_PSC(预分频器的输入时钟) / (PSC + 1) / (ARR + 1)
PSC预分频少，ARR自动重装多，以比较高频率计比较多的数
PSC预分频多，ARR自动重装少，以比较低频率计比较少的数
如果说我们现在要定时1s，定时频率为1Hz，CK_PSC选择内部时钟72MHz
就是说(PSC + 1) * (ARR + 1)=72MHz，自主分配
我们可以给PSC=7200-1，ARR=1000-1
在此给预分频是对72MHz进行7200分配，得到的是10K的计数频率，在10K的频率下，计10000数，就是1s的时间
### 计数器无预装时序
![[Pasted image 20260321194950.png]]
ARPE选择是否有预装功能，1为有，0为没有
### 计数器有预装时序
![[Pasted image 20260321195115.png]]
## RCC时钟树
![[Pasted image 20260322101840.png]]
## OC(Output Compare)输出比较
输出比较可以通过比较CNT与CCR寄存器值的关系，来对输出电平进行置1、置0或翻转的操作，用于==输出一定频率和占空比的PWM波形==
每个高级定时器和通用定时器都拥有4个输出比较通道
高级定时器的前3个通道额外拥有死区生成和互补输出的功能
输出比较：比较CNT与CCR的值
### PWM(Plus Width Modulation)脉冲宽度调制
在数字系统等效输出模拟量，实现LED亮度和控制电机
以一个很快的频率，给电机通电断电，那么电机的速度九可以维持在一个中等速度
在==**具有惯性的系统**==中，可以通过对一系列脉冲的宽度进行调制，来等效地获得所需要的模拟参量，常应用于==电机控速==等领域
PWM参数：
频率 = 1 / Ts  (越快越平稳但性能开销也越大)
占空比 = Ton(高电平实践)/ Ts(一整个周期的时间)
分辨率 = 占空比变化步距  (占空比变化的精细程度)
### 输出比较模式

| 模式            | 描述                                                                                       |
| ------------- | ---------------------------------------------------------------------------------------- |
| 冻结            | CNT=CCR时，CNT和CCR无效，REF保持为原状态                                                             |
| 匹配时置有效电平(高电平) | CNT=CCR时，REF置有效电平                                                                        |
| 匹配时置无效电平(低电平) | CNT=CCR时，REF置无效电平                                                                        |
| 匹配时电平翻转       | CNT=CCR时，REF电平翻转                                                                         |
| 强制为无效电平       | CNT与CCR无效，REF强制为无效电平                                                                     |
| 强制为有效电平       | CNT与CCR无效，REF强制为有效电平                                                                     |
| PWM模式1        | 向上计数：CNT<CCR时，REF置有效电平，CNT≥CCR时，REF置无效电平<br><br>向下计数：CNT>CCR时，REF置无效电平，CNT≤CCR时，REF置有效电平 |
| PWM模式2        | 向上计数：CNT<CCR时，REF置无效电平，CNT≥CCR时，REF置有效电平<br><br>向下计数：CNT>CCR时，REF置有效电平，CNT≤CCR时，REF置无效电平 |
### PWM基本结构
![[Pasted image 20260328164352.png]]





### 参数计算
PWM频率：	Freq = CK_PSC / (PSC + 1) / (ARR + 1)
PWM占空比：	Duty = CCR / (ARR + 1)
PWM分辨率：	Reso = 1 / (ARR + 1)
讲解：
因为始终对应，PWM频率始终等于计数器的更新频率

### 输出比较通道(高级)
![[Pasted image 20260328165447.png]]
两个MOS管用于驱动电机
三个MOS管用于驱动三相无刷电机
**死区生成电路**：在上下管切断关闭的时候，延时一会再另外一个管子换个导通，可以避免上下管同时导通的情况
## 舵机
- 根据输入的PWM信号占空比来控制输出角度的装置
- 输入PWM信号要求：周期为20ms，对应的是50Hz，高电平宽度(占空比)为0.5ms~2.5ms
![[Pasted image 20260328215555.png]]
对应关系都是线性分配，按照比例来，给一个PWM输出轴就固定在一个角度
### 应用场景
机器人机械臂，可以用舵机来控制关节
可以把PWM当成一个通信协议或是一个模拟输出来用
### 舵机硬件电路
![[Pasted image 20260328222837.png]]
## 直流电机及驱动
- 直流电机是一种将电能转换为机械能的装置，有两个电极，当电极正接时，电机正转，当电极反接时，电机反转
- 直流电机属于大功率器件，GPIO口无法直接驱动，需要配合**电机驱动电路**来操作
- TB6612是一款双路H桥型的直流电机驱动芯片，可以驱动两个直流电机并且控制其转速和方向
### 驱动板硬件电路
![[Pasted image 20260328223832.png]]
由两路推挽电路组成，O1O2接电机：
左上右下导通，电流从左流向右，右上左下导通，电流从右流向左
H桥控制电流流过的方向，可以控制电机正反转
### 直流电机硬件电路
![[Pasted image 20260328223002.png]]
三个引脚(如PWMA,AIN2,AIN1)给一个低功率的控制信号，驱动电路就会从VM汲取电流，来输出到电机，完成**低功率的控制信号控制大功率设备**的目的
STBY(Stand By)是待机控制脚，接**GND**，芯片不工作，处于**待机**状态；接**逻辑电源VCC**，芯片**正常工作**。
不需要待机模式，直接接VCC 3.3V，如果需要的话，任意接一个GPIO，给高低电平控制




## 定时器库函数
```c
void TIM_DeInit(TIM_TypeDef* TIMx);//恢复缺省配置
void TIM_TimeBaseInit(TIM_TypeDef* TIMx, TIM_TimeBaseInitTypeDef* TIM_TimeBaseInitStruct);//时基单元初始化



void TIM_TimeBaseStructInit(TIM_TimeBaseInitTypeDef* TIM_TimeBaseInitStruct);//把结构体变量赋一个默认值


void TIM_Cmd(TIM_TypeDef* TIMx, FunctionalState NewState);//使能计数器

void TIM_ITConfig(TIM_TypeDef* TIMx, uint16_t TIM_IT, FunctionalState NewState);//使能中断输出信号

----------------------------------
//以下六个函数是时基单元的时钟源选择
void TIM_InternalClockConfig(TIM_TypeDef* TIMx);//选择内部时钟
void TIM_ITRxExternalClockConfig(TIM_TypeDef* TIMx, uint16_t TIM_InputTriggerSource);//选择ITRx其他定时器的时钟
void TIM_TIxExternalClockConfig(TIM_TypeDef* TIMx, uint16_t TIM_TIxExternalCLKSource,
                               uint16_t TIM_ICPolarity, uint16_t ICFilter);//选择TIx捕获通道的时钟
void TIM_ETRClockMode1Config(TIM_TypeDef* TIMx, uint16_t TIM_ExtTRGPrescaler, uint16_t TIM_ExtTRGPolarity,
                             uint16_t ExtTRGFilter);//选择ETR外部时钟模式1输入的时钟
void TIM_ETRClockMode2Config(TIM_TypeDef* TIMx, uint16_t TIM_ExtTRGPrescaler, 
                             uint16_t TIM_ExtTRGPolarity, uint16_t ExtTRGFilter);//选择ETR外部时钟模式2输入的时钟
void TIM_ETRConfig(TIM_TypeDef* TIMx, uint16_t TIM_ExtTRGPrescaler, uint16_t TIM_ExtTRGPolarity,
                   uint16_t ExtTRGFilter);//单独配置ETR引脚的预分频器、极性、滤波器等参数
---------------------------------
void TIM_PrescalerConfig(TIM_TypeDef* TIMx, uint16_t Prescaler, uint16_t TIM_PSCReloadMode);//单独写预分频值
void TIM_CounterModeConfig(TIM_TypeDef* TIMx, uint16_t TIM_CounterMode);//改变计数器的计数模式

void TIM_ARRPreloadConfig(TIM_TypeDef* TIMx, FunctionalState NewState);//自动重装器预装功能配置

void TIM_SetCounter(TIM_TypeDef* TIMx, uint16_t Counter);//给计数器写入一个值//手动给一个计数值
void TIM_SetAutoreload(TIM_TypeDef* TIMx, uint16_t Autoreload);//给自动重装器写入一个值//手动给一个自动重装值

uint16_t TIM_GetCounter(TIM_TypeDef* TIMx);//获取当前计数器的值//想看预分频值

///以下四个是和标志位相关函数
FlagStatus TIM_GetFlagStatus(TIM_TypeDef* TIMx, uint16_t TIM_FLAG);
void TIM_ClearFlag(TIM_TypeDef* TIMx, uint16_t TIM_FLAG);
ITStatus TIM_GetITStatus(TIM_TypeDef* TIMx, uint16_t TIM_IT);
void TIM_ClearITPendingBit(TIM_TypeDef* TIMx, uint16_t TIM_IT);

```

## TIM基本定时
目的：定时执行程序
定一个固定时间，每隔这个时间产生中断
用处：时钟，秒表，或用一些程序算法
### 定时器定时中断
Timer.c
```C
#include "stm32f10x.h"                  // Device header
extern uint16_t Num;
void Timer_Init(void)
{
	//在此开启TIM2，TIM2在APB1线上
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);
	//选择时基单元时钟
	TIM_InternalClockConfig(TIM2);//选择内部时钟
	//配置时基单元
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
	TIM_TimeBaseInitStruct.TIM_ClockDivision=TIM_CKD_DIV1;//1分频
	TIM_TimeBaseInitStruct.TIM_CounterMode=TIM_CounterMode_Up;//向上计数
	TIM_TimeBaseInitStruct.TIM_Period=10000-1;//
	TIM_TimeBaseInitStruct.TIM_Prescaler=7200-1;//PSC预分频器
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter=0;//重复计数器
	TIM_TimeBaseInit(TIM2,&TIM_TimeBaseInitStruct);
	TIM_ClearFlag(TIM2,TIM_FLAG_Update);
	//使能更新中断
	TIM_ITConfig(TIM2, TIM_IT_Update, ENABLE);
	//配置NVIC
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
	NVIC_InitTypeDef NVIC_InitStruct;
	NVIC_InitStruct.NVIC_IRQChannel=TIM2_IRQn;
	NVIC_InitStruct.NVIC_IRQChannelCmd=ENABLE;
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority=2;
	NVIC_InitStruct.NVIC_IRQChannelSubPriority=1;
	NVIC_Init(&NVIC_InitStruct);
	//启动定时器
	TIM_Cmd(TIM2,ENABLE);
}
void TIM2_IRQHandler(void)
{
	if(TIM_GetITStatus(TIM2,TIM_IT_Update)==SET)
	{
		Num++;
		TIM_ClearITPendingBit(TIM2,TIM_IT_Update);
	}
}
```
main.c
```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "OLED.h"
#include "Timer.h"
uint16_t Num;
int main(void)
{
	OLED_Init();
	Timer_Init();
	OLED_ShowString(1,1,"Num:");
	OLED_ShowString(2,1,"CNT:");
	
	while(1)
	{
		OLED_ShowNum(1,5,Num,5);
		OLED_ShowNum(2,5,TIM_GetCounter(TIM2),5);
	}
}

```
### 定时器外部时钟
Timer.c
```c
#include "stm32f10x.h"                  // Device header
extern uint16_t Num;
void Timer_Init(void)
{
	//在此开启TIM2，TIM2在APB1线上//开启GPIO时钟
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode=GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin=GPIO_Pin_0;
	GPIO_InitStructure.GPIO_Speed=GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStructure);
	//选择时基单元时钟
	TIM_ETRClockMode2Config(TIM2,TIM_ExtTRGPSC_OFF, TIM_ExtTRGPolarity_NonInverted, 0x00);
	//配置时基单元
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
	TIM_TimeBaseInitStruct.TIM_ClockDivision=TIM_CKD_DIV1;//1分频
	TIM_TimeBaseInitStruct.TIM_CounterMode=TIM_CounterMode_Up;//向上计数
	TIM_TimeBaseInitStruct.TIM_Period=10-1;//
	TIM_TimeBaseInitStruct.TIM_Prescaler=1-1;//不需要分配
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter=0;//重复计数器
	TIM_TimeBaseInit(TIM2,&TIM_TimeBaseInitStruct);
	TIM_ClearFlag(TIM2,TIM_FLAG_Update);
	//使能更新中断
	TIM_ITConfig(TIM2, TIM_IT_Update, ENABLE);
	//配置NVIC
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
	NVIC_InitTypeDef NVIC_InitStruct;
	NVIC_InitStruct.NVIC_IRQChannel=TIM2_IRQn;
	NVIC_InitStruct.NVIC_IRQChannelCmd=ENABLE;
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority=2;
	NVIC_InitStruct.NVIC_IRQChannelSubPriority=1;
	NVIC_Init(&NVIC_InitStruct);
	//启动定时器
	TIM_Cmd(TIM2,ENABLE);
}
uint16_t Timer_GetCounter(void)
{
	return TIM_GetCounter(TIM2);
}
void TIM2_IRQHandler(void)
{
	if(TIM_GetITStatus(TIM2,TIM_IT_Update)==SET)
	{
		Num++;
		TIM_ClearITPendingBit(TIM2,TIM_IT_Update);
	}
}
```
main.c
```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "OLED.h"
#include "Timer.h"
uint16_t Num;
int main(void)
{
	OLED_Init();
	Timer_Init();
	OLED_ShowString(1,1,"Num:");
	OLED_ShowString(2,1,"CNT:");
	
	while(1)
	{
		OLED_ShowNum(1,5,Num,5);
		OLED_ShowNum(2,5,Timer_GetCounter(),5);
	}
}
```
因为是外部输入，所以加上GPIO_A0的初始化代码，然后再加上CNT的读取封装函数，规范化
## TIM定时器输出比较
产生PWM波形，用于驱动电机、舵机等设备
## TIM定时器输入捕获
使用输入捕获模块来实现测量方波频率
## TIM定时器的编码器接口
使用该接口，更方便的读取正交编码器的输出波形，在编码电机测速中应用广泛