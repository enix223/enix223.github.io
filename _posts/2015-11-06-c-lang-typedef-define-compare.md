---
layout: post
categories: C
published: true
title: typedef 与 define的区别
---
#define是预编译关键字，而typedef是编译器关键字。

  

typedef 就像是变量，而define定义的宏一直有效，直到文件末尾或者undef。

typedef只能定义类型，而不能定义常量。

  

  

举例，define 不能替代 typedef的例子：

  

`typedef int* int_p1;  
int_p1 a, b, c;  // a, b, and c are all int pointers.  
  
#define int_p2 int*  
int_p2 a, b, c;  // only the first is a pointer!`

  

`typedef int a10[10];  
a10 a, b, c; // create three 10-int arrays`

  

`typedef int (*func_p) (int);  
func_p fp // func_p is a pointer to a function that  
          // takes an int and returns an int`

  

typedef unsigned char MAC_ADDR[6];

MAC_ADDR  addr = {0x1A, 0x2B, 0x3C, 0x4C, 0x5D, 0x6E}; // MAC_ADDR is a alias
name of array unsigned char[6]

  

转自：<http://stackoverflow.com/questions/1666353/are-typedef-and-define-the-
same-in-c>

  

  

