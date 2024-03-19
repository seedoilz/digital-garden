---
aliases: 
date created: 五月 24日 2023, 2:28:42 下午
date modified: 三月 5日 2024, 4:07:12 下午
title: Bayesian Network
tags: [code/machine-learning]
---
>A Bayesian network is a data structure that represents the dependencies among random variables. Bayesian networks have the following properties.

### Instance
For this image, Rain is dependent on nothing while Maitenance is depentent on the Rain. 
As a result, the possibility of Maintenance is a conditional possibility which needs to be discussed in the certain Rain situation.
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230523223001.png)

### Inference
- Query **X**: the variable for which we want to compute the probability distribution.
- Evidence variables **E**: one or more variables that have been observed for event **e**. For example, we might have observed that there is light rain, and this observation helps us compute the probability that the train is delayed.
- Hidden variables **Y**: variables that aren’t the query and also haven’t been observed. For example, standing at the train station, we can observe whether there is rain, but we can’t know if there is maintenance on the track further down the road. Thus, Maintenance would be a hidden variable in this situation.
- The goal: calculate **P**(_X | e_). For example, compute the probability distribution of the Train variable (the query) based on the evidence **e** that we know there is light rain.