---
aliases: 
date created: 八月 1日 2023, 10:02:04 上午
date modified: 三月 5日 2024, 4:07:12 下午
title: Inside Python Objects
tags: [input, language/python]
---

## Dicts, Classes, Instances
>Each instance gets its own private dictionary to store its member variables
>Each class gets its own dictionary to store its methods

### Instances and Classes
Instances and classes are linked together by the **__class__** attribute.

The instance dictionary holds data unique to each instance whereas the class dictionary holds data collectively shared by all instances
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230801115530.png)

#### Attribute Access
Using the (.) operator helps to access data and methods, which is actually tied to the dictionaries.
When we read or write the attributes of an instance, we actually make actions to the dictionaries underlying.

- First check in local \_\_dict\_\_
- If not found, look in \_\_dict\_\_ of class
- If not found in class, look in base classes

## Inheritance
### How Inheritance Works
Bases classes are stored as a tuple in each class which provides a link to parent classes

![[#Attribute Access]]

### Single Inheritance

In inheritance hierarchies, attributes are found by walking up the inheritance tree.
With single inheritance, there is a single path to the top.
You stop with the first match.

#### The MRO
>Method Resolution Order

The inheritance chain is precomputed and stored in an "MRO" attribute on the class

![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230801124328.png)


### Multiple Inheritance
>It is like Single Inheritance. But it needs rules to determine the path.
#### Rules
Rule 1: Children before parents
Rule 2: Parents go in order

### Why super()?
> Always use super () when overriding methods
> super () delegates to the next class on the MRO

### Designing for Inheritance
#### Rule 1
>Compatible Method Arguments

Overridden methods must have a compatible signature across the entire hierarchy

#### Rule 2
>Method chains must terminate

There must be a implemented class to terminate the search chain

### Descriptor
>A _descriptor_ is an object with one or more of the following special methods

Whenever an attribute is accessed on a class, the attribute is checked to see if it is an object that looks like a so-called "descriptor"

Descriptors **always override \_\_dict\_\_**

#### Who Cares descriptor?
Every major feature of classes is implemented using descriptors
-  Instance methods
- Static methods (@staticmethod)
- Class methods (@classmethod)
- Properties (@property)
- \_\_slots\_\_

#### Descriptors in Action
##### Behind the scenes of method lookup
```python
>>> s = Stock('GOOG',100,490.10)
>>> value = Stock.__dict__['cost']
>>> value
<function cost at 0x378770>
>>> hasattr(value,"__get__")
True
>>> result = value.__get__(s,Stock)
>>> result
<bound method Stock.cost of <__main__.Stock object at0x37e250>>
>>> result()
49010.0
```

##### Descriptors and Properties
```python
>>> s = Stock()
>>> p = Stock.__dict__['shares']
>>> p
<property object at 0x3759c0>
>>> p.__set__(s, 100) # Same as s.shares = 100
>>> p.__get__(s, Stock) # Same as s.shares
100
>>> s.shares
100
```

##### Descriptors and \_\_slots__
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230802164937.png)
Each slot name is used to create a descriptor that simply gets or sets values in the appropriate array position (internals are implemented in C and hard to view though)

#### Descriptor Application
>Provide more precise control than properties.

```python
class Integer:
	def __init__(self, name):
		self.name = name
	def __get__(self, instance, cls):
		return instance.__dict__[self.name]
	def __set__(self, instance, value):
		if not isinstance(value, int):
			raise TypeError('Expected an integer')
		instance.__dict__[self.name] = value
```

>A weaker descriptor that only has \_\_get__ Only triggered if obj.\_\_dict__ doesn't match

### Attribute Access Methods
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230804085044.png)
#### \_\_getattribute__()
> Called every time an attribute is read

Default behavior looks for descriptors, checks the instance dictionary, checks bases classes (inheritance), etc.
If it can't find the attribute after all of those steps, it invokes \_\_getattr__(self,name)

#### \_\_getattr__() method
>A failsafe method. Called if an attribute can't be found using the standard mechanism

#### \_\_setattr__() method
>Called every time an attribute is set

Default behavior checks for descriptors, stores values in the instance dictionary, etc.

#### \_\_delattr__() method
>Called every time an attribute is deleted

Default behavior checks for descriptors and deletes from the instance dictionary