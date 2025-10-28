It would seem that with is like defining a function in place and running it
```python
def main():
	with open("payouts.txt", "r") as f: # it's got assignment and shit
		f.seek(0)
		bills = f.read()
	# f is destroyed by the end of this blocc
	print(f) # will prolly raise an excetion
```

an even clearer example of function likeness with [[async await]]:
```python
import asyncio as aio
import time

async def say_what(eep, what):
	aio.sleep(eep)
	print(what)

async def main():
	async with aio.TaskGroup() as tg: # it is async as if a function signature
		task1 = tg.createTask(say_what(2, "What?.."))
		task2 = tg.createTask(say_what(3, "What?!!.."))
		print(f"both tasks registered at {time.strftime('%X')}")
		
	# the await is FOCKEN IMPLICIT MAN BECAUSE THE "FUNCTION" IS IMMEDIATELY CALLED I GUESS
	print(f"both tasks finished at {time.strftime('%X')}")
	
aio.run(main())
```