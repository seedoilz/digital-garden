---
aliases: 
date created: 五月 19日 2023, 4:19:02 下午
date modified: 三月 5日 2024, 4:07:12 下午
title: Knowledge - Lecture 1
tags: [input]
---

## Model Checking
To determine if $KB \models \alpha$:
	- Enumerate all possible models
	- If in every model, $KB \models \alpha$ is True, then $KB \models \alpha$
	- Otherwise, it is not true

## Conversion to CNF
1. Elminate biconditionals
2. Elminate implications
3. Move NOT inwards using De Morgan's laws
4. Use distributive law to distribute v whenever possible