---
aliases: 
title: Functions
date created: 八月 4日 2023, 10:14:42 上午
date modified: 三月 5日 2024, 4:07:12 下午
tags: [language/python, input]
---

## Overview
### Default Arguments
>Default arguments must appear last in definition

#### Keyword Arguments
With default arguments in function definition, keywords should be pointed out for passing optional arguments.

In addition, people can force the use of keyword arguments.
```python
def read_data(filename, *, debug=False):
	...
```

#### Default Values

**Don't use mutable values as defaults**
The default value is only created once for the whole program--mutations are "sticky"

```python
def func(a, items=[]):
	items.append(a)
	return items
>>> func(1)
[1]
>>> func(2)
[1, 2]
>>> func(3)
[1, 2, 3]
```

**Advice**: Only use immutable values such as None, True, False, numbers, or strings

## Concurrency
### Concurrency
>Functions might execute concurrently (threads)

![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230804124216.png)

### Futures

![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230804125331.png)


## Lambda Functions
> One-expression functions can use lambda

### Usage
#### Alter function args
```python
def distance(x, y):
	return abs(x - y)
>>> dist_from10 = lambda y: distance(10, y)
```

```python
def map(func, values):  
    result = []  
    for x in values:  
       result.append(func(x))  
    return result  
def reduce(func, values, initial=0):  
    result = initial  
    for n in values:  
       result = func(n, result)  
    return result  
def sum(x, y):  
    return x + y  
def square(x):  
    return x * x

nums = [1, 2, 3, 4]  
result = reduce(sum, map(square, nums))
```

## Closures
Essential feature : A "closure" retains the values of all variables needed for the function to run properly later on.

### Usages
- Alternate evaluation (e.g., "delayed evaluation)
- Callback functions
- Code creation ("macros")

## Exceptions

### What Exceptions to Handle?
Functions should only handle exceptions where recovery is possible.
And Let other exceptions propagate

### No need to catch all errors
Never catch all exceptions unless you report/ record the actual exception that occurred

### What Exceptions to Raise?
Applications should have their own exceptions
```python
class ApplicationError(Exception):
	pass
```

### Logging
>Use logging for recording diagnostics

## Test
### Unittest
```python
import simple
import unittest
class TestAdd(unittest.TestCase):
	def test_simple(self):
		# Test with simple integer arguments
		r = simple.add(2, 2)
		self.assertEqual(r, 5)
	def test_str(self):
		# Test with strings
		r = simple.add('hello', 'world')
		self.assertEqual(r, 'helloworld')
if __name__ == '__main__':
	unittest.main()
```


