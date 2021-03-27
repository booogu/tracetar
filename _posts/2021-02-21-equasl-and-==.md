---
layout:     post
comments: true
title:      ==/equals/hasCode总结
subtitle:   ==、equals、hasCode
date:       2021-02-21 20:04:00
author:     "Booogu"
header-style: text
catalog: true
tags:
    - Java
    - 读书笔记
    - 基础
---

## ==与equals异同
==比较的都是值（Java只有值传递），对于基本数据类型比较的是数值，对于引用类型，比较的是引用对象的地址值
Object的equals方法，就是调用==，所以equals方法需要重写，比如String类做了重写，仅比较字符串的内容是否相同
uilder.append()，然后调用toString()，实现上会new String("HellowWorld")，但这个字符串不会在常量池中创建！

## 为什么重写equals必须同时重写hashCode？
为了保证：两个对象相当（.equals），则它们的hashCode必须相同（不重写则达不到这个效果）

### 底层原理
hashCode()的默认行为是对堆上的对象产生独特值，如果没有重写，则该class的两个对象无论如何都不会相等（即使指向相同的数据）

### 注意点
equals为true是两个对象hashCode相同的充分不必要条件

### hashCode设计初衷
是为了更好地支持Java中基于哈希机制的集合类，HashTable，HashSet，HashMap等，目的是在比较对象是否相等时，先检查HashCode，如果不相等直接就不再比对equals了，提高了不少的执行效率，如果相等再执行equals方法判断

