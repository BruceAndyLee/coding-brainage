В задаче есть преамбула: робот встает на нужное место и стреляет только когда космические крысы проходят перед ним шеренгой

0. Переиспользуемые вещи теперь можно выносить в константы
```toml
const tile_empty 0
const tile_crate 40
const tile_rat 33

const cmd_l 0b00000000
const cmd_f 0b00000001
const cmd_r 0b00000010
const cmd_w 0b00000011
const cmd_a 0b00000100
const cmd_s 0b00000101

const exe cpy | r0 | out
const see cpy | in | r2d
```

чтобы удобнее было ждать и стрелять несколько шагов подряд, вводится конструкция цикл:
1. в начале проги один раз заносим 1 в R2, чтобы использовать его как декремент в цикле
```toml
1 # decrement
cpy | r0 | r2d
```
- теперь можно будет положить длину цикла в r1 и после необходимого действия вычитать декремент из r1
- результат вычитания будет записан в r3 по дизайну арифметических команд
- r3 сравнится с нулем и при недостижении нуля команда сравнения переключит счетчик на начало цикла

2. Пример цикла
```toml
4 # длина цикла
cpy | r0 | r1d # заносится в r1

cmd_s # тело цикла
exe # тело цикла

sub # уменьшаем счетчик цикла
cpy | r3s | r1d # обновляем значение в регистре

20 # адрес первой инструкции в теле цикла (cmd_s в данном случае)
jg # проверяем достиг ли счетчик нуля, если нет, то прыгаем на адрес первой инструкции в теле цикла
```

3. Конечная программа переиспользует подход с циклом несколько раз
```toml
const tile_empty 0
const tile_crate 40
const tile_rat 33

const cmd_l 0b00000000
const cmd_f 0b00000001
const cmd_r 0b00000010
const cmd_w 0b00000011
const cmd_a 0b00000100
const cmd_s 0b00000101

const exe cpy | r0 | out
const see cpy | in | r2d

# 
cmd_r
exe

cmd_f
exe
exe

cmd_l
exe

cmd_f
exe
exe
exe
exe
exe

cmd_w
exe
exe

1 # prepare decrement
cpy | r0 | r2d # for all cycles

# shoot
4 # cycle length
cpy | r0 | r1d
cmd_s
exe
20 # iteration start to r0
sub # decrement
cpy | r3s | r1d
jg

11 # cycle length
cpy | r0 | r1d
cmd_w
exe
28 # iteration start to r0
sub # decrement
cpy | r3s | r1d
jg

# shoot again
4 # cycle length
cpy | r0 | r1d
cmd_s
exe
36 # iteration start to r0
sub # decrement
cpy | r3s | r1d
jg

# the long wait of 20
20 # cycle length
cpy | r0 | r1d
cmd_w
exe
44 # iteration start to r0
sub # decrement
cpy | r3s | r1d
jg

16 # return to the outer cycle
jump

```