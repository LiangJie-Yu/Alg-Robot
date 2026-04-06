## 同型号之间的修改
以STM32F103C8T6(最小系统板)到STM32F103ZET6(正点原子开发板)为例
1. **改芯片型号**：魔法棒-Device-STM32F103ZET6
2. **改芯片启动型号选择**：三个盒子(文件管理)-start-startup_stm32f10x_hd.s![[Pasted image 20260406115821.png]]
3. **改宏定义**：魔法棒-C/C++-Preprocessor Symbols-Define-"STM32F10X_HD,USE_STDPERIPH_DRIVER"![[Pasted image 20260406115757.png]]
因为C8T6是中容量芯片，ZET6是大容量芯片