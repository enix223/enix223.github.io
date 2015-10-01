---
category: blog
layout: blog
title: "xCopter开发日记 - STM32F4 LED example"
date: 2015-05-04 23:07:47 +0800
comments: true
---

本文阐述如何使用IAR，和STM32F401-Discovery开发板，实现最简单的LED灯点亮熄灭的操作。程序一开始初始化开发板上面的四个LED灯，这四个灯的定义请查看代码stm32f401-discovery.h, 它们分别为：

* LED4
* LED3
* LED5
* LED6

然后程序初始化SystemTick，并把Tick设定为1秒，然后程序进入循环，反转4个LED的状态，并进入1秒的等待。

## 准备工作

### 开发需要用到的软件 

1. [STM32F4Cube][STM32F4Cube] 由ST公司提供给用户的硬件抽象层(HAL)，和stm32f4所有开发板对应的例子和ARM公司的DSP库，非常实用。  
2. [ST-LINK][ST-LINK] ST-LINK的WINDOWS USB驱动
3. IAR 7.2

### 需要用到的硬件

1. STM32F401-Discovery 开发板，芯片为STM32F407VG。TB有售。

## IAR项目配置

1. General Options > Target > Processor variant > Device 选择STM32F407VG    
    
![stm32f4-1-1]({{site.url}}/img/stm32f4/1-1.jpg)

2. Runtime Checking > C/C++ Compiler > Preprocessor > Additional include directories, 添加头文件位置， Define Symbols输入芯片的型号`STM32F407xx` 
同时因为本项目需要用到内存运行调试，所以加入了使用内存作为中断向量表的标记`VECT_TAB_SRAM`    

![stm32f4-1-1]({{site.url}}/img/stm32f4/1-2.jpg)

3. Runtime Checking > Linker > Config > Linker configuration file
因调试需要，本例子在内存中运行程序，并调试，所以需要配置中断向量表的地址，和RAM 地址， ROM地址。这里勾选`Override default`，并选择stm32f407xG.icf (默认目录是IAR的安装目录下的flashloader文件夹，因需要更改配置，所以这里就把icf文件拷贝到项目文件夹)    
![stm32f4-1-1]({{site.url}}/img/stm32f4/1-3.png)

4. 点击下方`Edit`按钮，并按下图更改更改配置：    
![stm32f4-1-1]({{site.url}}/img/stm32f4/1-4.png)

5. Runtime Checking > Debugger > Driver 选择ST-LINK    
![stm32f4-1-1]({{site.url}}/img/stm32f4/1-5.png)

6. Runtime Checking > Debugger > Download取消勾选所有选项
![stm32f4-1-1]({{site.url}}/img/stm32f4/1-6.png)

## IAR项目文件结构

![stm32f4-1-1]({{site.url}}/img/stm32f4/2-1.png)

### Drivers 

开发板用到的HAL硬件抽象层的文件，这里只用到了GPIO，所以只引入了STM32F4Cube里面关于GPIO的文件。

#### CMSIS

ARM公司为Cortex-M系列处理器定义的与硬件厂商无关的硬件抽象层。其中包括：

1. core_cm4.h Cortex-M4内核及其设备定义
2. system_stm32f4xx.c SystemInit系统初始化函数，SystemCoreClock系统时钟和时钟更新函数。

#### BSP

与开发板相关的硬件定义，例如LED灯的GPIO管脚

1. stm32f401-discovery.c

#### STM32F4xx_HAL_Driver

STM32F4系列的硬件抽象层驱动接口

1. stm32f4xx_hal_gpio.c
2. stm32f4xx_hal.c

### App

1. main.c 主应用程序
2. stm32f4xx_it.c 中断函数定义
3. startup_stm32f407xx.s 编译器启动代码，弱定义的中断程序。

## 项目中遇到的问题及解决办法

### 项目程序无法运行

问题描述：项目无法正确编译和链接

解决办法： 必须勾选下方`Automatic runtime library`和`library configuration`的library选项选择`Normal`  
![stm32f4-1-1]({{site.url}}/img/stm32f4/3-1.png)
![stm32f4-1-1]({{site.url}}/img/stm32f4/3-2.png)


### 无法触发`SysTick_Handler`中断

问题描述：程序开始执行后，程序进入了`Delay`函数的while循环，一直无法进入`SysTick_Handler`的中断函数

解决办法：因为在main程序中，未加入System_Configuration函数的执行。

## 项目运行结果

![stm32f4-1-1]({{site.url}}/img/stm32f4/4-1.jpg)

## 项目主要代码

* main.c

{% highlight c %}
#include "stm32f401-discovery.h"

int main(void)
{
  SysTick_Configuration();
  
  LED_Init(LED4);
  LED_Init(LED3);
  LED_Init(LED5);
  LED_Init(LED6);
  
  for(;;)
  {
    LED_Toggle(LED4);
    LED_Toggle(LED3);
    LED_Toggle(LED5);
    LED_Toggle(LED6);
    
    Delay(1000);   
  }
}
{% endhighlight %}



* stm32f4xx_it.c

{% highlight c %}
...

/**
  * @brief  This function handles SysTick Handler.
  * @param  None
  * @retval None
  */
void SysTick_Handler(void)
{
  TimeDelay_Decrease();
}

...
{% endhighlight %}




* stem32f401-discovery.c

{% highlight c %}
#include "stm32f401-discovery.h"

static __IO uint32_t TimeDelay;

GPIO_TypeDef* GPIO_PORT[LEDn] = {LED4_GPIO_PORT, LED3_GPIO_PORT, LED5_GPIO_PORT, LED6_GPIO_PORT};
const uint16_t GPIO_PIN[LEDn] = {LED4_PIN, LED3_PIN, LED5_PIN, LED6_PIN};

void LED_Init(LED_Typedef led)
{
  GPIO_InitTypeDef GPIO_InitStruct;
  
  /* Enable the GPIO_LED Clock */
  LEDx_GPIO_CLK_ENABLE(led);
  
  GPIO_InitStruct.Pin = GPIO_PIN[led];
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_PULLUP;
  GPIO_InitStruct.Speed = GPIO_SPEED_FAST;
  
  HAL_GPIO_Init(GPIO_PORT[led], &GPIO_InitStruct);
  HAL_GPIO_WritePin(GPIO_PORT[led], GPIO_PIN[led], GPIO_PIN_RESET);
}

void LED_On(LED_Typedef led)
{
  HAL_GPIO_WritePin(GPIO_PORT[led], GPIO_PIN[led], GPIO_PIN_SET);
}

void LED_Off(LED_Typedef led)
{
  HAL_GPIO_WritePin(GPIO_PORT[led], GPIO_PIN[led], GPIO_PIN_RESET);
}

void LED_Toggle(LED_Typedef led)
{
  HAL_GPIO_TogglePin(GPIO_PORT[led], GPIO_PIN[led]);
}

void SysTick_Configuration()
{
  if(SysTick_Config(SystemCoreClock / 1000))
  {
    /* Capture error, loop forever */
    while(1);
  }
}

void Delay(__IO uint32_t nTime)
{
  TimeDelay = nTime;
  while(TimeDelay != 0); // Loop when counter not equal zero
}

void TimeDelay_Decrease()
{
  if(TimeDelay != 0x00)
  {
    TimeDelay --;
  }
}
{% endhighlight %}

[STM32F4Cube]: http://www.st.com/st-web-ui/static/active/en/st_prod_software_internet/resource/technical/software/firmware/stm32cubef4.zip
[ST-LINK]: http://www.st.com/st-web-ui/static/active/en/st_prod_software_internet/resource/technical/software/driver/st-link_v2_usbdriver.zip