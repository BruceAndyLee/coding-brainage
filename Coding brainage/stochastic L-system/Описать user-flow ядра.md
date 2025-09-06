---
tags:
  - docs
  - core
---
Пользователь предоставляет yaml файл, в котором описаны все свойтсва узлов, и правила переходов.
При генерации дерева будут использоваться наборы атрибутов по бинарному принципу:
- у узла каждый атрибут либо есть, либо нет
- в правилах перехода задается превращение аттрибута или группы атрибутов
Пример:
```yaml
attributes:
	- is_root # ? prolly redundant ?
	- is_assembly
	- configurable # aka CI
	- custom
	- procured
	- standard
	- has_material

# restrictions:
# if OR(assembly, has_material, configurable) -> custom

rules:
	is_assembly: # if parent has this attr, the children are gonna have:
		- probability: 0.8
		  attributes:
			- custom
		    - has_material
		- probability: 0.2
		  attributes:
		    - custom
		    - is_assembly
	
		
```