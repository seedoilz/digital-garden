---
aliases: [xgb]
date created: 四月 18日 2023, 5:08:41 下午
date modified: 三月 5日 2024, 4:07:12 下午
title: XGBoost
tags: [code/machine-learning]
---

## 原理
本质基于[[决策树]]

## 使用
### 引入基本库
```python
import numpy as np
import pandas as pd 
import xgboost as xgb
```

### 数据预处理
使用这些库进行数据的处理，形成 X-Y 的对应关系。X 可以为集合。
```python
dtrain=xgb.DMatrix(train_x,label=train_y)  
dtest=xgb.DMatrix(test_x)  
watchlist = [(dtrain,'train')]
```

### 模型参数
#### booster
默认为 gbtree，一般也最好选择 gbtree 。
1. gbtree（树模型）
2. gblinear（线性模型，适合高维度数据）
3. dart（树模型，引入随机性，防止过拟合，因此在某些情况下可以提高模型的泛化性能）

#### objective
1. Reg: squarederror       均方误差
2. reg: logistic           对数几率损失，参考对数几率回归 (逻辑回归)
3. binary: logistic        二分类对数几率回归，输出概率值
4. binary: hinge           二分类合页损失，此时不输出概率值，而是 0 或 1
5. multi: softmax          多分类 softmax 损失，此时需要设置 num_class 参数

#### max_depth
树的深度，**直接关系到了模型的复杂度以及过拟合等问题**。

#### min_child_weight
最小的叶子节点权重。
在普通的 GBM 中，叶子节点样本没有权重的概念，其实就是等权重的，也就相当于叶子节点样本个数。越小越没有限制，容易过拟合，太高容易欠拟合。

#### eval_metric
1. Rmse : root mean square error     也就是平方误差和开根号
2. mae  : mean absolute error        误差的绝对值再求平均
3. auc  : area under curve           roc 曲线下面积
4. aucpr: area under the pr curve    pr 曲线下面积
