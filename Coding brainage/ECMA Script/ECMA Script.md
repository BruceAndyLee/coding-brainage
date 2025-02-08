---
Status: In progress
tags:
  - bread
  - frontend
  - knowledge
---
[everything everything](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_Types)

# Specification

### Functions

- **Function wrapping +** **.call()** + **.apply()**
    
    `funcname.call`
    
    При вызове функции, принадлежащей объекту, в новом контексте, `this` из ее внутреннего лексического окружения будет перетёрт в значение `undefined`.
    
    ```JavaScript
    let worker = {
    	initial_phase() {
    		// some work
    	},
    	concluding_phase() {
    		// some more work
    	},
    	work() {
    		return this.initial_phase() + this.concluding_phase();
    	},
    };
    
    function addCaching(f) {
    	const cache = new Map();
    	return function(arg) {
    		if (cache.has(arg)) {
    			return cache.get(arg);
    		}
    		const result = f(arg);
    		cache.set(arg, result);
    		return result;
    	}
    };
    
    // rewriting original worker's method with the use of caching_decorator
    worker.work = addCaching(worker.work)
    ```
    
    В этом примере `this` функции [`worker.work`](http://worker.work) равен **undefined**.
    
    для решения проблемы используется метод Function.call:
    
    ```JavaScript
    // f.call(specified_this, ...args)
    
    function addCaching(f, context) {
    	const cache = new Map();
    	return function(...args) {
    		// it is presumed that the hash function knows how
    		// to turn a colletion of values into a single value
    		const cache_key = hash(args);
    		if (cache.has(arg)) {
    			return cache.get(arg);
    		}
    		const result = f.call(context, ...args);
    		cache.set(arg, result);
    		return result;
    	}
    };
    
    // the caching decorator needs to know what context to use
    // in this case for the decorated function to work properly
    worker.work = addCaching(worker.work, worker)
    ```
    
    `Function.apply`
    
    ```JavaScript
    f.call(context, ...args)
    // is equivalent to
    f.apply(context, args)
    ```
    
- **Method context loss +** **.bind()**
    
    ```JavaScript
    let user = {
    	name: "Blyat Ivanovic",
    	sayMyName() {
    		return "God damn right, I am " + this.name;
    	},
    };
    
    // the context is irretrievably lost
    setTimeout(user.sayMyName, 1000);
    
    // the two examples below work, because now
    // 1. the 'user' obj is retreived from the closure - anonymous function's lexical environment
    // 2. its method is called with proper 'this'
    setTimeout(function() {
    	user.sayMyName()
    }, 1000);
    setTimeout(() => user.sayMyName(), 1000);
    
    // this following example works, because the Function.bind method
    // explicitly sets a function's this to be the passed context object:
    setTimeout(user.sayMyName.bind(user), 1000);
    ```
    
    Подобную задачу решают методы всяких библиотек (типа lodash). Метод `bindAll` нужен для того, чтобы привязать все методы внутри объекта к нему. (Какого хера отвязка вообще происходит? почему все методы не привязаны по умолчанию? сложнаэ)
    
- **Constructor method and** `**F.prototype**`
    
    ```JavaScript
    function Elf(name) {
    	this.name = name;
    	// some Elf-initialization steps in here
    }
    
    let Legolas = new Elf("Legolas");
    
    // the object's constructor can be overriden
    // and or modified in run time.
    // to access it:
    let Glorfindel = new Legolas.constructor("Glorfindel");
    Legolas.contructor === Elf; // truthy
    Legolas.constructor === Glorfindel.constructor; // truthy just as well
    ```
    
    В коде может быть изменен прототип функции-конструктора. Это может привести к тому, что у создаваемых ей объектов конструктор будет отличаться от функции-конструктора (поразительная дичь конечно):
    
    ```JavaScript
    function Dwarf(name) {
    	this.name = name;
    }
    
    Dwarf.prototype = {
    	isMaya: false,
    	constructor: SomeOtherConstructorFunction,
    };
    
    let Gloin = new Dwarf("Gloin");
    Gloin.constructor == Dwarf; // FALSY!!!
    ```
    

### Prototype

- **`__proto__`**
    
    `[[Prototype]]` свойство есть у всех объектов. При чтении поля, которого нет в объекте, интерпрeтатор возьмет его из прототипа.
    
    ```JavaScript
    const Ainur = {
    	physical_being: false,
    	sing() {
    		// beautiful singing code
    	}
    };
    const Valar = {
    	bipedal: true;
    };
    
    // __proto__ the name of the getter/setter for the [[Prototype]] field
    Valar.__proto__ = Ainur;
    // alternative approach
    const Valar = {
    	bipedal: true;
    	__proto__: Ainur;
    };
    // more modern approach
    Object.setPrototypeOf(Valar, Ainur);
    Object.getPrototypeOf(Valar);
    
    Valar.sing(); // is accesible since the prototype of Valar is Ainur
    ```
    
    В то же время, при записи свойств в объект, прототип не используется
    
    ```JavaScript
    // this will prevent the Ainur's sing() from being called:
    Valar.sing = function() {
    	// Melkor is rewriting the Eru's singing code
    }
    ```
    
    Кроме случая, когда речь идет о сеттере.
    
    ```JavaScript
    const IsildursHeir = {
    	name: "Ciryon",
      surname: "Isildurovich",
    	set fullName(full_name) {
    		[this.name, this.surname] = full_name.split(" ");
    	},
    	get fullName() {
    		// 'this' will always equal to the object
    		// on which the getter is called;
    		return `${this.name} ${this.surname}`;
    	}
    }
    
    const Aratan = {
    	__proto__: IsildursHeir,
    	AncestorOfAragorn: false;
    };
    
    Aratan.fullName = "Aratan Isildurovich";
    console.log(Aratan.name); // "Aratan"
    console.log(Aratan.surname); // "Isildurovich"
    // the setter function, when accessed on a descendant
    // stays the same;
    ```
    
    Важно помнить, что внутри сеттера и геттера `this` всегда будет равен тому объекту, на котором сеттер/геттер был вызван. При этом не важно, где находится метод - в объекте или в прототипе.
    
    ---
    
    Обход ключей объекта:
    
    ```JavaScript
    for (let key in someObject) // iterates over both inherited and own properties
    Object.keys(someObject) // returns own properties only
    
    // to check if an object has its own property (any inherited - filtered out)
    someObject.hasOwnProperty("propertyName");
    ```
    
    Многие встроенные в прототип свойства не являются перечислимыми (_см. флаги свойств_), поэтому их не видно при использовании итераторов, которые не игнорируют унаследованные свойства. Именно поэтому, при вызове `for .. in` мы не увидим поле `hasOwnProperty()`.
    
- **more about** `**F.prototype**`
    
    В функцию конструктор можно добавить поле `prototype` (это не служебное поле, а просто свойство объекта с таким именем), тогда оператор `new` в `[[Prototype]]` созданного объекта запишет ссылку на этот объект.
    
    ```JavaScript
    let handcraft = {
    	wieldable: true,
    }
    
    function weapon(name) {
    	this.name = name;
    }
    
    // by default weapon's prototype is { constructor: weapon }
    weapon.prototype.constructor === weapon; // truthy
    
    
    weapon.prototype = handcraft;
    
    const sword = new weapon("the blade that was broken");
    sword.wieldable; // truthy
    sword.__proto__ === handcraft; // truthy
    ```
    
      
    
- **embedded/default prototypes**
    
    Встроенные в язык объекты `Function`, `Array`, `Number` имеют свои прототипы со своими функциями. Наверху иерархии прототипов - `Object`
    
    ![[Screenshot_2022-05-17_at_15.17.16 1.png|Screenshot_2022-05-17_at_15.17.16 1.png]]
    
- **primitives**
    
    К числам, строкам и логическим значениям можно обратиться как к объектам. Спецификация говорит, что в момент обращения интерпретатор создает объект, который живет не больше чем надо для обращения к свойствам.
    
- `Object.create()`, `Object.getPrototype()` and stuff
    
    ```JavaScript
    let somePrototype = {
    	// some very prototypical stuff
    }
    
    let yourObject = Object.create(somePrototype, {
    	field_name_1: {
    		value: "some value",
    		// descriptors
    		writable: true,
    		enumerable: false,
    		configurable: false,
    	}
    })
    
    // cloning the object with the righteous functions
    // for prototype/field/descriptors tinkering
    let objectCloned = Object.create(
    	Object.getPrototype(yourObject),
    	Object.getOwnPropertyDescriptors(yourObject)
    );
    ```
    
- **prototypeless object**
    
    Прикол такого объекта в том, что у него нет поля `__proto__`, то есть, это поле можно использовать для своих целей. Однако никаких полезных методов, которые получает объект из прототипа `Object`, у такого объекта тоже не будет.
    
    ```JavaScript
    // no prototype and no fields
    const simpleObject = Object.create(null);
    
    alert(simpleObject); //error - toString is undefined
    ```
    
    Так же надо понимать, что с таким объектом будут работать методы, доступные через `Object`, ибо не в прототипе у `Object`-а они хранятся, а в нем самом.
    

### Descriptors

### Proxy

- **basics**
    
    Прокси - это объект-обертка, который можно надстроить над чем угодно.
    
    Этот объект нужен для того, чтобы перехватывать вызов низкоуровневых методов объекта.
    
    ```JavaScript
    // initialize a proxy
    let target = {};
    let handler = { /* ...traps... */ };
    let proxy = new Proxy(target, handler);
    
    // if the handler object is empty, every function call works on proxy-object
    // the same way as it would if it was called on the target-object itself
    proxy.test = "Woah, mama!"
    
    console.log(target.test) // Woah, mama!
    console.log(proxy.test) // Woah, mama!
    ```
    
    Таблица соответствий между встроенными в язык методами по работе с объектами и ловушками, которые передаются через параметр `handler` в прокси:
    
    ![[Untitled 2 1.png|Untitled 2 1.png]]
    
    ограничения на поведение ловушек:
    
    - Метод `[[Set]]` должен возвращать `true`, если значение было успешно записано, иначе `false`.
    - Метод `[[Delete]]` должен возвращать `true`, если значение было успешно удалено, иначе `false`.
    - Метод `[[GetPrototypeOf]]`, применённый к прокси, должен возвращать то же значение, что и метод `[[GetPrototypeOf]]`, применённый к оригинальному объекту. Другими словами, чтение прототипа объекта прокси всегда должно возвращать прототип оригинального объекта.
    - полный список [https://tc39.es/ecma262/#sec-proxy-object-internal-methods-and-internal-slots](https://tc39.es/ecma262/#sec-proxy-object-internal-methods-and-internal-slots)
- **get-trap** default value
    
    Самый простой пример: значение по умолчанию
    
    ```JavaScript
    let books = {famous_book: "'Arry, Po'ar", good_book: "Пикник на обочине"};
    
    // redefining the initial variable is good practice
    // since the only object with access to the original one
    // will always be the proxy, and there is not going to be any confusion
    books = new Proxy(books, {
    	get(target, prop) {
    		if (prop in target) {
    			return target[prop];
    		} else {
    			return "There is no such book in our library";
    		}
    	}
    });
    console.log(famous_book); // 'Arry, Po'ar
    console.log(excellent_book); // There is no such book in our library
    ```
    
- **set-trap** validation
    
    Ловушка должна возвращать значение, чтобы можно было отследить ошибку на уровне языка\интерпритатора.
    
    ```JavaScript
    let numbers = [1,2,3];
    numbers = new Proxy(numbers, {
    	set(target, prop, val, receiver) {
    		if (typeof val === "number") {
    			target[prop] = val;
    			return true;
    		} else {
    			return false;
    		}
    	}
    })
    ```
    
    Важно понимать, что доступ к методам **массива** у проксированного объекта остается, при этом работают эти методы с учетом нашей ловушки: `push` и `unshift` используют внутри себя метод `[[Set]]`, так что `TypeError` вылетит, если надо будет.
    
- **keys and descriptors** reading and iteration
    
    `[[OwnPropertyKeys]]` используется методами, которые работают со списком свойств объекта (`for-in`, `Object.keys`). Эти методы различаются в деталях:
    
    - `getOwnPropertyNames()` - возвращает не-символьные ключи
    - `getOwnPropertySymbols()` - возвращает символьные ключи
    - `Object.keys()/Object.values()` - возвращает ключи или значения с флагом `enumerable` в дескрипторе
    - `for..in` - перебирает **не**символьные ключи с флагом `enumerable`, и ключи прототипов (тоже с флагом enumerable? несимвольные?)
    
    Пример со скрытыми/приватными/внутренними свойствами объекта:
    
    ```JavaScript
    let user = {
    	name: "Jeff",
    	age: 27,
    	_inn: "3429837646853544",
    };
    
    user = new Proxy(user, {
    	ownKeys(target) {
    		return Object.keys(target).filter(key => !key.startsWith("_"));
    	}
    });
    
    for (let key in user) console.log(key); // name, age
    console.log(Object.keys(user)); // ["name", "age"]
    console.log(Object.values(user)); // ["Jeff", 27]
    ```
    
    Пример с перечислением отсутствующих свойств в ловушке. `Object.keys()` перебирает только `enumerable` поля, значит, если внутри `ownKeys` ловушки вернуть несуществующие поля, то `Object.keys()` ничего не выдаст, так как у отсутствующего поля отсутствует дескриптор (`[[GetOwnProperty]]`). Однако, можно перехватить и свойство, возвращающее дескриптор поля:
    
    ```JavaScript
    user = new Proxy(user, {
    	ownKeys() {
    		return ["lol", "kek", "cheboorek"];
    	},
    	getOwnPropertyDescriptor(target, prop) {
    		return {
    			enumerable: true,
    			configurable: true,
    		}
    	}
    });
    
    console.log(Object.keys(user)); // ["lol", "kek", "cheboorek"]
    // what the literal fuf?? why?????? 
    ```
    
- **property deletion** security
    
    Речь идет о ловушке на внутренний метод `[[Delete]]`. Пример настройки прокси с полностью защищенными внутренними свойствами (аки в ООП), для контроля за удалением используем ловушку `deleteProperty`:
    
    ```JavaScript
    user = new Proxy(user, {
    	get(target, prop) {
    		if (prop.startsWith("_")) {
    			// the interpreter won't throw an error for you from the 'get' trap
    			throw new Error("Access denied.");
    		} else {
    			const value = target[prop];
    			// YYYEEESSSS!!!!!! WHY IS IT NOT DEFAULT?????? oh, it's not that s1mple
    			return typeof value  === "function" ? value.bind(target) : value;
    		}
    	},
    	set(target, prop, value) {
    		if (prop.startsWith("_")) {
    			return false;
    		} else {
    			target[prop] = value;
    			return true;
    		}
    	},
    	deleteProperty(target, prop) {
    		if (prop.startsWith("_")) {
    			// I guess throwing a specific error is preferable to returning false
    			// since the latter will trigger a TypeError, which may confuse the programemr
    			throw new Error("Access denied.");
    		} else {
    			delete target[prop];
    			return true;
    		}
    	},
    	// the same iteration-trap usage as in previous sub-section
    	ownKeys(target) {
    		return Object.keys(target).filter(key => !key.startsWith("_"));
    	},
    });
    ```
    
    ==**Очень важное замечание**==: если внутри объекта **user** изначально был какой-то метод, то при вызове этого метода, внутри этого метода объектом **this** будет хостящий его **user** (с которого началось это предложение). Однако, так как **user** был проксирован, то обращение к полю изнутри метода приведет к вызову ловушки **get.** Привязывание контекста к методам объекта решает эту проблему (если не передавать метод в другую среду без привязки, конечно, тогда, как мы знаем, **this** потеряется).
    
    Объяснения на примере:
    
    ```JavaScript
    let user = {
    	_password: "845602987534",
    	checkPassword(value) {
    		return value === this._password;
    	},
    };
    
    user = new Proxy( /* that big handler-object from the previous code snippet */ );
    
    // now, since we FIRST read the 'checkPassword' field,
    // it calls the 'get' trap which, in the case of an unprotected function,
    // returns a target-bound copy of the function. 
    // So, even though it is called off a user object which by this point is proxied,
    // the 'get' trap of that proxy gives us a function that is bound to the original
    // proxied object. kaCHUNK.
    user.checkPassword("rootandtools");
    ```
    
- **range**-like object
    
    Подобный функционал можно реализовать с помощью ловушки `has`, для встроенного метода `[[HasProperty]]`.
    
    ```run-javascript
    let range = {
    	start: 0,
    	end: 10,
    };
    
    range = new Proxy(range, {
    	has(target, prop) {
    		return prop >= target.start && prop <= target.end;
    	},
    });
    
    console.log(2 in range); // true
    console.log(11 in range); // false
    ```
    
- **function** proxying
    
    Встроенный метод `[[Call]]` описывает, как должна вызываться функция (которая сама по себе объект). Функционал переадресации вызова (_см._ _**Function: call и apply**_) позволяет создавать функции-обертки, однако, при этом доступ к полям оригинальной функции `name` и `length`, например, будет потерян.
    
    ```JavaScript
    function explainUniverse() {
    	return 42;
    }
    
    function delay(f, ms) {
    	return function delayed() {
    		setTimeout(() => f.apply(this, args), ms);
    	}
    }
    
    explainUniverse = delay(explainUniverse, 9999999999999);
    
    console.log(explainUniverse.length); // 0 - since the wrapper funtion does not take any arguments 
    ```
    
    Более гибкий и мощный подход с использованием проксирования (`apply` trap):
    
    ```JavaScript
    explainUniverse = new Proxy(explainUniverse, {
    	apply(targetFunction, targetContext, args) {
    		setTimeout(() => targetFunction.bind(targetContext)(args), 1000)
    	}
    })
    
    // or to be more robust, instead here's the wrapper maker
    function delay(f, ms) {
    	return new Proxy(f, {
    		apply(target, thisArg, args) {
    			// seems like f and target are interchangeable here
    			setTimeout(() => f.apply(thisArg, args), ms);
    		},
    	})
    }
    
    delay(explainUniverse, 1000).length; // 1 - as with original function
    ```
    
    В обоих случаях, однако, возвращаемое значение теряется. Чтобы сохранить возвращение значения надо соорудить более сложную конструкцию. В случае с перехватом вызова метода, вызывающий код получит то, что вернет ловушка `apply`.
    
    ```JavaScript
    function delay(f, ms) {
      return new Proxy(f, {
        apply(target, thisArg, args) {
          // seems like f and target are interchangeable here
          return new Promise((resolve) =>
            setTimeout(() => resolve(f.apply(thisArg, args)), ms)
          );
        },
      });
    }
    ```
    
- more concise proxy construction with `Reflect` object
    
      
    

### Generator

- **Basics**
    
    Особым образом объявляемая функция, которая может возвращать несколько значений, потоком исполнения которой можно управлять из места вызова. Её существование обусловлено необходимостью создавать управляемые стримы. С помощью такого инструмента можно, например, настроить поток загрузки данных.
    
    Функция-генератор не исполняет свой код, а возвращает объект-генератор. Вызывая методы этого объекта можно управлять исполнением кода функции-генератора.
    
    ```JavaScript
    function* generateSequence() {
      yield 1;
      yield 2;
      return 3;
    }
    
    /**
     * Generator object exposes next() method
     * {
     *   next()
     * }
     */
    let generator = generateSequence();
    console.log(generator); // [object Generator]
    
    generator.next(); // returns { value: 1, done: false }
    generator.next(); // returns { value: 2, done: false }
    // if another call of the next() method hits 'return' statement
    // the object contains done: true
    generator.next(); // returns { value: 3, done: true }
    ```
    
- **the** **`for..of`** **and** **`…`** **operators**
    
    ```JavaScript
    function* enumeratingGenerator(array) {
    	for (let ind = 0; ind < array.length; ind ++) {
    		yield [i, array[i]];
    	}
    }
    
    // the for..of operator calls next() method and passes
    // the yielded result into the cycle-body
    for (let [ind, el] of enumeratingGenerator(array)) {
    	console.log(end, el); // 0, array[0], ...
    }
    
    ///////////
    function* sequence() {
    	yield 1;
    	yield 2;
    	yield 3;
    }
    // spread operator makes use of generator's iterability
    console.log([0, ...sequenceGenerator()]); // [0, 1, 2, 3]
    ```
    
- `flatten()` with generator
    
    ```JavaScript
    function* flatten(array, depth) {
        if (depth === undefined) {
            depth = 1;
        }
        for (let item of array) {
            if (Array.isArray(item) && depth > 0) {
                yield* flatten(item, depth - 1);
            } else {
                yield item;
            }
        }
    }
    ```
    
- make any object **iterable** with generators
    
    Сами по себе объекты делаются перебираемыми с помощью `[Symbol.iterator]`
    
    ```JavaScript
    // without using the generator
    let range = {
    	from: 0,
    	to: 10,
    	[Symbol.iterator] = function() {
    		// return an object, that complies with the generator pattern
    		// i.e. it has the next() method that'll be yielding results
    
    		// the for..of operator will work with this object ONLY
    		return {
    			// initialize cycle configs 
    			current: this.from,
    			last: this.to
    			next() {
    				if (this.current < this.last) {
    					return { value: this.current++, done: false };
    				} else {
    					return { value: this.current, done: true };
    				}
    			}
    		}
    	}
    }
    ```
    
    А теперь, можно саму функцию итератор (`[Symbol.iterator]`) сделать генератором, то есть, практически, функции генераторы - это сокращение для создания кастомной логики перебора перебираемых объектов:
    
    ```JavaScript
    // be sure to note that independent calls for the iterator symbol
    // get independed generator-object instances, that can be used for
    // iteration independently (AS IN THE EXAMPLE ABOVE)
    let range = {
    	from: 0,
    	to: 10,
    	*[Symbol.iterator]() {
    		for (let i = this.from; i < this.to; i++) {
    			yield i;
    		}
    	}
    }
    ```
    
- **composition** of generators
    
      
    

### Class

- **basics and parallels**
    
    Объявление класса в js очень близко по сути к созданию функции-конструктора с методами прототипа:
    
    ```JavaScript
    class Guitar {
    	constructor(name) {
    		this.name = name;
    	}
    	play() {
    		// lol
    	}
    	changeStrings() {
    		// kek
    	}
    }
    
    typeof Guitar; // function
    Object.getOwnPropertyNames(Guitar.prototype); // constructor, play, changeStrings
    
    // pretty much equivalent to
    function Guitar(name) {
    	this.name = name;
    }
    
    Guitar.prototype.play = function() {
    	// lol
    }
    
    Guitar.prototype.changeString = function() {
    	// kek
    }
    ```
    
    **Различия**:
    
    - созданная с помощью слова `class` функция, помечена внутренним полем `[[IsClassConstructor]]: true`
    - конструктор (`className()`) не может быть вызван без слова `new`
    - строковое представление начинается со слова `"class"`
    - для оператора `for .. in` функции не видимы (да, как мы помним, поля прототипа норм оператором перечисляются, если только не скрыты дескрипторами)

### Побитовые операторы

побитовыми операторами можно округлять числа лол

13.5 * 14.6 ^ 0

приоритет у побитовых операций ниже чем приоритет операции сравнения

### console

- `console.log()`
- `console.trace()` - возвращает текущее состояние стека вызовов
- `console.dir()` - используется для отображения DOM элементов в виде объектов
- `console.time("timerName")` + `console.timeEnd("timerName")` - для проверки времени исполнения кода между
- `console.group()` + `console.groupEnd()` - раскрывающиеся группы с отступами и вложенностью
- `console.table()` - используется для отображения массива объектов в виде таблицы

- `Colors`
    
    ```JavaScript
    Reset = "\x1b[0m"
    Bright = "\x1b[1m"
    Dim = "\x1b[2m"
    Underscore = "\x1b[4m"
    Blink = "\x1b[5m"
    Reverse = "\x1b[7m"
    Hidden = "\x1b[8m"
    
    FgBlack = "\x1b[30m"
    FgRed = "\x1b[31m"
    FgGreen = "\x1b[32m"
    FgYellow = "\x1b[33m"
    FgBlue = "\x1b[34m"
    FgMagenta = "\x1b[35m"
    FgCyan = "\x1b[36m"
    FgWhite = "\x1b[37m"
    
    BgBlack = "\x1b[40m"
    BgRed = "\x1b[41m"
    BgGreen = "\x1b[42m"
    BgYellow = "\x1b[43m"
    BgBlue = "\x1b[44m"
    BgMagenta = "\x1b[45m"
    BgCyan = "\x1b[46m"
    BgWhite = "\x1b[47m"
    ```
    

### xmlRequest

[https://learn.javascript.ru/xmlhttprequest](https://learn.javascript.ru/xmlhttprequest)


# Utils (self concocted)

- **Date formatting**
    
    [https://learn.javascript.ru/datetime](https://learn.javascript.ru/datetime)
    
    ручное форматирование
    
    ```JavaScript
    /**
     * Formats passed date by replacing characters in
     * the passet @format parameter
     *
     * defaults to
     *  2022-01-27T10:40:07.057373+00:00
     * @param {Date} date
     * @param {String} format
     * @returns
     */
    export function format_date(date, format) {
      if (format) {
        let date_string = format;
        if (date_string.indexOf("hh") !== -1) {
          date_string = date_string.replace("hh", date.getHours());
        }
        if (date_string.indexOf("mm") !== -1) {
          date_string = date_string.replace("mm", date.getMinutes());
        }
        if (date_string.indexOf("ss") !== -1) {
          date_string = date_string.replace("ss", date.getSeconds());
        }
        if (date_string.indexOf("DD") !== -1) {
          date_string = date_string.replace("DD", date.getDate());
        }
        if (date_string.indexOf("MM" !== -1)) {
          date_string = date_string.replace("MM", date.getMonth());
        }
        if (date_string.indexOf("YYYY" !== -1)) {
          date_string = date_string.replace("YYYY", date.getFullYear());
        }
        if (date_string.indexOf("YY" !== -1)) {
          date_string = date_string.replace("YY", date.getFullYear() % 1000);
        }
        return date_string;
      }
      return `${date.getUTCFullYear()}-${date.getUTCMonth()}-${date.getUTCDate()}T${date.getUTCHours()}:${date.getMinutes()}:${date.getSeconds()}.${date.getMilliseconds()}${date.getTimezoneOffset()}`;
    }
    ```
    
    Добавление к дате целого количества минут, часов, дней и недель
    
    ```JavaScript
    /**
     * @param {Date} date
     * @param {Number} number
     * @param {String} unit
     * @returns
     */
    export function add_to_date(date, number, unit) {
      if (number && unit) {
        let milli_seconds_addition = 1000 * parseInt(number);
        switch (unit.toLowerCase()) {
          case "minutes":
            milli_seconds_addition *= 60;
            break;
          case "hours":
            milli_seconds_addition *= 3600;
            break;
          case "days":
            milli_seconds_addition *= 3600 * 24;
            break;
          case "weeks":
            milli_seconds_addition *= 3600 * 24 * 7;
            break;
          default:
            break;
        }
        return new Date(date.getTime() + milli_seconds_addition);
      }
      return date;
    }
    ```
    
- **is_<js_type>** functions PLZ DONT USE THIS ONE
    
    ```JavaScript
    const isFunction = (x) =>
      Object.prototype.toString.call(x) === '[object Function]';
    
    const isArray = (x) =>
    	Object.prototype.toString.call(x) === '[object Array]';
    
    const isDate = (x) =>
    	Object.prototype.toString.call(x) === '[object Date]';
    
    const isObject = (x) =>
      Object.prototype.toString.call(x) === '[object Object]';
    
    const isValue = (x) =>
      !isObject(x) && !isArray(x);
    ```
    
- Deep object comparator
    
    ```JavaScript
    var deepDiffMapper = function () {
      return {
        VALUE_CREATED: 'created',
        VALUE_UPDATED: 'updated',
        VALUE_DELETED: 'deleted',
        VALUE_UNCHANGED: 'unchanged',
        map: function(obj1, obj2) {
          if (this.isFunction(obj1) || this.isFunction(obj2)) {
            throw 'Invalid argument. Function given, object expected.';
          }
          if (this.isValue(obj1) || this.isValue(obj2)) {
            return {
              type: this.compareValues(obj1, obj2),
              data: obj1 === undefined ? obj2 : obj1
            };
          }
    
          var diff = {};
          for (var key in obj1) {
            if (this.isFunction(obj1[key])) {
              continue;
            }
    
            var value2 = undefined;
            if (obj2[key] !== undefined) {
              value2 = obj2[key];
            }
    
            diff[key] = this.map(obj1[key], value2);
          }
          for (var key in obj2) {
            if (this.isFunction(obj2[key]) || diff[key] !== undefined) {
              continue;
            }
    
            diff[key] = this.map(undefined, obj2[key]);
          }
    
          return diff;
    
        },
        compareValues: function (value1, value2) {
          if (value1 === value2) {
            return this.VALUE_UNCHANGED;
          }
          if (this.isDate(value1) && this.isDate(value2) && value1.getTime() === value2.getTime()) {
            return this.VALUE_UNCHANGED;
          }
          if (value1 === undefined) {
            return this.VALUE_CREATED;
          }
          if (value2 === undefined) {
            return this.VALUE_DELETED;
          }
          return this.VALUE_UPDATED;
        },
        isFunction: function (x) {
          return Object.prototype.toString.call(x) === '[object Function]';
        },
        isArray: function (x) {
          return Object.prototype.toString.call(x) === '[object Array]';
        },
        isDate: function (x) {
          return Object.prototype.toString.call(x) === '[object Date]';
        },
        isObject: function (x) {
          return Object.prototype.toString.call(x) === '[object Object]';
        },
        isValue: function (x) {
          return !this.isObject(x) && !this.isArray(x);
        }
      }
    }();
    
    
    var result = deepDiffMapper.map({
      a: 'i am unchanged',
      b: 'i am deleted',
      e: {
        a: 1,
        b: false,
        c: null
      },
      f: [1, {
        a: 'same',
        b: [{
          a: 'same'
        }, {
          d: 'delete'
        }]
      }],
      g: new Date('2017.11.25')
    }, {
      a: 'i am unchanged',
      c: 'i am created',
      e: {
        a: '1',
        b: '',
        d: 'created'
      },
      f: [{
        a: 'same',
        b: [{
          a: 'same'
        }, {
          c: 'create'
        }]
      }, 1],
      g: new Date('2017.11.25')
    });
    console.log(result);
    ```
    
- `mm_per_pix()`
    
    ```JavaScript
    funtion mm_to_px(mm) {
      let div = document.createElement("div");
      div.style.display = "block";
      div.style.height = "100mm";
      document.body.appendChild(div);
      let converted = (div.offsetHeight * mm) / 100;
      div.parentNode.removeChild(div);
      return converted;
    }
    ```
    
- base64 image display
    
    ```JavaScript
    <img src="`data:image/png;base64, ${image_b64_content}`" />
    ```
    
- file downloader function
    
    ```JavaScript
    export function save_file(content, filename, mime = "text/plain") {
      const byteNumbers = Array.from({ length: content.length });
      for (let i = 0; i < content.length; i++) {
        byteNumbers[i] = content.codePointAt(i);
      }
      const byteArray = new Uint8Array(byteNumbers);
      const a = document.createElement("a");
      document.body.append(a);
      a.style = "display: none";
      const blob = new Blob([byteArray], { type: mime });
      const url = window.URL.createObjectURL(blob);
      a.href = url;
      a.download = filename;
      a.click();
      window.URL.revokeObjectURL(url);
    }
    ```
    
- template function calling
    
      
    

# Utils (from the community)

[axios + ts](https://bobbyhadz.com/blog/typescript-http-request-axios)

[[reactivity]]

Мемоизация [https://habr.com/ru/company/ruvds/blog/332384/](https://habr.com/ru/company/ruvds/blog/332384/)

[[ES JS ECMAScript]]

[[Browser APIs]]

[[html + html5]]

```Plain
Reset = "\x1b[0m"
Bright = "\x1b[1m"
Dim = "\x1b[2m"
Underscore = "\x1b[4m"
Blink = "\x1b[5m"
Reverse = "\x1b[7m"
Hidden = "\x1b[8m"

FgBlack = "\x1b[30m"
FgRed = "\x1b[31m"
FgGreen = "\x1b[32m"
FgYellow = "\x1b[33m"
FgBlue = "\x1b[34m"
FgMagenta = "\x1b[35m"
FgCyan = "\x1b[36m"
FgWhite = "\x1b[37m"

BgBlack = "\x1b[40m"
BgRed = "\x1b[41m"
BgGreen = "\x1b[42m"
BgYellow = "\x1b[43m"
BgBlue = "\x1b[44m"
BgMagenta = "\x1b[45m"
BgCyan = "\x1b[46m"
BgWhite = "\x1b[47m"
```