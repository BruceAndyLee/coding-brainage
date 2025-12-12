```bash
while $# -gt 0; do
	case $1 in
		case1|case11|case12)
			# case1-specific actions
			;;
		case2|case21|case22)
			# case2-specific actions
			;;
	esac
done
```

пример с обработкой флагов скрипта
```bash
SOURCE="default/destination/"
DESTINATION="default/destination/"

while $# -gt 0; do
	case $1 in
		"-s"|"--source")
			shift # make is so that $1 points to the next argument
			SOURCE=$1
			shift # shift for the next iteration of while to take a look at the next flag/argument
			;;
		"-d"|"--destination")
			shift
			DESTINATION=$1
			shift
			;;
	esac
done

# the actual script
cp $SOURCE $DESTINATION

```

пример с передачей всех аргументов скрипта в функцию для более явного разделения ответственности
```bash
SOURCE="default/destination/"
DESTINATION="default/destination/"

parse_arguments() {
	while $# -gt 0; do
		case $1 in
			"-s"|"--source")
				shift # make is so that $1 points to the next argument
				SOURCE=$1
				shift # shift for the next iteration of while to take a look at the next flag/argument
				;;
			"-d"|"--destination")
				shift
				DESTINATION=$1
				shift
				;;
		esac
	done
}


# the actual script
parse_arguments $*

cp $SOURCE $DESTINATION
```