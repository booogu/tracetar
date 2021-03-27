---
layout:     post
comments: true
title:      Java基本类型与包装类，装箱拆箱
subtitle:   基本类型、包装类、装箱拆箱
date:       2021-02-21 22:04:00
author:     "Booogu"
header-style: text
catalog: true
tags:
    - Java
    - 读书笔记
    - 基础
---

## Java基本类型的特点
Java中8中基本类型及其长度与默认值，注意Java中的基本类型长度与硬件架构无关，这种所占空间大小不变性是Java程序比其他大多数语言更具有可移植性的原因之一
![](http://booogu.top/img/in-post/java-primiteive-type.jpg)


## 装箱拆箱实现原理
反编译字节码得知：
- 自动装箱：调用包装类的valueOf()方法
- 自动拆箱：调用包装类的xxxValue()方法

## 基本类型的valueOf()
- Integer：-127-128之间的数值会被缓存到IntegerCache中，不会重复创建新对象，同Integer实现类似的有：Short、Byte、Long、Character
- Double：没有做缓存，因为不同于Integer在某个范围内是有限的，一个范围区间的浮点数是无法枚举的
同Double实现类似的有：Float
- Boolean：是直接存了两个静态成员属性，所有的Boolean，只有两个对象。true和false

## `Integer i = new Integer(2)` 和 `Integer i = 2`的区别
- 第二种会触发自动装箱，第一种则会直接创建对象
- 第二种某些情况下会复用缓存，而第一种则一定会创建对象。
- 在执行效率和资源占用上不同。一般来说第二种的执行效率和资源占用要更好


## 包装器、基本类型的比较"=="注意事项
- 包装器类型的==，会比较引用对象的地址（==介绍中解释过，值比较）
- 如果==两端有一个操作数是表达式，则先出发自动拆箱，比较数值（a+b，会先出发自动拆箱，调用intValue来计算），如果==没有遇到运算符，则不会触发自动拆箱（比较的是地址）
- equals括号中是数值，则会按照左侧包装器类型进行自动装箱，调用valueOf，来进行比较
- 对于包装器类型，equals方法不会进行类型转换（不会把Long转为Integer）
- equals括号中的数值，是什么类型，则触发什么类型的自动装箱

### 核心点
对于基本类型：==比基本类型的值（不是基本类型则自动拆箱），equals比较包装器对象的地址（不是包装器类型则自动装箱）

### 一图解惑

![](http://booogu.top/img/in-post/java-primiteive-type.jpg)

**结果及解释：**
- True 	127以下，比较的是缓存，相同
- False	127以上，比较的是两个对象，不同
- True	a+b表达式先自动拆箱计算数值，然后c自动拆箱与右侧比较数值，相同
- True	a+b表达式先自动拆箱计算数值，然后按Integer.intValue，包装成Integer类型，与左侧比较，相同
- True	a+表达式先自动拆箱计算数值，然后g自动拆箱得到数值，与右侧比较，疑问：int数值和long数值比对怎么比？
- False	a+b表达式先自动拆箱计算数值，然后按Integer装箱，与左侧Long比对，不相同
- True	a+h表达式先自动拆箱计算数值，int与long相加得到的是long，然后按Long装箱，与左侧Long比对，相同


> 后记：参考资料（常查常看）
https://www.cnblogs.com/dolphin0520/p/3780005.html

