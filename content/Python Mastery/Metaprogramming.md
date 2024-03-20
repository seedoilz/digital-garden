---
aliases: 
title: Metaprogramming
date created: 2023-08-08 18:08:00
date modified: 2024-03-20 11:03:09
tags: [language/python, input]
---
  

## Decorators
Add new behaviors to an existing method

```python
def logged(func):
	# Define a wrapper function around func
	def wrapper(*args, **kwargs):
		print('Calling', func.__name__)
		return func(*args, **kwargs)
	return wrapper
```

```python
@logged
def add(x, y):
	return x + y
```

### Applications
- Debugging and diagnostics
- Avoiding code replication
- Enabling/disabling optional features

### Copying Metadata
>Decorators should copy metadata

```python
def logged(func):
	@wraps(func)
	def wrapper(*args, **kwargs):
		print('Calling', func.__name__)
		return func(*args, **kwargs)
	# wrapper.__name__ = func.__name__
	# wrapper.__doc__ = func.__doc__
	return wrapper
```

### Class Decorator
```python
def logged_getattr(cls):
	# Get the original implementation
	orig_getattribute = cls.__getattribute__
	# Replacement method
	def __getattribute__(self, name):
		print('Getting:', name)
		return orig_getattribute(self, name)
	# Attach to the class
	cls.__getattribute__ = __getattribute__
	return cls
	
@logged_getattr
class MyClass:
	def foo(self):
		pass
	def bar(self):
		pass
```

