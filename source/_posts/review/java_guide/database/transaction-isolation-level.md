---
title: MySQL事务隔离级别详解
description: 隔离级别
categories:
  - 学习
tags:
  - 复习
  - 复习-javaGuide
  - 复习-javaGuide-database
date: 2023-01-16 01:00:44
updated: 2023-01-16 01:00:44
---

> 转载自https://github.com/Snailclimb/JavaGuide（添加小部分笔记）感谢作者!

# 事务隔离级别总结

- SQL标准定义了**四个隔离级别**

  1. **READ-UNCOMMITTED(读取未提交)**：**最低**的隔离级别，允许读取**尚未提交的数据**变更，可能会导致**脏读、幻读或不可重复读**
  2. **READ-COMMITED(读取已提交)**：允许读取**并发事务** **已经提交**的数据，可以阻止**脏读**，但是**幻读**或**不可重复读**仍有可能发生
  3. **REPEATABLE-READ(可重复读)**：对**同一字段的多次读取**结果都是一致的，除非数据是被**本身事务自己**所修改，可以**阻止脏读**和**不可重复读**，但**幻读**仍有可能发生
  4. **SERIALIZABLE(可串行化)**：**最高**的隔离级别，**完全服从ACID**的隔离级别。所有的**事务依次逐个**执行，这样事务之间就**完全不可能产生干扰**，也就是说，该级别可以防止**脏读**、**不可重复读**以及**幻读**。

  |     隔离级别     | 脏读 | 不可重复读 | 幻读 |
  | :--------------: | :--: | :--------: | :--: |
  | READ-UNCOMMITTED |  √   |     √      |  √   |
  |  READ-COMMITTED  |  ×   |     √      |  √   |
  | REPEATABLE-READ  |  ×   |     ×      |  √   |
  |   SERIALIZABLE   |  ×   |     ×      |  ×   |

- MySQL InnoDB 存储引擎的默认支持的隔离级别是 **REPEATABLE-READ（可重读）**

  > 使用命令查看，通过```SELECT @@tx_isolation;```。  
  > MySQL 8.0 该命令改为```SELECT @@transaction_isolation;```
  >
  > ```shell
  > MySQL> SELECT @@tx_isolation;
  > +-----------------+
  > | @@tx_isolation  |
  > +-----------------+
  > | REPEATABLE-READ |
  > +-----------------+ 
  > ```

- 从上面对SQL标准定义了**四个隔离级别**的介绍可以看出，标准的SQL隔离级别里，**REPEATABLE-READ(可重复读)**是不可以防止幻读的。但是，**InnoDB实现的REPEATABLE-READ** 隔离级别其实是可以**解决幻读**问题发生的，分两种情况

  1. **快照读**：由**MVCC**机制来保证不出现幻读
  2. **当前读**：使用**Next-Key Lock**进行加锁来保证不出现幻读，Next-Key Lock是**行锁（Record Lock ）和间隙锁（Gap Lock）的结合**，行锁只能锁住已经存在的行，为了避免插入新行，需要依赖**间隙锁**  (**只用间隙锁不行，因为间隙锁是 > 或 < ，不包括等于，所以再可重复读下原记录可能会被删掉**)

  因为**隔离级别越低，事务请求的锁越少**，所以大部分数据库系统的隔离级别都是 **READ-COMMITTED** ，但是你要知道的是 InnoDB 存储引擎默认使用 **REPEATABLE-READ** 并不会有任何性能损失。

- InnoDB 存储引擎在**分布式事务**的情况下一般会用到 **SERIALIZABLE 隔离级别**

  1. InnoDB 存储引擎提供了对 **XA 事务**的支持，并**通过 XA 事务来支持分布式事务**的实现。
  2. 分布式事务指的是允许**多个独立的事务资源（transactional resources）**参与到**一个全局的事务**中。事务资源通常是关系型数据库系统，但也可以是其他类型的资源。**全局事务**要求在其中的**所有参与的事务要么都提交**，要么**都回滚**，这对于事务原有的 ACID 要求又有了提高。
  3. 在**使用分布式事务时**，InnoDB 存储引擎的**事务隔离级别必须设置为 SERIALIZABLE**。

# 实际情况演示

- 下面会使用2个命令行MySQL，模拟多线程（多事务）对同一份数据的(脏读等)问题

  > MySQL 命令行的默认配置中事务都是自动提交的，即执行 SQL 语句后就会马上执行 COMMIT 操作。如果要显式地开启一个事务需要使用命令：**`START TRANSACTION`**

- 通过下面的命令来设置隔离级别

  ```shell
  SET [SESSION|GLOBAL] TRANSACTION ISOLATION LEVEL [READ UNCOMMITTED|READ COMMITTED|REPEATABLE READ|SERIALIZABLE] 
  ```

- 实际操作中使用到的一些并发控制的语句

  1. ```START TRANSACTION | BEGIN```：显示地开启一个事务

## 脏读（读未提交）

## 避免脏读（读已提交）

## 不可重复读

## 可重复度

## 幻读