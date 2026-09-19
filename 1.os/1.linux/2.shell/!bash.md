# Bash — шпаргалка

> Практическая шпаргалка по Bash: от базового синтаксиса до рабочих скриптов.
> 
> Цель: знать конструкции, которые покрывают большую часть повседневной автоматизации Linux/macOS.

---

# Содержание

[[#1. Запуск Bash-скрипта]]
[[#2. Комментарии]]
[[#3. Вывод]]
[[#4. Переменные]]
[[#5. Кавычки]]
[[#6. Спецпеременные]]
[[#7. Команды внутри переменных]]
[[#8. Математика]]
[[#9. Условия if]]
[[#10. Проверка файлов]]
[[#11. Сравнение строк]]
[[#12. Сравнение чисел]]
[[#13. case]]
[[#14. Цикл for]]
[[#15. Цикл while]]
[[#16. Чтение файла построчно]]
[[#17. break / continue]]
[[#18. Функции]]
[[#19. Аргументы скрипта]]
[[#20. getopts]]
[[#21. Массивы]]
[[#22. Ассоциативные массивы]]
[[#23. Строки]]
[[#24. Параметры по умолчанию]]
[[#25. Код возврата]]
[[#26. set -euo pipefail]]
[[#27. Перенаправление]]
[[#28. Pipeline]]
[[#29. grep]]
[[#30. find]]
[[#31. find + действия]]
[[#32. sed]]
[[#33. awk]]
[[#34. sort / uniq / wc]]
[[#35. cut / tr]]
[[#36. xargs]]
[[#37. Here Document]]
[[#38. Временные файлы]]
[[#39. trap]]
[[#40. Логи]]
[[#41. Проверка команд]]
[[#42. Процессы]]
[[#43. Фоновые процессы]]
[[#44. SSH]]
[[#45. scp / rsync]]
[[#46. tar]]
[[#47. curl]]
[[#48. jq]]
[[#49. Cron]]
[[#50. systemd]]
[[#51. Отладка]]
[[#52. ShellCheck]]
[[#53. Безопасный Bash]]
[[#54. Практический шаблон Bash-скрипта]]
[[#55. Проверка сервиса]]
[[#56. Проверка URL]]
[[#57. Проверка диска]]
[[#58. Backup]]
[[#59. Backup + удаление старых файлов]]
[[#60. Обработка всех файлов]]
[[#61. Поиск больших файлов]]
[[#62. Поиск ошибок в логах]]
[[#63. Самые частые ошибки]]
[[#64. Проверка нескольких серверов]]
[[#65. Выполнить команду на нескольких серверах]]
[[#66. Lock от повторного запуска]]
[[#67. Ограничение времени]]
[[#68. Process substitution]]
[[#70. Git из Bash]]
[[#71. Deploy-скрипт]]
[[#72. Bash + JSON API]]
[[#73. Bash + SQL]]
[[#74. Чего НЕ стоит делать]]
[[#75. Мини-справочник операторов]]
[[#76. Самые полезные конструкции — в одном месте]]
[[#77. Универсальный шаблон скрипта]]
[[#78. Что выучить в первую очередь]]
[[#79. Ментальная модель Bash]]
[[#80. Практический порядок изучения]]
[[#81. Главный принцип Bash]]
[[#82. Финальный чеклист перед запуском скрипта]]
[[#83. Самая короткая шпаргалка]]
[[#84. Золотые правила Bash]]
[[#85. Минимальный набор для 90% задач]]
[[#Итог]]


---

# 1. Запуск Bash-скрипта

Файл:

```bash
#!/usr/bin/env bash

echo "Hello"
```

Сделать исполняемым:

```bash
chmod +x script.sh
```

Запустить:

```bash
./script.sh
```

Или:

```bash
bash script.sh
```

---

# 2. Комментарии

Однострочный комментарий:

```bash
# Это комментарий
```

Пример:

```bash
#!/usr/bin/env bash

# Создаём каталог
mkdir -p backup
```

---

# 3. Вывод

```bash
echo "Hello"
```

Лучше для форматированного вывода:

```bash
printf 'Hello, %s\n' "$name"
```

Несколько строк:

```bash
printf '%s\n' \
    "Line 1" \
    "Line 2" \
    "Line 3"
```

---

# 4. Переменные

Создание:

```bash
name="Alex"
age=30
```

Использование:

```bash
echo "$name"
echo "$age"
```

### Важно

Пробелы вокруг `=` нельзя:

```bash
name="Alex"   # правильно
```

```bash
name = "Alex" # неправильно
```

### Константа

```bash
readonly APP_NAME="myapp"
```

---

# 5. Кавычки

## Двойные кавычки

Переменные раскрываются:

```bash
name="Alex"

echo "Hello $name"
```

Результат:

```text
Hello Alex
```

## Одинарные кавычки

Переменные не раскрываются:

```bash
echo 'Hello $name'
```

Результат:

```text
Hello $name
```

## Главное правило

Переменные почти всегда заключай в:

```bash
"$variable"
```

Например:

```bash
rm -- "$file"
cp "$source" "$destination"
echo "$name"
```

---

# 6. Спецпеременные

```bash
$0      # имя скрипта
$1      # первый аргумент
$2      # второй аргумент
$#      # количество аргументов
$@      # все аргументы
$?      # код возврата последней команды
$$      # PID текущего процесса
$!      # PID последнего фонового процесса
$HOME   # домашний каталог
$USER   # текущий пользователь
$PWD    # текущий каталог
$PATH   # PATH
```

Пример:

```bash
echo "Script: $0"
echo "Arg 1: $1"
echo "Args: $#"
```

---

# 7. Команды внутри переменных

Используй:

```bash
result="$(command)"
```

Например:

```bash
current_dir="$(pwd)"
today="$(date +%F)"
hostname="$(hostname)"
```

Пример:

```bash
echo "Host: $(hostname)"
echo "Date: $(date +%F)"
```

---

# 8. Математика

В Bash:

```bash
a=10
b=20

sum=$((a + b))

echo "$sum"
```

Операции:

```bash
a=$((10 + 5))
a=$((10 - 5))
a=$((10 * 5))
a=$((10 / 5))
a=$((10 % 3))
```

Увеличить:

```bash
((counter++))
```

Уменьшить:

```bash
((counter--))
```

---

# 9. Условия if

Базовый синтаксис:

```bash
if [[ condition ]]; then
    command
fi
```

Полный:

```bash
if [[ condition ]]; then
    command
elif [[ another_condition ]]; then
    command
else
    command
fi
```

Пример:

```bash
if [[ "$USER" == "root" ]]; then
    echo "Ты root"
else
    echo "Ты не root"
fi
```

---

# 10. Проверка файлов

|Проверка|Значение|
|---|---|
|`-e`|существует|
|`-f`|обычный файл|
|`-d`|каталог|
|`-r`|доступен для чтения|
|`-w`|доступен для записи|
|`-x`|исполняемый|
|`-s`|не пустой|

Примеры:

```bash
if [[ -f "$file" ]]; then
    echo "Файл существует"
fi
```

```bash
if [[ -d "$dir" ]]; then
    echo "Каталог существует"
fi
```

```bash
if [[ ! -f "$file" ]]; then
    echo "Файла нет"
fi
```

---

# 11. Сравнение строк

Равно:

```bash
if [[ "$name" == "Alex" ]]; then
    echo "Hello Alex"
fi
```

Не равно:

```bash
if [[ "$name" != "Alex" ]]; then
    echo "Not Alex"
fi
```

Пустая:

```bash
if [[ -z "$name" ]]; then
    echo "Empty"
fi
```

Не пустая:

```bash
if [[ -n "$name" ]]; then
    echo "Not empty"
fi
```

---

# 12. Сравнение чисел

Лучше использовать:

```bash
(( ... ))
```

Пример:

```bash
a=10
b=20

if (( a < b )); then
    echo "a меньше b"
fi
```

Операторы:

```text
==    равно
!=    не равно
<     меньше
>     больше
<=    меньше или равно
>=    больше или равно
```

Пример:

```bash
if (( $# < 1 )); then
    echo "Нужен аргумент"
    exit 1
fi
```

---

# 13. case

Когда много вариантов:

```bash
case "$1" in
    start)
        echo "Starting"
        ;;
    stop)
        echo "Stopping"
        ;;
    restart)
        echo "Restarting"
        ;;
    *)
        echo "Unknown command"
        exit 1
        ;;
esac
```

Использование:

```bash
./app.sh start
./app.sh stop
./app.sh restart
```

---

# 14. Цикл for

Простой:

```bash
for item in one two three; do
    echo "$item"
done
```

Числа:

```bash
for i in {1..10}; do
    echo "$i"
done
```

С шагом:

```bash
for i in {0..20..2}; do
    echo "$i"
done
```

Файлы:

```bash
for file in *.txt; do
    echo "$file"
done
```

Аргументы:

```bash
for arg in "$@"; do
    echo "$arg"
done
```

---

# 15. Цикл while

```bash
counter=1

while (( counter <= 5 )); do
    echo "$counter"
    ((counter++))
done
```

---

# 16. Чтение файла построчно

Правильный базовый вариант:

```bash
while IFS= read -r line; do
    echo "$line"
done < file.txt
```

Почему `IFS=` и `-r`:

- сохраняются пробелы;
    
- не обрабатывается `\` как escape.
    

---

# 17. break / continue

`break` — выйти из цикла:

```bash
for i in {1..10}; do
    if (( i == 5 )); then
        break
    fi

    echo "$i"
done
```

`continue` — перейти к следующей итерации:

```bash
for i in {1..10}; do
    if (( i % 2 == 0 )); then
        continue
    fi

    echo "$i"
done
```

Результат:

```text
1
3
5
7
9
```

---

# 18. Функции

Создание:

```bash
hello() {
    echo "Hello"
}
```

Вызов:

```bash
hello
```

Аргументы:

```bash
greet() {
    local name="$1"

    echo "Hello, $name"
}

greet "Alex"
```

### `local`

Всегда старайся использовать:

```bash
local variable="value"
```

внутри функции.

---

# 19. Аргументы скрипта

Скрипт:

```bash
#!/usr/bin/env bash

echo "First: $1"
echo "Second: $2"
```

Запуск:

```bash
./script.sh hello world
```

Все аргументы:

```bash
for arg in "$@"; do
    echo "$arg"
done
```

Количество:

```bash
echo "$#"
```

Проверка:

```bash
if (( $# != 2 )); then
    echo "Usage: $0 <source> <destination>"
    exit 1
fi
```

---

# 20. getopts

Для параметров вида:

```bash
./script.sh -v -f file.txt
```

Пример:

```bash
#!/usr/bin/env bash

set -euo pipefail

verbose=false
file=""

while getopts "vf:" opt; do
    case "$opt" in
        v)
            verbose=true
            ;;
        f)
            file="$OPTARG"
            ;;
        *)
            echo "Usage: $0 [-v] -f file"
            exit 1
            ;;
    esac
done

echo "file=$file"
echo "verbose=$verbose"
```

---

# 21. Массивы

Создание:

```bash
servers=("web1" "web2" "web3")
```

Получить элемент:

```bash
echo "${servers[0]}"
```

Все элементы:

```bash
printf '%s\n' "${servers[@]}"
```

Количество:

```bash
echo "${#servers[@]}"
```

Добавить:

```bash
servers+=("web4")
```

Цикл:

```bash
for server in "${servers[@]}"; do
    echo "$server"
done
```

### Важно

Используй:

```bash
"${array[@]}"
```

а не:

```bash
$array
```

---

# 22. Ассоциативные массивы

```bash
declare -A ports

ports[ssh]=22
ports[http]=80
ports[https]=443
```

Получить:

```bash
echo "${ports[https]}"
```

Перебрать:

```bash
for service in "${!ports[@]}"; do
    echo "$service -> ${ports[$service]}"
done
```

---

# 23. Строки

```bash
text="Hello World"
```

Длина:

```bash
echo "${#text}"
```

Замена:

```bash
echo "${text/World/Bash}"
```

Заменить все:

```bash
echo "${text//World/Bash}"
```

Начинается с:

```bash
[[ "$text" == Hello* ]]
```

Заканчивается:

```bash
[[ "$text" == *World ]]
```

Убрать расширение:

```bash
file="report.txt"

name="${file%.txt}"
```

Получить имя файла:

```bash
path="/home/user/file.txt"

basename="${path##*/}"
```

Результат:

```text
file.txt
```

---

# 24. Параметры по умолчанию

Если переменная не задана или пустая:

```bash
name="${NAME:-Guest}"
```

Если переменная не задана — ошибка:

```bash
: "${API_TOKEN:?API_TOKEN is required}"
```

Проверить:

```bash
if [[ -z "${API_TOKEN:-}" ]]; then
    echo "API_TOKEN не задан"
    exit 1
fi
```

---

# 25. Код возврата

В Unix:

```text
0     успех
!= 0  ошибка
```

Пример:

```bash
ls /tmp

echo "$?"
```

Проверка:

```bash
if command; then
    echo "OK"
else
    echo "ERROR"
fi
```

Или:

```bash
if ! command; then
    echo "Ошибка" >&2
    exit 1
fi
```

---

# 26. set -euo pipefail

Для большинства серьёзных скриптов:

```bash
set -euo pipefail
```

### `-e`

Остановиться при ошибке.

### `-u`

Ошибка при использовании несуществующей переменной.

### `pipefail`

Ошибка внутри pipeline не будет потеряна.

Типичный старт:

```bash
#!/usr/bin/env bash

set -euo pipefail
```

---

# 27. Перенаправление

stdout:

```bash
command > output.txt
```

Добавить:

```bash
command >> output.txt
```

stderr:

```bash
command 2> error.log
```

stdout + stderr:

```bash
command > output.log 2>&1
```

В Bash:

```bash
command &> output.log
```

Выкинуть вывод:

```bash
command >/dev/null 2>&1
```

---

# 28. Pipeline

Передать результат команды следующей:

```bash
ps aux | grep nginx
```

Например:

```bash
cat access.log | grep ERROR
```

Но если файл можно передать напрямую:

```bash
grep ERROR access.log
```

Ещё:

```bash
ps aux |
    grep nginx |
    grep -v grep
```

---

# 29. grep

Поиск:

```bash
grep "ERROR" app.log
```

Без регистра:

```bash
grep -i "error" app.log
```

С номером строки:

```bash
grep -n "ERROR" app.log
```

Рекурсивно:

```bash
grep -R "TODO" .
```

Исключить:

```bash
grep -v "DEBUG" app.log
```

Несколько вариантов:

```bash
grep -E 'ERROR|WARN' app.log
```

Только имена файлов:

```bash
grep -l "ERROR" *.log
```

---

# 30. find

Найти все `.log`:

```bash
find . -type f -name "*.log"
```

Каталоги:

```bash
find . -type d
```

Файлы больше 100 MB:

```bash
find . -type f -size +100M
```

Изменённые за последние сутки:

```bash
find . -type f -mtime -1
```

Старше 30 дней:

```bash
find . -type f -mtime +30
```

---

# 31. find + действия

Сначала проверить:

```bash
find /tmp -type f -name "*.tmp" -print
```

Удалить:

```bash
find /tmp -type f -name "*.tmp" -delete
```

Выполнить команду:

```bash
find . -type f -name "*.log" -exec gzip {} \;
```

---

# 32. sed

Замена:

```bash
sed 's/foo/bar/g' file.txt
```

Удалить строки:

```bash
sed '/DEBUG/d' app.log
```

Показать строки 10–20:

```bash
sed -n '10,20p' file.txt
```

Изменить файл:

```bash
sed -i 's/foo/bar/g' file.txt
```

> `sed -i` отличается между GNU/Linux и macOS.

---

# 33. awk

Вывести первую колонку:

```bash
awk '{print $1}' file.txt
```

Вторую:

```bash
awk '{print $2}' file.txt
```

Первая + вторая:

```bash
awk '{print $1, $2}' file.txt
```

Условие:

```bash
awk '$2 > 100 {print $1}' file.txt
```

Сумма:

```bash
awk '{sum += $2} END {print sum}' file.txt
```

---

# 34. sort / uniq / wc

Сортировка:

```bash
sort file.txt
```

Обратная:

```bash
sort -r file.txt
```

Числовая:

```bash
sort -n numbers.txt
```

Уникальные:

```bash
sort file.txt | uniq
```

Количество повторений:

```bash
sort file.txt | uniq -c
```

Самые частые:

```bash
sort file.txt |
    uniq -c |
    sort -nr
```

Количество строк:

```bash
wc -l file.txt
```

Количество слов:

```bash
wc -w file.txt
```

---

# 35. cut / tr

`cut`:

```bash
cut -d: -f1 /etc/passwd
```

Здесь:

```text
-d:   разделитель :
-f1   первая колонка
```

Первые 10 символов:

```bash
cut -c1-10 file.txt
```

`tr`:

```bash
echo "hello" | tr 'a-z' 'A-Z'
```

Результат:

```text
HELLO
```

Удалить цифры:

```bash
echo "abc123" | tr -d '0-9'
```

---

# 36. xargs

Например:

```bash
find . -type f -name "*.log" -print0 |
    xargs -0 wc -l
```

`-print0` и `-0` позволяют безопаснее обрабатывать имена с пробелами.

---

# 37. Here Document

```bash
cat <<EOF
Hello
World
User: $USER
EOF
```

Переменные раскрываются.

Без раскрытия:

```bash
cat <<'EOF'
Hello
$USER
EOF
```

---

# 38. Временные файлы

Плохо:

```bash
tmp="/tmp/myfile"
```

Лучше:

```bash
tmp="$(mktemp)"
```

Пример:

```bash
tmp="$(mktemp)"

echo "data" > "$tmp"

cat "$tmp"

rm -f "$tmp"
```

Для каталога:

```bash
tmp_dir="$(mktemp -d)"
```

---

# 39. trap

Автоматическая очистка:

```bash
tmp="$(mktemp)"

cleanup() {
    rm -f "$tmp"
}

trap cleanup EXIT
```

Полный пример:

```bash
#!/usr/bin/env bash

set -euo pipefail

tmp="$(mktemp)"

cleanup() {
    rm -f "$tmp"
}

trap cleanup EXIT

echo "Работаем..."
```

---

# 40. Логи

Функция:

```bash
log() {
    printf '[%s] %s\n' "$(date '+%F %T')" "$*"
}
```

Использование:

```bash
log "Script started"
log "Processing files"
log "Done"
```

Ошибка:

```bash
error() {
    printf '[%s] ERROR: %s\n' \
        "$(date '+%F %T')" \
        "$*" >&2
}
```

---

# 41. Проверка команд

Проверить наличие:

```bash
command -v curl
```

В условии:

```bash
if command -v jq >/dev/null 2>&1; then
    echo "jq installed"
else
    echo "jq missing"
fi
```

Функция:

```bash
require_command() {
    command -v "$1" >/dev/null 2>&1 || {
        echo "Required command not found: $1" >&2
        exit 1
    }
}

require_command curl
require_command jq
require_command git
```

---

# 42. Процессы

Все процессы:

```bash
ps aux
```

Поиск:

```bash
pgrep -a nginx
```

Проверить:

```bash
if pgrep -x nginx >/dev/null; then
    echo "nginx running"
fi
```

Завершить:

```bash
kill PID
```

Принудительно:

```bash
kill -9 PID
```

> Сначала используй обычный `kill`. `kill -9` — крайний вариант.

---

# 43. Фоновые процессы

Запустить в фоне:

```bash
long_command &
```

Получить PID:

```bash
long_command &
pid=$!
```

Дождаться:

```bash
wait "$pid"
```

Несколько задач:

```bash
task1 &
pid1=$!

task2 &
pid2=$!

wait "$pid1"
wait "$pid2"
```

---

# 44. SSH

Подключиться:

```bash
ssh user@server
```

Выполнить команду:

```bash
ssh user@server 'hostname'
```

Несколько:

```bash
ssh user@server '
    cd /app
    git pull --ff-only
    ./deploy.sh
'
```

---

# 45. scp / rsync

Скопировать файл:

```bash
scp file.txt user@server:/tmp/
```

Каталог:

```bash
scp -r ./project user@server:/tmp/
```

`rsync`:

```bash
rsync -av ./project/ user@server:/app/
```

Проверить без изменений:

```bash
rsync -av --dry-run ./project/ user@server:/app/
```

Синхронизация с удалением:

```bash
rsync -av --delete ./project/ user@server:/app/
```

> Перед `--delete` обязательно проверь направление и сделай `--dry-run`.

---

# 46. tar

Создать архив:

```bash
tar -czf backup.tar.gz project/
```

Распаковать:

```bash
tar -xzf backup.tar.gz
```

Посмотреть:

```bash
tar -tzf backup.tar.gz
```

---

# 47. curl

GET:

```bash
curl https://example.com
```

Скачать:

```bash
curl -o file.html https://example.com
```

Следовать redirect:

```bash
curl -L https://example.com
```

Для скриптов:

```bash
curl -fsS https://example.com
```

Основные параметры:

```text
-f   ошибка при HTTP 4xx/5xx
-s   silent
-S   показывать ошибки вместе с -s
-L   redirects
-o   output
-I   только headers
```

POST JSON:

```bash
curl -fsS \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"name":"Alex"}' \
    https://example.com/api
```

---

# 48. jq

Для JSON используй `jq`.

Получить поле:

```bash
jq -r '.name' data.json
```

Массив:

```bash
jq -r '.users[] | .name' data.json
```

Фильтр:

```bash
jq -r '.users[] | select(.active == true) | .name' data.json
```

API:

```bash
response="$(curl -fsS https://example.com/api)"

status="$(echo "$response" | jq -r '.status')"

if [[ "$status" == "ok" ]]; then
    echo "OK"
fi
```

---

# 49. Cron

Посмотреть:

```bash
crontab -l
```

Редактировать:

```bash
crontab -e
```

Каждый день в 03:00:

```cron
0 3 * * * /home/user/scripts/backup.sh >> /home/user/logs/backup.log 2>&1
```

Каждые 5 минут:

```cron
*/5 * * * * /home/user/scripts/check.sh
```

Формат:

```text
┌──────── минута
│ ┌────── час
│ │ ┌──── день месяца
│ │ │ ┌── месяц
│ │ │ │ ┌ день недели
│ │ │ │ │
* * * * *
```

Для cron используй абсолютные пути.

---

# 50. systemd

Service:

```ini
[Unit]
Description=My App

[Service]
ExecStart=/usr/local/bin/myapp.sh
Restart=on-failure
```

Timer:

```ini
[Unit]
Description=Backup timer

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

После изменения:

```bash
sudo systemctl daemon-reload
```

Запустить:

```bash
sudo systemctl enable --now backup.timer
```

Проверить:

```bash
systemctl status backup.timer
```

Логи:

```bash
journalctl -u backup.service
```

---

# 51. Отладка

Проверка синтаксиса:

```bash
bash -n script.sh
```

Запустить с debug:

```bash
bash -x script.sh
```

Включить внутри:

```bash
set -x
```

Выключить:

```bash
set +x
```

---

# 52. ShellCheck

Установи ShellCheck и запускай:

```bash
shellcheck script.sh
```

Это одна из самых полезных привычек при написании Bash.

---

# 53. Безопасный Bash

## Всегда кавычь переменные

Хорошо:

```bash
rm -- "$file"
```

Плохо:

```bash
rm $file
```

---

## Используй `[[ ]]`

Хорошо:

```bash
if [[ -f "$file" ]]; then
```

---

## Для чисел используй `(( ))`

```bash
if (( count > 10 )); then
```

---

## Проверяй входные данные

```bash
[[ -f "$file" ]] || {
    echo "File not found: $file" >&2
    exit 1
}
```

---

## Не используй `eval` без крайней необходимости

Плохо:

```bash
eval "$user_input"
```

---

## Перед удалением проверяй

Сначала:

```bash
find "$dir" -type f -name "*.tmp" -print
```

Потом:

```bash
find "$dir" -type f -name "*.tmp" -delete
```

---

# 54. Практический шаблон Bash-скрипта

Это один из главных шаблонов, который можно копировать:

```bash
#!/usr/bin/env bash

set -euo pipefail

readonly SCRIPT_NAME="$(basename "$0")"

log() {
    printf '[%s] %s\n' "$(date '+%F %T')" "$*"
}

die() {
    printf '[%s] ERROR: %s\n' \
        "$(date '+%F %T')" \
        "$*" >&2

    exit 1
}

usage() {
    cat <<EOF
Usage:
    $SCRIPT_NAME <file>

Example:
    $SCRIPT_NAME data.txt
EOF
}

if (( $# != 1 )); then
    usage
    exit 1
fi

file="$1"

[[ -f "$file" ]] ||
    die "File not found: $file"

log "Processing: $file"

# Основная логика

log "Done"
```

---

# 55. Проверка сервиса

```bash
#!/usr/bin/env bash

set -euo pipefail

service="nginx"

if systemctl is-active --quiet "$service"; then
    echo "$service: OK"
else
    echo "$service: DOWN" >&2
    exit 1
fi
```

---

# 56. Проверка URL

```bash
#!/usr/bin/env bash

set -euo pipefail

url="${1:-}"

if [[ -z "$url" ]]; then
    echo "Usage: $0 <url>"
    exit 1
fi

if curl -fsS --max-time 10 "$url" >/dev/null; then
    echo "OK: $url"
else
    echo "FAIL: $url" >&2
    exit 1
fi
```

Запуск:

```bash
./check-url.sh https://example.com
```

---

# 57. Проверка диска

```bash
#!/usr/bin/env bash

set -euo pipefail

threshold=80

usage="$(
    df / |
        awk 'NR==2 {
            gsub("%", "", $5)
            print $5
        }'
)"

if (( usage >= threshold )); then
    echo "WARNING: disk usage ${usage}%"
    exit 1
fi

echo "Disk usage: ${usage}%"
```

---

# 58. Backup

```bash
#!/usr/bin/env bash

set -euo pipefail

SOURCE="/home/user/project"
DEST="/backup"

mkdir -p "$DEST"

timestamp="$(date '+%Y%m%d-%H%M%S')"
archive="$DEST/project-$timestamp.tar.gz"

tar -czf "$archive" "$SOURCE"

echo "Created: $archive"
```

---

# 59. Backup + удаление старых файлов

```bash
#!/usr/bin/env bash

set -euo pipefail

SOURCE="/home/user/project"
DEST="/backup"

mkdir -p "$DEST"

timestamp="$(date '+%Y%m%d-%H%M%S')"
archive="$DEST/project-$timestamp.tar.gz"

tar -czf "$archive" "$SOURCE"

find "$DEST" \
    -type f \
    -name 'project-*.tar.gz' \
    -mtime +30 \
    -delete

echo "Backup: $archive"
```

---

# 60. Обработка всех файлов

```bash
#!/usr/bin/env bash

set -euo pipefail

directory="${1:-.}"

while IFS= read -r -d '' file; do
    echo "Processing: $file"

    # обработка "$file"

done < <(
    find "$directory" -type f -print0
)
```

---

# 61. Поиск больших файлов

```bash
find /var -type f -size +500M -print
```

На GNU/Linux можно получить размер:

```bash
find /var -type f -printf '%s %p\n' 2>/dev/null |
    sort -nr |
    head -20
```

---

# 62. Поиск ошибок в логах

```bash
grep -E 'ERROR|CRITICAL' app.log
```

Количество:

```bash
grep -Ec 'ERROR|CRITICAL' app.log
```

Последние 100:

```bash
grep -E 'ERROR|CRITICAL' app.log | tail -100
```

---

# 63. Самые частые ошибки

```bash
grep "ERROR" app.log |
    sed 's/.*ERROR: //' |
    sort |
    uniq -c |
    sort -nr |
    head -20
```

---

# 64. Проверка нескольких серверов

```bash
#!/usr/bin/env bash

set -euo pipefail

servers=(
    web1
    web2
    web3
)

for server in "${servers[@]}"; do
    if ssh -o ConnectTimeout=5 "$server" 'echo OK' >/dev/null; then
        echo "$server: OK"
    else
        echo "$server: FAIL"
    fi
done
```

---

# 65. Выполнить команду на нескольких серверах

```bash
servers=(
    server1
    server2
    server3
)

for server in "${servers[@]}"; do
    echo "=== $server ==="

    ssh "$server" '
        hostname
        uptime
        df -h /
    '
done
```

---

# 66. Lock от повторного запуска

Если скрипт запускается через cron:

```bash
exec 9>/tmp/my-script.lock

if ! flock -n 9; then
    echo "Already running"
    exit 1
fi
```

Теперь второй экземпляр не запустится одновременно.

---

# 67. Ограничение времени

```bash
timeout 30s ./script.sh
```

Например:

```bash
if timeout 10s curl -fsS https://example.com >/dev/null; then
    echo "OK"
else
    echo "Timeout or error"
fi
```

---

# 68. Process substitution

Можно передать результат команды как файл:

```bash
diff <(sort file1.txt) <(sort file2.txt)
```

Ещё:

```bash
while IFS= read -r line; do
    echo "$line"
done < <(find . -type f)
```

---

# 69. Работа с переменными окружения

Посмотреть:

```bash
env
```

Получить:

```bash
echo "$HOME"
echo "$PATH"
```

Экспортировать:

```bash
export API_URL="https://example.com"
```

Дочерние процессы увидят переменную.

---

# 70. Git из Bash

Текущая ветка:

```bash
git branch --show-current
```

Есть ли изменения:

```bash
if [[ -n "$(git status --porcelain)" ]]; then
    echo "Changes exist"
fi
```

Последний commit:

```bash
git log -1 --oneline
```

---

# 71. Deploy-скрипт

```bash
#!/usr/bin/env bash

set -euo pipefail

APP_DIR="/opt/myapp"

cd "$APP_DIR"

echo "Pulling..."
git pull --ff-only

echo "Installing..."
npm ci

echo "Restarting..."
sudo systemctl restart myapp

echo "Checking..."

if systemctl is-active --quiet myapp; then
    echo "Deploy successful"
else
    echo "Deploy failed" >&2
    exit 1
fi
```

---

# 72. Bash + JSON API

```bash
#!/usr/bin/env bash

set -euo pipefail

API_URL="https://example.com/api"

response="$(curl -fsS "$API_URL")"

status="$(jq -r '.status' <<< "$response")"

if [[ "$status" == "ok" ]]; then
    echo "API OK"
else
    echo "API ERROR"
    exit 1
fi
```

---

# 73. Bash + SQL

Если нужно выполнить SQL:

```bash
psql "$DATABASE_URL" <<'SQL'
SELECT
    id,
    name
FROM users
LIMIT 10;
SQL
```

Обрати внимание на:

```bash
<<'SQL'
```

Если переменные Bash не должны раскрываться.

---

# 74. Чего НЕ стоит делать

## Не парси `ls`

Плохо:

```bash
for file in $(ls); do
    ...
done
```

Лучше:

```bash
for file in *; do
    ...
done
```

или:

```bash
find . -type f
```

---

## Не делай `cat | grep`

Плохо:

```bash
cat file.txt | grep ERROR
```

Лучше:

```bash
grep ERROR file.txt
```

---

## Не используй `$@` без кавычек

Плохо:

```bash
for arg in $@; do
```

Хорошо:

```bash
for arg in "$@"; do
```

---

## Не используй `eval`

Без крайней необходимости:

```bash
eval "$command"
```

не нужен.

---

# 75. Мини-справочник операторов

## Логические

```bash
&&      # AND
||      # OR
!       # NOT
```

Примеры:

```bash
command1 && command2
```

```bash
command1 || command2
```

```bash
if ! command; then
    echo "failed"
fi
```

---

## Файлы

```bash
-f file
-d dir
-e path
-r file
-w file
-x file
-s file
```

---

## Строки

```bash
[[ "$a" == "$b" ]]
[[ "$a" != "$b" ]]
[[ -z "$a" ]]
[[ -n "$a" ]]
```

---

## Числа

```bash
(( a == b ))
(( a != b ))
(( a > b ))
(( a < b ))
(( a >= b ))
(( a <= b ))
```

---

# 76. Самые полезные конструкции — в одном месте

## Переменная

```bash
name="Alex"
```

## Команда → переменная

```bash
result="$(command)"
```

## Условие

```bash
if [[ condition ]]; then
    ...
fi
```

## Числовое условие

```bash
if (( count > 10 )); then
    ...
fi
```

## Цикл

```bash
for item in "${items[@]}"; do
    ...
done
```

## While

```bash
while condition; do
    ...
done
```

## Функция

```bash
function_name() {
    local value="$1"
    ...
}
```

## Проверка ошибки

```bash
if ! command; then
    ...
fi
```

## Выход

```bash
exit 0
```

или ошибка:

```bash
exit 1
```

## Лог

```bash
printf '[%s] %s\n' "$(date '+%F %T')" "$message"
```

## Временный файл

```bash
tmp="$(mktemp)"
```

## Очистка

```bash
trap 'rm -f "$tmp"' EXIT
```

---

# 77. Универсальный шаблон скрипта

Если не знаешь, с чего начать новый скрипт — копируй это:

```bash
#!/usr/bin/env bash

set -euo pipefail

readonly SCRIPT_NAME="$(basename "$0")"

log() {
    printf '[%s] %s\n' "$(date '+%F %T')" "$*"
}

die() {
    printf '[%s] ERROR: %s\n' \
        "$(date '+%F %T')" \
        "$*" >&2
    exit 1
}

usage() {
    cat <<EOF
Usage:
    $SCRIPT_NAME <argument>

Example:
    $SCRIPT_NAME test
EOF
}

# -------------------------
# Проверка аргументов
# -------------------------

if (( $# < 1 )); then
    usage
    exit 1
fi

argument="$1"

# -------------------------
# Проверка зависимостей
# -------------------------

require_command() {
    command -v "$1" >/dev/null 2>&1 ||
        die "Command not found: $1"
}

# require_command curl
# require_command jq

# -------------------------
# Основная логика
# -------------------------

log "Started"

echo "Argument: $argument"

# ...

log "Finished"
```

---

# 78. Что выучить в первую очередь

Если не хочется учить всё сразу, запомни сначала эти конструкции:

```bash
#!/usr/bin/env bash
set -euo pipefail
```

```bash
variable="value"
```

```bash
echo "$variable"
```

```bash
result="$(command)"
```

```bash
if [[ condition ]]; then
    ...
fi
```

```bash
if (( number > 10 )); then
    ...
fi
```

```bash
for item in "${array[@]}"; do
    ...
done
```

```bash
while IFS= read -r line; do
    ...
done < file
```

```bash
function_name() {
    local value="$1"
}
```

```bash
case "$1" in
    start) ... ;;
    stop)  ... ;;
    *)     ... ;;
esac
```

```bash
find . -type f -name "*.log"
```

```bash
grep "ERROR" file.log
```

```bash
sed 's/old/new/g' file
```

```bash
awk '{print $1}' file
```

```bash
command1 | command2 | command3
```

```bash
command > file
```

```bash
command >> file
```

```bash
command >/dev/null 2>&1
```

```bash
ssh user@server 'command'
```

```bash
curl -fsS URL
```

```bash
jq -r '.field' file.json
```

---

# 79. Ментальная модель Bash

Большинство скриптов можно мыслить так:

```text
Вход
 │
 ├── аргументы
 ├── переменные окружения
 ├── файлы
 └── API
       │
       ▼
   Проверка
       │
       ▼
   Основная логика
       │
       ├── if
       ├── case
       ├── for
       ├── while
       └── functions
       │
       ▼
   Команды Linux
       │
       ├── grep
       ├── find
       ├── sed
       ├── awk
       ├── curl
       ├── ssh
       └── systemctl
       │
       ▼
   Результат
       │
       ├── stdout
       ├── файл
       ├── exit code
       └── лог
```

---

# 80. Практический порядок изучения

Рекомендуемый порядок:

```text
1. Команды Linux
      ↓
2. Переменные
      ↓
3. Кавычки
      ↓
4. if
      ↓
5. for
      ↓
6. while
      ↓
7. функции
      ↓
8. аргументы
      ↓
9. массивы
      ↓
10. grep
      ↓
11. find
      ↓
12. sed
      ↓
13. awk
      ↓
14. pipeline
      ↓
15. redirection
      ↓
16. trap
      ↓
17. обработка ошибок
      ↓
18. SSH / rsync
      ↓
19. curl / jq
      ↓
20. cron / systemd
```

---

# 81. Главный принцип Bash

Не пытайся сделать всё на Bash.

Bash особенно хорош, когда задача выглядит примерно так:

```text
найти файлы
    ↓
проверить условие
    ↓
запустить команду
    ↓
отфильтровать результат
    ↓
сохранить результат
    ↓
записать лог
```

Например:

```bash
find /var/log \
    -type f \
    -name "*.log" \
    -mtime +30 \
    -print
```

или:

```bash
grep ERROR app.log |
    awk '{print $1}' |
    sort |
    uniq -c |
    sort -nr |
    head
```

Когда логика превращается в большое количество сложных структур данных, парсинг или бизнес-логику — обычно пора переходить на Python/Go/другой язык, а Bash оставить для orchestration.

---

# 82. Финальный чеклист перед запуском скрипта

Перед production-запуском:

```text
[ ] Есть #!/usr/bin/env bash
[ ] Есть set -euo pipefail
[ ] Переменные заключены в "$..."
[ ] Используется [[ ]] для условий
[ ] Числа сравниваются через (( ))
[ ] Проверены аргументы
[ ] Проверены входные файлы
[ ] Проверены необходимые команды
[ ] Ошибки идут в stderr
[ ] Есть понятные exit codes
[ ] Временные файлы очищаются через trap
[ ] Удаление сначала протестировано через -print
[ ] Нет eval
[ ] Нет опасного парсинга ls
[ ] Проверен ShellCheck
[ ] Скрипт протестирован на копии данных
```

---

# 83. Самая короткая шпаргалка

```bash
#!/usr/bin/env bash
set -euo pipefail

# variable
name="Alex"

# command
date_now="$(date '+%F %T')"

# if
if [[ -f "$file" ]]; then
    echo "file"
fi

# numeric
if (( count > 10 )); then
    echo "many"
fi

# for
for file in *.txt; do
    echo "$file"
done

# while
while IFS= read -r line; do
    echo "$line"
done < file.txt

# function
greet() {
    local name="$1"
    echo "Hello $name"
}

# case
case "$1" in
    start)  echo "start" ;;
    stop)   echo "stop" ;;
    *)      echo "unknown" ;;
esac

# find
find . -type f -name "*.log"

# grep
grep -n "ERROR" app.log

# sed
sed 's/old/new/g' file

# awk
awk '{print $1}' file

# pipeline
grep ERROR app.log | sort | uniq -c | sort -nr

# redirect
command > output.log 2>&1

# temp file
tmp="$(mktemp)"
trap 'rm -f "$tmp"' EXIT

# ssh
ssh user@server 'hostname'

# curl
curl -fsS https://example.com

# json
jq -r '.name' data.json

# exit
exit 0
```

---

# 84. Золотые правила Bash

1. **Всегда думай о пробелах в именах файлов.**
    
2. **Кавычь переменные:** `"$var"`.
    
3. **Используй `[[ ]]` для условий Bash.**
    
4. **Используй `(( ))` для арифметики.**
    
5. **Используй `"$@"`, а не `$@`.**
    
6. **Не парсь `ls`.**
    
7. **Не используй `eval`, если можно обойтись без него.**
    
8. **Для JSON используй `jq`.**
    
9. **Для поиска файлов используй `find`.**
    
10. **Для сложной обработки текста используй `awk`/`sed`, а не огромные цепочки `grep`.**
    
11. **Перед опасным `rm` сначала делай `-print`.**
    
12. **Используй `set -euo pipefail`, понимая его особенности.**
    
13. **Используй `shellcheck`.**
    
14. **Для временных файлов используй `mktemp`.**
    
15. **Для cleanup используй `trap`.**
    
16. **Для повторяемых задач используй функции.**
    
17. **Для сложных скриптов делай `usage`.**
    
18. **Не запускай скрипты от root без необходимости.**
    
19. **Проверяй команды и входные данные.**
    
20. **Если Bash начинает превращаться в полноценную программу — рассмотрись Python/Go.**
    

---

# 85. Минимальный набор для 90% задач

Если нужно запомнить буквально **20 вещей**, запомни:

```bash
#!/usr/bin/env bash
set -euo pipefail
```

```bash
"$variable"
```

```bash
result="$(command)"
```

```bash
if [[ ... ]]; then ... fi
```

```bash
if (( ... )); then ... fi
```

```bash
for x in "${array[@]}"; do ... done
```

```bash
while IFS= read -r line; do ... done < file
```

```bash
function_name() { ... }
```

```bash
case "$1" in ... esac
```

```bash
find ...
```

```bash
grep ...
```

```bash
sed ...
```

```bash
awk ...
```

```bash
command1 | command2
```

```bash
command > file
```

```bash
command 2> error.log
```

```bash
command >/dev/null 2>&1
```

```bash
tmp="$(mktemp)"
trap 'rm -f "$tmp"' EXIT
```

```bash
ssh user@server 'command'
```

```bash
curl -fsS URL
```

---

## Итог

Для практического Bash тебе не нужно помнить сотни команд наизусть.

Достаточно уверенно владеть:

```text
переменные
    +
if / case
    +
for / while
    +
функции
    +
аргументы
    +
массивы
    +
pipes
    +
redirects
    +
grep / find / sed / awk
    +
curl / jq
    +
ssh / rsync
    +
trap / обработка ошибок
```

А остальные команды можно находить по мере необходимости через:

```bash
man command
```

или:

```bash
command --help
```

и проверять готовый код через:

```bash
shellcheck script.sh
```