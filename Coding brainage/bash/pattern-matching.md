
Проверка строки на соответствие шаблону

```bash
filenames=(file_01 file_02 file_03 yoomba yoomba_2)

for filename in "${filenames[@]}"; do
	if [[ "$filename" == *_0* ]]; then
		echo "$filename"
	fi
done

# file_01
# file_02
# file_03
```

Пройдемся по файлам в папочке
```bash
files=(./folder/with_tests/*)

for ((i=0; i < ${#files[@]}; i++)); do
	echo ${files[i]}
done
```

Оставим от каждого файла только его имя без относительного пути
```bash
for ((i=0; i < ${#files[@]}; i++)); do
	filename=$(basename "${files[i]}")
	echo ${files[i]}
done
```

откусим от имени каждого файла префикс `test_` и добавим дополнительную шаблонную информацию (номер файла в папке)

```bash
for ((i=0; i < ${#files[@]}; i++)); do
	filename=$(basename "${files[i]}")
	echo ${filename#test_}
done
```

Полная спека для получения огрызков "строки"
```bash
${var#pattern} - remove prefix (shortest)
${var##pattern} - remove prefix (longest/greedy)
${var%pattern} - remove suffix (shortest)
${var%%pattern} - remove suffix (longest/greedy)
${var/pattern/replacement} - replace first match
${var//pattern/replacement} - replace all matches
```