### Команды (пригодяться)

**Вывести список пользователей  с passwd**:
```bash
$ cut -d: -f1 /etc/passwd | sort
```

Сделаю 10 команд
```bash
seq 10 | awk '{print "touch file" $0 ".txt"}' | bash
```


`find` +  *удаление*
```bash
$  find $HOME/Pictures/Screenshots -type f -name "*.png" -exec echo rm {} ";"  # Используем echo для вывода списка удаляеммых
rm /home/vslsk/Pictures/Screenshots/snapshot_2026-01-01_01-37-25.png

$  find $HOME/Pictures/Screenshots -type f -name "*.png" -exec rm {} ";" # Удалить по-настоящему
```


Поиск дублей
```bash
md5sum *.jpg \
| awk '{counts[$1]++; names[$1]=names[$1] " " $2} \
END {for (key in counts) print counts[key] " " key ":" names[key]}' \
| grep -v '^1 ' \
| sort -nr
```