---
title: rpc基础及面试题
description: rpc基础及面试题
categories:
  - 学习
tags:
  - 复习
  - 复习-javaGuide
  - 复习-javaGuide-distributed_system
date: 2023-02-11 19:41:34
updated: 2023-02-11 19:41:34
---

> 转载自https://github.com/Snailclimb/JavaGuide（添加小部分笔记）感谢作者!

简单介绍一下 RPC 相关的基础概念。

## 何为 RPC?

**RPC（Remote Procedure Call）** 即远程过程调用，通过名字我们就能看出 RPC 关注的是远程调用而非本地调用。

**为什么要 RPC ？** 因为，两个不同的服务器上的服务提供的方法不在一个内存空间，所以，需要通过网络编程才能传递方法调用所需要的参数。并且，方法调用的结果也需要通过网络编程来接收。但是，如果我们自己手动网络编程来实现这个调用过程的话工作量是非常大的，因为，我们需要考虑底层传输方式（TCP还是UDP）、序列化方式等等方面。

**RPC 能帮助我们做什么呢？** 简单来说，通过 RPC 可以帮助我们调用远程计算机上某个服务的方法，这个过程就像调用本地方法一样简单。并且！我们不需要了解底层网络编程的具体细节。

举个例子：两个不同的服务 A、B 部署在两台不同的机器上，服务 A 如果想要调用服务 B 中的某个方法的话就可以通过 RPC 来做。

一言蔽之：**RPC 的出现就是为了让你调用远程方法像调用本地方法一样简单。**

## RPC 的原理是什么?

为了能够帮助小伙伴们理解 RPC 原理，我们可以将整个 RPC的 核心功能看作是下面👇 5 个部分实现的：

1. **客户端（服务消费端）** ：调用远程方法的一端。
2. **客户端 Stub（桩）** ： 这其实就是一代理类。代理类主要做的事情很简单，就是把你调用方法、类、方法参数等信息传递到服务端。
3. **网络传输** ： 网络传输就是你要把你调用的方法的信息比如说参数啊这些东西传输到服务端，然后服务端执行完之后再把返回结果通过网络传输给你传输回来。网络传输的实现方式有很多种比如最近基本的 Socket或者性能以及封装更加优秀的 Netty（推荐）。
4. **服务端 Stub（桩）** ：这个桩就不是代理类了。我觉得理解为桩实际不太好，大家注意一下就好。这里的服务端 Stub 实际指的就是接收到客户端执行方法的请求后，去指定对应的方法然后返回结果给客户端的类。
5. **服务端（服务提供端）** ：提供远程方法的一端。

具体原理图如下，后面我会串起来将整个RPC的过程给大家说一下。

[![RPC原理图](attachments/img/lyx-20241126133917319.jpg)](https://camo.githubusercontent.com/5e8198844ec97b718219e83fec147089c73daadc5c987752787bfa4d4d538c24/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d31322d362f33373334353835312e6a7067)

1. 服务消费端（client）以本地调用的方式调用远程服务；
2. 客户端 Stub（client stub） 接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体（序列化）：`RpcRequest`；
3. 客户端 Stub（client stub） 找到远程服务的地址，并将消息发送到服务提供端；
4. 服务端 Stub（桩）收到消息将消息反序列化为Java对象: `RpcRequest`；
5. 服务端 Stub（桩）根据`RpcRequest`中的类、方法、方法参数等信息调用本地的方法；
6. 服务端 Stub（桩）得到方法执行结果并将组装成能够进行网络传输的消息体：`RpcResponse`（序列化）发送至消费方；
7. 客户端 Stub（client stub）接收到消息并将消息反序列化为Java对象:`RpcResponse` ，这样也就得到了最终结果。over!

相信小伙伴们看完上面的讲解之后，已经了解了 RPC 的原理。

虽然篇幅不多，但是基本把 RPC 框架的核心原理讲清楚了！另外，对于上面的技术细节，我会在后面的章节介绍到。

**最后，对于 RPC 的原理，希望小伙伴不单单要理解，还要能够自己画出来并且能够给别人讲出来。因为，在面试中这个问题在面试官问到 RPC 相关内容的时候基本都会碰到。**

## 有哪些常见的 RPC 框架？

我们这里说的 RPC 框架指的是可以让客户端直接调用服务端方法，就像调用本地方法一样简单的框架，比如我下面介绍的 Dubbo、Motan、gRPC这些。 如果需要和 HTTP 协议打交道，解析和封装 HTTP 请求和响应。这类框架并不能算是“RPC 框架”，比如Feign。

### Dubbo

[![img](attachments/img/lyx-20241126133917889.jpg)](https://camo.githubusercontent.com/339828865c8318aeea323c1c693417d8a1afca2329dffd0db201c0a5992502fb/68747470733a2f2f67756964652d626c6f672d696d616765732e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f6769746875622f6a61766167756964652f64697374726962757465642d73797374656d2f7270632f696d6167652d32303232303731363131313035333038312e706e67)

Apache Dubbo 是一款微服务框架，为大规模微服务实践提供高性能 RPC 通信、流量治理、可观测性等解决方案， 涵盖 Java、Golang 等多种语言 SDK 实现。

Dubbo 提供了从服务定义、服务发现、服务通信到流量管控等几乎所有的服务治理能力，支持 Triple 协议（基于 HTTP/2 之上定义的下一代 RPC 通信协议）、应用级服务发现、Dubbo Mesh （Dubbo3 赋予了很多云原生友好的新特性）等特性。

[![img](https://camo.githubusercontent.com/4819aa47ea3d35b239480b466ca82b600cd4df196101b08e5953e387375af3ac/68747470733a2f2f67756964652d626c6f672d696d616765732e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f6769746875622f6a61766167756964652f64697374726962757465642d73797374656d2f7270632f696d6167652d32303232303731363131313534353334332e706e67)](https://camo.githubusercontent.com/4819aa47ea3d35b239480b466ca82b600cd4df196101b08e5953e387375af3ac/68747470733a2f2f67756964652d626c6f672d696d616765732e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f6769746875622f6a61766167756964652f64697374726962757465642d73797374656d2f7270632f696d6167652d32303232303731363131313534353334332e706e67)

Dubbo 是由阿里开源，后来加入了 Apache 。正是由于 Dubbo 的出现，才使得越来越多的公司开始使用以及接受分布式架构。

Dubbo 算的是比较优秀的国产开源项目了，它的源码也是非常值得学习和阅读的！

- Github ：https://github.com/apache/incubator-dubbo
- 官网：https://dubbo.apache.org/zh/

### Motan

Motan 是新浪微博开源的一款 RPC 框架，据说在新浪微博正支撑着千亿次调用。不过笔者倒是很少看到有公司使用，而且网上的资料也比较少。

很多人喜欢拿 Motan 和 Dubbo 作比较，毕竟都是国内大公司开源的。笔者在查阅了很多资料，以及简单查看了其源码之后发现：**Motan 更像是一个精简版的 Dubbo，可能是借鉴了 Dubbo 的思想，Motan 的设计更加精简，功能更加纯粹。**

不过，我不推荐你在实际项目中使用 Motan。如果你要是公司实际使用的话，还是推荐 Dubbo ，其社区活跃度以及生态都要好很多。

- 从 Motan 看 RPC 框架设计：http://kriszhang.com/motan-rpc-impl/
- Motan 中文文档：https://github.com/weibocom/motan/wiki/zh_overview

### gRPC

[![img](https://camo.githubusercontent.com/5bdb69cd978e45cefb178305acd5ef334843e2ce281bc6f219f4dde79ebfd9e6/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323032302d382f32383433623130642d306332662d346237652d396333652d6561343436363739326138622e706e67)](https://camo.githubusercontent.com/5bdb69cd978e45cefb178305acd5ef334843e2ce281bc6f219f4dde79ebfd9e6/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323032302d382f32383433623130642d306332662d346237652d396333652d6561343436363739326138622e706e67)

gRPC 是 Google 开源的一个高性能、通用的开源 RPC 框架。其由主要面向移动应用开发并基于 HTTP/2 协议标准而设计（支持双向流、消息头压缩等功能，更加节省带宽），基于 ProtoBuf 序列化协议开发，并且支持众多开发语言。

**何谓 ProtoBuf？** [ProtoBuf（ Protocol Buffer）](https://github.com/protocolbuffers/protobuf) 是一种更加灵活、高效的数据格式，可用于通讯协议、数据存储等领域，基本支持所有主流编程语言且与平台无关。不过，通过 ProtoBuf 定义接口和数据类型还挺繁琐的，这是一个小问题。

[![img](attachments/img/lyx-20241126133918341.jpg)](https://camo.githubusercontent.com/e8a72b2bce1f9f1cdcf1d9d87ac01a4efa9d830bfb968dd012affb152577eccd/68747470733a2f2f67756964652d626c6f672d696d616765732e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f6769746875622f6a61766167756964652f64697374726962757465642d73797374656d2f7270632f696d6167652d32303232303731363130343330343033332e706e67)

不得不说，gRPC 的通信层的设计还是非常优秀的，[Dubbo-go 3.0](https://dubbogo.github.io/) 的通信层改进主要借鉴了 gRPC。

不过，gRPC 的设计导致其几乎没有服务治理能力。如果你想要解决这个问题的话，就需要依赖其他组件比如腾讯的 PolarisMesh（北极星）了。

- Github：https://github.com/grpc/grpc
- 官网：https://grpc.io/

### Thrift

Apache Thrift 是 Facebook 开源的跨语言的 RPC 通信框架，目前已经捐献给 Apache 基金会管理，由于其跨语言特性和出色的性能，在很多互联网公司得到应用，有能力的公司甚至会基于 thrift 研发一套分布式服务框架，增加诸如服务注册、服务发现等功能。

`Thrift`支持多种不同的**编程语言**，包括`C++`、`Java`、`Python`、`PHP`、`Ruby`等（相比于 gRPC 支持的语言更多 ）。

- 官网：https://thrift.apache.org/
- Thrift 简单介绍：https://www.jianshu.com/p/8f25d057a5a9

### 总结

gRPC 和 Thrift 虽然支持跨语言的 RPC 调用，但是它们只提供了最基本的 RPC 框架功能，缺乏一系列配套的服务化组件和服务治理功能的支撑。

Dubbo 不论是从功能完善程度、生态系统还是社区活跃度来说都是最优秀的。而且，Dubbo在国内有很多成功的案例比如当当网、滴滴等等，是一款经得起生产考验的成熟稳定的 RPC 框架。最重要的是你还能找到非常多的 Dubbo 参考资料，学习成本相对也较低。

下图展示了 Dubbo 的生态系统。

[![img](https://camo.githubusercontent.com/cca8f0a0f0aecb206130f302115d639a3c6c9b1bccb8d579997f19d2c50b437b/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323032302d382f65656539386666322d386530362d343632382d613432622d6433306666636432383331652e706e67)](https://camo.githubusercontent.com/cca8f0a0f0aecb206130f302115d639a3c6c9b1bccb8d579997f19d2c50b437b/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323032302d382f65656539386666322d386530362d343632382d613432622d6433306666636432383331652e706e67)

Dubbo 也是 Spring Cloud Alibaba 里面的一个组件。

[![img](attachments/img/lyx-20241126133918770.jpg)](https://camo.githubusercontent.com/227e0ade4d8a92d222f5abee729d335262339823826abf26cd9c0c562a3db850/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323032302d382f30643139356461652d373262632d343935362d383435312d3365616636646431316362642e706e67)

但是，Dubbo 和 Motan 主要是给 Java 语言使用。虽然，Dubbo 和 Motan 目前也能兼容部分语言，但是不太推荐。如果需要跨多种语言调用的话，可以考虑使用 gRPC。

综上，如果是 Java 后端技术栈，并且你在纠结选择哪一种 RPC 框架的话，我推荐你考虑一下 Dubbo。

## 如何设计并实现一个 RPC 框架？

**《手写 RPC 框架》** 是我的[知识星球](https://javaguide.cn/about-the-author/zhishixingqiu-two-years.html)的一个内部小册，我写了 12 篇文章来讲解如何从零开始基于 Netty+Kyro+Zookeeper 实现一个简易的 RPC 框架。

麻雀虽小五脏俱全，项目代码注释详细，结构清晰，并且集成了 Check Style 规范代码结构，非常适合阅读和学习。

**内容概览** ：

[![img](attachments/img/lyx-20241126133919244.jpg)](https://camo.githubusercontent.com/ae931acda3a182ad4fb3759844a86c132d2f2734ffa398256b55385bc0d35b37/68747470733a2f2f67756964652d626c6f672d696d616765732e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f6769746875622f6a61766167756964652f696d6167652d32303232303330383130303630353438352e706e67)

## 既然有了 HTTP 协议，为什么还要有 RPC ？

[HTTP 和 RPC 详细对比](https://github.com/Snailclimb/JavaGuide/blob/main/docs/distributed-system/rpc/http&rpc.md) 。