---
aliases: 
tags: [code/big-data, code/database]
title: Rollup
date created: 2024-05-16 17:05:00
date modified: 2024-05-21 10:05:86
---
## Rollup上卷
ROLLUP 在多维分析中是“上卷”的意思，即将数据按某种指定的粒度进行进一步聚合(**从细粒度到粗粒度**)。

### 基本概念
在 Doris 中，我们将用户通过建表语句创建出来的表称为 Base 表（Base Table）。Base 表中保存着按用户建表语句指定的方式存储的基础数据。

在 Base 表之上，我们可以创建任意多个 ROLLUP 表。这些 ROLLUP 的数据是基于 Base 表产生的，并且在**物理上是独立存储**的。

ROLLUP 表的基本作用，在于在 Base 表的基础上，获得**更粗粒度**的聚合数据。

### Roll使用
#### Aggregate 和 Unique 模型中的 ROLLUP
比如需要查看某个用户的总消费，那么可以建立一个只有user_id和cost的rollup
##### 创建Rollup
```sql
alter table example_site_visit add rollup rollup_cost_userid(user_id,cost);
```
##### 查询Rollup
```sql
--直接查询即可，这里是用来查看执行计划
explain SELECT user_id, sum(cost) FROM example_site_visit GROUP BY user_id;
```
![图片1.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/%E5%9B%BE%E7%89%871.png)

#### Duplicate 模型中的 ROLLUP
因为 Duplicate 模型没有聚合的语意。所以该模型中的 ROLLUP，已经失去了“上卷”这一层含义。而仅仅是作为调整列顺序，以命中**前缀索引**的作用。下面详细介绍前缀索引，以及如何使用ROLLUP改变前缀索引，以获得更好的查询效率。

##### 前缀索引
不同于传统的数据库设计，Doris 不支持在任意列上创建索引。Doris 这类 MPP 架构的 OLAP 数据库，通常都是通过提高并发，来处理大量数据的。

本质上，Doris 的数据存储在类似 SSTable（Sorted String Table）的数据结构中。该结构是一种有序的数据结构，可以**按照指定的列进行排序存储**。在这种数据结构上，**以排序列作为条件进行查找，会非常的高效**。

在 Aggregate、Unique 和 Duplicate 三种数据模型中。底层的数据存储，是按照各自建表语句中，**AGGREGATE KEY、UNIQUE KEY 和 DUPLICATE KEY 中指定的列**进行**排序存储**的。

而前缀索引，即在**排序的基础上**，实现的一种根据给定前缀列，**快速查询数据**的索引方式。
##### Rollup 调整前缀索引
因为建表时已经指定了列顺序，所以一个表只有一种前缀索引。这对于使用其他不能命中前缀索引的列作为条件进行的查询来说，效率上可能无法满足需求。因此，我们可以通过创建 ROLLUP 来人为的调整列顺序。举例说明。
**Base 表结构如下：**

| ColumnName     | Type         |
| -------------- | ------------ |
| **user_id**    | BIGINT       |
| **age**        | INT          |
| message        | VARCHAR(100) |
| max_dwell_time | DATETIME     |
| min_dwell_time | DATETIME     |

**我们可以在此基础上创建一个 ROLLUP 表：**

| ColumnName     | Type         |
| -------------- | ------------ |
| **age**        | INT          |
| **user_id**    | BIGINT       |
| message        | VARCHAR(100) |
| max_dwell_time | DATETIME     |
| min_dwell_time | DATETIME     |

可以看到，ROLLUP和Base表的列完全一样，只是将**user_id和age的顺序调换**了。那么当我们进行如下查询时：

```sql
SELECT * FROM table where age=20 and message LIKE "%error%";
```

会优先选择ROLLUP 表，因为ROLLUP的前缀索引匹配度更高。