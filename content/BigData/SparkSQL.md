---
aliases: 
title: SparkSQL
date created: 2024-04-07 11:04:00
date modified: 2024-04-07 11:04:24
tags: [code/big-data]
---
## RDD、DataFrame、DataSet 三者的关系
### 三者的共性
- 本质上都属于RDD
- 三者都有partition 的概念

### 三者的区别
1. RDD
	1. RDD 一般和 spark mllib 同时使用
	2. RDD 不支持 sparksql 操作
	3. RDD 不支持 sparksql 操作
2. DataFrame
	1. 与RDD 和 Dataset 不同，DataFrame 每一行的类型固定为Row，每一列的值没法直接访问，只有通过解析才能获取各个字段的值
	2. DataFrame 与DataSet 均支持 SparkSQL 的操作，比如select，groupby 之类，还能注册临时表/视窗，进行 sql 语句操作
	3. DataFrame 也可以叫Dataset\[Row\],每一行的类型是 Row，不解析，每一行究竟有哪些字段，各个字段又是什么类型都无从得知

## UDF
