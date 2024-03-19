---
aliases: 
title: Iterators
date created: 八月 14日 2023, 3:51:23 下午
date modified: 三月 5日 2024, 4:07:11 下午
tags: [language/python, input]
---

## Generators

### Reusable Generators
```python
class Countdown:
    def __init__(self, n):
        self.n = n
        
    def __iter__(self):
        n = self.n
        while n > 0:
            yield n
            n -= 1
            
countdown_instance = Countdown(5)
# 第一次迭代
for value in countdown_instance:
    print(value)
# 创建一个新的迭代器并进行第二次迭代
for value in countdown_instance:
    print(value)
```

## Generator Pipelines

![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230814192011.png)

## Coroutines

```python
def match(pattern):  
    print('Looking for %s' % pattern)  
    while True:  
        line = yield  
        if pattern in line:  
            print(line)
g = match('python')
g.send("python generators rock!")
```

### Dataflow
> With coroutines, you can "fan out"

![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230814195058.png)

### Control Flow

- .close() method to shutdown a generator
- .throw() method to raise exception

### Managed Generators
>A manager will coordinate the execution of a collection of executing generators

```python
tasks = deque([
	countdown(10),
	countdown(5),
	countup(20)
])
def run():
	while tasks:
		t = tasks.popleft() # Get a task
		try:
			t.send(None) # Run to yield
			tasks.append(t) # Reschedule
		except StopIteration:
			pass
```

### Delegating Generation
#### Drive the generator yourself
```python
def up_and_down(n):
	for x in countup(n):
		yield x
	for x in countdown(n):
		yield x
```

#### Let Python drive it
```python
def up_and_down(n):
	yield from countup(n)
	yield from countdown(n)
```