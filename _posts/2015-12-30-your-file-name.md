---
layout: blog
categories: blog
published: true
title: KVC键值编码和KVO观察者模式
---
KVC键值编码

  

定义

  

KVC键值访问是指对象包含的信息的字符串作为键值使用，间接访问该信息的方式。

  

基本处理方式

  

键值编码的必须方法在非正式协议NSKeyValueCoding中定义：

  

**\- (id) valueForKey: (NSString *key)**

返回表示属性的键值字符串对应的值，如果无法取得该值，则调用valueForUndefinedKey:

  

**\- (void) setValue: (id) value forKey: (NSString *)key**

设置键值字符串key对应的属性的值为value。如果无法取得该值，则调用setValue:ForUndefinedKey

  

**\- (id) valueForKeyPath: (NSString *)keyPath**

返回属性路径对应的值。这个方法适用于属性是对象的情况。该方法以”.”切分keyPath，然后循环的向每一个键发送valueForyKey:消息。

  

**\- (void) setValue: (id) value forKeyPath: (NSString *)keyPath**

设置属性路径keyPath对应的值为value

  

**\- (BOOL) validateValue: (inout id *) ioValue forKey: (NSString *) key error: (out NSError *) outError**

使用键值寻找validate_ _ _: error的验证方法，并调用，如果不存在该方法则返回YES

  

**\- (BOOL) validateValue: (inout id *) ioValue forKeyPath: (NSString *) keyPath error: (out NSError *) outError**

使用键值路径中，最后一个键值对应的validate_ _ _: error的验证方法，并调用，如果不存在该方法则返回YES

  

  

键值编码的内部实现

  

假设我们向obj传递valueForKey:消息： [obj valueForKey: @“name”]，那么objective-c内部会按如下方式查找：

1\. 消息接收者obj如果有name访问器（getName, _name, isName, _getName）则调用它。

2\. 没有name访问器时，使用消息接收者obj的类方法：accessInstanceVariableDirectyly来查找instance变量。返回Y
ES时，如果存在实例变量name（或_name, isName, _isName）则返回其值

3\. 既没有访问器，也没有实例变量的情况下，则调用valueForDefinedKey:

4\. 返回的值如果不是对象（例如int, char, long等），则返回该值对应的适当对象的包装值（例如，NSNumber）

  

一对多和一对一关系

  

如果需要访问的属性是集合对象，例如NSArray, NSDictionary，

1\. 那么调用valueForyKey和valueForKeyPath就会返回数组或集合。

2\. 如果调用setValueForKey:forKey和setValueForyKeyPath:forKeyPath的时候，该属性对应的集合里的对象都会
被设置为value

  

用户自定义的集合对象：

  

用户如果定义了自己的集合对象，那么需要实现如下两个方法，才能使用键值操作

  

**\- (NSUInteger) countOf_ _ _;**

**\- (id) objectIn_ _ _AtIndex: (NSUInteger) index;**

  

例如我们定义了一个Team类，需要键值访问member属性，则需要定义：

  

**\- (NSUInteger) countOfMember;**

**\- (id) objectInMemberAtIndex: (NSUInteger) index;**

  

这样，我们就可以以键值访问方式访问member属性：

[ aTeam valueForKey: @“member”]

  

  

KVO键值观察者模式

  

  

KVO机制是，对某个对象的属性的观察，当其值改变时，通知其他对象的机制。KVO可以建立一对一，一对多的观察模式。

  

KVO的准则

  

1\. 随访问器方法而改变

2\. setValue:forKey:和键进行改变。此时可能不经过访问器

3\. setValue:forKeyPath:和键路径进行改变。此时可能不经过访问器。不仅仅是最终的监视对象属性，路径属中的属性变化时，也会被通知。

  

实现方法

  

1\. 注册观察者

**\- (void) addObserver: (NSObject *)anObject**

**               forKeyPath: (NSString *) keyPath**

**               options: (NSKeyValueObservingOptions) options**

**               context: (void *) context**

注册观察者，当keyPath中任何一个属性发生变化，都会通知onObject，属性变化的通知中，包含变化的内容，参数context制定任意的对象指针或对象。
options指定字典数据中包含什么值。

options可取值为：

NSKeyValueObservingOptionNew  改变后的值

NSKeyValueObservingOptionOld    改变前的值

  

2\. 观察者实现消息接受方法

**\- (void) observeValueForKeyPath: (NSString *) keyPath**

**               ofObject: (id) object**

**               change: (NSDictionary *) change**

**               context: (void *) context**

字典change保存如下信息：

NSKeyValueChangeKindKey           属性改变的种类

KSKeyValueChangeNewKey           属性更新后的值

NSKeyValueChangeOldKey

NSKeyValueChangeIndexesKey      如果属性是动态数组，则为这些集合的索引集合

  

3\. 移除观察者

**\- (void) removeObserver: (NSObject *) anObserver**

**               forKeyPath: (NSString *) keyPath**

  

  

  

