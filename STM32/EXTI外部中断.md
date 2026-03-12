## 中断系统
中断：在主程序运行过程中，出现了特定的中断触发条件(中断源)，使得CPU暂停当前正在运行的程序，转而去处理中断程序，处理完成后又返回原来被暂停的位置继续运行
中断优先级：当前有多个中断源同时申请时，CPU会根据中断源的轻重缓急进行裁决，优先响应更加紧急的中断源
中断嵌套：当一个中断程序正在运行时，又有新的更高优先级的中断源申请中断，CPU再次暂停当前中断程序，转而去处理新的中断程序，处理完成后依次进行返回
## STM32中断
68个可屏蔽中断通道，包含EXTI、TIM、ADC、USART、SPI、I2C、RTC等多个外设
使用NVIC统一管理中断，每个中断通道都拥有16个可编程的优先等级，可对优先级进行分组，进一步设置抢占优先级和响应优先级
## NVIC优先级
- 响应优先级：等上一个运行完立即运行
- 抢占优先级：不等上一个运行完，立即执行
- ==优先级数越**小**，优先级越**高**，0是最高优先级==
- NVIC的中断优先级由优先级寄存器的4位（0~15）决定，这4位可以进行切分，分为高n位的抢占优先级和低4-n位的响应优先级
- 抢占优先级高的可以中断嵌套，响应优先级高的可以优先排队，抢占优先级和响应优先级均相同的按中断号排队


| 分组方式 | 抢占优先级      | 响应优先级      |
| ---- | ---------- | ---------- |
| 分组0  | 0位，取值为0    | 4位，取值为0~15 |
| 分组1  | 1位，取值为0~1  | 3位，取值为0~7  |
| 分组2  | 2位，取值为0~3  | 2位，取值为0~3  |
| 分组3  | 3位，取值为0~7  | 1位，取值为0~1  |
| 分组4  | 4位，取值为0~15 | 0位，取值为0    |
## EXTI外部中断
- XTI可以监测指定GPIO口的电平信号，当其指定的GPIO口产生电平变化时，EXTI将立即向NVIC发出中断申请，经过NVIC裁决后即可中断CPU主程序，使CPU执行EXTI对应的中断程序
- ==引脚电平变化，申请中断==
- 支持的触发方式：上升沿/下降沿/双边沿/软件触发
- 支持的GPIO口：所有GPIO口，但相同的Pin不能同时触发中断
- 通道数：16个GPIO_Pin，外加PVD输出、RTC闹钟、USB唤醒、以太网唤醒
- 触发响应方式：中断响应(正常流程，引脚电平触发中断)/事件响应(不触发中断，而是触发别的外设操作，属于外设之间的联合工作)

### 用在突发不能等待的场景
比如旋钮，是突发的，并且转瞬即逝，就需要外部中断

## EXTI基本结构

![[Pasted image 20260310195707.png]]
工作路径如上图所示
根据该图可写代码
1. 配置RCC，开启相关的外设时钟(需要开启的是GPIO与AFIO，EXTI和NVIC不需要开启，因为NVIC是内核的外设，而RCC管的是外设的时钟)
2. 配置GPIO，选择我们的端口为输入模式
3. 配置AFIO外部中断引脚选择，选择我们用的这一路的GPIO，连接到后面的EXTI
4. 配置EXTI，选择边沿触发方式和选择触发响应方式(中断响应和事件响应)
5. 配置NVIC，给中断选择一个合适的优先级
6. 通过NVIC，外部中断信号就能进入CPU了
7. CPU收到中断信号，跳转到中断函数里执行中断程序
## AFIO的函数在GPIO中，
```c

void GPIO_AFIODeInit(void);//复位AFIO外设//调用函数后，AFIO外设的配置就会全部清除
void GPIO_PinLockConfig(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);//锁定GPIO配置//调用函数，指定某个引脚，引脚的配置会被锁定，放置意外更改
void GPIO_EventOutputConfig(uint8_t GPIO_PortSource, uint8_t GPIO_PinSource);//配置AFIO的事件输出功能
void GPIO_EventOutputCmd(FunctionalState NewState);//配置AFIO的事件输出功能

------
void GPIO_PinRemapConfig(uint32_t GPIO_Remap, FunctionalState NewState);//引脚重映射（第一个参数重映射方式，第二个参数新的状态）
void GPIO_EXTILineConfig(uint8_t GPIO_PortSource, uint8_t GPIO_PinSource);//外部中断需要用的函数//调用这个函数，可以配置AFIO的数据选择器，来选择我们想要的中断引脚
------

void GPIO_ETH_MediaInterfaceConfig(uint32_t GPIO_ETH_MediaInterface);//和以太网有关
```
## EXTI
```c
-----------
//以下三个基本每个外设都有
void EXTI_DeInit(void);//清空EXTI配置，恢复成上电默认状态
void EXTI_Init(EXTI_InitTypeDef* EXTI_InitStruct);//根据结构体里的参数配置EXTI外设//初始化EXTI用
void EXTI_StructInit(EXTI_InitTypeDef* EXTI_InitStruct);//把参数传递的结构体变量赋一个默认值
-----------


void EXTI_GenerateSWInterrupt(uint32_t EXTI_Line);//软件触发外部中断//调用这个函数，参数给一个指定的中断线，就能软件触发一次这个外部中断



-----------
//以下函数也是库函数模板函数
//本质上都是对状态寄存器的读写
FlagStatus EXTI_GetFlagStatus(uint32_t EXTI_Line);//可以获取指定的标志位是否被置1了
void EXTI_ClearFlag(uint32_t EXTI_Line);//对置1的标志位进行清除
ITStatus EXTI_GetITStatus(uint32_t EXTI_Line);//获取中断标志位是否被置1了
void EXTI_ClearITPendingBit(uint32_t EXTI_Line);//清楚中断挂起标志位
//想在主程序中查看和清除标志位，用上面两个
//想在中断函数里查看和清除标志位，用下面两个
```

## NVIC
在misc中
```c
void NVIC_PriorityGroupConfig(uint32_t NVIC_PriorityGroup);//用来中断分组，参数是中断分组的方式
void NVIC_Init(NVIC_InitTypeDef* NVIC_InitStruct);//初始化
void NVIC_SetVectorTable(uint32_t NVIC_VectTab, uint32_t Offset);//设置中断向量表
void NVIC_SystemLPConfig(uint8_t LowPowerMode, FunctionalState NewState);//系统低功耗配置
void SysTick_CLKSourceConfig(uint32_t SysTick_CLKSource);

```

## 对射式红外传感器计次
### CountSensor.c文件
```c
#include "stm32f10x.h"                  // Device header
uint16_t CountSensor_Count;
void CountSensor_Init(void)
{
	//配置时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB,ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO,ENABLE);
	//配置GPIO
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode=GPIO_Mode_IPU;//上拉输入，默认高电平
	GPIO_InitStructure.GPIO_Pin=GPIO_Pin_14;
	GPIO_InitStructure.GPIO_Speed=GPIO_Speed_50MHz;
	GPIO_Init(GPIOB,&GPIO_InitStructure);
	//配置AFIO外部中断引脚选择
	GPIO_EXTILineConfig(GPIO_PortSourceGPIOB, GPIO_PinSource14);
	//配置EXTI
	EXTI_InitTypeDef EXTI_InitStructure;
	EXTI_InitStructure.EXTI_Line=EXTI_Line14;//EXTI的14线路配置为中断模式
	EXTI_InitStructure.EXTI_LineCmd=ENABLE;//开启中断
	EXTI_InitStructure.EXTI_Mode=EXTI_Mode_Interrupt;
	EXTI_InitStructure.EXTI_Trigger=EXTI_Trigger_Falling;//下降沿触发
	EXTI_Init(&EXTI_InitStructure);
	//配置NVIC
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
	
	NVIC_InitTypeDef NVIC_InitStructure;
	NVIC_InitStructure.NVIC_IRQChannel=EXTI15_10_IRQn;
	NVIC_InitStructure.NVIC_IRQChannelCmd=ENABLE;
	NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority=1;
	NVIC_InitStructure.NVIC_IRQChannelSubPriority=1;
	NVIC_Init(&NVIC_InitStructure);
}
uint16_t CountSensor_Get(void)
{
	return CountSensor_Count/2;
}
//配置中断函数，中断函数不需要调用，是自动执行
//尽量不要在中断程序中加延时，会“阻塞主程序”
void EXTI15_10_IRQHandler(void)//中断函数
{
	if(EXTI_GetITStatus(EXTI_Line14)==SET)//中断标志位判断//确保是想要的中断触发的函数
	{
		CountSensor_Count++;
		EXTI_ClearITPendingBit(EXTI_Line14);//清除中断标志位
	}
}
```
虽然选择了下降沿触发，但是出现了上升沿和下降沿同时触发的情况，在此基础上
对于uint16_t CountSensor_Get(void)的返回值进行修改
变成return CountSensor_Count/2;
在中断程序中应避免添加中断函数，会“阻塞主程序”
### main.c
```c
#include "stm32f10x.h"                  // Device header
#include "OLED.h"
#include "CountSensor.h"
int main(void)
{
	OLED_Init();
	CountSensor_Init();
	OLED_ShowString(1,1,"Count:");
	while(1)
	{
		OLED_ShowNum(1,7,CountSensor_Get(),5);
	}
}
```
## 旋转编码器