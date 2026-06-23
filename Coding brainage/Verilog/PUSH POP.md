
ASM:
```toml
IA -> 1000 0000 # immediate value 
add 
```


```toml
IA | add 0 0 r0

# start at byte 4
add input r0 r1

# compare input to 0
IB | eq r1 0 24

# push if not 0
# (value is already in r1)
I | add STACK_PUSH 0 r5
I | add 0 0 r5
eq 0 0 4

# pop if 0
I | add STACK_POP 0 r5
I | add 0 0 r5
eq 0 0 4

```