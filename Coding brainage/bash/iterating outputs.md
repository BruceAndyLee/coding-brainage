```bash

files_str=$(ls -la | grep "Dec 28")

# to get only the name of the la -la
# files_str=$(ls -la | grep "Dec 28" | awk '{print $9}')

files=($files_str)

for file in "${files[@]}"; do
	echo file
done
```