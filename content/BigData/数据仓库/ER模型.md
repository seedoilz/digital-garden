---
aliases: 
title: ER模型
date created: 2024-05-01 21:05:00
date modified: 2024-05-01 21:05:46
tags: [code/database, code/big-data]
---
数据仓库之父Bill Inmon提出的建模方法是从全企业的高度，用实体关系（Entity Relationship，ER）模型来描述企业业务，并用规范化的方式表示出来，在范式理论上符合3NF。

## 实体关系模型

实体关系模型将复杂的数据抽象为两个概念——实体和关系。实体表示一个对象，例如学生、班级，关系是指两个实体之间的关系，例如学生和班级之间的从属关系。

## 数据库规范化

数据库规范化是使用一系列范式设计数据库（通常是关系型数据库）的过程，其目的是减少数据冗余，增强数据的一致性。

这一系列范式就是指在设计关系型数据库时，需要遵从的不同的规范。关系型数据库的范式一共有六种，分别是第一范式（1NF）、第二范式（2NF）、第三范式（3NF）、巴斯-科德范式（BCNF）、第四范式(4NF）和第五范式（5NF）。遵循的范式级别越高，数据冗余性就越低。

## 三范式

### 第一范式
![CleanShot 2024-05-01 at 21.27.41.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-05-01%20at%2021.27.41.png)

### 第二范式
![CleanShot 2024-05-01 at 21.28.23.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-05-01%20at%2021.28.23.png)

### 第三范式
![CleanShot 2024-05-01 at 21.28.46.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-05-01%20at%2021.28.46.png)

### 函数依赖
![CleanShot 2024-05-01 at 21.29.21.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-05-01%20at%2021.29.21.png)

