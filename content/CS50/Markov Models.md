---
aliases: 
date created: 五月 24日 2023, 2:29:24 下午
date modified: 三月 5日 2024, 4:07:12 下午
title: Markov Models
tags: [code/machine-learning]
---

### Markov Assumption
The current state depends on only a finite fixed number of previous states.
Usingt he Markov assumption, we restrict our previous states, thereby making the task manageable. What we get from this is a more rough approxiamation of the probabilities of interest, but this is often good enough for our needs.

### Markov Chain
> A sequence of random variables where the distribution of each variable follows the Markov assumption.
> Each event in the chain occurs based on the probability of the event before it.

#### How to use?
##### A transition model
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230524134829.png)
