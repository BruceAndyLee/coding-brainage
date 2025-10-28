If you want to name your entrypoint script something other than `main.py` and still have it function essentially as a `main.py`, do this:
```python
def my_entry_point_func():
	pass

if __name__ == "__main__":
	my_entry_point_func()
```

When python runs files directly - i.e. `python3 my_script.py` - it sets the `__name__` variable to `'__main__'` - for that purpose exactly (it seems like)