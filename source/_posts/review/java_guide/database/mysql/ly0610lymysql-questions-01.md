---
title: MySQL常见面试题总结
description: MySQL常见面试题总结
categories:
  - 学习
tags:
  - 复习
  - 复习-javaGuide
  - 复习-javaGuide-database
date: 2023-01-20 11:36:06
updated: 2023-01-22 22:46:06
---

> 转载自https://github.com/Snailclimb/JavaGuide（添加小部分笔记）感谢作者!====

# MySQL基础
## 关系型数据库介绍

- **关系型数据库**，建立在**关系模型**的基础上的数据库。表明数据库中所**存储**的数据之间的**联系**（一对一、一对多、多对多）
- 关系型数据库中，我们的数据都被**存放在各种表**中（比如用户表），表中的**每一行**存放着**一条数据（比如一个用户的信息）**
  ![关系型数据库表关系](attachments/img/lyx-20241126133523797.png)
- 大部分关系型数据库都使用**SQL**来操作数据库中的数据，并且大部分**关系型数据库**都支持**事务**的**四大特性（ACID）**

**常见的关系型数据库**  
**MySQL**、**PostgreSQL**、**Oracle**、**SQL Server**、**SQLite**（**微信本地的聊天记录**的存储就是用的 SQLite） ......

## MySQL介绍

![img](attachments/img/lyx-20241126133524335.png)

- MySQL是一种**关系型数据库**，主要用于**持久化存储**我们系统中的一些数据比如**用户信息**

- 由于 MySQL 是**开源**免费并且比较**成熟**的数据库，因此，MySQL 被大量使用在各种系统中。任何人都可以在 **GPL(General Public License 通用性公开许可证)** 的许可下下载并根据**个性化的需要**对其进行**修改**。MySQL 的默认端口号是**3306**。

  

# MySQL基础架构

- MySQL的一个**简要机构图**，客户端的一条**SQL语句**在MySQL内部如何执行
  ![lyx-20241126133524826](attachments/img/lyx-20241126133524826.png)
- MySQL主要由几部分构成
  1. **连接器**：**身份认证**和**权限相关**（登录MySQL的时候）
  2. **查询缓存**：执行**查询**语句的时候，会先**查询缓存**（MySQL8.0版本后**移除**，因为这个功能不太实用）
  3. **分析器**：**没有命中缓存**的话，SQL语句就会经过分析器，分析器说白了就是要先看你的SQL语句**要干嘛**，再检查你的**SQL语句语法**是否正确
  4. **优化器**：按照**MySQL认为最优的方案**去执行
  5. **执行器**：**执行**语句，然后从**存储引擎返回**数据。执行语句之前会**先判断是否有权限**，如果没有权限，就会报错
  6. **插件式存储引擎**：主要负责**数据**的**存储**和**读取**，采用的是**插件式架构**，支持**InnoDB**、**MyISAM**、**Memory**等多种存储引擎

# MySQL存储引擎

MySQL**核心**在于**存储引擎**

## MySQL支持哪些存储引擎？默认使用哪个？

- MySQL支持**多种存储引擎**，可以通过```show engines```命令来**查看MySQL支持的所有存储引擎**
  ![查看 MySQL 提供的所有存储引擎](attachments/img/lyx-20241126133525279.png)

- **默认**存储引擎为InnoDB，并且，所有存储引擎中**只有InnoDB是事务性存储引擎**，也就是说**只有InnoDB支持事务**

- **这里使用MySQL 8.x**
  MySQL 5.5.5之前，MyISAM是MySQL的默认存储引擎；5.5.5之后，InnoDB是MySQL的默认存储引擎，可以通过```select version()```命令查看你的MySQL版本

  ```mysql
  mysql> select version();
  +-----------+
  | version() |
  +-----------+
  | 8.0.27    |
  +-----------+
  1 row in set (0.00 sec) 
  ```

  使用```show variables like %storage_engine%```命令直接查看MySQL**当前默认的存储引擎**    
  ![查看 MySQL 当前默认的存储引擎](attachments/img/lyx-20241126133525810.png)

  如果只想**查看数据库中某个表使用的存储引擎**的话，可以使用```show table status from db_name where name = 'table_name'```命令  
  ![查看表的存储引擎](attachments/img/lyx-20241126133526287.png)

> 如果你想要深入了解每个存储引擎以及它们之间的区别，推荐你去阅读以下 MySQL 官方文档对应的介绍(面试不会问这么细，了解即可)：
>
> - InnoDB 存储引擎详细介绍：https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html 。
> - 其他存储引擎详细介绍：https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html 。
>
> ![img](attachments/img/lyx-20241126133526708.png)

## MySQL存储引擎架构了解吗？

- MySQL 存储引擎采用的是**插件式架构**，支持**多种存储引擎**，我们甚至可以**为不同的数据库表设置不同的存储引擎**以**适应不同场景的需要**。存储引擎是**基于表**的，**而不是数据库**
- 可以**根据 MySQL 定义的存储引擎实现标准接口**来编写一个**属于自己的存储引擎**。这些**非官方提供的存储引擎**可以称为**第三方存储引擎**，**区别于官方存储引擎**

> 像目前最常用的 InnoDB 其实刚开始就是一个第三方存储引擎，后面由于过于优秀，其被 Oracle 直接收购了。
>
> MySQL 官方文档也有介绍到如何编写一个自定义存储引擎，地址：https://dev.mysql.com/doc/internals/en/custom-engine.html 

## MyISAM和InnoDB的区别是什么？

- ISAM全称：**Indexed Sequential Access Method**(索引 顺序 访问 方法)
- 虽然，MyISAM 的性能还行，各种特性也还不错（比如**全文索引**、**压缩**、**空间函数**等）。但是，**MyISAM 不支持事务**和**行级锁**，而且最大的缺陷就是**崩溃后无法安全恢复**

1. **是否支持行级锁**
   **MyISAM** 只有**表级锁(table-level locking)**，而 **InnoDB** 支持**行级锁(row-level locking)**和**表级锁**,**默认为行级锁**。

   > MyISAM 一锁就是锁住了整张表，这在并发写的情况下是多么滴憨憨啊！这也是为什么 InnoDB 在并发写的时候，性能更牛皮了！

2. **是否支持事务**

   MyISAM不支持事务，InnoDB提供事务支持  

   - InnoDB实现了SQL标准，定义了**四个隔离级别**，具有**提交（commit）**和**回滚（rollback）事务**的能力
   - InnoDB默认使用的**REPEATABLE-READ(可重复读)**隔离级别是可以解决**幻读问题发生的（部分幻读）**，基于**MVCC**和**Next-Key Lock（间隙锁）**  

   详细可以查看**MySQL 事务隔离级别详解**

3. **是否支持外键**

   MyISAM不支持，而InnoDB支持

   > 外键对于维护数据一致性非常有帮助，但是对性能有一定的损耗。因此，通常情况下，我们是不建议在实际生产项目中使用外键的，在业务代码中进行约束即可！
   >
   > 阿里的《Java 开发手册》也是明确规定禁止使用外键的。
   >
   > ![img](attachments/img/lyx-20241126133527148.png)
   >
   > 不过，在代码中进行约束的话，对程序员的能力要求更高，具体是否要采用外键还是要根据你的项目实际情况而定

   - 一般我们也是**不建议在数据库层面使用外键**的，**应用层面可以解决**。不过，这样会对数据的一致性造成威胁。具体要不要使用外键还是要根据你的项目来决定

4. **是否支持数据库异常崩溃后的安全恢复**
   MyISAM 不支持，而 InnoDB 支持。  

   - 使用 InnoDB 的数据库在异常崩溃后，数据库重新启动的时候会**保证数据库恢复到崩溃前的状态**。这个**恢复的过程依赖于 `redo log`** 

5. **是否支持MVCC**
   MyISAM 不支持，而 InnoDB 支持。  

   - MyISAM 连行级锁都不支持。**MVCC 可以看作是行级锁的一个升级**，可以有效**减少加锁**操作，**提高性能**。

6. **索引实现不一样**

   - 虽然 **MyISAM 引擎和 InnoDB 引擎都是使用 B+Tree** 作为索引结构，但是两者的**实现方式不太一样**。
   - InnoDB 引擎中，其**数据文件本身就是索引文件**。而 **MyISAM中，索引文件和数据文件是分离**的
   - InnoDB引擎中，表数据文件本身就是按 B+Tree 组织的一个索引结构，**树的叶节点 data 域保存了完整的数据记录**。

详细区别，推荐 ： MySQL 索引详解 

## MyISAM和InnoDB 如何选择

- 大多数时候我们使用的都是 **InnoDB 存储引擎**，在某些**读密集的情况**下，使用 MyISAM 也是合适的。不过，前提是你的项目**不介意 MyISAM 不支持事务**、**崩溃恢复等缺点**（可是~我们一般都会介意啊！）

- 《MySQL 高性能》上面有一句话这样写到:

  > 不要轻易相信“MyISAM 比 InnoDB 快”之类的经验之谈，这个结论往往不是绝对的。在很多我们已知场景中，InnoDB 的速度都可以让 MyISAM 望尘莫及，尤其是用到了聚簇索引，或者需要访问的数据都可以放入内存的应用。

- 一般情况下我们选择 InnoDB 都是没有问题的，但是某些情况下你并不在乎**可扩展能力**和**并发能力**，也不需要**事务支持**，也不在乎**崩溃后的安全恢复问题**的话，选择 MyISAM 也是一个不错的选择。但是一般情况下，我们都是需要考虑到这些问题的。

- 对于咱们日常开发的业务系统来说，你几乎**找不到什么理由再使用 MyISAM** 作为自己的 MySQL 数据库的存储引擎

# MySQL 索引

MySQL 索引相关的问题比较多，对于面试和工作都比较重要，于是，我**单独抽了一篇文章**专门来总结 MySQL 索引相关的知识点和问题： MySQL 索引详解] 

# MySQL查询缓存

执行查询语句的时候，会**先查询缓存**。不过**，MySQL 8.0 版本后移除**，因为这个功能不太实用

- `my.cnf` 加入以下配置，重启 MySQL **开启查询缓存**  

  ```properties
  query_cache_type=1
  query_cache_size=600000
  ```

- 执行以下命令也可以**开启查询缓存**

  ```mysql
  set global  query_cache_type=1;
  set global  query_cache_size=600000;
  ```

**开启查询缓存后在同样的查询条件以及数据情况下，会直接在缓存中返回结果**：  

> 查询条件包括查询本身、当前要查询的数据库、客户端协议版本号等一些可能影响结果的信息

**查询缓存不命中的情况**：  

1. 任何两个查询在**任何字符上的不同**都会导致缓存不命中
2. 如果查询中包含任何**用户自定义函数**、**存储函数**、**用户变量**、**临时表**、**MySQL 库中的系统表**，其查询结果也不会被缓存
3. **缓存建立之后**，MySQL 的**查询缓存系统会跟踪查询中涉及的每张表**，如果**这些表（数据或结构）发生变化**，那么和这张表相关的所有缓存数据都将失效

**缓存虽然能够提升数据库的查询性能，但是缓存同时也带来了额外的开销，每次查询后都要做一次缓存操作，失效后还要销毁。** 因此，开启查询缓存要谨慎，尤其对于写密集的应用来说更是如此。如果开启，要注意合理控制缓存空间大小，一般来说其大小设置为几十 MB 比较合适。此外，**还可以通过 sql_cache 和 sql_no_cache 来控制某个查询语句是否需要缓存  

```sql
select sql_no_cache count(*) from usr;
```

# MySQL事务
## 何谓事务

> 我们设想一个场景，这个场景中我们需要插入**多条相关联**的数据到数据库，不幸的是，这个**过程**可能会遇到下面这些问题：
>
> - 数据库中途突然因为某些原因挂掉了。
> - 客户端突然因为网络原因连接不上数据库了。
> - 并发访问数据库时，多个线程同时写入数据库，覆盖了彼此的更改。
> - ......
>
> 上面的任何一个问题都可能会**导致数据的不一致性**。为了保证数据的一致性，系统必须能够处理这些问题。事务就是我们抽象出来简化这些问题的**首选机制**。**事务的概念起源于数据库**，目前，已经成为一个比较广泛的概念

- 事务是**逻辑上的一组操作**，要么**都执行**，要么**都不执行**

- 最经典的就是**转账**，假如小明要给小红转账1000元，这个转账涉及到**两个关键操作**，这两个操作必须**都成功**或者**都失败**  

  1. 将小明的余额减少1000元
  2. 将小红的余额增加1000元

  事务会把两个操作看成**逻辑上的一个整体**，这个整体包含的操作**要么都成功**，**要么都失败**。这样就不会出现**小明余额减少**而**小红余额却没有增加**的情况  
  ![lyx-20241126133527692](attachments/img/lyx-20241126133527692.png)

## 何谓数据库事务

- 多数情况下，我们谈论事务的时候，如果没有特指**分布式事务**，往往指的是**数据库事务**

- **数据库事务**在日常开发中**接触最多**，如果项目属于**单体架构**，接触的往往就是**数据库事务**

- 数据库事务的作用  
  可以保证**多个对数据库的操作（也就是SQL语句）构成一个逻辑上的整体**，构成这个逻辑上整体的这些数据库操作遵循：**要么全部执行成功，要么全部不执行**  

  ```mysql
  # 开启一个事务
  START TRANSACTION;
  # 多条 SQL 语句
  SQL1,SQL2...
  ## 提交事务
  COMMIT;
  ```

  ![lyx-20241126133528147](attachments/img/lyx-20241126133528147.png)

- **关系型数据库**（比如MySQL、SQLServer、Oracle等）事务都有**ACID**特性  
  ![lyx-20241126133528592](attachments/img/lyx-20241126133528592.png)

  1. **原子性**（`Atomicity`） ： 事务是**最小的执行单位**，**不允许分割**。事务的原子性确保动作**要么全部完成**，要么**完全不起作用**；
  2. **一致性**（`Consistency`）： 执行事务前后，数据保持一致，例如转账业务中，无论事务是否成功，转账者和收款人的总额应该是不变的；（**其实一致性是结果**）
  3. **隔离性**（`Isolation`）： **并发访问数据库**时，一个用户的事务不被其他事务所干扰，**各并发事务之间数据库是独立的**；
  4. **持久性**（`Durability`）： 一个**事务被提交之后**。它**对数据库中数据的改变是持久的**，**即使数据库发生故障**也不应该对其有任何影响。

  **只有保证了事务的持久性、原子性、隔离性之后，一致性才能得到保障。也就是说 A、I、D 是手段，C 是目的！** 想必大家也和我一样，被 ACID 这个概念被误导了很久! 我也是看周志明老师的公开课[《周志明的软件架构课》open in new window](https://time.geekbang.org/opencourse/intro/100064201)才搞清楚的（多看好书！！）  
  ![lyx-20241126133529013](attachments/img/lyx-20241126133529013.png)

- 另外，DDIA 也就是 [《Designing Data-Intensive Application（数据密集型应用系统设计）》open in new window](https://book.douban.com/subject/30329536/) 的作者在他的这本书中如是说：

  > Atomicity, isolation, and durability are properties of the database, whereas consis‐ tency (in the ACID sense) is a property of the application. The application may rely on the database’s atomicity and isolation properties in order to achieve consistency, but it’s not up to the database alone.
  >
  > 翻译过来的意思是：**原子性**，**隔离性**和**持久性**是**数据库的属性**，而**一致性（在 ACID 意义上）是应用程序的属性**。应用可能依赖数据库的原子性和隔离属性来实现一致性，但这并不仅取决于数据库。因此，字母 C 不属于 ACID 《Designing Data-Intensive Application（数据密集型应用系统设计）》这本书强推一波，值得读很多遍！豆瓣有接近 90% 的人看了这本书之后给了五星好评。另外，中文翻译版本已经在 Github 开源，地址：[https://github.com/Vonng/ddiaopen in new window](https://github.com/Vonng/ddia)
  >
  > ![img](attachments/img/lyx-20241126133529502.png)

## 并发事务带来了哪些问题

典型应用程序中，**多个事务并发运行**，经常会**操作相同数据**来完成**各自任务**(多个用户对统一数据进行操作)。**并发**虽然是必须的，但是会导致一下的问题

1. **脏读（Dirty read） **  
   一个事务读取数据并且对数据进行了修改，这个修改对其他事务来说是可见的(**其实就是读未提交**），即使当前事务没有提交。这时另外一个事务读取了这个还未提交的数据，但第一个事务突然回滚，导致**数据并没有被提交到数据库，那第二个事务读取到的就是脏数据**，这也就是脏读的由来。  

   > 例如：事务 1 读取某表中的数据 A=20，事务 1 修改 A=A-1，事务 2 读取到 A = 19,事务 1 回滚导致对 A 的修改并为提交到数据库， A 的值还是 20  
   > ![lyx-20241126133529916](attachments/img/lyx-20241126133529916.png)

2. **丢失修改（Lost to modify）**
   在一个事务读取一个数据时，另外一个事务也访问了该数据，那么**在第一个事务中修改了这个数据后，第二个事务也修改了这个数据**。这样第一个事务内的修改结果就被丢失，因此称为丢失修改。

   > 例如：事务 1 读取某表中的数据 A=20，事务 2 也读取 A=20，事务 1 先修改 A=A-1，事务 2 后来也修改 A=A-1，最终结果 A=19，事务 1 的修改被丢失。
   > (这里例子举得不好，用**事务2进行了A = A - 2 操作**会比较明显)
   >  ![lyx-20241126133530355](attachments/img/lyx-20241126133530355.png)

3. **不可重复读（Unrepeatable read)**  
    指在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，由于**第二个事务的修改导致第一个事务两次读取的数据可能不太一样**。这就发生了在**一个事务内两次读到的数据是不一样**的情况，因此称为**不可重复读**。  

   > 例如：事务 1 读取某表中的数据 A=20，事务 2 也读取 A=20，事务 1 修改 A=A-1，事务 2 再次读取 A =19，此时读取的结果和第一次读取的结果不同。  
   > ![lyx-20241126133530807](attachments/img/lyx-20241126133530807.png)

4. **幻读**  
   幻读与不可重复读类似。它发生在一个事务读取了**几行**数据，接着另一个并发事务**插入**了一些数据时。在随后的查询中，第一个事务就会发现多了一些**原本不存在的记录**，就好像发生了幻觉一样，所以称为**幻读**。  

   > 例如：事务 2 读取某个范围的数据，事务 1 在这个范围插入了新的数据，事务 1 再次读取这个范围的数据发现相比于第一次读取的结果多了新的数据。  
   > ![lyx-20241126133531242](attachments/img/lyx-20241126133531242.png)

## 不可重复读和幻读有什么区别  

- 不可重复读的重点是**内容修改**或者**记录减少**。比如多次读取一条记录发现其中**某些记录的值被修改**；

- 幻读的重点在于**记录新增**比如多次执行同一条查询语句（DQL）时，发现查到的**记录增加**了。

- 幻读其实可以看作是不可重复读的一种特殊情况，单独把区分幻读的原因主要是**解决幻读**和**不可重复读**的方案不一样。

  > 1. 举个例子：执行 `delete` 和 `update` 操作的时候，可以直接对记录加锁，保证事务安全。而执行 `insert` 操作的时候，由于记录锁（Record Lock）只能锁住已经存在的记录，为了避免插入新记录，需要依赖间隙锁（Gap Lock）。也就是说执行 `insert` 操作的时候需要依赖 **Next-Key Lock（Record Lock+Gap Lock）** 进行加锁来**保证不出现幻读**。  (**这里说的是完全解决幻读，其实也可以依靠MVCC部分解决幻读**)
  > 2. 使用MVCC机制（只**在事务第一次select的时候生成ReadView解决不可重复读的问题**）

## SQL标准定义了哪些事务隔离级别

SQL标准定义了**四个隔离级别  **

1. **READ-UNCOMMITTED(读取未提交)** ： 最低的隔离级别，允许读取尚未提交的数据变更，可能会**导致脏读**、**幻读**或**不可重复读**。
2. **READ-COMMITTED(读取已提交)** ： 允许**读取并发事务已经提交**的数据，可以**阻止脏读**，但是**幻读**或不可重复读仍有可能发生。
3. **REPEATABLE-READ(可重复读)** ： 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以**阻止脏读和不可重复读**，但**幻读仍有可能发生**。
4. **SERIALIZABLE(可串行化)** ： 最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以**防止脏读、不可重复读以及幻读**。

|     隔离级别     | 脏读 | 不可重复读 | 幻读 |
| :--------------: | :--: | :--------: | :--: |
| READ-UNCOMMITTED |  √   |     √      |  √   |
|  READ-COMMITTED  |  ×   |     √      |  √   |
| REPEATABLE-READ  |  ×   |     ×      |  √   |
|   SERIALIZABLE   |  ×   |     ×      |  ×   |

## MySQL的隔离级别是基于锁实现的吗

MySQL 的隔离级别基于**锁**和 **MVCC** 机制共同实现的。

- **SERIALIZABLE** 隔离级别，是**通过锁**来实现的。**除了 SERIALIZABLE** 隔离级别，**其他的隔离级别都是基于 MVCC** 实现。
- 不过， SERIALIZABLE 之外的其他隔离级别可能也需要用到锁机制，就比如 **REPEATABLE-READ 在当前读情况下需要使用加锁读来保证不会出现幻读**（这就是MVCC不能解决幻读的例外之一）。

## 上述总结

![lyx-20241126133531671](attachments/img/lyx-20241126133531671.png)

## MySQL的默认隔离级别是什么

MySQL InnoDB 存储引擎的默认支持的隔离级别是 **REPEATABLE-READ（可重读）**。我们可以通过**`SELECT @@tx_isolation;`**命令来查看，MySQL 8.0 该命令改为`SELECT @@transaction_isolation;`

```mysql
mysql> SELECT @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
------ 
```

关于 MySQL 事务隔离级别的详细介绍，可以看看我写的这篇文章： MySQL 事务隔离级别详解 

# MySQL锁
## 表级锁和行级锁了解吗？有什么区别

- MyISAM 仅仅支持**表级锁(table-level locking)**，一锁就**锁整张表**，这在**并发写的情况下性非常差**。
- InnoDB 不光**支持表级锁(table-level locking)**，还**支持行级锁(row-level locking)**，**默认为行级锁**。行级锁的粒度更小，**仅对相关的记录上锁**即可（对一行或者多行记录加锁），所以**对于并发写入**操作来说， **InnoDB 的性能更高**。

**表级锁和行级锁对比** ：

- **表级锁：** MySQL 中**锁定粒度最大的一种锁（全局锁除外）**，是**针对非索引字段**加的锁，对当前操作的**整张表加锁**，实现简单，**资源消耗也比较少**，**加锁快，不会出现死锁**。其锁定粒度最大，**触发锁冲突的概率最高，并发度最低**，MyISAM 和 InnoDB 引擎都支持表级锁。

- **行级锁：** MySQL 中**锁定粒度最小**的一种锁，是**针对索引字段加的锁**，**只针对当前操作的行记录进行加锁**。 行级锁能**大大减少数据库操作的冲突**。其加锁粒度最小，**并发度高**，但**加锁的开销也最大，加锁慢，会出现死锁**。

## 行级锁的使用有什么注意事项

- InnoDB 的行锁是**针对索引**字段加的锁，表级锁是**针对非索引**字段加的锁。

  当我们执行 `UPDATE`、`DELETE` 语句时，**如果 `WHERE`条件中字段没有命中唯一索引**或者**索引失效**的话，就会**导致扫描全表对表中的所有行记录进行加锁**。这个在我们日常工作开发中经常会遇到，一定要多多注意！！！

- 不过，很多时候即使用了索引也有可能会走全表扫描，这是**因为 MySQL 优化器**的原因。

## 共享锁和排他锁

**不论是表级锁还是行级锁**，都存在**共享锁（Share Lock，S 锁）**和**排他锁（Exclusive Lock，X 锁）**这两类  

- **共享锁（S 锁）** ：又称读锁，事务在**读取记录的时候获取共享锁**，允许**多个事务同时获取（锁兼容）**。
- **排他锁（X 锁）** ：又称**写锁/独占锁**，事务在**修改记录的时候获取排他锁**，**不允许多个事务同时获取**。如果一个记录已经被加了排他锁，那其他事务不能再对这条事务加任何类型的锁**（锁不兼容）**。

**排他锁与任何的锁都不兼容，共享锁仅和共享锁兼容。**

|      | S 锁   | X 锁 |
| :--- | :----- | :--- |
| S 锁 | 不冲突 | 冲突 |
| X 锁 | 冲突   | 冲突 |

**由于 MVCC** 的存在，对于**一般的 `SELECT` 语句，InnoDB 不会加任何锁**。不过， 你可以通过以下语句**显式加共享锁**或**排他锁**：  

```mysql
# 共享锁
SELECT ... LOCK IN SHARE MODE;
# 排他锁
SELECT ... FOR UPDATE;
```



## 意向锁有什么作用

★★ **重点** ：**如果需要用到表锁的话，如何判断表中的记录没有行锁**呢？一行一行遍历肯定是不行，性能太差。我们需要用到一个叫做**意向锁**的东东**来快速判断是否可以对某个表使用表锁**。

- **意向锁是表级锁（这句话很重要，意向锁是描述某个表的某个属性（这个表是否有记录加了共享锁/或者排他锁））**，共有两种：
  - **意向共享锁（Intention Shared Lock，IS 锁）**：事务有意向对**表中的某些**记录加共享锁（S 锁），加共享锁前必须**先取得该表的 IS 锁**。
  - **意向排他锁（Intention Exclusive Lock，IX 锁）**：事务有意向对**表中的某些**记录加排他锁（X 锁），加排他锁之前必须**先取得该表的 IX 锁**。

    > 意向锁是有数据引擎自己维护的，用户无法手动操作意向锁，在**为数据行加共享 / 排他锁之前**，InooDB 会**先获取（如果获取到了，其实就是“加了锁”）该数据行所在在数据表的对应意向锁**。
  
- 意向锁之间是互相兼容的 ：  

  > 理由很简单，表里某一条记录加了排他锁（即这个表加了意向排他锁），不代表不能操作其他记录

  |       | IS 锁 | IX 锁 |
  | ----- | ----- | ----- |
  | IS 锁 | 兼容  | 兼容  |
  | IX 锁 | 兼容  | 兼容  |

- 意向锁和共享锁和排它锁互斥（这里指的是**表级别**的共享锁和排他锁，意向锁不会与**行级**的共享锁和排他锁互斥，★★括号里这句话极其重要，要不然就看不懂下面的表了）。  

  |      | IS 锁 | IX 锁 |
  | ---- | ----- | ----- |
  | S 锁 | 兼容  | 互斥  |
  | X 锁 | 互斥  | 互斥  |

  > 《MySQL 技术内幕 InnoDB 存储引擎》这本书对应的描述应该是笔误了。  
  > ![img](attachments/img/lyx-20241126133532110.png)

## InnoDB 有哪几类行锁

**MySQL InnoDB 支持三种行锁定方式**：

- **记录锁（Record Lock）** ：也被称为记录锁，属于**单个行记录上的锁**。
- **间隙锁（Gap Lock）** ：**锁定一个范围，不包括记录本身**。
- **临键锁（Next-key Lock）** ：Record Lock+Gap Lock，**锁定一个范围，包含记录本身**。记录锁只能锁住已经存在的记录，为了避免插入新记录，需要依赖间隙锁。

InnoDB 的**默认隔离级别 RR（可重读）是可以解决幻读**问题发生的，主要有下面两种情况：

- **快照读**（**一致性非锁定读**） ：由 **MVCC 机制**来保证不出现幻读。
- **当前读** （**一致性锁定读**）： 使用 **Next-Key Lock** 进行加锁来保证不出现幻读。

## 当前读和快照读有什么区别

- **快照读**（一致性非锁定读）就是**单纯的 `SELECT` 语句**，但不包括下面这两类 `SELECT` 语句：  

  ```mysql
  SELECT ... FOR UPDATE
  SELECT ... LOCK IN SHARE MODE
  ```

  快照即**记录的历史版本**，每行记录可能存在**多个历史版本**（多版本技术）。

  快照读的情况下，**如果读取的记录正在执行 UPDATE/DELETE 操作，读取操作不会因此去等待记录上 X 锁的释放，而是会去读取行的一个快照**。

  只有在**事务隔离级别 RC(读取已提交，ReadCommit)** 和 **RR（可重读，RepeatableCommit）**下，InnoDB 才会使用一致性非锁定读：

  - 在 RC 级别下，对于快照数据，一致性非锁定读**总是读取被锁定行的最新一份（可见）快照**数据。
  - 在 RR 级别下，对于快照数据，一致性非锁定读总是读取**本事务开始时的行数据版本**。

  快照读比较适合对于**数据一致性要求不是特别高**且**追求极致性能**的业务场景。

- **当前读** （一致性锁定读）就是**给行记录加 X 锁**或 **S 锁**。（使用当前读的话在RR级别下就无法解决幻读）

  当前读的一些常见 SQL 语句类型如下：  

  ```mysql
  # 对读的记录加一个X锁
  SELECT...FOR UPDATE
  # 对读的记录加一个S锁
  SELECT...LOCK IN SHARE MODE
  # 对修改的记录加一个X锁
  INSERT...
  UPDATE...
  DELETE... 
  ```

# MySQL 性能优化

关于 MySQL 性能优化的建议总结，请看这篇文章： MySQL 高性能优化规范建议总结 

## 能用MySQL直接存储文件（比如图片）吗

- 可以是可以，直接存储文件对应的**二进制数据**即可。不过，还是建议不要在数据库中存储文件，会**严重影响数据库性能**，**消耗过多存储空间**。

- **数据库只存储文件地址信息，文件由文件存储服务负责存储。**

  - 可以选择使用**云服务厂商提供的开箱即用的文件存储服务**，成熟稳定，价格也比较低。
    ![img](attachments/img/lyx-20241126133532565.png)

  - 也可以选择**自建文件存储服务**，实现起来也不难，基于 **FastDFS**、**MinIO（推荐）** 等开源项目就可以实现**分布式文件服务**。

    > 相关阅读：[Spring Boot 整合 MinIO 实现分布式文件服务](https://www.51cto.com/article/716978.html)

## MySQL如何存储IP 地址

- 可以**将 IP 地址转换成整形数据**存储，**性能更好**，**占用空间也更小**。

- MySQL 提供了两个方法来**处理 ip 地址**
  - `INET_ATON()` ： 把 **ip 转为无符号整型** (4-8 位)
  - `INET_NTOA()` :把**整型的 ip 转为地址**

- **插入数据前**，先用 `INET_ATON()` 把 ip 地址转为整型，**显示数据时**，使用 `INET_NTOA()` 把整型的 ip 地址转为地址显示即可

## 有哪些常见的SQL优化手段吗

>  [《Java 面试指北》open in new window](https://www.yuque.com/docs/share/f37fc804-bfe6-4b0d-b373-9c462188fec7) 的「技术面试题篇」有一篇文章详细介绍了常见的 SQL 优化手段，非常全面，清晰易懂！
>
> ![常见的 SQL 优化手段](attachments/img/lyx-20241126133532990.png)

