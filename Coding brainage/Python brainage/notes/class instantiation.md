- Classes have methods `__new__` and `__init__`
- When classes are called, python calls the `__call__` method of their parent classes - it's `type.__call__`.
- `type.__call__` calls the `__new__` and `__init__` and the class that has been called (if overridden)


`MyClass.__call__ -> type.__call__ -> [MyClass.__new__(cls), MyClass.__init__(inst)]`

- the responsibility of `__new__` is the return an instance
- the responsibility of `__init__` is to setup an existing instance

you can interject:
```python
class Piano:
	def play(self, note: str):
		print(note)

original_new = Piano.__new__

def substitue_new(cls):
	print("__new__ got hacked:", cls)
	return original_piano_new(cls)

Piano.__new__ = subsitute_new

p = Piano()
# __new__ got hacked <class __main__.Piano>
```

you can fucking break the fucking thing:
```python
class Piano:
	def play(self, note: str):
		print(note)

def substitue_new(cls):
	print("__new__ got hacked:", cls)
	return cls.__new__(cls)

Piano.__new__ = subsitute_new

p = Piano()
# __new__ got hacked <class __main__.Piano> x10000
# RecursionError: maximum recursion depth exceeded while calling a Python object
```

it seems you cannot intergect the `__call__` :
```python
Piano = type('Lol', (), dict(strings=['a', 'b', 'c', 'd']))
original_call = Piano.__call__

def substitute_call():
	print("__call__ got hacked!")
	return original_call()
	
Piano.__call__ = substitue_call

p = Piano() # no outpuot ensues 
```

