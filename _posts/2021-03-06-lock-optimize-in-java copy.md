---
layout:     post
comments: true
title:      Java中的锁优化
subtitle:   Java中锁的优化方式
date:       2021-03-06 21:44:00
author:     "Booogu"
header-style: text
catalog: true
tags:
    - Java
    - 读书笔记
    - CS基础
---

## Java中的各种锁优化技术：
- 自旋锁：阻塞时，挂起和恢复线程都需要转入内核态完成，代价很高，所以可以暂时不放弃处理器执行时间，自己等自己一会，看看持有锁的线程是不是很快就释放了，这就是自选锁优化
- 适应性自旋锁：自旋的时间不再固定，而是由前一次在同一个锁上的自旋时间以及锁的拥有者的状态来决定的，虚拟机堆程序锁的状况预测会越来越精准、JVM越来越聪明（需要随着程序运行时间的增长以及性能监控信息的不断完善）
- 锁消除：基于逃逸分析技术，需要分析出这段代码不可能存在共享数据竞争，就当作栈上数据处理，比如StringBuffer sb.append(1).append(2).append(3)这段代码在一个方法里，sb从未被被外部方法引用访问（没有逃逸），所以stringbuffer内部的同步锁都可以去掉
- 锁粗化：多次执行堆同一对象的加锁（比如在循环里，或者一段比较密集的代码里），因为涉及到了多次互斥同步，用户态核心态切换，所以可以合并这些操作，将锁变粗，同③中的例子，如果需要加锁，直接在方法外加一个即可，不需要每次append都加、释放锁。

- 轻量级锁：本质是乐观锁（CAS），可能涉及锁膨胀，即变成重量级锁
- 偏向锁：本质时无锁，也会发生锁膨胀。

以上`轻量级锁`以及`偏向锁`待详细学习