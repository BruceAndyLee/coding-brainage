Prolly from the first talk but the bullet-points of a good language design are claimed as:
- Don't have headers. Implement imports or something.
- support good programmers (don't include safety measures that minimise the number if mistakes rookies do if they're imparing the DX of good programmers - so, in a lot of ways like globals - make your language terser)
- language-level error-message generation (remembering stack frames of each of the exceptions generated)
- generailzed: support lambdas very well
	- unhindered static type-checking: factorability: easy to move code around (e.g. FD is no different from lambda, typed the same way) - NO NEED TO CHANGE CODE WHEN CHANGING SCOPE between
		- inline
		- in-block
		- in-file
		- in-project
	- unhindered static type-checking: factorability: no need to use libs to move pieces of code around (see previous, cpp only has lambda typing support in the std, not the language itself)
- think about captures:
	- they should not be a part of the type, take place AFTER the type. Thus make captures a property of a block of code and RESTRICT it to using a specific variable from above that code block.
	  Maturity cycle gets another iteration of factoring: a code-block with captures. Better error messages?
	- capture process is orthogonal to the function mechanics
	- capture makes for a good starting point for thread-safety by default by giving developer a compile error when a code block is written to access anything outside of the things specified in the capture
	- captures can assert purity of functions
	- use captures on global functions for doc purposes and stuff (easily removable for dev-ex)
	- a serious version of captures that enforces that callees (sub-calls) are also subject to the same or tighter insurance policy - a hard restriction, a no-negotiation compile-time thread-safety mechanism
	- 

Performant code:
- outer scope to be allocated on the stack

> elide - игнорировать (ctx: not use the return value of a func-call)
> tack (on) - добавлять, приметать
> nebulous - vague, uncertain, foggy

 
Code hygiene:
- think about how much of the same stuff you'd need to modify if something updated
- don't globalize anything initially bc you never know what you actually need to do. Abstract afterwards
- static function in cpp: for disabling linking a file-scoped functino to a different file.
- remember maturation cycle
- in games the leaf code is probably the code that needs to be performant.
- language has to let you make safety optional for easier and cheaper introduction of it into the code-base


capture example:
```c
													  { ... }
						[captured outer scope subset] { ... }
	 (float x) -> float [captured outer scope subset] { ... }
f := (float x) -> float [captured outer scope subset] { ... }
```

Q&A:
- Contract-oriented programming
- GC until memory marked-up for collection
- different memory-allocators
- coroutines vs threads ???
- drink OOP cool-aid? lean into overload and stuff...
- what to add to std: vecs, matrices?
- language has to have specs about the allocators, memory organization.

---

Actual fucking thoughts on why languages may be considered sucky

CPP
- lambda-func declaration for inline and scoped version has different syntax
- DIFFERENT TYPING SYNTAX is required if you want to pass lambda or a fun-dec function as an argument
	- you have to import std (compile-time increases, error-msg quality)
	- btw rust has different syntax but at least the same type(?)

Java:
- no globals. Jump through a hoop: create a static class
	- but isn't it more documentation-prone tho?
- rust: the same but put it into `unsafe {}` - wrapping a lot of stuff in that will result in throwing away some of the mechanisms - potentially - you'd want by default at compile-time.

