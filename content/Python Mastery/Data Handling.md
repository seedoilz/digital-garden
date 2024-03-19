---
aliases: 
date created: 七月 25日 2023, 4:47:52 下午
date modified: 三月 5日 2024, 4:07:12 下午
title: Data Handling
tags: [input, language/python]
---

## Tuple
- can be used as an array
```python
s = ('Goog', 100, 490.1)
name = s[0]
cost = s[1] * s[2]
```
- unpack into separate variables
```python
name, shares, price = s
```
- **Immutable (can not be modified)**

## Slots
### Effect
- Saves memory
```python
class Stock:
	__slots__ = ('name', 'shares', 'price')
	def __init__(self, name, shares, price):
		self.name = name
		self.shares = shares
		self.price = price
```
don't need extra space to store the additional attribute
However, the son class will not inherit the slots from the father class

## Dataclasses
### Effect
This can reduce the amount of code that must be written

```python
from dataclasses import dataclass

@dataclass
class Stock:
	name : str
	shares : int
	price: float
```

### Points
Types are NOT enforced (the type of the variables in the class can be changed)

## Named Tuples
### Effect
- Immutability/tuple behavior
- another variant on class definition

```python
from collections import namedtuple # Define a named tuple type called "Point" with fields "x" and "y" Point = namedtuple('Point', 'x y')
```

## Summary
From the exercise in python-mastery, **named tuples** and **classes with slots** can save a lot of memory. Dictionaries will cost much more memory than common classes.
$$
Slots < NamedTuple < Tuple < Class < Dictionary
$$
However, named tuples and slots are able to save memory since they are determined while others can change during running. In addition, I thought that Class will cost more memory than dictionary. BUT opposite.

## Dicts
### Composite Keys
Use tuples for multi-part keys
```python
prices = {
	('ACME','2017-01-01') : 513.25,
	('ACME','2017-01-02') : 512.10,
	('ACME','2017-01-03') : 512.85,
	('SPAM','2017-01-01') : 42.1,
	('SPAM','2017-01-02') : 42.34,
	('SPAM','2017-01-03') : 42.87,
}
```

### Lookup with default value
```python
p = prices.get('AAPL', 0.0) # Lookup with default value
```

### defaultdict
If I ask for a missing dict key, it will generate a default data type for me to use.
```python
from collections import defaultdict
d = defaultdict(list)
d = defaultdict(int)...
```

### Counter
>A dictionary specialized for counting items

```python
from collections import Counter
totals = Counter()
totals['IBM'] += 20
totals['AA'] += 50
```

### ChainMap
```python
from collections import ChainMap
allprices = ChainMap(dict1, dict2)
```


## deque
>Double-ended queue

```python
from collections import deque
q = deque()
q.append(1)
q.appendleft(3)
q.pop()
q.popleft()
```

### Keeping a History
>Keep a history of the last N things
```python
history = deque(maxlen = N)
```

## Iteration
### Wildcard unpacking
```python
prices = [  
    ['GOOG', 490.1, 485.25, 487.5 ],  
    ['IBM', 91.5],  
    ['HPE', 13.75, 12.1, 13.25, 14.2, 13.5 ],  
    ['CAT', 52.5, 51.2]  
]
for name, *values in prices:
	print(name, values)
```
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/20230727171533.png)

### Unpacking Iterables
```python
a = (1, 2, 3)
b = [4, 5]
c = [ *a, *b ] # c = [1, 2, 3, 4, 5] (list)
d = ( *a, *b ) # d = (1, 2, 3, 4, 5) (tuple)
```

### Unpacking Dictionaries
```python
a = { 'name': 'GOOG', 'shares': 100, 'price':490.1 }
b = { 'date': '6/11/2007', 'time': '9:45am' }
c = { **a, **b }
{ 'name': 'GOOG', 'shares':100, 'price': 490.1,
'date': '6/11/2007,'time': '9:45am' }
```

### Argument Passing
#### Iterables can expand to positional args
```python
a = (1, 2, 3)
b = (4, 5)
func(*a, *b) # func(1,2,3,4,5)
```

#### Dictionaries can expand to keyword args
```python
c = {'x': 1, 'y': 2 }
func(**c) func(x=1, y=2)
```

#### Combinations fine as long as positional go first
```python
func(*a, **c)
func(*a, *b, **c)
func(0, *a, *b, 6, spam=37, **c)
```

## Generator
>A generator can only be consumed once

```python
nums = [1,2,3,4]  
squares = (x*x for x in nums)
for n in squares:  
    print(n, end=' ')
```

### Generator Arguments
```python
sum(x*x for x in nums)
print(','.join(str(x) for x in items))
if any(name.endswith('.py') for name in filenames):
```

Generator acts as a filter/transform on an iterable.

### Generator Functions
In Python, yield is a keyword used to define generator functions. The generator is a special iterator that allows values to be generated on demand instead of all values at once, thus effectively saving memory and computing resources.

## Builtin
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230727221037.png)

### TO make new "Builtins"
Probably the key thing to keep in mind is that you can customize almost every aspect of how an object interacts with the rest of Python if you know the underlying protocols. If you're going to do this, it's advisable to look at the existing code for something similar to what you're trying to make.
```python
class MutInt:  
    __slots__ = ['value']  
  
    def __init__(self, value):  
        self.value = value  
  
    def __add__(self, other):  
        if type(other) == int:  
            return MutInt(self.value + other)  
        return MutInt(self.value + other.value)  
  
    def __str__(self):  
        return str(self.value)  
  
    def __repr__(self):  
        return f'MutInt({self.value!r})'  
  
    def __format__(self, fmt):  
        return format(self.value, fmt)  
  
    def __eq__(self, other):  
        if isinstance(other, MutInt):  
            return self.value == other.value  
        elif isinstance(other, int):  
            return self.value == other  
        else:  
            return NotImplemented  
  
    def __lt__(self, other):  
        if isinstance(other, MutInt):  
            return self.value < other.value  
        elif isinstance(other, int):  
            return self.value < other  
        else:  
            return NotImplemented  
  
    def __int__(self):  
        return self.value  
  
    def __float__(self):  
        return float(self.value)  

    __index__ = __int__  # Make indexing work  
  
    __radd__ = __add__
```

## Over-allocation
In Python, all mutable containers tend to over-allocate memory so that there are always some free slots available. It is a performance optimization.
- Lists : Increase by ~12.5% when full
- Sets : Increases by factor 4 when 2/3 full
- Dicts : Increases by a factor 2 when 2/3 full

## Key Restrictions
Sets/dict keys restricted to “hashable” objects
This usually means you can only use strings, numbers, or tuples (no lists, dicts, sets, etc.)