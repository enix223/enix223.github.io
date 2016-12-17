**定时器寄存器**

  

相关寄存器：

  * 计数寄存器（counter register TIMx_CNT）
  * 预分频寄存器（prescaler register TIMx_PSC）
  * 自动重载寄存器（auto reload register TIMx_ARR）
  * 重复计数寄存器（ repetition counter register TIMx_RCR）

  

TIMx_ARR首先预载入。预载入寄存器会在每一次计时器更新事件发生的时候，写入影子寄存器，这由TIMx_CR1的ARPE位决定
。当计时器溢出，并且TIMx_CR1的UDIS位为0的时候，就会产生一次UEV更新事件。

  

定时器的时钟由系统时钟的预分频后的时钟（CK_CNT）提供，但只是在TIMx_CR1的CEN位为1的时候才起作用。

  

**关于预分频**

  

预分频是用于将时钟频率进行切分，切分因子介于1至65536之间。它时基于一个由16位寄存器控制的16位计数器（TIMx_PSC）。它可以随时改变，并在下一次
UEV产生的时候起作用。

  

**计算方法**

  

TIM的时钟是由PCKx提供，根据stm32f4xx的reference 手册，RCC关于Timmer时钟的描述如下：

  1. When TIMPRE bit of the RCC_DCKCFGR register is reset, if APBx prescaler is 1, then TIMxCLK = PCLKx, otherwise TIMxCLK = 2x PCLKx.

  2. When TIMPRE bit in the RCC_DCKCFGR register is set, if APBx prescaler is 1,2 or 4, then TIMxCLK = HCLK, otherwise TIMxCLK = 4x PCLKx.

  

所以，根据需要，我们可以设置TIM的时钟为2*PCLKx或者4*PCLKx。

  

假如TIM2的时钟是2 * PCLK1，而

  

**_PCK1 = HCLK / 4 =&gt; PCLK1 = SystemCoreClock / 4_**

  

那么

  

**_TIM2CLK ＝ SystemCoreClock  / 4 * 2 = SystemCoreClock / 2_**

  

Prescaler与输入输出频率的关系如下：

  

_**fout = fin / ( prescaler + 1 )**_

  

也就可以推导出：

  

**_prescaler = fin / fout - 1_**

  

而此时，输入时钟TIM2CLK为 SystemCoreClock / 2，假设我们需要输出10 KHz的时钟频率，那么：

  

**_prescaler = SystemCoreClock / 2 / 10000 - 1_**

  

而TIM.Period则控制计数多少次会产生一次中断。

  

综上所述，prescaler可以理解为设置定时器1秒计数多少次的设置参数，而period控制计数达到多少次之后，会产生一次中断。假设，prescaler为1
000, 而period为4000，那么定时器需要4秒之后，就会产生一次中断。

  

**编程实现**

  

  uwPrescalerValue = (uint32_t)((SystemCoreClock / 2) / 10000) - 1;

  TimHandle.Instance = TIMx;

  TimHandle.Init.Period = 1000 - 1;

  TimHandle.Init.Prescaler = uwPrescalerValue;

  TimHandle.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;

  TimHandle.Init.CounterMode = TIM_COUNTERMODE_UP;

  

Instance 设置初始化那个TIMER,
而Period设置计数多少次后会产生一次中断。Prescaler设置定时器1秒计数多少次的参数。而ClockDivision设置CK_INT与dead
time和sample clock的关系：

  

00: tDTS=tCK_INT

01: tDTS=2*tCK_INT

10: tDTS=4*tCK_INT

11: Reserved, do not program this value

  

最后，CounterMode设置计时模式，例如上向上计时，向下计时等。

  

**关于 PWM**

  

PWM有两种模式PWM mode 1 ( 110 in OCxM bits TIMx_CCMRx寄存器) 和 PWM mode 2 ( 111 in
OCxM bits TIMx_CCMRx寄存器)

  

PWM mode 1 - 向上计数模式，channel x 只有在TIMx_CNT &lt;
TIMx_CCR1满足的情况下才有效（active），否则为inactive；而在向下计数模式中，channel x 只有在TIMx_CNT &gt;
TIMx_CCR1满足的情况下才有效，否则为inactive。

  

PWM mode 2 - 向上计数模式，channel x 只有在TIMx_CNT &lt;
TIMx_CCR1满足的情况下才无效（inactive），否则为active；而在向下计数模式中，channel x 只有在TIMx_CNT &gt;
TIMx_CCR1满足的情况下才无效，否则为active。

  

TIMx_CCR1/2/3/4 - Channel 1/2/3/4比较寄存器。

  



  

  

