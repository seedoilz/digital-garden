---
aliases: 
title: Doris分区分桶
date created: 2024-05-16 16:05:00
date modified: 2024-05-16 17:05:89
---
---
Doris支持两层的数据划分。第一层是 Partition，支持 Range和List的划分方式。第二层是 Bucket（Tablet），仅支持Hash的划分方式。
## 分区（Partition）
Partition列可以指定一列或多列。分区列必须为KEY列。多列分区的使用方式在后面介绍。
不论分区列是什么类型，在写分区值时，都需要加双引号。
分区数量理论上没有上限。
当不使用Partition建表时，系统会自动生成一个和表名同名的，全值范围的 Partition。该Partition对用户不可见，并且不可删改。
创建分区时不可添加范围重叠的分区。

### Range分区
分区列通常为**时间列**( PARTITION BY RANGE(`date`) )，以方便的管理新旧数据。
VALUES LESS THAN (...) 仅指定上界，系统会将前一个分区的上界作为该分区的下界，生成一个左闭右开的区间。
VALUES \[...) 指定上下界，生成一个左闭右开的区间。
```sql
CREATE TABLE IF NOT EXISTS students (
    student_id INT,
    name STRING,
    age INT,
    score DOUBLE
)
DUPLICATE KEY(student_id)
DISTRIBUTED BY HASH(student_id) BUCKETS 10;

ALTER TABLE students
ADD PARTITION p1 VALUES LESS THAN ("60"),
ADD PARTITION p2 VALUES LESS THAN ("70"),
ADD PARTITION p3 VALUES LESS THAN ("80"),
ADD PARTITION p4 VALUES LESS THAN ("90"),
ADD PARTITION p5 VALUES LESS THAN ("100"),
ADD PARTITION p6 VALUES LESS THAN ("MAXVALUE");
```

### List分区
分区列支持 BOOLEAN, TINYINT, SMALLINT, INT, BIGINT, LARGEINT, DATE, DATETIME, CHAR, VARCHAR 数据类型，分区值为枚举值。**只有当数据为目标分区枚举值其中之一时，才可以命中分区**
```sql
CREATE TABLE IF NOT EXISTS employees (
    employee_id INT,
    name STRING,
    age INT,
    department STRING,
    salary DOUBLE
)
DUPLICATE KEY(employee_id)
DISTRIBUTED BY HASH(employee_id) BUCKETS 10;

ALTER TABLE employees
ADD PARTITION p_sales VALUES IN ("Sales"),
ADD PARTITION p_engineering VALUES IN ("Engineering"),
ADD PARTITION p_hr VALUES IN ("HR"),
ADD PARTITION p_marketing VALUES IN ("Marketing"),
ADD PARTITION p_admin VALUES IN ("Admin"),
ADD PARTITION p_others VALUES IN ("Finance", "Legal", "Support");
```

## 分桶
- 如果**使用了 Partition**，则 DISTRIBUTED ... 语句描述的是数据在**各个分区内**的划分规则。如果**不使用 Partition**，则描述的是对**整个表**的数据的划分规则。
- 分桶列可以是多列，但**必须为 Key** 列。分桶列可以和 Partition 列相同或不同。
- 分桶的数量理论上没有上限。
- 分桶列的选择，是在查询吞吐和查询并发之间的一种权衡：
	- 如果选择多个分桶列，则数据分布更均匀。如果一个查询条件不包含所有分桶列的等值条件，那么该查询会触发所有分桶同时扫描，这样查询的吞吐会增加，单个查询的延迟随之降低。这个方式适合大吞吐低并发的查询场景。
	- 如果仅选择一个或少数分桶列，则对应的点查询可以仅触发一个分桶扫描。此时，当多个点查询并发时，这些查询有较大的概率分别触发不同的分桶扫描，各个查询之间的IO影响较小（尤其当不同桶分布在不同磁盘上时），所以这种方式适合高并发的点查询场景。
```sql
DISTRIBUTED BY HASH(`user_id`) BUCKETS 16
```

## 动态分区
动态分区是在 Doris 0.12 版本中引入的新功能。旨在对表级别的分区实现生命周期管理(TTL)，减少用户的使用负担。
目前实现了**动态添加**分区及**动态删除**分区的功能。
动态分区只**支持 Range 分区**。

### 原理
在某些使用场景下，用户会将表按照天进行分区划分，每天定时执行例行任务，这时需要使用方**手动管理分区**，否则可能由于使用方没有创建分区导致数据导入失败，这给使用方带来了额外的维护成本。

通过动态分区功能，用户可以在建表时设定动态分区的规则。**FE 会启动一个后台线程，根据用户指定的规则创建或删除分区**。用户也可以在运行时对现有规则进行变更。

### 使用方式
动态分区的规则可以在建表时指定，或者在运行时进行修改。当前仅支持对**单分区列**的分区表设定动态分区规则。

#### 建表时指定
```sql
CREATE TABLE tbl1
(...)
PROPERTIES
(
    "dynamic_partition.prop1" = "value1",
    "dynamic_partition.prop2" = "value2",
    ...
)
```

#### 运行时修改
```sql
ALTER TABLE tbl1 SET
(
    "dynamic_partition.prop1" = "value1",
    "dynamic_partition.prop2" = "value2",
    ...
)
```

#### 示例
```sql
create table student_dynamic_partition1
(
id int,
time date,
name varchar(50),
age int
)
duplicate key(id,time)
PARTITION BY RANGE(time)()
distributed by hash(`id`)
PROPERTIES(
"dynamic_partition.enable" = "true",
"dynamic_partition.time_unit" = "DAY",
"dynamic_partition.create_history_partition" = "true",
"dynamic_partition.history_partition_num" = "3",
"dynamic_partition.start" = "-7",
"dynamic_partition.end" = "3",
"dynamic_partition.prefix" = "p",
"dynamic_partition.buckets" = "10",
 "replication_num" = "1"
 );
```