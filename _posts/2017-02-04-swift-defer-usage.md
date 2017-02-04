---
layout: blog
categories: blog
published: true
title: Swift defer的用法
---
defer的字面意思就是__延迟执行__，在swift中用于修饰一个block，并声明该block里面的代码在当前函数的所有代码执行完毕后，才被执行。官方对defer的说明如下：

> “Use defer to write a block of code that is executed after all other code in the function, just before the function returns.”

> Excerpt From: Apple Inc. “The Swift Programming Language (Swift 3).” iBooks.

defer通常用于对数据的清理工作，例如关闭文件。我们通过如下例子，说明defer的用法：

    var fileOpen = false

    func processFile() {
        fileOpen = true
        defer {
            // defer包含的代码在processFile函数的最后才会被调用
            fileOpen = false
        }
        
        // 此时 fileOpen 仍然为true
		assert(fileOpen == true)
        
        print('processing file')
    }

    processFile()

	// fileOpen 在processFile的最后才被调用
    assert(fileOpen == false)
