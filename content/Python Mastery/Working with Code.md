---
aliases: 
title: Working with Code
date created: 八月 8日 2023, 2:51:16 下午
date modified: 三月 5日 2024, 4:07:11 下午
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

