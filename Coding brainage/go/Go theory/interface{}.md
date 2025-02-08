---
tags:
  - cheatsheets
---
1. It’s a smallest building block in Go. It’s size is literally 0 bytes.
2. If it has zero size. you may create a slice of 1000’s empty  
    structures and this slice will be very tiny. Because really Go stores  
    only a number of them in the slice but not them itself. The same story  
    with channels.  
    
3. All pointers to it always point to the same special place in memory.
4. Very useful in channels when you notify about some event but you  
    don’t need to pass any information about it, only a fact. Best solution  
    is to pass an empty structure because it will only increment a counter  
    in the channel but not assign memory, copy elements and so on.  
    Sometime people use Boolean values for this purpose, but it’s much  
    worse.  
    
5. Zero size container for methods. You may want have a mock for  
    testing interfaces. Often you don’t need data on it, just methods with  
    predefined input and output.  
    
6. Go has no `Set` object. But can be easily realized as a `map[keyType]struct{}`. This way map keeps only keys and no values.
7. An empty struct is used as a type to implement an interface. This is seen in receiver methods.