---
title: SQL语句在MySQL中的执行过程
description: SQL语句在MySQL中的执行过程
categories:
  - 学习
tags:
  - 复习
  - 复习-javaGuide
  - 复习-javaGuide-database
date: 2023-01-19 10:20:57
updated: 2023-01-19 10:20:57
---

> 转载自https://github.com/Snailclimb/JavaGuide（添加小部分笔记）感谢作者!
>
> 原文 https://github.com/kinglaw1204 感谢作者

- 本篇文章会分析**一个SQL语句**在MySQL的**执行流程**，包括**SQL的查询**在MySQL内部会怎么**流转**，**SQL语句的更新**是怎么完成的
- 分析之前先看看**MySQL的基础架构**，知道了MySQL由**哪些组件**组成以及**这些组件的作用**是什么，可以帮助我们**理解**和**解决**这些问题

 # MySQL基础架构分析

![lyx-20241126133519051](attachments/img/lyx-20241126133519051.png)

## MySQL基本架构概览

- 下图是MySQL的简要架构图，从下图可以看到**用户的SQL语句**在MySQL内部是**如何执行的**
- 先简单介绍一个下图涉及的一些组件的基本作用
  ![lyx-20241126133519184](attachments/img/lyx-20241126133519184.png)
  1. **连接器**： **身份认证**和**权限相关**（登录MySQL的时候）
  2. **查询缓存**：执行查询语句的时候，会**先查询缓存**（MySQL8.0版本后移除，因为这个功能不太实用）
  3. **分析器**：**没有命中**缓存的话，SQL语句就会经过分析器，分析器说白了就是要**先看**你的SQL语句干嘛，再**检查**你的SQL语句**语法**是否正确
  4. **优化器**：按照**MySQL认为最优的方案**去执行
  5. **执行器**：**执行语句**，然后从**存储引擎**返回数据
- 简单来说 MySQL 主要分为 Server 层和存储引擎层：
  - **Server 层**：主要包括**连接器**、**查询缓存**、**分析器**、**优化器**、**执行器**等，所有**跨存储引擎**的功能都在这一层实现，比如**存储过程**、**触发器**、**视图**，**函数**等，还有一个**通用的日志模块 binlog 日志模块**。
  - **存储引擎**： 主要负责数据的**存储**和**读取**，采用**可以替换的插件式架构**，支持 InnoDB、MyISAM、Memory 等多个存储引擎，其中 InnoDB 引擎有自有的日志模块 **redolog 模块**。**现在最常用的存储引擎是 InnoDB，它从 MySQL 5.5 版本开始就被当做默认存储引擎了**

## Server层基本组件介绍

![lyx-20241126133519678](attachments/img/lyx-20241126133519678.png)

1. **连接器**
   连接器主要和**身份认证**和**权限相关**的功能相关，就好比一个级别很高的门卫一样  

   > 主要负责用户登录数据库，进行用户的身份认证，包括校验账户密码，权限等操作，如果用户**账户密码**已通过，连接器会到**权限表**中查询**该用户的所有权限**，之后在这个连接里的权限逻辑判断都是会依赖**此时读取到的权限数据**，也就是说，**后续只要这个连接不断开**，**即使**管理员**修改**了该用户的权限，该用户也是不受影响的。

2. **查询缓存（MySQL8.0 版本后移除）**  
   查询缓存**主要用来缓存**我们所执行的 **SELECT 语句**以及该**语句的结果集**。

   > -  连接建立后，执行查询语句的时候，会先查询缓存，MySQL 会先校验这个 SQL 是否执行过，以 **Key-Value** 的形式缓存在内存中，Key 是查询预计，Value 是结果集。如果缓存 key 被命中，就会直接返回给客户端，如果没有命中，就会执行后续的操作，完成后也会把结果缓存起来，方便下一次调用。当然在真正执行缓存查询的时候还是会校验用户的权限，是否有该表的查询条件。
   >
   > - MySQL 查询不建议使用缓存，因为查询缓存失效在实际业务场景中可能会非常频繁，假如**你对一个表更新**的话，**这个表上的所有的查询缓存都会被清空**。对于**不经常更新的数据**来说，**使用缓存还是可以**的。所以，一般在大多数情况下我们都是不推荐去使用查询缓存的。

   MySQL **8.0 版本后删除了缓存**的功能，官方也是认为该功能在实际的应用场景比较少，所以干脆直接删掉了

3. **分析器**MySQL 没有命中缓存，那么就会进入分析器，分析器主要是用来**分析 SQL 语句是来干嘛**的，分析器也会分为几步：

   **第一步，词法分析**，一条 SQL 语句有多个字符串组成，首先要**提取关键字**，比如 select，提出查询的表，提出字段名，提出查询条件等等。做完这些操作后，就会进入第二步。

   **第二步，语法分析**，主要就是判断你输入的 SQL 是否正确，**是否符合 MySQL 的语法**。

   完成这 2 步之后，MySQL 就准备开始执行了，但是**如何执行，怎么执行是最好的结果**呢？这个时候就需要优化器上场了。

4. **优化器**  
   优化器的作用就是**它认为的最优的执行**方案去执行（有时候可能也不是最优，这篇文章涉及对这部分知识的深入讲解），比如**多个索引**的时候该如何选择索引，**多表查询**的时候**如何选择关联顺序**等。

   可以说，经过了优化器之后可以说这个语句具体该如何执行就已经定下来

5. **执行器**  
   当选择了执行方案后，MySQL 就准备开始执行了，首先**执行前会校验该用户有没有权限**，如果没有权限，就会返回错误信息，**如果有**权限，就会去**调用引擎的接口**，返回接口执行的结果

# 语句分析  

SQL分为两种，一种是**查询**，一种是**更新（增加、修改、删除）**

## 查询语句

```select * from tb_student  A where A.age='18' and A.name=' 张三 ';```

结合上面说明，分析下面这个语句的**执行流程**：  

1. 先**检查该语句是否有权限**，如果没有权限，直接返回错误信息，如果有权限，在 MySQL8.0 版本以前，会先查询缓存，以这条 SQL 语句为 key 在内存中查询是否有结果，如果有直接缓存，如果没有，执行下一步。

2. 通过**分析器**进行**词法分析**，提取 SQL 语句的关键元素，比如提取上面这个语句是查询 select，提取需要查询的表名为 tb_student，需要查询所有的列，查询条件是这个表的 id='1'。然后判断这个 SQL 语句是否有**语法错误**，比如关键词是否正确等等，如果检查没问题就执行下一步。

3. 接下来就是优化器进行确定执行方案，上面的 SQL 语句，可以有两种执行方案：  

   ```
   a.先查询学生表中姓名为“张三”的学生，然后判断是否年龄是 18。
   b.先找出学生中年龄 18 岁的学生，然后再查询姓名为“张三”的学生。
   ```

   那么优化器根据自己的优化算法进行选择执行效率最好的一个方案（**优化器认为，有时候不一定最好**）。那么确认了执行计划后就准备开始执行了

4. 进行**权限校验**，如果没有权限就会返回错误信息，如果有权限就会调用**数据库引擎接口**，**返回引擎的执行结果**

## 更新语句

以上就是一条查询 SQL 的执行流程，那么接下来我们看看一条更新语句如何执行的呢？SQL 语句如下：  
```update tb_student A set A.age='19' where A.name=' 张三 ';```

> - 我们来给张三修改下年龄，在实际数据库肯定不会设置年龄这个字段的，不然要被技术负责人打的 (因为可能有生日，**年龄是不可人为手动修改**)
> - 其实这条语句也基本上会沿着上一个查询的流程走，只不过执行更新的时候肯定要记录日志啦，这就会**引入日志模块**了，MySQL 自带的日志模块是 **binlog（归档日志）** ，所有的存储引擎都可以使用，我们常用的 InnoDB 引擎还自带了一个日志模块 **redo log（重做日志）**，我们就以 InnoDB 模式下来探讨这个语句的执行流程

1. 先**查询**到张三这一条数据，如果有缓存，也是会用到缓存。
2. 然后拿到查询的语句，把 **age 改为 19**，然后调用引擎 API 接口，写入这一行数据，**InnoDB 引擎把数据保存在内存**中，同时**记录 redo log**，此时 redo log 进入 prepare 状态，然后告诉执行器，执行完成了，随时可以提交。
3. 执行器收到通知后**记录 binlog**，然后调用引擎接口，提交 redo log 为提交状态。
4. 更新完成

**这里肯定有同学会问，为什么要用两个日志模块，用一个日志模块不行吗?**

- 这是因为最开始 MySQL 并没有 InnoDB 引擎（InnoDB 引擎是其他公司以插件形式插入 MySQL 的），MySQL 自带的引擎是 MyISAM，但是我们知道 **redo log 是 InnoDB 引擎特有**的，其他存储引擎都没有，这就导致会没有 **crash-safe** 的能力(**crash-safe 的能力即使数据库发生异常重启，之前提交的记录都不会丢失**)，binlog 日志只能用来归档。

- 并不是说只用一个日志模块不可以，只是 InnoDB 引擎就是通过 redo log 来支持事务的。那么，又会有同学问，我用两个日志模块，但是不要这么复杂行不行，为什么 redo log 要引入 prepare 预提交状态？这里我们用反证法来说明下为什么要这么做？
  - **先写 redo log 直接提交，然后写 binlog**，假设写完 redo log 后，机器挂了，binlog 日志没有被写入，那么机器重启后，这台机器会通过 redo log 恢复数据，但是这个时候 binlog 并没有记录该数据，**后续进行机器备份(从机)**的时候，就会丢失这一条数据，同时主从同步也会丢失这一条数据。
  - **先写 binlog，然后写 redo log**，假设写完了 binlog，机器异常重启了，由于没有 redo log，**本机是无法恢复**这一条记录的，但是 binlog 又有记录，那么和上面同样的道理，就会产生数据不一致的情况。

- 如果采用 redo log 两阶段提交的方式就不一样了，**写完 binlog 后，然后再提交 redo log** 就会防止出现上述的问题，从而保证了数据的一致性。那么问题来了，有没有一个极端的情况呢？假设 **redo log 处于预提交状态，binlog 也已经写完**了，这个时候发生了异常重启会怎么样呢？ 这个就要依赖于 **MySQL 的处理机制**了，MySQL 的处理过程如下：
  - 判断 redo log 是否完整，如果判断是完整的，就立即提交。
  - 如果 **redo log 只是预提交但不是 commit 状态，这个时候就会去判断 binlog 是否完整，如果完整就提交 redo log, 不完整就回滚事务**。

这样就解决了数据一致性的问题

# 总结

- MySQL 主要分为 **Server 层**和**引擎层**，Server 层主要包括**连接器**、**查询缓存**、分析器、**优化器**、**执行器**，同时还有一个**日志模块（binlog）**，这个日志模块所有执行引擎都可以共用，**redolog 只有 InnoDB 有**。

- 引擎层是插件式的，目前主要包括，**MyISAM**,**InnoDB**,**Memory** 等。

- **查询**语句的执行流程如下：**权限校验（如果命中缓存）--->查询缓存--->分析器--->优化器--->权限校验--->执行器--->引擎**

  **更新**语句执行流程如下：**分析器---->权限校验---->执行器--->引擎---redo log(prepare 状态)--->binlog--->redo log(commit状态**

  

