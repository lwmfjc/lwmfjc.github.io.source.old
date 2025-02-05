---
title: 数据库基础
description: 数据库基础
categories:
  - 学习
tags:
  - 复习
  - 复习-javaGuide
  - 复习-javaGuide-database
date: 2022-12-20 11:19:14
updated: 2023-01-10 13:19:14
---

> 转载自https://github.com/Snailclimb/JavaGuide（添加小部分笔记）感谢作者!

> 这部分内容由于涉及太多概念性内容，所以参考了维基百科和百度百科相应的介绍。

# 什么是数据库，数据库管理系统，数据库系统，数据库管理员

- **数据库**：数据库（**DataBase 简称DB**）就是**信息的集合**或者说**数据库管理系统管理的数据的集合**。
- **数据库管理系统**：数据库管理系统（**Database Management System 简称DBMS**）是一种**操纵和管理数据库**的大型软件，通常用于建立、使用和维护 **数据库**。
- **数据库系统（范围最大）**：数据库系统（Data Base System，简称DBS）通常由**软件、数据和数据管理员（DBA）**组成。
- **数据库管理员**：数据库管理员（Database Adminitrator，简称DBA）负责全面**管理和控制**数据库系统 **(是一个人)**

**数据库系统基本构成**如下图所示  
![lyx-20241126133445802](attachments/img/lyx-20241126133445802.png)

# 什么是元组，码，候选码，主码，外码，主属性，非主属性

- **元组**：元组（tuple）是**关系数据库**中的**基本概念**，**关系**是一张表，表中的**每行**（即数据库中的每条**记录**）就是一个元组，每列就是一个属性。在**二维表**里，元组也成为**行**
- **码**：码就是能**唯一标识实体**的属性，对应表中的**列**
- **候选码**：若关系中的**某一属性**或**属性组的值**能**唯一的标识一个元组**，而**其任何、子集都不能再标识**，则称该**属性组**为**候选码**。例如：在学生实体中，**“学号”**是能唯一的区分学生实体的，同时又假设**“姓名”、“班级”的属性组合**足以区分学生实体，那么**{学号}**和**{姓名，班级}**都是**候选码**。
- **主码**：主码也叫**主键**，主码是**从候选码**中选出来的。一个实体集中只能有**一个主码**，但可以有**多个候选码**
- **外码**：外码也叫**外键**。如果**一个关系中的一个属性**是**另外一个关系中的主码**则这个属性为外码。
- **主属性** ： **候选码中出现过的属性**称为主属性(**这里强调单个**）。比如关系 工人（工号，身份证号，姓名，性别，部门）. 显然**工号和身份证号**都能够唯一标示这个关系，所以都是候选码。**工号、身份证号这两个属性就是主属性**。如果主码是一个属性组，那么属性组中的属性都是主属性。
- **非主属性：** **不包含在任何一个候选码中的属性**称为非主属性。比如在关系——学生（学号，姓名，年龄，性别，班级）中，主码是“学号”，那么其他的“姓名”、“年龄”、“性别”、“班级”就都可以称为非主属性。

# 主键和外键有什么区别

- **主键(主码)** ：主键用于**唯一标识一个元组**，不能有重复，不允许为空。一个表只能有一个主键。
- **外键(外码)** ：外键用来和其他表建立联系用，**外键是另一表的主键**，外键是可以有重复的，可以是空值。一个表可以有多个外键

# 为什么不推荐使用外键与级联

- 对于外键和级联，阿里巴巴开发手册这样说道

  > 【强制】不得使用外键与级联，一切**外键概念**必须在应用层解决。
  >
  > 说明: 以学生和成绩的关系为例，学生表中的 student_id 是主键，那么成绩表中的 student_id 则为外键。如果更新学生表中的 student_id，同时触发成绩表中的 student_id 更新，即为级联更新。
  >
  > 缺点： **外键与级联更新适用于单机低并发，不适合分布式、高并发集群; 级联更新是强阻塞，存在数据库更新风暴的风 险; 外键影响数据库的插入速度**

- 为什么不要使用外键

  1. 增加了复杂性

     > a. 每次做DELETE 或者UPDATE都必须考虑外键约束，会导致开发的时候很痛苦, **测试数据极为不方便**; b. 外键的主从关系是定的，假如那天需求有变化，**数据库中的这个字段根本不需要和其他表有关联的话就会增加很多麻烦**。

  2. 增加了额外操作

     > 数据库需要增加维护外键的工作，比如当我们做一些涉及外键字段的增，删，更新操作之后，需要触发相关操作去检查，保证数据的的一致性和正确性，这样会不得不消耗资源；（**个人觉得这个不是不用外键的原因**，因为即使你不使用外键，你在应用层面也还是要保证的。所以，我觉得这个影响可以忽略不计。）

  3. 对分库分表很不友好：因为分库分表下外键无法生效

  4. ...

- 外键的一些好处

  1. 保证了**数据库数据的一致性和完整性**；
  2. 级联操作方便，**减轻了程序代码量**；
  3. ......

  **如果系统不涉及分库分表**，**并发量不是很高**的情况还是可以考虑使用外键的

# 什么是ER图

> 做一个项目的时候一定要试着**画 ER 图来捋清数据库设计**，这个也是面试官问你项目的时候经常会被问道的。

- E-R图，也称 **实体-联系图（Entity Relationship Diagram）**，提供表示**实体类型**、**属性**和**关系**，用来描述现实世界的**概念模型**。它是描述**现实世界关系概念模型**的有效方法，是**表示概念关系模型**的一种方式
- 下图是一个学生选课的 ER 图，**每个学生可以选若干门课程**，**同一门课程也可以被若干人选择**，所以它们之间的关系是**多对多（M: N）**。另外，还有**其他两种关系**是：**1 对 1（1:1）**、**1 对多（1: N）**
  ![lyx-20241126133446325](attachments/img/lyx-20241126133446325.png)
  将**ER图**转换成**数据库实际的关系模型**（实际设计中，我们通常会将任课教师也作为一个实体来处理）  
  ![lyx-20241126133446796](attachments/img/lyx-20241126133446796.png)

# 数据库范式了解吗

1. **1NF(第一范式)**
   属性（对应于表中的字段）不能再被分割，也就是这个字段只能是一个值，**不能再分为多个其他的字段**了。**1NF 是所有关系型数据库的最基本要求** ，也就是说关系型数据库中创建的表一定满足第一范式。

2. **2NF(第二范式)**
   2NF 在 1NF 的基础之上，**消除了非主属性对于码的部分函数依赖**。如下图所示，展示了第一范式到第二范式的过渡。第二范式在第一范式的基础上增加了一个列，这个列称为主键，非主属性都依赖于主键。

   >第二范式要求，在满足第二范式的基础上，还要满足数据表里得每一条数据记录，都是可唯一标识的。而且所有**非主键字段**，都必须完全依赖主键，不能只依赖主键的一部分，如下，主键为商品名称、供应商名称，是主码是**属性组**。而供应商电话只依赖于供应商id，商品价格只依赖于价格。所以不满足第二范式


   ![lyx-20241126133447298](attachments/img/lyx-20241126133447298.png)

3. **3NF(第三范式)**3NF 在 2NF 的基础之上，消除了**非主属性对于码的传递函数依赖** 。符合 3NF 要求的数据库设计，**基本**上解决了数据冗余过大，插入异常，修改异常，删除异常的问题。比如在关系 R(学号 , 姓名, 系名，系主任)中，学号 → 系名，系名 → 系主任，所以存在非主属性系主任对于学号的传递函数依赖，所以该表的设计，不符合 3NF 的要求。

   > 确保数据表中的每一个非主键字段都和主键字段相关，也就是说，要求数据表中的所有**非主键字段**不能依赖于其他**非主键字段**。（即，不能存在非主属性A依赖于非主属性B，非主属性B依赖于主键C的情况，即存在A --> B --> C 的决定关系），规则的意思是**所有非主键属性之间**不能有依赖关系，必须互相独立
   >
   > > 简单举例：   
   > > 部门信息表：每个部门有**部门编号(dept_id)**、**部门名称**、**部门简介**等消息  
   > > 员工信息表：每个员工有**员工编号**、**姓名**、**部门编号**。（注意，列出部门编号就不能再将部门名称、部门简介等部门相关的信息再加入员工信息表中，否则将不满足第3范式（但其实是满足第二范式的））

总结  

- 1NF：属性**不可再分**。
- 2NF：1NF 的基础之上，**消除了非主属性对于码的部分函数依赖**。
- 3NF：3NF 在 2NF 的基础之上，消除了**非主属性对于码的传递函数依赖** 。

> 一些概念：  
>
> - **函数依赖（functional dependency）** ：若在一张表中，在属性（或属性组）X 的值确定的情况下，必定能确定属性 Y 的值，那么就可以说 Y 函数依赖于 X，写作 X → Y。
> - **部分函数依赖（partial functional dependency）** ：如果 X→Y，并且存在 X 的一个真子集 X0，使得 X0→Y，则称 Y 对 X 部分函数依赖。比如学生基本信息表 R 中（学号，身份证号，姓名）当然学号属性取值是唯一的，在 R 关系中，（学号，身份证号）->（姓名），（学号）->（姓名），（身份证号）->（姓名）；所以**姓名部分函数依赖与（学号，身份证号）**；**（感觉这个例子虽然是对的，但是不利于理解第二范式）**
> - **完全函数依赖(Full functional dependency)** ：在一个关系中，若某个非主属性数据项依赖于全部关键字称之为完全函数依赖。比如学生基本信息表 R（学号，班级，姓名）假设不同的班级学号有相同的，班级内学号不能相同，在 R 关系中，（学号，班级）->（姓名），但是（学号）->(姓名)不成立，（班级）->(姓名)不成立，所以姓名完全函数依赖与（学号，班级）；
> - **传递函数依赖** ： 在关系模式 R(U)中，设 X，Y，Z 是 U 的不同的属性子集，如果 X 确定 Y、Y 确定 Z，且有 X 不包含 Y，Y 不确定 X，（X∪Y）∩Z=空集合，则称 Z 传递函数依赖(transitive functional dependency) 于 X。传递函数依赖会导致数据冗余和异常。传递函数依赖的 Y 和 Z 子集往往同属于某一个事物，因此可将其合并放到一个表中。比如在关系 R(学号 , 姓名, 系名，系主任)中，学号 → 系名，系名 → 系主任，所以存在非主属性系主任对于学号的传递函数依赖。。

# 什么是存储过程

- 作用：我们可以把存储过程看成是一些 **SQL 语句的集合**，中间加了点逻辑控制语句。存储过程在业**务比较复杂的时候是非常实用**的，比如很多时候我们完成一个操作可能需要写一大串 SQL 语句，这时候我们就可以写有一个存储过程，这样也方便了我们下一次的调用。**存储过程一旦调试完成通过后就能稳定运行**，另外，使用**存储过程比单纯 SQL 语句执行要快**，因为**存储过程是预编译过**的。

- 存储过程在互联网公司应用不多，因为存储过程**难以调试**和**扩展**，而且**没有移植性**，还会消耗**数据库资源**

  > 阿里巴巴Java开发手册要求禁止使用存储过程
  > ![阿里巴巴Java开发手册: 禁止存储过程](attachments/img/lyx-20241126133447819.png)

# drop、delete与truncate区别

## 用法不同

- drop(丢弃数据): `drop table 表名` ，直接**将表都删除掉**，在删除表的时候使用。
- truncate (清空数据) : `truncate table 表名` ，只**删除表中的数据**，再**插入数据的时候自增长 id 又从 1 开始**，在清空表中数据的时候使用。
- delete（删除数据） : `delete from 表名 where 列名=值`，**删除某一行的数据，如果不加 where 子句和`truncate table 表名`作用类似**。

truncate 和不带 where 子句的 delete、以及 drop 都会删除表内的数据，但是 **truncate 和 delete 只删除数据不删除表的结构(定义)，执行 drop 语句，此表的结构也会删除，也就是执行 drop 之后对应的表不复存在。**

## 属于不同的数据库语言

- **truncate 和 drop 属于 DDL(数据定义语言)语句，操作立即生效**，原数据**不放到 rollback segment** 中，**不能回滚**，操作**不触发 trigger**。而 **delete 语句是 DML (数据库操作语言)**语句，这个操作会放到 **rollback segement** 中，**事务提交之后才生效**。
- DML语句和DDL语句区别
  1. DML 是**数据库操作语言（Data Manipulation Language）**的缩写，是指对数据库中表记录的操作，主要包括表记录的插入（insert）、更新（update）、删除（delete）和查询（select），是开发人员日常使用最频繁的操作。
  2. **DDL （Data Definition Language）**是数据定义语言的缩写，简单来说，就是对数据库内部的对象进行创建、删除、修改的操作语言。它和 DML 语言的最大区别是 DML 只是对表内部数据的操作，而不涉及到表的定义、结构的修改，更不会涉及到其他对象。**DDL 语句更多的被数据库管理员（DBA）所使用，一般的开发人员很少使用**。
- 由于`select`不会对表进行破坏，所以有的地方也会把`select`单独区分开叫做数据库查询语言**DQL（Data Query Language）**

## 执行速度不同

- 一般来说：drop > truncate > delete（这个我没有设计测试过）

  > `delete`命令执行的时候会产生数据库的`binlog`日志，而日志记录是需要消耗时间的，但是也有个好处方便数据回滚恢复。
  >
  > `truncate`命令执行的时候不会产生数据库日志，因此比`delete`要快。除此之外，还会把表的自增值重置和索引恢复到初始大小等。
  >
  > `drop`命令会把表占用的空间全部释放掉。
  >
  > Tips：你应该更多地关注在使用场景上，而不是执行效率。

# 数据库设计通常分为哪几步

1. **需求分析** : 分析用户的需求，包括数据、功能和性能需求。
2. **概念结构设计** : 主要采用 E-R 模型进行设计，包括画 E-R 图。
3. **逻辑结构设计** : 通过将 E-R 图转换成表，实现从 E-R 模型到关系模型的转换。
4. **物理结构设计** : 主要是为所设计的数据库选择合适的存储结构和存取路径。
5. **数据库实施** : 包括编程、测试和试运行
6. **数据库的运行和维护** : 系统的运行与数据库的日常维护。