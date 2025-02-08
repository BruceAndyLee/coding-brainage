---
Status: In progress
tags:
- bread
- frontend
- backend
---
```TypeScript
function getLastFrom<T>(arr: T[]): T | undefined {
	return arr.at(-1);
}
```

> To me, the most intuitive mental model for types (in TypeScript) is that of (mathematical) sets. 'never' is simply the empty set, 'any' is the set of everything, 'string' is the infinite set of all possible strings, while any particular string (e.g. 'abc') when used as a type is simply the set consisting of a single element that is that particular string, and so on. Generics act somewhat like functions at the level of the type system (with the types listed between < and > acting as the arguments), and the 'extends' and 'infer' operators are there for type pattern matching and extraction.

---

### Cheatsheet

![[Untitled 6.png|Untitled 6.png]]

### Typing functions

### Type expression

Для объявления типа функции, описываем её сигнатуру в обобщенном виде. Используется символ `⇒`:

```run-typescript
function makeSomeLogCalls(
	messages: string[],
	logger: (message: string) => void
) {
	messages.forEach(message => logger(message));
}

// the loggerFunction argument's type is defined by function type expression

// you can also keep the type separately
type LoggerFunction = (message: string) => void;
function makeSomeLogCalls(
	messages: string[],
	loggerFunction: LoggerFunction,
) {
	messages.forEach(message => loggerFunction(message));
}

makeSomeLogCalls(["a", "b", "c"], (msg) => { console.log(msg) })
```

### Call signature

Этот инструмент нужен для создания более сложной структуры с несколькими полями, у которой, как и у функции, есть свойство “вызываемость”. То есть, инстансы такого типа будут вызываемы как обычная функция и иметь дополнительные свойства.

```TypeScript
type Human = {
	name: string;
	surname: string;
	age: number;
}
type HumanCreatorFunction = {
	argumentsType: [string, string, number];
	(name: string, surname: string, age: number) => Human;
};
```

Самое смешное: typescript не будет ругаться, если рандомной функции просто задать какое-то свойство от балды. Но вот если мы захотим это свойство прочитать в функции высшего порядка, передав в нее первую функцию, то тогда эта функция высшего порядка будет ругаться, если не объяснить ей, что в нее передали функцию с типом, объявленным с помощью CallSignature

```TypeScript
// невалидно!
// Компилятор не знает, откуда у типа Function могло взяться поле spec
function inspectFunction(fn: Function) {
	console.log(`function inspector: got function ${fn.specs}`);
}
function susFunction() {}
susFunction.specs = "This function is very insiduous!"

inspectFunction(susFunction);


// А вот так - валидно! Потому что теперь insectFunction знает
// что именно за функция в него передается.
type FunctionWithSpecs = {
	specs: string;
	(args: any[]): void;
};
function inspectFunction(fn: FunctionWithSpecs) {
	console.log(`function inspector: got function ${fn.specs}`);
}
function susFunction() {}
susFunction.specs = "This function is very insiduous!"

inspectFunction(susFunction);
```

### Construct signatures

У инстансов какого-то типа может быть конструктор. Таким образом, можно использовать тип и как конструктор и как функцию (привет `Date` из javascript).

Наличие в типе конструктора можно

```TypeScript
type DateObject = {
	seconds: () => number;
	minutes: () => number;
	hours: () => number;
	day: () => string;
	month: () => string;
	year: () => string;
};
type Date = {
	new (epochMilliSeconds: number): DateObject;
};
```

### Generic functions (basis)

Функции можно писать для нескольких типов сразу, превращая их в код, удаленно по функциональности напоминающий javascript, настолько оно может показаться неограниченным.

Функция для получения последнего элемента массива

```TypeScript
// javascript
function getLastFrom(arr) {
	return arr.at(-1);
}

// typescript
// Практически, никакой разницы, но выглядит очень солидно.
function getLastFrom<T>(arr: T[]): T | undefined {
	return arr.at(-1);
}
```

Версия на typescript накладывает всего одно ограничение: в функцию можно передать только массив. _Вроде как еще напрашивается, что все элементы в массиве должны быть_ _одного_ _типа, но нет._

```TypeScript
function getLastFrom<T>(arr: T[]): T | undefined {
	return arr[arr.length - 1];
}

// оба вызова валидны, несмотря на то, что во втором случае
// не все элементы массива одного типа
console.log(getLastFrom([1, 2, 3]))
console.log(getLastFrom([1, 2, "3"]))
```

При этом, typescript занимается выведением типа - **type inference** - самостоятельно. Вовсе не обязательно указывать обобщенной-функции тип аргумента, с которым она будет работать:

```TypeScript
// все эти вызовы отработают без ошибок
console.log(getLastFrom([1, 2, 3]))
console.log(getLastFrom([1, 2, "3"]))
console.log(getLastFrom([]))
console.log(getLastFrom([{}, {}, {}]))
```

Наконец, типов в обобщенной функции можно использовать несколько, и inference всё еще будет автоматический:

```TypeScript
function functionalStyleMap<Input, Output>(
	arr: Input[],
	mapper: (el: Input, index: number) => Output,
) {
	return arr.map(mapper);
}

function functionalStyleFilter<Input>(
	arr: Input[],
	filter: (el: Input, index: number) => boolean,
) {
	return arr.filter(filter);
}

//function functionalStyleReduce<Input, Output>(
//	arr: Input[],
//	reducer: (acc: Output, cur: Input, index: number) => Output,
//) {
//	return arr.reduce(reducer);
//}
```

### Generic functions (with constraints)

Продолжение предыдующей темы: судя по type inference, typescript очень хорошо умеет работать с типами переменных, которые мы пытаемся пробросить в дженерики.

Логично предположить, что он может так же следить за дополнительными ограничениями, которые мы накладываем на те самые произвольные типы.

Например, что тот или иной аргумент может быть не просто рандомного типа, а типа, который является суперсетом какого-то из типов.

Это всегда нужно, чтобы сузить множество допустимых типов:

```TypeScript
// ограничиваем всё множество any/unknown теми
// типами, у которых есть некоторые свойства
function getLongest<T extends { length: number }>(a: T, b: T) {
	return a.length >= b.length ? a : b;
}

// а вот пример, в которм может показаться, что область определения функции
// расширилась:
function getLastFrom<T>(arr: T[]): T | undefined {
	return arr.at(-1);
}

function getLastFrom<T extends Array<any>>(arr: T): T | undefined {
	return arr[arr.length - 1];
}

// но на самом деле, во втором случае обобщенный тип используется
// как массив, а не как элемент массива. То есть, всё равно
// произошло сужение относительно варианта, в котором не было extends:
function getLastFrom<T>(arr: T): T | undefined {
	return arr[arr.length - 1];
}
// потому что теперь компилятор точно знает, что надо работать с массивами
```

Логично предположить, что наличие ограничений на тип должно быть обусловлено ограничениями возможности самого метода. Таким образом, в отличие от javascript, в котором все ограничения функции обсуловлены её реализацией - _сломается она если вместо ожидавшегося массива подкинуть ей объект или число?_- в typescript у разработчика есть возможность ограничить область определения функции на уровне типа и узнать о том, что может вылететь exception, еще на этапе написания кода.

### Generic functions* (matching the constraint)

Еще одна важная особенность работы с типами: typescript следит за используемым обобщенным типом. В следующем примере надо сравнить две реализации функции с одной и той же сигнатурой:

```TypeScript
function <T extends { length: number }>getMin(a: T, b: T): T {
	if (a.length <= b.length) {
		return a;
	}
	return b;
}

function <T extends { length: number }>getMin(a: T, b: T): T {
	if (a.length <= b.length) {
		return { length: a.length };
	}
	return { length: b.length };
}
```

Второй пример приведет к ошибке компиляции и это, на самом деле логично. Потому что в сигнатуре функции четко указано, что тип возвращаемого значения именно тот же, что и у аргументов, а во втором случае, этот тип не сохраняется.

Во втором случае, получается объект другого типа, где этот другой тип тоже расширяет тип `{ length: number }`, но не является типом `T`

  

### Infer and pattern/type-matching

### Keys to values to union

```TypeScript
const LogLevels = {
    DEBUG: "debug",
    WARNING: "warning",
    ERROR: "error",
} as const;

type KeysOf<T> = keyof T;
type ValuesOf<T> = T[keyof T];

type LogLevelKey = KeysOf<typeof LogLevels> // "DEBUG" | "WARNING" | "ERROR"
type LogLevel = ValuesOf<typeof LogLevels> // "debug" | "warning" | "error"
```

### Loose objects

```TypeScript
// a big no-no
// this is bad cuz it doesn't add any extra control of the code
// that typescript has to offer, instead it's just a fallback to JS but with
// extra noise in that cast to any
let obj: any = {};
obj.literally_any_prop_name = "Hello world";

// interface
interface LooseObject {
  [key: string]: any
}

const object: LooseObject<string> = some_crazy_code_to_generate_an_object()
```

Don’t put your project types in .d.ts.

### TS pulls all the types together

```TypeScript
type SizeOptions = "xs" | "sm" | string

// writing the next line won't be supported by any type-augmented autocomplete
// bc TS rightfuly presumes that "xs" and "sm" are string, so they don't need to
// be put in separately from the string type.
const componentSize = ..

// INSTEAD
type LooseAutocomplete<T extend string> = T | Omit<string, T>
type SizeOptions = LooseAutocomplete<"xs" | "sm"> // "xs" | "sm" | Omit<string, "xs" | "sm">
```

### Remove an element from a union

```TypeScript
type Letter = "a" | "b" | "c"

type RemoveC<T extends string> = T extends "c" ? never : T; // as if each member of the union is regarded as a full type

type NoC = RemoveC<Letter> // "a" | "b"
```

### Replace an element in a union

```TypeScript
type Letter = "a" | "b" | "c"

type ReplaceC<T extends string, R> = T extends "c" ? R : T; // as if each member of the union is regarded as a full type

type CinsteadofD = ReplaceC<Letter, "d"> // "a" | "b" | "d"
```

#### Narrow the union (Extract)
```TypeScript
type Extract<T, U> = T extends U ? T : never;

type Action = "log-in" | "log-out" | "send-message" | "delete-user"

type LoginAction = Extract<Action, "log-in" | "log-out>
// "log-in" | "log-out"

```
#### Infer and pattern/type-matching
### Infer different object-’interfaces’ for different keys in those objects

---

### tasks

- is phrase a pangram
    
    ```TypeScript
    // an okay solution I guess. Employs Set
    export const isPangram = (phrase: string): boolean => {
      
      const letterSet: Set<string> = new Set<string>("abcdefghijklmnopqrstuvwxyz".split(""))
    
      let char_ind: number = 0;
      while(letterSet.size && char_ind < phrase.length) {
        letterSet.delete(phrase.charAt(char_ind++).toLowerCase())  
      }
      
      return letterSet.size === 0;
    }
    
    
    // an overloaded solution with the use of LooseObject, just for the sake of practising it
    export const isPangram = (phrase: string): boolean => {
      
      interface LooseObject {
        [key: string]: any
      }
      
      const letterMap: LooseObject = ("abcdefghijklmnopqrstuvwxyz" as string)
        .split("")
        .reduce((map: LooseObject, char: string) => ({ ...map, [char]: char}), {} as LooseObject)
    
      type LetterEntry = keyof typeof letterMap;
      
      let char_ind: number = 0;
      while(Object.getOwnPropertyNames(letterMap).length && char_ind < phrase.length) {
        delete letterMap[phrase.charAt(char_ind++).toLowerCase() as LetterEntry];    
      }
      
      return Object.getOwnPropertyNames(letterMap).length === 0;
    }
    ```
    
- number digit rearranger
    
    ```TypeScript
    export function descendingOrder(n: number): number {
      const digits: number[] = [];
      
      while (n > 0) {
        digits.push(n % 10)
        n = Math.trunc(n / 10)
      }
      
      return digits
    	.sort((a: number, b: number) => a - b)
    	.reduce((res, digit, pow) => res + digit * Math.pow(10, pow), 0)
    }
    ```
    
- narcissistic number
    
    ```TypeScript
    // Number is narcissistic if the sum of its digits,
    // each raised to the digits.length degree, 
    // equals the number itself
    export function narcissistic(value: number): boolean {
      const digits: number[] = [];
      
      let value_ = value;
      while (value_ > 0) {
        digits.push(value_ % 10)
        value_ = Math.trunc(value_ / 10)
      }
      
      const sum = digits.reduce((sum: number, digit: number) => sum + Math.pow(digit, digits.length), 0);
      
      return sum === value
    }
    ```