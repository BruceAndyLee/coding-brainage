### to dict
```python
@dataclass
class Config:
	field: Union[0, 1] = 1
	
obj = Config()

obj.__dict__
# {"field": 1}
```

### serialization

