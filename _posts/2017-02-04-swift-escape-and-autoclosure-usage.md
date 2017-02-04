---
layout: blog
categories: blog
published: false
title: 'Swift 闭包 @escape, @nonescape和@autoclosure的作用和用法'
tags: ''
---
在Swift中，关于闭包closure有3个声明属性修饰符(Declaration Attributes)，它们分别是：@escape, @nonescape 和 @autoclosure。

刚开始接触Swift的时候，会比较迷惑，它们的各自的作用到底是什么？为什么需要使用这个声明属性？

带着这些问题，我阅读了Apple的官方教程《The Swift Programming Language》。以下是我对于这3个声明属性的浅薄的个人理解：

这3个修饰符都是用于修饰函数定义中的闭包参数。@escape和@nonescape是用于声明闭包可以在何时调用；@autoclosure则是用于把一个表达式作为参数，自动打包为闭包，并传递给接受闭包参数的函数。

### @escape 和 @nonescape

当闭包closure作为函数的参数时，Swift允许我们对闭包参数添加声明注解，其中@escape和@nonescape是对闭包的调用范围作出声明。

官方的对@escape的解释是这样的：

```
“A closure is said to escape a function when the closure is passed as an argument to the function, but is called after the function returns.”
 
Excerpt From: Apple Inc. “The Swift Programming Language (Swift 3).” iBooks.
```

它的意思是：__用@escape修饰的闭包参数，允许闭包在函数的作用域之外调用__。通常使用@escape场合是，我们需要把闭包closure保存为类的属性，并在稍后作为回调函数的方式调用。也就是说，闭包并不在当前函数中调用，而是在当前函数的作用域之外调用。

看看如下例子：


    class AsyncReader {
      // 定义Callback
      typealias Callback (Bool Success) -> ()

      // 定义一个回调函数属性
      var callback: Callback

      func setReadCallback(@escape callback Callback) {
          // 我们并不需要callback在当前函数的作用域中调用，而是在稍后才调用callback函数
          // 所以，我们需要对callback参数作@escape的声明
          self.callback = callback
      }

      func read() {
		print("Reading...")
        print("Read finished")
        
        // callback在此时才被调用
        callback(true)
      }
    }

    let reader = AsyncReader()
    reader.setReadCallback {
        success in 
        let result = success ? "Success" : "Failed"
        print("Read \(result)")
    }
    reader.read()


@nonescape的作用和@escape相反，也就是闭包必须在当前函数的作用域中被调用。请看例子:

    class SyncReader {

      typealias Callback (Bool success) -> ()

      var callback: Callback

      // 以下函数会在编译过程报错 
      func setReadCallback(@escape callback Callback) {
          // 如果callback没有被立即调用，那么编译器会报错，
          // 因为callback没有在setReadCallback作用域中被调用.
          self.callback = callback
      }

      func readSync(callback Callback) {
         print("Reading...")
         print("Read finished")

         // callback闭包必须在readSync的调用声明过程中被调用。
         callback(true)

         // self.callback = callback
      }
    }

### @autoclosure


