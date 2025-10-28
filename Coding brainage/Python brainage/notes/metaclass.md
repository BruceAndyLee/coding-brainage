All classes **IMPLICITLY** inherit from `type`.
Inheriting **IMPLICITLY** is not the same as inheriting from it **EXPLICITLY**.

You may want to redefine how classes are instatiated, but you are not allowed to add functionality to `type`:
```python
def new_type_new(cls):
	instance = type.__new__(cls)
	instance.birth_mark = "dovakin"
	return instance

type.__new__ = new_type_new
# TypeError: cannot set '__new__' attribute of immutable type 'type'
```
which is totally legit since doing that would create an infinite recursion of calling the `__new__` method on the `typee` class-object.

Instead, you create new meta class, that **EXPLICITLY** inherits from the type:
```python
class SkyrimProtagonist(type):
	def __new__(cls, name, bases, dct):
		# add some printing just to see what the hecc
		print("SkyrimProtagonist::__new__(): environment")
		for attr, val in zip(
			['super', 'type(super)', 'super()', 'type(super())'],
			[super, type(super), super(), type(super())],
		):
			print(f'\t{attr}: {val}')
		# super: <class 'super'>
        # type(super): <class 'type'>
        # super(): <super: <class 'SkyrimProtagonist'>, <SkyrimProtagonist object>>
        # type(super()): <class 'super'>		
		
		print("SkyrimProtagonist::__new__(): arguments")
		for attr, val zip(
			['cls', 'name', 'base', 'dct'],
			[cls, name, base, dct]
		):
			print(f'\t{attr}: {val}')
			# cls: <class '__main__.SkyrimProtagonist'>
	        # name: Khajit
	        # bases: ()
	        # dct: {'__module__': '__main__', '__qualname__': 'Khajit', '__init__': <function Khajit.__init__ at 0x10311cae0>}

		# instance = type.__new__(cls) wrong! type.__new__ expects 3 arguments
		# instance = type.__new__(name, bases, dct) won't work bc it's for explicit type inheritants 
		instance = super().__new__(cls, name, base, dct)
		instance.birth_mark = "dovakin"
		return instance
		
class Khajit(SkyrimProtagonist):
	def __init__(self):
		self.perks = ["cat ears"]
		
me = Khajit()
print("me birth mark", me.birth_mark)
# me birth mark dovakin
print("me", me._class)
# me <class '__main__.Khajit'>
print("me perks:", me.perks)
# me perks: ['his ears']
```

type is a metaclass so deriving from it explicitly means creating a new metaclass.

It seems as though `class ClassName(metaclass=CustomMetaclass)` is sugar for `ClassName = type.__new__(CustomMetaclass, 'ClassName', (), {})`:
```python
# this section is buggy, return to it later
class SomeGuyInSolitude(type):
	citizen_of: 'Solitude'
	
Imperial = type.__new__(SomeGuyInSolitude, 'Imperial', (), dict(drinks_for='Uriel Septim'))
some_guard_without_knee = Imperial()
print('this guy is:', some_guard_without_knee.__class__)
# Imperial
print('this guy is the citizen of', some_guard_without_knee.citizen_of)
# Solitude
```

Metaclass with state:

Нерабочий пример на `__new__`
```python
class Singleton(type):
    class_instances = {}
    def __new__(cls, name, bases, data):
        print("Singleton::__new__ called")
        if cls.__name__ in Singleton.class_instances:
            return Singleton.class_instances[cls.__name__]

        instance = super().__new__(cls, name, bases, data)
        Singleton.class_instances[cls.__name__] = instance

        return instance

```

Рабочий пример на `__call__`
```python
class Singleton(type):
    class_instances = {}
    def __call__(cls, *args, **kwargs):
        print("Singleton::__call__ called")
        if cls.__name__ in Singleton.class_instances:
            return Singleton.class_instances[cls.__name__]

        instance = super().__new__(cls, name, bases, data)
        Singleton.class_instances[cls.__name__] = instance

        return instance


class LolService(metaclass=Singleton):

	@classmethod
	def count_init(cls):
		if not hasattr(cls, 'init_count'):
			setattr(cls, 'init_count', 0)
		
		cls.init_count += 1
	
	@classmethod
	def get_init_count(cls):
		if not hasattr(cls, 'init_count'):
			return 0
		
		return cls.init_count

	def __init__(self):
		self.lol_message = "lol"
		

print("before instantiating the first service", LolService.get_init_count())
# Singleton::__new__ called  IS THIS REALLY HAPPENING BECAUSE WE ACCES A CLASS METHOD ON AN UNINSTANTIATED CLASS?
# before instantiating the first service: 0

lol_service = LolService()
# Singleton::__call__ called
print("after instantiating the first service:", LolService.get_init_count())
# after instantiating the first service: 1

lol_serice_2 = SomeService()
# Singleton::__call__ called

print("after instantiating the second service:", LolService.get_init_count())
# after instantiating the second service: 1
# lol



```