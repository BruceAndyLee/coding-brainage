`type` is not a type checker
```python
# main.py

class Bass:
	def strum(self):
		print("Dabadadibadada")
	
adam_neely_fender = Bass()
adam_neely_fender.strum()
# Dabadadibadada

Bass_2 = type(adam_neely_fender)
# <class __main__,'Bass'>

adam_neely_five_string = Bass_2()
adam_neely_five_string.strum()
# Dabadadibadada
```

`type` is a meta class
```python
type(Bass)
# <class 'type'>

type(type(Bass))
# <clas 'type'>
```

`type` is a class creator function 
```python
class MusicalInstrument:
	def make_sound(self):
		print("pluck")
		

class BuildingTool:
	def make_sound(self):
		print("thunk")
		

# btw this is how you make a tuple with only one element (element,)
(MusicalInstrument,)
# btw dict can take an arbitrary amount of amed paramenters
dict(attr=123)
# { 'attr': 123 }

# ANYWAY

Bass = type('Bass', (MusicalInstrument,), dict(strings=['e', 'a', 'd', 'g']))
type(Bass)
# <class 'type'>
Bass.strings
# ['e', 'a', 'd', 'g']
jaco_fretless = Bass()
jaco_fretless.make_sound()
# pluck
jaco_fretless.strings
# ['e', 'a', 'd', 'g']
type(Hammer)
# <class 'type'>
Hammer.weight_pounds
# 6
chris_cornells_hammer = Hammer()
chris_cornells_hammer.make_sound()
# thunk
chris_cornells_hammer.weight_pounds
# 6

# lets inherit from both classes
WTFIsThis = type('WTFIsThis', (MusicalInstrument, BuildingTool), dict(marks=['d', 'e', 'a', 'd', 'b', 'e', 'e', 'f']))

wha = WTFIsThis()
wha.make_sound()
# place your bets

OhMyGod = type('OhMyGod', (BuildingTool, MusicalInstrument), dict(marks=['d', 'e', 'a', 'd', 'b', 'e', 'e', 'f']))

wait_no = OhMyGod()
wait_no.make_sound()
# place your bets [2]


```