### Команды (пригодяться)

**Вывести список пользователей  с passwd**:
```bash
$ cut -d: -f1 /etc/passwd | sort
```

```bash
$  echo {A..Z} | tr -d ' '      # Удалить пробелы
ABCDEFGHIJKLMNOPQRSTUVWXYZ 
$  echo {A..С} | tr ' ' '\n'    # Заменить на символ новой строки  
A
B
C
```

`find` +  *удаление*
```bash
$  find $HOME/Pictures/Screenshots -type f -name "*.png" -exec echo rm {} ";"  # Используем echo для вывода списка удаляеммых
rm /home/vslsk/Pictures/Screenshots/snapshot_2026-01-01_01-37-25.png

$  find $HOME/Pictures/Screenshots -type f -name "*.png" -exec rm {} ";" # Удалить по-настоящему
```
