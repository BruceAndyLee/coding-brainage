add `async` to the function signature in front and import a sleeper from asyncio
```python
import asyncio as aio

async say_what(delay, what):
	await aio.sleep(delay)
	print(what)
```

This is still function but it no longer returns a value, but a coroutine:
```python
type(say_what)
# <class 'function'>

type(say_what())
# <class 'coroutine'>
```

The result will be lost unless you use a coroutine runner that awaits its completion
```python
aio.run(say_what(2, "What?.."))
# dos segundas despues
# > What?..
```

Just calling a function is creating a coroutine but not scheduling it to be executed.
Of course you cannot use await outside of function (bc everything in python is an object but not the module so the module execution cannot be awaited by the python interpreter itself, there has to be like an explicit awaiter idk or something), which is what `aio.run` is for. It's probably just
```python
async def run(func):
	await func()
```

or something (bruh ofc it isn't. That `run` function needs to be awaited by somebody too)

---

You can schedule the coroutines to be run concurrently (try to spot the difference):
```python
async say_what(delay, what):
	await aio.sleep(delay)
	print(what)

# this is sequential
async def main():
	await say_what(2, 'whot?..')
	await say_what(2, 'You WHOT now??..')

# this is concurrent	
async def main():
	say_what_task = aio.create_task(say_what(2, 'whot?..'))
	say_what_but_longer_task = aio.create_task(say_what(2, 'You WHOT now??..'))
	
	await say_what_task
	await say_what_but_longer_task

# this is concurrent but with with
async def main():
	async with aio.TaskGroup() as tg:
		say_what_task = tg.create_task(say_what(2, 'whot?..'))
		say_what_but_longer_task = tg.create_task(say_what(2, 'You WHOT now??..'))
	
	# ... and the fucking await were implicit ...
	print("This is printed out with a delay")
```

The second variation (2nd and 3d actually) will take less time (ideally two times less), but in reality there is some coroutine-sync overhead.
So to make sure the difference is visible you kinda have to give it a bigger delay:
```python
import time
import asyncio as aio

async def main():
	async with aio.TaskGroup():
		say_what_task = tg.create_task(say_what(5, 'whot?..'))
		say_what_but_longer_task = tg.create_task(say_what(5, 'You WHOT now??..'))
		print(f"Tasks registered at {time.strftime('%X')}") # no fucking clue what is this formatting
	
	# ... oh my gaaawd, the await were implicit ...
	print(f"Tasks completed at {time.strftime('%X')}")


# Tasks registered at 18:34:16
# whot?.. <- shows up simultaneously with the last print
# You WHOT now??.. <- shows up simultaneously with the last print
# both tasks finished at 18:34:21
```
So the time difference is about 5 seconds, not 10 as in a sequential case.