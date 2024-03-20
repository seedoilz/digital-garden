---
aliases: 
date created: 2023-04-14 16:04:00
date modified: 2024-03-20 11:03:14
title: EntitySet
tags: [code/machine-learning]
---

***（目前基于 [[Featuretools]]）***
一个 [[EntitySet]] 是 dataframes 及其关系的集合。

#### 原始数据的处理
首先初始化一个 [[EntitySet]]，并且可以提供一个 id
```python
es = ft.EntitySet(id="customer_data")
```

#### 添加数据帧
利用 `add_dataframe`，向 [[EntitySet]] 中添加 dataframe。
其中有三个重要的参数需要指定。
-   The `index` parameter specifies the column that uniquely identifies rows in the dataframe.
-   The `time_index` parameter tells [[Featuretools]] when the data was created.
-   The `logical_types` parameter indicates that “product_id” should be interpreted as a Categorical column, even though it is just an integer in the underlying data.

add_dataframe 方法将数据框架中的每列与 [[Woodwork]] (special dataframe) Logical Type 相关联。每个逻辑类型都可以有一个相关的标准语义标签，以帮助定义列数据类型。如果您没有为列指定逻辑类型，则会根据基础数据推断出来。

对于每一个 [[EntitySet]] 可以添加多个 dataframe。

#### 添加关系
我们希望通过每个数据框架中名为“product_id”的列将这两个数据帧联系起来。每个产品都有多个与之关联的事务，因此它被称为**父数据框架**，而事务数据框架被称为**子数据框架**。指定关系时，我们需要四个参数：<font color="#9bbb59">父数据帧名称、父列名称、子数据帧名称和子列名称</font>。请注意，每个关系必须表示一对多的关系，而不是一对一或多对多的关系。

#### 规范化
在 [[Featuretools]] 中，重复实体是指数据表中的多个行表示相同的实体（具有相同的实体标识符）。这种情况通常发生在实体之间存在多对多关系时，例如一个订单可以包含多个商品，而每个商品又可以出现在多个订单中。
`normalize_dataframe()` 函数可以将这样的数据表进行规范化，即将包含相同实体的多个行分割成独立的数据表，并创建新的实体和关系。这样，我们就可以更好地利用 [[Featuretools]] 中自动生成的特征，以便更好地进行机器学习建模。