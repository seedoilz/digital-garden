---
aliases: 
title: Working with Code
date created: 2023-08-08 14:08:00
date modified: 2024-03-20 11:03:09
tags: [input, language/python]
---

## Function Attributes
```python
def func(a, b):
	'This function does something.'
	...
func.threadsafe = False
func.blah = 42
```

## eval ()  and exec ()
```python
# eval()
>>> x = 10
>>> eval('3*x - 2')
28
# exec()
>>> exec('for i in range(5): print(i)')
0
1
2
3
4
```

