---
aliases: 
date created: 2023-06-21 21:06:00
date modified: 2024-03-20 11:03:14
title: Optimization - Lecture 3
tags: [input]
---

## state-space landscape
> find the max or min value based on the objective function or the cost function

### Hill Climbing
```
function Hill-Climb(problem):
	current = initial state of problem
	repeat:
		neighbor = highest valued neighbor of current
		if neighbor not better than current:
			return current
		current = neighbor
```

Sometimes, we may get stuck in local maxima. How to solve?
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230621221029.png)

### Simulated Annealing
- Early on, higher "temperature": more likely to accept neighbors that are worse than current state
- ﻿﻿Later on, lower "temperature": less likely to accept neighbors that are worse than current state

```
function Simulated-Annealing(problem, max):
	current = initial state of problem
	for t = 1 to max:
		T = Temperature(t)
		neighbor = random neighbor of current
		∆E = how much better neighbor is than current
		if ∆E > 0:
			current = neighbor
		with probability e^(∆E/T) set current = neighbor
```

## Constraint Satisfaction Problem
### Arc Consistent

### Backtracking Search