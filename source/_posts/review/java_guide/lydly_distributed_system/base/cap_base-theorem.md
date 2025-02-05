---
title: CAP&BASE 理论
description: CAP&BASE 理论
categories:
  - 学习
tags:
  - 复习
  - 复习-javaGuide
  - 复习-javaGuide-distributed_system
date: 2023-02-10 15:03:48
updated: 2023-02-10 15:03:48
---

> 转载自https://github.com/Snailclimb/JavaGuide（添加小部分笔记）感谢作者!

经历过技术面试的小伙伴想必对 CAP & BASE 这个两个理论已经再熟悉不过了！

我当年参加面试的时候，不夸张地说，只要问到分布式相关的内容，面试官几乎是必定会问这两个**分布式相关的理论**。一是因为这两个分布式基础理论是学习分布式知识的**必备前置基础**，二是因为很多面试官自己比较熟悉这两个理论（方便提问）。

我们非常有必要将这两个理论搞懂，并且能够用自己的理解给别人讲出来。

## CAP 理论

[CAP 理论/定理](https://zh.wikipedia.org/wiki/CAP定理)起源于 2000 年，由加州大学伯克利分校的 Eric Brewer 教授在分布式计算原理研讨会（PODC）上提出，因此 CAP 定理又被称作 **布鲁尔定理（Brewer’s theorem）**

2 年后，麻省理工学院的 Seth Gilbert 和 Nancy Lynch 发表了布鲁尔猜想的证明，CAP 理论正式成为**分布式领域的定理**。

### 简介

> ```[kənˈsɪstənsi] consistency 一致性```   
> ```[əˌveɪlə'bɪləti] availability 可用性 ```,  
>  ```[pɑːˈtɪʃn] 分割 [ˈtɒlərəns] 容忍, ```

**CAP** 也就是 **Consistency（一致性）**、**Availability（可用性）**、**Partition Tolerance（分区容错性）** 这三个单词首字母组合。

![lyx-20241126133857502](attachments/img/lyx-20241126133857502.png)

CAP 理论的提出者布鲁尔在提出 CAP 猜想的时候，并没有详细定义 **Consistency**、**Availability**、**Partition Tolerance** 三个单词的明确定义。

因此，对于 CAP 的民间解读有很多，一般比较被大家推荐的是下面 👇 这种版本的解读。

在理论计算机科学中，CAP 定理（CAP theorem）指出对于一个分布式系统来说，当设计**读写操作**时，只能同时满足以下三点中的两个：

- **一致性（Consistency）** : 所有节点**访问同一份最新的数据副本**
- **可用性（Availability）**: **非故障的节点在合理的时间内返回合理的响应**（不是错误或者超时的响应）。
- **分区容错性（Partition Tolerance）** : 分布式系统出现网络分区的时候，**仍然能够对外提供服务**。

**什么是网络分区？**

分布式系统中，多个节点之前的网络本来是连通的，但是因为某些故障（比如**部分节点网络出了问题**）某些节点之间不连通了，整个网络就分成了几块区域，这就叫 **网络分区**。

![partition-tolerance](attachments/img/lyx-20241126133858050.jpg) 

### 不是所谓的“3 选 2”

大部分人解释这一定律时，常常简单的表述为：“一致性、可用性、分区容忍性三者你只能同时达到其中两个，不可能同时达到”。实际上这是一个非常具有误导性质的说法，而且在 CAP 理论诞生 12 年之后，CAP 之父也在 2012 年重写了之前的论文。

> **当发生网络分区的时候，如果我们要继续服务（也就是P)，那么强一致性和可用性只能 2 选 1。也就是说当网络分区之后 P 是前提，决定了 P 之后才有 C 和 A 的选择。也就是说分区容错性（Partition tolerance）我们是必须要实现的。**
>
> 简而言之就是：CAP 理论中分区容错性 P 是一定要满足的，在此基础上，只能满足可用性 A 或者一致性 C。

因此，**分布式系统理论上不可能选择 CA 架构，只能选择 CP 或者 AP 架构。** 比如 ZooKeeper、HBase 就是 CP 架构，Cassandra、Eureka 就是 AP 架构，Nacos 不仅支持 CP 架构也支持 AP 架构。
> P，分区容错性，就是一定要保证能提供服务

**为啥不可能选择 CA 架构呢？** 举个例子：若系统出现“分区”，系统中的某个节点在进行写操作。为了保证 C， 必须要禁止其他节点的读写操作，这就和 A 发生冲突了。如果为了保证 A，其他节点的读写操作正常的话，那就和 C 发生冲突了。

**选择 CP 还是 AP 的关键在于当前的业务场景，没有定论，比如对于需要确保强一致性的场景如银行一般会选择保证 CP 。**

另外，需要补充说明的一点是： **如果网络分区正常的话（系统在绝大部分时候所处的状态），也就说不需要保证 P 的时候，C 和 A 能够同时保证。**

### CAP 实际应用案例

我这里以注册中心来探讨一下 CAP 的实际应用。考虑到很多小伙伴不知道注册中心是干嘛的，这里简单以 Dubbo 为例说一说。

下图是 Dubbo 的架构图。**注册中心 Registry 在其中扮演了什么角色呢？提供了什么服务呢？**

注册中心负责**服务地址的注册与查找**，相当于目录服务，**服务提供者**和**消费者**只在**启动时与注册中心交互**，注册中心不转发请求，压力较小。

![img](attachments/img/lyx-20241126133858524.jpg)](https://camo.gith

常见的可以作为注册中心的组件有：ZooKeeper、Eureka、Nacos...。

1. **ZooKeeper 保证的是 CP。** 任何时刻对 ZooKeeper 的读请求都能得到**一致性**的结果，但是， ZooKeeper **不保证每次请求的可用性**比如在 Leader **选举过程中**或者**半数以上的机器不可用**的时候**服务就是不可用**的。
2. **Eureka 保证的则是 AP。** Eureka 在设计的时候就是**优先保证 A （可用性）**。在 Eureka 中不存在什么 Leader 节点，每个节点都是一样的、平等的。因此 Eureka 不会像 ZooKeeper 那样出现选举过程中或者半数以上的机器不可用的时候服务就是不可用的情况。 Eureka 保证**即使大部分节点挂掉也不会影响正常提供服务**，只要有一个节点是可用的就行了。只不过**这个节点上的数据可能并不是最新**的。
3. **Nacos 不仅支持 CP 也支持 AP。**

### 总结

在进行分布式系统设计和开发时，我们不应该仅仅局限在 CAP 问题上，还要关注系统的扩展性、可用性等等

**在系统发生“分区”的情况下，CAP 理论只能满足 CP 或者 AP**。要注意的是，这里的前提是系统发生了“分区”

如果系统没有发生“分区”的话，**节点间的网络连接通信正常**的话，也就不存在 P 了。这个时候，我们就可以同时保证 C 和 A 了。

总结：**如果系统发生“分区”，我们要考虑选择 CP 还是 AP。如果系统没有发生“分区”的话，我们要思考如何保证 CA 。**

### 推荐阅读

1. [CAP 定理简化](https://medium.com/@ravindraprasad/cap-theorem-simplified-28499a67eab4) （英文，有趣的案例）
2. [神一样的 CAP 理论被应用在何方](https://juejin.im/post/6844903936718012430) （中文，列举了很多实际的例子）
3. [请停止呼叫数据库 CP 或 AP ](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html)（英文，带给你不一样的思考）

## BASE 理论

[BASE 理论](https://dl.acm.org/doi/10.1145/1394127.1394128)起源于 2008 年， 由 eBay 的架构师 Dan Pritchett 在 ACM 上发表。

### 简介

**BASE** 是 **Basically Available（基本可用）** 、**Soft-state（软状态）** 和 **Eventually Consistent（最终一致性）** 三个短语的缩写。BASE 理论是对 CAP 中一致性 C 和可用性 A 权衡的结果，其来源于对大规模互联网系统分布式实践的总结，是基于 CAP 定理逐步演化而来的，它**大大降低了我们对系统的要求**。

### BASE 理论的核心思想

即使无法做到强一致性，但每个应用都可以根据自身业务特点，采用适当的方式来使系统达到**最终一致性**。

> 也就是牺牲数据的一致性来满足系统的高可用性，系统中**一部分数据不可用**或者**不一致**时，仍需要保持**系统整体“主要可用”**。

**BASE 理论本质上是对 CAP 的延伸和补充，更具体地说，是对 CAP 中 AP 方案的一个补充。**

**为什么这样说呢？**

CAP 理论这节我们也说过了：

> 如果系统没有发生“分区”的话，节点间的网络连接通信正常的话，也就不存在 P 了。这个时候，我们就可以同时保证 C 和 A 了。因此，**如果系统发生“分区”，我们要考虑选择 CP 还是 AP。如果系统没有发生“分区”的话，我们要思考如何保证 CA 。**

因此，AP 方案只是在系统**发生分区的时候放弃一致性**，而**不是永远放弃一致性**。在**分区故障恢复**后，系统应该**达到最终一致性**。这一点其实就是 BASE 理论延伸的地方。

### BASE 理论三要素

 ![lyx-20241126133858952.png](attachments/img/lyx-20241126133858952.png)


#### 基本可用

基本可用是指分布式系统在出现不可预知故障的时候，允许**损失部分可用性**。但是，这绝不等价于系统不可用。

**什么叫允许损失部分可用性呢？**

- **响应时间上的损失**: 正常情况下，处理用户请求需要 0.5s 返回结果，但是由于系统出现故障，处理用户请求的时间变为 3 s。
- **系统功能上的损失**：正常情况下，用户可以使用系统的全部功能，但是由于系统访问量突然剧增，系统的部分非核心功能无法使用。

#### 软状态

软状态指允许系统中的数据存在中间状态（**CAP 理论中的数据不一致**），并认为**该中间状态的存在不会影响系统的整体可用性**，即**允许系统在不同节点的数据副本之间进行数据同步的过程存在延时**。

#### 最终一致性

最终一致性强调的是**系统中所有的数据副本，在经过一段时间的同步后，最终能够达到一个一致的状态**。因此，最终一致性的本质是需要系统保证**最终数据能够达到一致**，而**不需要实时保证系统数据的强一致性**。

> 分布式一致性的 3 种级别：
>
> 1. **强一致性** ：系统写入了什么，读出来的就是什么。
> 2. **弱一致性** ：不一定可以读取到最新写入的值，也不保证多少时间之后读取到的数据是最新的，只是会**尽量保证某个时刻达到数据一致**的状态。
> 3. **最终一致性** ：弱一致性的升级版，系统会保证**在一定时间内达到数据一**致的状态。
>
> **业界比较推崇是最终一致性级别，但是某些对数据一致要求十分严格的场景比如银行转账还是要保证强一致性。**

那实现最终一致性的具体方式是什么呢? [《分布式协议与算法实战》](http://gk.link/a/10rZM) 中是这样介绍：

> - **读时修复** : 在**读取数据时，检测数据的不一致，进行修复**。比如 Cassandra 的 Read Repair 实现，具体来说，在向 Cassandra 系统查询数据的时候，如果检测到不同节点的副本数据不一致，系统就自动修复数据。
> - **写时修复** : 在**写入数据，检测数据的不一致时，进行修复**。比如 Cassandra 的 Hinted Handoff 实现。具体来说，Cassandra 集群的节点之间远程写数据的时候，如果**写失败 就将数据缓存下来，然后定时重传，修复数据的不一致性**。
> - **异步修复** : 这个是最常用的方式，通过**定时对账检测副本数据的一致性，并修复**。

比较推荐 **写时修复**，这种方式对性能消耗比较低。

### 总结

**ACID 是数据库事务完整性的理论，CAP 是分布式系统设计理论，BASE 是 CAP 理论中 AP 方案的延伸。**