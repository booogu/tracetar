---
layout:     post
comments: true
title:      内源InnerSource介绍
subtitle:   开源社区的发展策略讲座分享
date:       2021-03-21 22:26:00
author:     "谭中意"
header-style: text
catalog: true
tags:
    - 开源
    - 大咖讲座整理
---

> 整理自开放原子开源基金会TOC副主席谭中意老师2020年11月来浪潮做开源治理分享讲座


## 什么是内部开源
内源其实很简单，是说在公司内采用开源社区项目做法来开发项目，简单说就是，我内部代码仓库是开放的，是允许接收其他部门贡献的，就是这么简单。


## 内源能给企业带来的好处
然后我说下内源可以给公司带来什么好处，第一是，它可以更好为对外开源做准备，第二是说有助于打破部门墙，减少重复造轮子，第三是有助于提升代码质量，第四是有助于提升人员能力，第五是有助于提高员工满意度。之后我会每一条每一页都讲一讲。

### 更好地为对开源做准备
首先我们说一下为什么内源可以更好为对开开源做准备，就在于对外开源是一个广泛的在开源社区里协作，接收贡献；对内是在部门内（协作、接收贡献）。  
这两者要做得好，都应该遵守相同的价值观，也是俊平刚才提的很多次的，Community Over Code，社区的协作比代码的质量更重要，所以内源要做的好，需要有一个很好的内部协同社区，外源做得好也是一样的（要有好的协同社区），而要做得好都需要有相同价值观：开放、透明、协同，这在内源和外源是相同的，同时也有一些类似的协同原则，比如版本管理下的多人协同，比如异步沟通，存档，可追溯，刚才看有问题说怎么处理邮件和IM的沟通，开源社区都推行：重要事情都通过邮件列表/Issue的方式去沟通，好处就在于这种方式是异步沟通，所有信息可追溯，这个追溯是可以追溯很多很多年的，就像我当初给Mozilla贡献的代码是20年前贡献的，现在我还能追溯到，这个历史就很清楚

### 有助于打破部门墙，减少重复造轮子
第二个就是说的一个很多大公司特别看重的，就是有助于打破部门墙。

大家知道一个公司如果要高效，那么它的边界会切分得非常清晰，但切割非常清晰后，往往会产生很多协作问题，比如你提的一个需求，是需要让另一个部门提供的，那么你需求的排期，就完全取决与他的排期，如果得不到他的支持，那么你死定了，或者你可以去找他的老板去沟通，但这个沟通效率就会很低，而越是在大公司，部门墙就越厚，所以我们把代码开放出来后，是有助于打破部门墙的，这是第一点。

第二点就是重复造轮子，这是每个大企业都会存在的问题，每一个老板都希望下面的人越来越多，希望自己的团队覆盖的底盘越多越好，这样就会导致每个公司内，有大量重复的工作分散在各个团队中，有时老板看不到也无从管起，看得到的，也解决不了，而在内部开源情况下，第一是老板能看到，第二，老板可以挑选或扶持一家做得最好的部门，其他的需求由这个部门来采取内部开源的方式去支撑，这样可以减少重复造轮子的问题。


### 有助于提升代码质量
首先大家知道Linux’s Law，为什么Linux内核的质量好？原因很简单，越多人看到代码，Bug就会越少，一般来说，著名的开源项目（开源协同很好的项目），它的质量比在公司内部很多私有系统的软件要好很多，有问题后，修复的速度也快得多，原因就在于更多人能看到它，而且在内源过程中我们也能看到一个现象，很多工程师为了修复问题，为了工期的紧张，都是先解决再说，先上线，先搞完排期再说，这样就导致，代码会写的像狗屎一样，不仅小公司，大公司也一样，越早的代码越ugly，但是如果能把代码开出去，工程师觉得自己代码能被全公司看到后，他会注重得多，外源代码也是，我印象中有很多大公司项目在对外开源的时候，会至少有两到三个月时间在代码重构，把代码写得好看点，我印象中的华为，Facebook，很多著名项目，甚至在开出去的前一夜，还在改代码注释，他们会想，至少代码开出去别丢人，这种思考和顾虑，对工程师内部研发群体做内源，也是一样的。

第二在于代码评审，做过研发管理的都知道，代码评审是提升代码质量最佳方式之一，但很遗憾，在很多公司，绝大多数代码评审做都是走过场，有的根本不做，有的是自己给自己做，有的是给别人看了，但1分钟不到，就催人家赶紧看完。大家可以看看，代码数据上的代码评审记录是什么样子的，我觉得能有超过10%的代码评审记录，就已经很高了，有10%，review时间大于5分钟就很高了，Review时，写的Comment是有意义的Comment，那就更好了（极少数）。

但在代码社区，开源社区里面，你的每一行代码提交都是通过代码Review的方式来提交的，有的还是2级Review，第一级是通过当前功能owner的Review，第二级还要通过项目owner的Review。

而且在代码内部开源后，你需要接受其他部门的patch，对于你这个部门来说，你就要非常care代码的质量问题，因为他们的代码提交进来了，但这个服务还是你在维护，（如果代码出了问题）你会背他的锅，所以代码评审会很严肃，因此只有在内部开源的项目中，代码评审才不会成为摆设，这是在很多公司的内源实践中提到的一点，所以把内源做起来，把接收其他部门贡献做起来，就很容易提升代码质量。

### 有助于提升人员能力
再一个就是有助于提升人员能力。

一般工程师的能力提升是从工作中来，70%是日常工作，20%是业余学习，10%来自与其他人的交流学习，如果项目本身的团队氛围很差，或者能力不够，那么这个团队能力提升就很慢，但如果当这个同学可以参与到更多内源项目中，他就可以跟更多高手过招，他可以了解其他高手的代码，可以了解别人做的架构（进而得到成长和提升）。

而且内部开源的好处在于，我可以很方便的跟心目中的高手进行沟通。

我举个例子，像谷歌的内部工程师们，他们最喜欢做的事情就是，过一段时间看看Jack ben写得代码，Jack ben是谷歌的一个杰出工程师，他有名在于他开创了两个时代，两个时代他都站在了潮头，一个是大数据时代，一个是人工智能时代，这两个时代他都是领军人物，所以内部工程师很喜欢做的一件事情就是，每周去看看jack ben本周干了啥，他写的代码怎么样。所以内部开源后，可以把“墙”打开，能直接看到厂内其他人的工作，可以跟一些高手直接去沟通，比如可以给他提个patch，然后项目负责人就可以跟你说，你这里写得不好，那里还可以修改，经过这样的沟通和交流后，你的能力可以飞速增长。

### 有助于提升员工满意度
再一个可以提升员工满意度

这点在于，现在工程师，尤其是刚毕业的工程师，都是开源软件的受益者，从大学开始到毕业都习惯了开源软件，也习惯了开源社区的工作方式，我们称之为数字原生代（的工程师群体），这些工程师进入企业后会发现，团队用的陈旧的技术栈、一堆屎一样的代码、一个特别封闭的环境，你觉得这个工程师的心情是什么样的？只要稍微有点诱惑，他就会跳槽。

所以，员工，尤其是对自己有较高要求的工程师员工，他们更接受一种开源协同的开源社区的工作模式，而内源非常有助于让他们感受到这种工作模式。

## 国内多个公司的内源实践
我简单说一下国内几家公司做内源的情况。

腾讯从12年开始，百度从16年开始，华为从14年开始，滴滴从19年开始。

每一家的做法都不一样，目的也不一样，原因就在于，内部开源其实是跟提升（公司）效益相关的，跟每家公司的组织架构、企业文化都是相关的，所以每家都不一样。

我们简单看一下腾讯的做法，腾讯的内部开源，从2012开始，可以分为2个阶段，2012-2018年就是从下到上，做法很简单，就是鼓励大家多做好工程师的本分，怎么算做好工程师本分呢？很简单，Show me the code，把代码开出去让大家做品牌、让大家做贡献，这就是好的工程师。这样做的好处是，内部出现了很多很多的小项目，也培养了一些工程师，而且很多小项目后来也有一部分对外开源了，但缺点是什么？这种模式下，不会有真正对业务有影响力的项目出现，因为没有团队来从管理层去把关，都是靠工程师兴趣做的内源项目，这些项目往往集中在一些小工具、小库上面，虽然这个很好，影响范围广，但其实对业务影响不大。所以从2018年开始，腾讯内源的变革开始，他们开始大力推开源协同，在全公司内，推开源协同，目前他们有80%的代码仓库，是内部开源，内部可见的，除了一些机密的、策略相关的代码仓是保密的，其他大部分代码都是能看得到的，这就是腾讯的做法，先从下到上，再从上到下。

再简单说一下华为，华为比较有意思，他们是先从上到下，2014-2020年，他们做内源，是因为老板要求，必须要代码内部开源，这是上方的红头文件，我问过他们当时的初创人员，为什么这样做？原因很简单，华为内部很多项目都是互相调用的，出现问题后，就会出现“连环夺命call”的现象：项目现场发现A的问题，A发现是引用的B的，然后项目B的人进来，然后等等一长串，会有一个很长的链条最后都在一个电话会议内，而且问题的定位时间会非常长，虽然华为工程师很努力，但是在代码封闭情况下，只能这样（去排查问题），所以当代码开放后，可以很快定位到谁的原因，可以响应更快，然后满意度更高，这是华为做内部开源的初衷。（华为内源模式的）问题在于，很多代码虽然开源了，但基本上没有内部来自其他部门的贡献，（现象就是）就是代码放在那里，但研发模式和以前一样，大家会问，So What？

还有的公司我听过也做内源，他们是代码分两部分，一部分是小团队内部开源的，这是应付老板（要求）的内部开源，（他们）把一个版本拿出来放到那里（开放源码），这个开源很简单，但是别人看到代码没有任何意义。因为一那不是最新版本的代码，二我不能给你反馈，三我不清楚你项目的Roadmap、发展规划，而且（你这个项目）还不一定好用，那我凭什么用你的？所以这条路是走不下去的。

现在华为在做内部开源2.0，从上到下（的政策），结合从下到上，上面指令继续，下面（工程师们）建立广泛的开源内部社区，采用开源社区的玩法，鼓励大家做贡献。

百度的内部开源是我亲手经历的，（等会）会给大家讲一张图，说明是如何做的。

滴滴是从19年做的内部开源，当时是他们的CTO亲自推动的，他们做开源的目的有两点，一是希望内部开源能够成为外部开源的孵化器，二是希望内部开源能促进一些平台的更广泛的代码复用。

以上是这四家典型的（内源的）做法。

## InnerSource推荐架构图
然后简单说一下，这是一个InnerSource的推荐架构图，这也是我在百度做内部开源时，实行的策略。

要把内部开源做好，首先需要领导的支持，需要从上到下制定各种规章制度、奖惩制度来促进代码开放，促进部门间的协同，这是从上到下（的含义）。

下面是内部工程师的社区，这个社区必须要活跃，必须要有创造力，这个可以从下到上给整个内源做很好的支撑。不然的话从上到下的命令很好下，一纸公文发下去，但却没有变化，因为内部工程师不买单，他凭什么参与你？

然后实施的步骤呢，第一步可以先找一个项目来试点，第二步再找一个部门来试点，第三步，推广到全公司。每一步都需要相应的政策、都需要相应的流程，还需要相应的工具，大家可千万不要小看这个工具，工具很重要很重要。

这是一个推荐的架构图。今天时间不那么多，我把更多时间留给大家的问答，看看大家还有什么问题？

![](http://booogu.top/img/in-post/inner-source-architecture.png)