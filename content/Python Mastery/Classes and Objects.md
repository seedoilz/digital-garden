---
aliases: 
date created: 2023-07-29 09:07:00
date modified: 2024-03-20 11:03:12
title: Classes and Objects
tags: [input, language/python]
---

## Attribute Access Functions
```python
getattr(obj, 'name') # Same as obj.name
setattr(obj, 'name', value) # Same as obj.name = value
delattr(obj, 'name') # Same as del obj.name
hasattr(obj, 'name') # Tests if attribute exists
```

## Bound Methods
>A method that has not yet been invoked by the function call operator () is known as a "bound method"

## Class
### Class Variables
The class and the instances created from the class share the same class variables. 
It can be changed via inheritance.

- Often used for settings applied to all instances
```python
class Date:
	datefmt = '{year}-{month}-{day}'
	def __init__(self, year, month, day):
		self.year = year
		self.month = month
		self.day = day
	def __str__(self):
		return self.datefmt.format(year=self.year,
									month=self.month,
									day=self.day)
```

### Class Methods
>A method that operates on the class itself

Class methods are often used as a tool for **defining alternate initializers**
Class methods solve some tricky problems with features like inheritance
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230729132615.png)

#### Use
```python
class Row:
	 def __init__(self, route, date, daytype, numrides):
		 self.route = route
		 self.date = date
		 self.daytype = daytype
		 self.numrides = numrides
	 @classmethod
	 def from_row(cls, row):
		 return cls(row[0], row[1], row[2], int(row[3]))

def read_csv_as_instances(filename, cls):
    '''
    Read a CSV file into a list of instances
    '''
    records = []
    with open(filename) as f:
        rows = csv.reader(f)
        headers = next(rows)
        for row in rows:
            records.append(cls.from_row(row))
    return records

rides = read_csv_as_instances('Data/ctabus.csv', Row)
```

### Static Methods
>A function that's defined as part of a class, but does not operate on instances or the class

Uses vary:
- Utility functions used by various methods
- Instance management/tracking
- Finalization, resource management
- Certain design patterns

## Python Encapsulation
在 Python 中，以单个下划线 `_` 作为前缀的变量或方法被认为是一种约定，表明它们是类的内部使用，但并没有强制限制子类的访问权限。子类仍然可以访问以单个下划线作为前缀的变量或方法。

然而，对于以双下划线 `__` 作为前缀的变量或方法，Python 会对其名称进行一定的名称修饰（name mangling），以防止子类意外地覆盖父类的私有成员。名称修饰会在变量或方法名前添加一个下划线和类名，以此实现私有成员的隐私保护。这使得子类无法直接访问父类的双下划线开头的私有成员。

### Properties
> Like a attribute, but it can be computed by attribute.

Without @property, anyone can set any type to the variable which could be dangerous. SO, with @property, everyone must use the SET method to the set the value and the SET method can check the type of the value.

```python
class cls:
	...
	@property
	def cost(self):
		return self.shares * self.price
```
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230729172226.png)

### \_\_slots\_\_ Attribute
> A performance optimization
> However, it can cause strange interaction with other parts of Python that are related to objects.
> **Don't use it** except with classes that serve as simple data structures.

## Special Methods
### String Conversions
> Objects have two string representations

#### str (x)
>Printable output

```python
>>> str(d)
'2012-12-21'
```

#### repr (x)
> For programmers

```python
>>> repr(d)
'datetime.date(2012, 12, 21)'
```

### Item Access
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230730102302.png)

### Mathematics
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230730102324.png)

### Instance Creation
> Instances are created in two steps

```python
d = Date.__new__(Date, 2012, 12, 21)
d.__init__(2012, 12, 21)
```

### \_\_del\_\_ method

Typical uses:
- Proper shutdown of system resources (e.g.,network connections)
- Releasing locks (e.g., threading)

## Code Reuse
### Handler Classes
> The handler only contains the methods that need to be implemented/customized

```python
class TableFormatter:
	def headings(self, headings):
		raise NotImplementedError
	def row(self, rowdata):
		raise NotImplementedError
```

### Classes as a Template
> A class might implement a general-purpose algorithm, but delegate certain steps to a subclass

![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230730120436.png)

## Mixin Classes
> A class with a fragment of code

装饰类？

```python
# Mixin Class
class JsonMixin:
    def to_json(self):
        import json
        return json.dumps(self.__dict__)

# 主要类
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

# 使用 Mixin 扩展主要类
class JsonPerson(JsonMixin, Person):
    pass

# 创建实例
person = JsonPerson("Alice", 30)

# 调用扩展的方法
print(person.to_json())  # 输出: {"name": "Alice", "age": 30}
```