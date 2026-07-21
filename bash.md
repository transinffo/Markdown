# Полезные команды bash


## Вывести файлы и каталоги с размером выше 100 Мб

```bash
find . \( -type f -o -type d \) -exec du -sb {} + 2>/dev/null | awk '$1 > 100*1024*1024' | numfmt --field=1 --to=iec
```


## Вывести дерево каталогов\файлов кроме node_modules

```bash
//tree - вывод всего
//tree -I 'node_modules|test' - если не хотим выводить несколько
tree -I 'node_modules'
```

