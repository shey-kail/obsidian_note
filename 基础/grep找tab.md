用c风格的字符串$'...'
```bash
grep $'\t' file.txt
```


用perl风格的正则表达式：
```
grep --perl-regex "\t"
grep -P "\t"
```


用sed也行：
```
sed -n '/\t/p' file.txt
```

