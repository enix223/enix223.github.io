---
layout: blog
tags: 
  - "null"
published: true
title: "Definition Guide to Cortex M4 Note - 1"
date: "2015-10-01 15:33:08 +0800"
categories: blog
---

## 关于Data structure alignment (字节对齐)

### 1. 背景

在C语言编程中，我们往往会遇到字节对齐的问题。因为对于现代的计算机，对内存的读写都是以字（word）的大小为基本操作。例如，对于32位CPU，一个字（word）= 4 个字节，也就是32位。

字节对齐就是在数据中插入无意义的占位字节，使数据结构struct的大小正好是CPU字的整数倍。这样可以提高计算机读取写入内存的效率。

举个例子，对于`int`类型，一般占用4个字节。如果需要读取一个`int`类型的变量，但是这个变量是在偶数地址（例如 `0x04`），那么CPU可以在一个周期就可以读出该int变量；若该int类型存储与奇数地址（例如 `0x03`），那么CPU必须读取两次内存，并需要对读取的两次值做拼凑，才可以得到该int的值。这样，读取效率就会变得很低。而通过字节对齐，CPU可以减少读取和写入的次数。对于struct，这个尤为明显。所以字节对齐会经常和struct一起出现。

  而C语言的基本数据类型占用的内存不一定为一个字，对于传统的编译器，包括Microsoft (Visual C++), Borland/CodeGear (C++Builder), Digital Mars (DMC) and GNU (GCC)，在x86的平台编译，数据类型占用的内存大小分别为：

- `char` 1个字节，
- `short` 2个字节.
- `int` 4个字节 will be 4-byte aligned.
- `long` 4个字节 will be 4-byte aligned.
- `float` 4个字节 will be 4-byte aligned.
- `double` 8个字节 will be 8-byte aligned on Windows and 4-byte aligned on Linux (8-byte with -malign-double compile time option).
- `long long` 8个字节 will be 8-byte aligned.
- `long double` (C++Builder 和 DMC为10个字节, 对于Visual C++为8个字节, 对于GCC为12个字节) will be 8-byte aligned with C++Builder, 2-byte aligned with DMC, 8-byte aligned with Visual C++ and 4-byte aligned with GCC.
- `任何指针类型` 4个字节 will be 4-byte aligned. (e.g.: char*, int*)              

了解了这个问题后背景知识后，接下来我们看看计算机是如何解决这个问题的。
    
### 2. 字节填充

虽然编译器会自动填充字节以解决以上问题，但是对于struct结构，我们往往有一些自己的需求，而不让编译器帮我们自动完成。同时，编译器会在struct成员之间插入无意义的字节，以达到字节对齐的目的，另外，对于struct数组，编译器也会添加额外的字节，以实现数组成员间的对齐。所以通过成员的重新排序，会改变填充字节的多少。例如，如果一个结构是以占用内存大小的降序排列成员，那么这种排列将会是，所有排列中，所需的填充字节最小的一种。

字节填充只会出现在一种情形：就是struct的前一个成员后跟一个比它所占用更多内存的成员，或者在struct的最后。

尽管C和C++编译器不允许在编译阶段重新排列结构的成员，以达到降低结构占用空间的目的，但我们仍然可以告诉编译器，使用什么样的对齐方式来封装`pack`结构的成员。例如，`pack(2)`表示结构成员将以2字节对齐。说的有点抽象，看看以下例子：

假如我们定义了一个这样的结构：  

	struct PackData
	{
	  char MA;
	  int MB;
	  char MC;
	}

在没有任何`pack`指令的前提下，编译器会自动按最大的成员做填充，`sizeof(PackData)`就会等于4×3=12，内存填充结果就是：      

	|  1   |   2   |   3   |   4   |
	| MA(1) | 填充字节..............|
	| MB(1) | MB(2) | MB(3) | MB(4) |
	| MC(1) | 填充字节..............|

如果我们修改结构的定义，加上一个`pack(2)`指令：

IAR编译器写法

	#pragma pack(2)
	struct PackData
	{
	  char MA;
	  int MB;
	  char MC;
	}

MDK和GCC编译器写法：
	
	struct PackData
	{
	  char MA;
	  int MB __attribute__ ((aligned(2)));
	  char MC;
	}

那么，`sizeof(PackData)`=4×2=8，编译后的填充结果为：

	|  1   |   2   |
	| MA(1) | 填充..|
	| MB(1) | MB(2) | 
	| MB(3) | MB(4) |
	| MC(1) | 填充..|

如果我们使用的是`pack(1)`，

IAR编译器写法

	#pragma pack(1)
	struct PackData
	{
	  char MA;
	  int MB;
	  char MC;
	}


MDK和GCC编译器写法：
	
	struct PackData
	{
	  char MA;
	  int MB __attribute__ ((packed));
	  char MC;
	}
    
那么`sizeof(PackData)`=1×6=6，内存不再有字节填充：

	|  1   |
	| MA(1) |
	| MB(1) |
	| MB(2) | 
	| MB(3) |
	| MB(4) |
	| MC(1) |

### 3. 编译器相关指令

对于字节对齐，各个编译器都有自己适用的指令，现在主要介绍我使用过的两种：

对于IAR的toolchain，`#pragma pack`就是用于指定如何对齐字节。

而对于MDK和GCC，__attribute__((packed)) 和 __attribute__((aligned))来实现相同的功能。

- __attribute__((packed)) 对于变量，则为1个字节；对于field，则为1 bit。
- __attribute__((aligned)) 以字节为单位


### 4. 参考
1. <https://en.wikipedia.org/wiki/Data_structure_alignment>
2. <http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0491e/CJAFJHJD.html>
3. <http://stackoverflow.com/questions/14332633/attribute-packed-v-s-gcc-attribute-alignedx>
