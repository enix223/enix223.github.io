**中断响应过程**

  1. 外设向处理器发起一个中断请求
  2. 处理器挂起当前的任务
  3. 处理器执行中断服务程序（ISR），并通过软件按需要清楚中断中断请求标志
  4. 处理器返回到上一个被挂起的任务继续执行。

![](NVIC%E4%B8%AD%E6%96%AD%E5%90%91%E9%87%8F%E6%8E%A7%E5%88%B6%E5%99%A8.resour
ces/CE61C7B1-8E33-4F0F-8D61-F7CC39AF2603.png)

  

NVIC中断处理器，专门用于处理中断请求。

cortex m3和cortex m4 NVIC支持最多240个IRQ, 一个NMI（不可屏蔽中断），SysTick（系统时钟），和一些其他系统中断。NMI
一半通过外设发起，例如看门狗，BOD。其他中断来自处理器，中断可以由软件发起。

  

cortex m处理器可以自动保存和恢复被中断的任务，所以无需编程过程中无需特别处理。

  

1-15为系统异常中断，16以上为外部中断。每一个中断源都有个中断编号（IRQ，1-240）。

  

**中断管理**

  

大部分NVIC寄存器都在SCB当中。NVIC和SCB在SCS地址0xE000E000，大小为4KB。

CMSIS NVIC相关函数（在头文件core_cm4.h/core_cm3.h）

![](NVIC%E4%B8%AD%E6%96%AD%E5%90%91%E9%87%8F%E6%8E%A7%E5%88%B6%E5%99%A8.resour
ces/17F53782-8591-4B66-B04B-24FB69B1AE4B.png)

  

每个中断有如下几种状态：

  * 启用enable或者禁用disabled
  * 等待pending处理器和非等待，pending可以在禁用状态下被置为1，且当中断启用后，立刻被处理
  * 正在执行active和非正在执行inactive

  

一个中断被响应，需要满足以下条件：

  * pending被置为1
  * 中断已启用
  * 中断的优先级比当前的级别要高

  

NVIC_Type数据结构：

  

**typedef** **struct**  
{  
  __IO uint32_t ISER[8];                 /*!&lt; Offset: 0x000 (R/W)
Interrupt Set Enable Register           */  
       uint32_t RESERVED0[24];  
  __IO uint32_t ICER[8];                 /*!&lt; Offset: 0x080 (R/W)
Interrupt Clear Enable Register         */  
       uint32_t RSERVED1[24];  
  __IO uint32_t ISPR[8];                 /*!&lt; Offset: 0x100 (R/W)
Interrupt Set Pending Register          */  
       uint32_t RESERVED2[24];  
  __IO uint32_t ICPR[8];                 /*!&lt; Offset: 0x180 (R/W)
Interrupt Clear Pending Register        */  
       uint32_t RESERVED3[24];  
  __IO uint32_t IABR[8];                 /*!&lt; Offset: 0x200 (R/W)
Interrupt Active bit Register           */  
       uint32_t RESERVED4[56];  
  __IO uint8_t  IP[240];                 /*!&lt; Offset: 0x300 (R/W)
Interrupt Priority Register (8Bit wide) */  
       uint32_t RESERVED5[644];  
  __O  uint32_t STIR;                    /*!&lt; Offset: 0xE00 ( /W)  Software
Trigger Interrupt Register     */

}  NVIC_Type;

  

重置之后，所有中断都处于禁用状态，并且优先级为0。所以在使用中断前，必须执行以下步骤：

  1. 设置中断的优先级（可选）
  2. 启用外设的中断
  3. 启用NVIC里面的中断

  

ISR程序可以在启动代码中找到，并且ISR程序的名字必须和中断表（vector table）一致。

  

**两个优先级**

  * 抢占优先级 priority group （旧称pre-emption ，高优先级可以抢占低优先级，即嵌套中断）
  * 响应优先级 subpriority （在相同抢占优先级的前提下，高响应优先级的中断必须等待低响应优先级的中断完成后才能执行，也就是不能嵌套中断）
  * 优先级取值 0 －15，数字越小，优先级越高

  

**优先级冲突处理：**

当两个中断同时发生，按如下逻辑判断哪个先执行：

  1. 比较两个中断的抢占优先级，高优先级的中断先被响应
  2. 相同抢占优先级的前提下，比较响应优先级，高响应优先级的中断先被响应
  3. 当抢占优先级和响应优先级都相同的情况下，则根据中断表的排位顺序决定哪一个先执行。（IRQ号小的优先级高）

  

中断优先级通过优先级寄存器控制，通常长度为3-8 bits

  

例如，如下例子，LSB 0-4bits均没实现，那么优先级可取值为0x20, 0x40, 0x80, 0xA0, 0xC0, 0xE0

![](NVIC%E4%B8%AD%E6%96%AD%E5%90%91%E9%87%8F%E6%8E%A7%E5%88%B6%E5%99%A8.resour
ces/8801E78B-4671-450B-BCCC-3CAAD3C0FB7F.png)

  

之所以忽略LSB而不是MSB，是为了兼容性问题。例如实现了4位的优先级寄存器程序，可运行于只实现了3位的优先级寄存器平台。

  

**设置priority group**

  

例子1， priority group设置为5，那么就有4个抢占优先级，每个抢占优先级有2个响应优先级

![](NVIC%E4%B8%AD%E6%96%AD%E5%90%91%E9%87%8F%E6%8E%A7%E5%88%B6%E5%99%A8.resour
ces/A8C911DA-1A61-415C-976B-EDA4660EF597.png)

例子2，如果priority group设置为1，那么就只有8个抢占优先级，而没有响应优先级。

![](NVIC%E4%B8%AD%E6%96%AD%E5%90%91%E9%87%8F%E6%8E%A7%E5%88%B6%E5%99%A8.resour
ces/DA68F37E-1E19-44D5-B531-E2E82D4CF915.png)

例如3， 如果实现了所有的寄存器位，那么该处理器最多只有2^7=128个抢占优先级，每个抢占优先级下有2个响应优先级

![](NVIC%E4%B8%AD%E6%96%AD%E5%90%91%E9%87%8F%E6%8E%A7%E5%88%B6%E5%99%A8.resour
ces/B7D3D468-F22C-44BD-B93E-787A9C50E779.png)

**中断向量表**

  

中断向量表一般置于地址0，中断向量表包括了异常处理程序的起始地址，还包括了MSP的初始值。但中断向量表可以在启动代码中重新放置于内存或者flash，可是不能
在运行过程中移动。

  

中断向量表relocation是通过VTOR寄存器实现。

  

