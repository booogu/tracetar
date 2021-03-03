---
layout:     post
comments: true
title:      <code>WIP</code>Seata客户端启动过程剖析（二）
subtitle:   注册中心 | 配置中心
date:       2021-03-04 1:35:01
author:     "Booogu"
header-style: text
catalog: true
tags:
    - Seata
---


> “刚上手Seata不久，对其各个模块了解还不够深入？ <br>
想深入研究下Seata源码但却还未付诸实践？<br>
想探究下在集成Seata后，自己的应用在启动过程中“偷偷”干了些啥？<br>
如果你有上述问题，那么今天这篇文章，就是为你量身打造的~


### 从<code>注册中心</code>获取TC Server集群地址
上面已提到，Seata支持多种注册中心实现（本文限定使用File文件作为注册中心），那么，Seata首先需要从一个地方先获取到“使用File作为注册中心”这个信息。
#### 从<code>Seata配置文件</code>获取注册中心的类型
从哪里获取呢？Seata用到了一个“配置文件”作为其框架内部配置信息的获取根入口，此配置文件也支持多种格式，Seata内置的默认文件为——registry.conf，从其中获取到注册中心的类型信息：[registry.type = file]
