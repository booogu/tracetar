---
layout:     post
comments: true
title:      <code>WIP</code>Seata客户端启动过程剖析
subtitle:   注册中心 | 配置中心与启动流程的关系
date:       2021-02-28 21:08:00
author:     "Booogu"
header-style: text
catalog: true
tags:
    - Seata
---

> “刚上手Seata不久，对其各个模块了解还不够深入？ || 想探究下应用在集成Seata后的启动过程中“偷偷”干了些啥？|| 想探究下Seata源码但却还未付诸实践？今天这篇文章，就是为你量身打造的~

## 盖个帽段
  看过官网README的第一张的同学都应该清楚，Seata作为一个分布式事务中间件，核心原理便在于通过其**协调器侧**的（TC），来协调参与分布式事务的各个**应用侧**（RM），来保证多个事务参与者的数据一致性。那么协调器端与应用端是如何建立连接并进行通信的呢？

  没错，答案就是Netty，Netty作为一款高性能的RPC通信框架，保证了TC与RM之间的高效通信，关于Netty的详细介绍，大家可以移步至[!Netty介绍](http://sd)进行学习，而本文今天探究的重点，在于**应用侧在启动过程中，如何通过一系列Seata关键模块之间的协作（比如Config/Registry、Discovery、LoadBalance等），来建立与协调器侧之间的通信**

## 给个限定
Seata作为一款中间件级的底层组件，是很谨慎地引入第三方框架具体实现的，感兴趣的同学可以深入了解下Seata的SPI机制，（可参考[Seata SPI核心原理与设计](http://)）看看Seata是如何通过大量扩展点（Extension），来将具体组件的实现倒置出去，转而依赖抽象接口的，同时，Seata为了更好地融入微服务、云原生等流行架构所衍生出来的生态中，也对主流的微服务框架、注册中心、配置中心以及Java开发框架界“扛把子”——SpringBoot等做了主动集成，在保证微内核架构、松耦合、可扩展的同时，又可以很好地与大家“打成一片”，使得应用了各种技术栈的用户都可以很方便地引入Seata。

本文为了贴近大家刚引入Seata试用的场景，在以下介绍中，选择**应用侧**的限定条件如下：使用**File（文件）作为配置中心与注册中心**，并基于**SpringBoot**启动。

有了这个限定条件，接下来就让我们深入Seata，一探究竟吧。

## 从数据源代理说起
Seata目前支持的事务模式共有4种：AT、TCC、Saga以及XA事务模式，其中的TCC、Saga、XA等概念在业内由来已久，也有不少其他框架实现了这些设计，而AT模式则是Seata推出之初重点打造的“无侵入式”事务模式（个人认为，AT是Seata的核心优势特性之一）。

Seata AT模式的实现基于其对应用侧数据源的代理，Seata代理数据源的过程是通过一个类型为GlobalTransactionScanner（简称Scanner）的Spring Bean来完成的，此外，Scanner还承担着与TC Server进行首次握手连接的“重任”，在Scanner的后事件中（afterPropertiesSet）,应用侧需要进行TM、RM的初始化行为，来建立与TC Server的连接。

## <code>重点:</code>多模块交替协作的TM & RM 初始化
  
### 从<code>注册中心</code>获取TC Server集群地址
上面已提到，Seata支持多种注册中心实现（本文限定使用File文件作为注册中心），那么，Seata首先需要从一个地方先获取到“使用File作为注册中心”这个信息。
#### 从<code>Seata配置文件</code>获取注册中心的类型
从哪里获取呢？Seata用到了一个“配置文件”作为其框架内部配置信息的获取根入口，此配置文件也支持多种格式，Seata内置的默认文件为——registry.conf，从其中获取到注册中心的类型信息：【registry.type = file】

### 通过<code>负载均衡</code>从集群中选择特定Server

