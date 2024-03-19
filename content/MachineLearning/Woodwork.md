---
aliases: 
date created: 四月 15日 2023, 9:49:21 上午
date modified: 三月 5日 2024, 4:07:12 下午
title: Woodwork Dataframe
tags: [code/machine-learning]
---

# Woodwork Dataframe
在 Woodwork Dataframe 中，每一列有三种主要内容组成。
- 物理类型 Physical Type：定义数据如何存储在磁盘或内存中。
- 逻辑类型 Logical Type：定义如何解析或解释数据。
- 语义标签 Semantic Tag：提供有关数据含义或应如何使用数据的额外数据。
如果不手动填充 Logical Type 的话，Woodwork 会自动推断。

## [Semantic Tag](https://woodwork.alteryx.com/en/stable/guides/logical_types_and_semantic_tags.html#Semantic-Tags)

## [Logical Type](https://woodwork.alteryx.com/en/stable/guides/logical_types_and_semantic_tags.html#Logical-Types)
### 指定 Logical Type 方法
```python
df.ww.init(
    logical_types={
        "latlongs": "LatLong",
        "dates": ww.logical_types.Datetime(datetime_format="%Y/%m/%d"),
    }
)
strings_df.ww.init(
    logical_types={
        "natural_language": "NaturalLanguage",
        "addresses": "Address",
        "filepaths": "FilePath",
        "full_names": "PersonFullName",
        "phone_numbers": "PhoneNumber",
        "urls": "URL",
        "ip_addresses": "IPAddress",
    }
)
```

## ColumnSchema objects
### schema 和 dataframe 的区别
```python
# Woodwork typing info for a DataFrame
retail_df.ww
# A Woodwork TableSchema
retail_df.ww.schema
```
Woodwork Dataframe 和 TableSchema 的唯一区别就在于：Dataframe 有 Physical Type 而 TableSchema 没有。

## 检查 Logical Type 是否能够为空值
```python
df.ww["bools_nullable"].ww.nullable
# 其返回值就是是否可以
```