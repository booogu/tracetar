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
````java
    @Override
    public void afterPropertiesSet() {
        if (disableGlobalTransaction) {
            if (LOGGER.isInfoEnabled()) {
                LOGGER.info("Global transaction is disabled.");
            }
            return;
        }
        //初始化TM、RM
        initClient();

    }
````

## <code>重点:</code>多模块交替协作的TM & RM 初始化
RMClient.init()为例说明，
查看源码，大家会发现，这里的init，涉及到了很深层次的类的构造器以及初始化，层层嵌套，看起来很复杂，但大家只需要明白，这些类是为了将客户端的层层职责划分的更清晰，更加专注于各自逻辑，更好实现扩展性（这块各自类的职责以及设计原则，请大家参考Seata RPC重构操刀者乘辉兄的文章[Seata-RPC重构](https://sda)）
这里我只将最为重点的，与TC Server建立连接最紧密的两个类梳理出来展现给大家。
````java
    public static void init(String applicationId, String transactionServiceGroup) {
        //首先从RmRpcClient类开始，依次调用父类的构造器，链路为：RmRpcClient->AbstractRpcRemotingClient->AbstractRpcRemoting
        RmRpcClient rmRpcClient = RmRpcClient.getInstance(applicationId, transactionServiceGroup);
        rmRpcClient.setResourceManager(DefaultResourceManager.get());
        rmRpcClient.setClientMessageListener(new RmMessageListener(DefaultRMHandler.get(), rmRpcClient));
        //然后从RmRpcClient类开始，依次调用父类的init()，链路为：RmRpcClient->AbstractRpcRemotingClient->AbstractRemoting
        rmRpcClient.init();
    }
````

* 核心类1：AbstractRpcRemotingClient
  - 初始化并持有了RpcClientBootstrap:具有启动、停止能力
  - 持有获取具体Client的NettyPoolKey的回调
初始化NettyPoolableFactory，制造Channel的工厂类，在初始化创建Channel的对象池时需要，是与Channel对象池交互的核心对象。
  - 使用factory及key，初始化了NettyClientChannelManager：管理客户端Channel（获取、释放、销毁、缓存），并负责通过Channel与TC Server建立连接
* 核心类2：NettyClientChannelManager，初始化了这样几个类：
  - 关键的GenericKeyedObjectPool，泛型对象池，简单理解为入参为K，出参为V，用于根据key，调用factory创建具体的对象
  - 持有获取具体Client对应的NettyPoolKey的回调

  在了解了Client初始化过程，以及涉及到连接Server的两个核心类有了大致了解后，大家可能还比较疑惑，不是说初始化时执行Client与Server的连接吗，怎么还没有看到哪里触发呢？

  别急，上面是在深入细节之前，先让大家对初始化过程有个概观，并对核心类的作用有个大体认识，接下来，我们开始深入探究代码细节，手撕连接的过程。
  并不难找，顺着init的方法看下去，很容易发现在好几个类的初始化方法中都定义了定时任务，Seata应用侧与协调器侧的重连机制，也是通过定时任务实现的。
  ````java
    /**
     * Class io.seata.core.rpc.netty.AbstractRpcRemotingClient
     */
    public void init() {
        clientBootstrap.setChannelHandlers(new ClientHandler());
        clientBootstrap.start();
        //设置定时器，定时重连TC Server
        timerExecutor.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                clientChannelManager.reconnect(getTransactionServiceGroup());
            }
        }, SCHEDULE_INTERVAL_MILLS, SCHEDULE_INTERVAL_MILLS, TimeUnit.SECONDS);
        //以下代码略
    }
````
  这里我们以第一次执行reconnect方法为抓手，看看客户端这几个类之间是如何协作，完成于TC的连接的（实际上首次连接应该发生在registerResource的过程中，但核心流程都是如下所示，并无差别）

  [图]
  这个图中，大家可以重点关注这几个点：
  （1）AbstractRpcRemotingClient具备执行具体Client获取NettyPoolKey回调的能力，此能力由RMRpcClient/TMRpcClient具体实现，此Key中具有两个主要信息：要链接的Server 地址，以及重连时要发送的RPC消息体
  （2）构造对象池GenericKeyedObjectPool时传入的NettyPoolableFactory，由在通过对象池获取Channel对象时发挥作用：根据NettyPoolKey来从工厂中构造一个对象（用到了抽象工厂模式）
  （3）RpcClientBootstrap是对Netty原生Bootstrap对象的封装，负责于Netty直接交互，获取新的Channel（同时还有启动。停止的能力）
  （4）在NettyPoolableFactory的makeObject方法中，不仅获取到了新的Channel，还直接使用此Channel发送了重连消息（分別用到了NettyPoolKey的address和message）
  以上流程，就是Seata客户端初始化时如何与TC Server连接的全过程，具体的细节，大家可以来顺着本文的梳理的脉络和重点详细阅读下源码，相信大家会有更深入的理解和全新的收获！

  
### 从<code>注册中心</code>获取TC Server集群地址
上面已提到，Seata支持多种注册中心实现（本文限定使用File文件作为注册中心），那么，Seata首先需要从一个地方先获取到“使用File作为注册中心”这个信息。
#### 从<code>Seata配置文件</code>获取注册中心的类型
从哪里获取呢？Seata用到了一个“配置文件”作为其框架内部配置信息的获取根入口，此配置文件也支持多种格式，Seata内置的默认文件为——registry.conf，从其中获取到注册中心的类型信息：【registry.type = file】

### 通过<code>负载均衡</code>从集群中选择特定Server

