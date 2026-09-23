# Bash: академическое практическое руководство

> **Версия документа:** 1.0  
> **Назначение:** систематическое изучение Bash от основ shell до разработки надёжных production-скриптов.  
> **Охват:** Bash 5.x как основной ориентир; отдельно отмечены особенности переносимости между Linux, macOS и разными версиями Bash.

---

## Содержание

1. [Что такое Bash](#1-что-такое-bash)
2. [Версии и совместимость](#2-версии-и-совместимость)
3. [Запуск скриптов](#3-запуск-скриптов)
4. [Комментарии, команды и exit status](#4-комментарии-команды-и-exit-status)
5. [Переменные и окружение](#5-переменные-и-окружение)
6. [Кавычки и разбор команд](#6-кавычки-и-разбор-команд)
7. [Подстановки и расширения](#7-подстановки-и-расширения)
8. [Специальные параметры и аргументы](#8-специальные-параметры-и-аргументы)
9. [Условия: `test`, `[ ]`, `[[ ]]`](#9-условия-test---)
10. [Арифметика](#10-арифметика)
11. [Массивы](#11-массивы)
12. [Строки и parameter expansion](#12-строки-и-parameter-expansion)
13. [Globbing](#13-globbing)
14. [Управляющие конструкции](#14-управляющие-конструкции)
15. [Функции](#15-функции)
16. [Потоки, pipeline и файловые дескрипторы](#16-потоки-pipeline-и-файловые-дескрипторы)
17. [Here documents, here strings и process substitution](#17-here-documents-here-strings-и-process-substitution)
18. [Подshell и группировка команд](#18-подshell-и-группировка-команд)
19. [Фоновые процессы и job control](#19-фоновые-процессы-и-job-control)
20. [Сигналы и `trap`](#20-сигналы-и-trap)
21. [`set`, `shopt` и режимы Bash](#21-set-shopt-и-режимы-bash)
22. [Обработка ошибок](#22-обработка-ошибок)
23. [CLI-интерфейсы и `getopts`](#23-cli-интерфейсы-и-getopts)
24. [Работа с файлами и каталогами](#24-работа-с-файлами-и-каталогами)
25. [Текстовые утилиты](#25-текстовые-утилиты)
26. [`find` и `xargs`](#26-find-и-xargs)
27. [Процессы](#27-процессы)
28. [Права и безопасность](#28-права-и-безопасность)
29. [SSH, rsync и tar](#29-ssh-rsync-и-tar)
30. [HTTP, JSON и API](#30-http-json-и-api)
31. [Git из Bash](#31-git-из-bash)
32. [Cron, systemd и launchd](#32-cron-systemd-и-launchd)
33. [Временные файлы, блокировки и timeout](#33-временные-файлы-блокировки-и-timeout)
34. [Логирование](#34-логирование)
35. [Отладка и статический анализ](#35-отладка-и-статический-анализ)
36. [Переносимость Linux/macOS](#36-переносимость-linuxmacos)
37. [Производительность](#37-производительность)
38. [Антипаттерны](#38-антипаттерны)
39. [Production-шаблон](#39-production-шаблон)
40. [Тестирование и идемпотентность](#40-тестирование-и-идемпотентность)
41. [Когда Bash перестаёт быть подходящим инструментом](#41-когда-bash-перестаёт-быть-подходящим-инструментом)
42. [Краткий справочник](#42-краткий-справочник)

---

# 1. Что такое Bash

**Bash (Bourne Again SHell)** — командный интерпретатор и язык сценариев семейства Unix. Он используется одновременно как:

- интерактивная оболочка;
- интерпретатор shell-скриптов;
- средство композиции Unix-команд;
- язык автоматизации;
- инструмент системного администрирования и orchestration.

Важно различать **Bash** и **Unix/GNU utilities**. Bash предоставляет shell-язык, расширения, переменные, функции, условия, циклы, job control и т. д. Утилиты `grep`, `sed`, `awk`, `find`, `tar`, `curl`, `jq`, `rsync` и `git` являются отдельными программами.

---

# 2. Версии и совместимость

Проверка версии:

```bash
bash --version
echo "$BASH_VERSION"
```

Возможности Bash зависят от версии. Ассоциативные массивы требуют Bash 4.0+.

## macOS

Системный Bash в старых версиях macOS — Bash 3.2. Поэтому скрипт, использующий возможности Bash 4/5, должен либо явно требовать современный Bash, либо учитывать совместимость.

Проверка:

```bash
command -v bash
bash --version
```

Если нужна конкретная версия, shebang:

```bash
#!/usr/bin/env bash
```

не гарантирует конкретную версию — он выбирает `bash` из `PATH`.

## GNU и BSD

Даже если shell один и тот же, утилиты могут различаться.

Например, `sed -i` имеет различия между GNU `sed` и BSD/macOS `sed`.

Поэтому переносимость следует рассматривать на двух уровнях:

1. совместимость Bash;
2. совместимость внешних Unix-утилит.

---

# 3. Запуск скриптов

Минимальный скрипт:

```bash
#!/usr/bin/env bash

printf '%s\n' "Hello, Bash"
```

Права:

```bash
chmod +x script.sh
```

Запуск:

```bash
./script.sh
```

или:

```bash
bash script.sh
```

Это не полностью эквивалентные способы.

`./script.sh` использует shebang, а:

```bash
bash script.sh
```

явно запускает файл интерпретатором `bash`.

Проверка синтаксиса без запуска:

```bash
bash -n script.sh
```

---

# 4. Комментарии, команды и exit status

Комментарий начинается с `#`:

```bash
# Это комментарий
printf '%s\n' "hello"
```

Каждая команда имеет **exit status** — целое число от 0 до 255.

По соглашению:

```text
0     успех
!= 0  ошибка или специальное состояние
```

Проверка:

```bash
some_command
status=$?
printf 'status=%s\n' "$status"
```

Условие часто проверяется непосредственно:

```bash
if some_command; then
    printf '%s\n' "success"
fi
```

Не следует без необходимости делать:

```bash
some_command
if [[ $? -eq 0 ]]; then
    ...
fi
```

Непосредственная форма обычно яснее.

---

# 5. Переменные и окружение

Присваивание:

```bash
name="Alex"
age=30
```

Пробелы вокруг `=` недопустимы:

```bash
# Неправильно
name = "Alex"
```

Локальная переменная shell:

```bash
NAME="Alex"
```

Экспорт в окружение дочерних процессов:

```bash
export NAME="Alex"
```

Проверка:

```bash
printf '%s\n' "$NAME"
env
printenv
```

Локальная переменная функции:

```bash
greet() {
    local name="$1"
    printf 'Hello, %s\n' "$name"
}
```

Константа:

```bash
readonly APP_NAME="myapp"
```

Удаление:

```bash
unset NAME
```

Проверка наличия команды:

```bash
command -v bash
command -v jq
```

---

# 6. Кавычки и разбор команд

Кавычки — фундаментальная тема Bash.

## Двойные кавычки

```bash
name="Alex"
printf '%s\n' "$name"
```

Переменная раскрывается, но word splitting и pathname expansion внутри двойных кавычек не выполняются.

## Одинарные кавычки

```bash
printf '%s\n' '$HOME'
```

Содержимое трактуется буквально.

## Главное правило

Практически всегда:

```bash
"$variable"
```

а не:

```bash
$variable
```

Например:

```bash
file="my document.txt"
cat "$file"
```

Без кавычек shell может разделить значение по пробелам.

---

# 7. Подстановки и расширения

Bash выполняет несколько видов expansion.

## Command substitution

```bash
current_dir="$(pwd)"
today="$(date +%F)"
```

Старый синтаксис:

```bash
`pwd`
```

не рекомендуется; используйте `$(...)`.

## Arithmetic expansion

```bash
a=10
b=20
sum=$((a + b))
```

## Parameter expansion

```bash
name="${NAME:-Guest}"
```

## Tilde expansion

```bash
cd ~/projects
```

## Pathname expansion (globbing)

```bash
echo *.txt
```

---

# 8. Специальные параметры и аргументы

Основные параметры:

| Параметр | Значение |
|---|---|
| `$0` | имя/путь запускаемого скрипта |
| `$1`, `$2`, ... | позиционные аргументы |
| `$#` | количество позиционных аргументов |
| `"$@"` | позиционные аргументы как отдельные слова |
| `"$*"` | позиционные аргументы, объединённые в одно расширение |
| `$?` | status предыдущей команды |
| `$$` | PID текущего shell-процесса |
| `$!` | PID последнего фонового процесса |
| `$-` | текущие shell options |
| `$?` | exit status последней команды |

### `$@` и `$*`

Предпочтительная передача всех аргументов:

```bash
for arg in "$@"; do
    printf '<%s>\n' "$arg"
done
```

`"$@"` сохраняет границы аргументов.

`"$*"` обычно объединяет их в одну строку с первым символом `IFS`.

### `!$`

Не путать:

```bash
$!
```

с:

```bash
!$
```

`$!` — PID последнего background-процесса.

`!$` — history expansion в интерактивном Bash и не является обычной переменной shell-скрипта.

---

# 9. Условия: `test`, `[ ]`, `[[ ]]`

Для современного Bash предпочтительна конструкция:

```bash
if [[ condition ]]; then
    ...
fi
```

## Строки

```bash
[[ "$a" == "$b" ]]
[[ "$a" != "$b" ]]
[[ -z "$a" ]]
[[ -n "$a" ]]
```

## Файлы

| Оператор | Значение |
|---|---|
| `-e` | существует |
| `-f` | обычный файл |
| `-d` | каталог |
| `-r` | доступен для чтения |
| `-w` | доступен для записи |
| `-x` | исполняемый |
| `-s` | размер больше нуля |
| `-L` | symbolic link |

Пример:

```bash
if [[ -f "$file" ]]; then
    printf '%s\n' "File exists"
fi
```

## Числа

В Bash:

```bash
if (( a < b )); then
    ...
fi
```

Не следует использовать `[[ a < b ]]` как числовое сравнение.

## `[[ ]]` против `[ ]`

`[[ ]]` — Bash-конструкция с более безопасной и богатой семантикой.

`[ ]` — команда `test`/её синтаксическая форма, более близкая к POSIX shell.

---

# 10. Арифметика

Bash поддерживает целочисленную арифметику.

```bash
a=10
b=3

printf '%s\n' "$((a + b))"
printf '%s\n' "$((a - b))"
printf '%s\n' "$((a * b))"
printf '%s\n' "$((a / b))"
printf '%s\n' "$((a % b))"
```

Арифметический контекст:

```bash
((counter++))
((counter += 10))

if (( counter >= 100 )); then
    ...
fi
```

Bash не является языком для произвольной точной арифметики с плавающей точкой. Для неё обычно используют `awk`, `bc`, Python и т. п.

---

# 11. Массивы

## Индексированный массив

```bash
servers=("web1" "web2" "web3")

printf '%s\n' "${servers[0]}"
printf '%s\n' "${servers[@]}"
printf '%s\n' "${#servers[@]}"

servers+=("web4")
```

Итерация:

```bash
for server in "${servers[@]}"; do
    printf '%s\n' "$server"
done
```

## Ассоциативный массив

Требует Bash 4+:

```bash
declare -A ports

ports[ssh]=22
ports[http]=80

for service in "${!ports[@]}"; do
    printf '%s -> %s\n' \
        "$service" \
        "${ports[$service]}"
done
```

## `[@]` и `[*]`

В двойных кавычках:

```bash
"${array[@]}"
```

раскрывает элементы массива как отдельные слова.

```bash
"${array[*]}"
```

объединяет элементы в одно слово с разделителем `IFS`.

Для передачи элементов массива обычно используйте:

```bash
"${array[@]}"
```

---

# 12. Строки и parameter expansion

Длина:

```bash
text="Hello Bash"
printf '%s\n' "${#text}"
```

Замена первого совпадения:

```bash
printf '%s\n' "${text/World/Bash}"
```

Замена всех:

```bash
printf '%s\n' "${text//World/Bash}"
```

Удаление префикса:

```bash
"${path#*/}"
"${path##*/}"
```

Удаление суффикса:

```bash
"${file%.txt}"
"${file%%.tar.gz}"
```

Подстрока:

```bash
"${text:0:5}"
```

Значение по умолчанию:

```bash
"${name:-Guest}"
```

Назначение значения по умолчанию:

```bash
"${name:=Guest}"
```

Использовать альтернативу:

```bash
"${name:+defined}"
```

Ошибка, если переменная отсутствует/пуста:

```bash
: "${API_TOKEN:?API_TOKEN is required}"
```

---

# 13. Globbing

Основные шаблоны:

```bash
*
?
[abc]
[a-z]
[^a]
```

Пример:

```bash
for file in *.log; do
    printf '%s\n' "$file"
done
```

## `nullglob`

По умолчанию при отсутствии совпадений:

```bash
*.log
```

может остаться буквальным шаблоном.

Безопаснее:

```bash
shopt -s nullglob
files=( *.log )
```

## `globstar`

Рекурсивный glob:

```bash
shopt -s globstar

for file in **/*.txt; do
    ...
done
```

## `dotglob`

Позволяет glob-выражениям учитывать скрытые файлы:

```bash
shopt -s dotglob
```

Изменение `shopt` следует делать осознанно: поведение globbing становится частью контракта скрипта.

---

# 14. Управляющие конструкции

## `if`

```bash
if [[ -f "$file" ]]; then
    printf '%s\n' "file"
elif [[ -d "$file" ]]; then
    printf '%s\n' "directory"
else
    printf '%s\n' "missing"
fi
```

## `case`

```bash
case "${1:-}" in
    start)
        start_service
        ;;
    stop)
        stop_service
        ;;
    restart)
        restart_service
        ;;
    *)
        printf 'Usage: %s {start|stop|restart}\n' "$0" >&2
        exit 2
        ;;
esac
```

## `for`

```bash
for item in one two three; do
    printf '%s\n' "$item"
done
```

C-style:

```bash
for ((i = 0; i < 10; i++)); do
    printf '%s\n' "$i"
done
```

## `while`

```bash
while (( counter < 10 )); do
    ((counter++))
done
```

## `until`

```bash
until command; do
    sleep 1
done
```

## Чтение файла

Надёжная форма:

```bash
while IFS= read -r line; do
    printf '%s\n' "$line"
done < file.txt
```

`IFS=` предотвращает удаление ведущих/конечных разделителей, а `-r` запрещает `read` интерпретировать обратные слэши как escape-последовательности.

## Управление циклом

```bash
break
continue
```

---

# 15. Функции

Объявление:

```bash
greet() {
    local name="$1"
    printf 'Hello, %s\n' "$name"
}
```

Вызов:

```bash
greet "Alex"
```

Функция получает собственные позиционные параметры:

```bash
process() {
    local first="$1"
    local second="$2"
}
```

Возврат status:

```bash
validate() {
    [[ -n "$1" ]]
}
```

или:

```bash
validate() {
    if [[ -n "$1" ]]; then
        return 0
    fi
    return 1
}
```

Результат функции обычно передают через stdout:

```bash
result="$(calculate)"
```

а не через глобальную переменную.

---

# 16. Потоки, pipeline и файловые дескрипторы

Стандартные потоки:

```text
0 stdin
1 stdout
2 stderr
```

Перенаправление stdout:

```bash
command > output.log
```

Дозапись:

```bash
command >> output.log
```

stderr:

```bash
command 2> error.log
```

stdout + stderr:

```bash
command > output.log 2>&1
```

В Bash также:

```bash
command &> output.log
```

Подавление:

```bash
command >/dev/null 2>&1
```

## Pipeline

```bash
grep 'ERROR' app.log | sort | uniq -c
```

Pipeline передаёт stdout одной команды на stdin следующей.

## `PIPESTATUS`

После pipeline:

```bash
command1 | command2 | command3

printf '%s\n' "${PIPESTATUS[@]}"
```

`PIPESTATUS` содержит status каждого элемента pipeline.

## `pipefail`

```bash
set -o pipefail
```

Тогда pipeline считается неуспешным, если неуспешен соответствующий компонент согласно правилам `pipefail`, а не только последняя команда.

---

# 17. Here documents, here strings и process substitution

## Here document

```bash
cat <<EOF
User: $USER
Home: $HOME
EOF
```

Кавычки у delimiter отключают интерполяцию:

```bash
cat <<'EOF'
$USER
$HOME
EOF
```

## Here string

```bash
read -r value <<< "$input"
```

## Process substitution

Позволяет представить результат команды как файловый интерфейс:

```bash
diff <(sort file1) <(sort file2)
```

Также:

```bash
while IFS= read -r line; do
    ...
done < <(find . -type f)
```

Process substitution — Bash-функция и не является POSIX shell feature.

---

# 18. Подshell и группировка команд

## Subshell

Скобки создают subshell:

```bash
(
    cd /tmp
    printf '%s\n' "$PWD"
)
```

Изменение каталога внутри не изменяет текущий shell.

## Group command

Фигурные скобки выполняют команды в текущем shell:

```bash
{
    printf '%s\n' "one"
    printf '%s\n' "two"
}
```

Нужно учитывать синтаксис:

```bash
{
    command
}
```

Последняя команда перед `}` должна быть завершена `;` или переводом строки.

## Pipeline и subshell

Конструкция:

```bash
cat file | while IFS= read -r line; do
    ...
done
```

может выполнять `while` в subshell в зависимости от shell/контекста. Поэтому для изменения переменных после цикла предпочтительно:

```bash
while IFS= read -r line; do
    ...
done < file
```

---

# 19. Фоновые процессы и job control

Запуск в фоне:

```bash
long_command &
```

PID:

```bash
long_command &
pid=$!
```

Ожидание:

```bash
wait "$pid"
```

Несколько процессов:

```bash
cmd1 &
pid1=$!

cmd2 &
pid2=$!

wait "$pid1"
wait "$pid2"
```

Интерактивный job control:

```bash
jobs
fg
bg
```

`Ctrl-Z` приостанавливает foreground job.

Другие механизмы:

```bash
nohup command &
disown
```

Современный Bash также предоставляет:

```bash
wait -n
```

для ожидания завершения любого из ожидаемых jobs/processes.

---

# 20. Сигналы и `trap`

Часто используемые сигналы:

```text
SIGINT   Ctrl-C
SIGTERM  корректное завершение
SIGHUP   hangup
```

Обработчик:

```bash
cleanup() {
    rm -f "$tmp"
}

trap cleanup EXIT
```

Несколько событий:

```bash
trap 'cleanup' EXIT INT TERM
```

Для серьёзных скриптов предпочтительнее именованная функция:

```bash
cleanup() {
    ...
}

trap cleanup EXIT
```

а не сложная строка кода внутри `trap`.

`EXIT` — специальное событие Bash, выполняющее trap при завершении shell.

---

# 21. `set`, `shopt` и режимы Bash

Проверка:

```bash
set -o
```

Популярные параметры:

```bash
set -e
set -u
set -o pipefail
```

## `set -e`

Просит shell завершаться при определённых ненулевых status. Это полезно, но **не является простым правилом «любая ошибка немедленно завершает скрипт»**. У `errexit` есть исключения и контекстные правила, особенно внутри `if`, `while`, `until`, `&&`, `||`, `!`, pipeline и некоторых подстановок.

Поэтому `set -e` не заменяет явное управление ошибками.

## `set -u`

Обращение к неустановленной переменной может завершить shell:

```bash
set -u
```

Для необязательных переменных используйте:

```bash
"${OPTIONAL:-}"
```

## `pipefail`

```bash
set -o pipefail
```

Комбинация:

```bash
set -euo pipefail
```

часто используется как стартовая конфигурация production-скрипта, но требует понимания её семантики.

## `shopt`

Bash-specific options:

```bash
shopt
shopt -s nullglob
shopt -s globstar
shopt -s dotglob
shopt -s extglob
```

---

# 22. Обработка ошибок

Надёжный скрипт должен различать:

1. ожидаемое условие;
2. recoverable error;
3. fatal error.

Пример:

```bash
if ! curl -fsS --max-time 10 "$url" -o "$output"; then
    printf '%s\n' "Download failed" >&2
    exit 1
fi
```

Для диагностики полезно:

```bash
printf 'ERROR: %s\n' "$message" >&2
```

Не следует полагаться исключительно на `$?` спустя несколько команд — status нужно проверять сразу или использовать условную конструкцию.

---

# 23. CLI-интерфейсы и `getopts`

Для простых аргументов:

```bash
if (( $# < 1 )); then
    printf 'Usage: %s FILE\n' "$0" >&2
    exit 2
fi
```

Для short options:

```bash
while getopts ':f:v' opt; do
    case "$opt" in
        f)
            file="$OPTARG"
            ;;
        v)
            verbose=true
            ;;
        \?)
            printf 'Invalid option: -%s\n' "$OPTARG" >&2
            exit 2
            ;;
        :)
            printf 'Option -%s requires an argument\n' "$OPTARG" >&2
            exit 2
            ;;
    esac
done

shift $((OPTIND - 1))
```

`getopts` — стандартный и переносимый способ обработки short options в shell.

Для сложных CLI с длинными GNU-style options (`--foo`, `--bar=value`) часто требуется собственный parser или внешняя программа.

---

# 24. Работа с файлами и каталогами

Создание:

```bash
mkdir -p "$dir"
touch "$file"
```

Удаление:

```bash
rm -- "$file"
rm -rf -- "$dir"
```

Перемещение:

```bash
mv -- "$source" "$destination"
```

Копирование:

```bash
cp -- "$source" "$destination"
```

Права:

```bash
chmod 600 "$file"
chmod +x "$script"
```

Не используйте `rm -rf` с непроверенными путями.

Особенно опасны конструкции, где пустая переменная может привести к неожиданному пути. Валидируйте критические пути до destructive operation.

---

# 25. Текстовые утилиты

## `grep`

```bash
grep 'pattern' file
grep -i 'pattern' file
grep -n 'pattern' file
grep -v 'pattern' file
grep -R 'pattern' directory
grep -E 'regex' file
```

## `sed`

Замена:

```bash
sed 's/foo/bar/g' file
```

Удаление:

```bash
sed '/DEBUG/d' file
```

Диапазон:

```bash
sed -n '10,20p' file
```

`-i` требует внимания к различиям GNU/BSD.

## `awk`

Первая колонка:

```bash
awk '{print $1}' file
```

Условие:

```bash
awk '$2 > 100' file
```

Агрегация:

```bash
awk '{sum += $2} END {print sum}' file
```

## Остальные

```bash
sort
uniq
wc
cut
tr
head
tail
tee
xargs
```

Примеры:

```bash
sort -n numbers.txt
uniq -c
wc -l file.txt
cut -d: -f1 /etc/passwd
tr 'a-z' 'A-Z'
```

---

# 26. `find` и `xargs`

Поиск:

```bash
find . -type f -name '*.log'
```

Размер:

```bash
find . -type f -size +100M
```

Возраст:

```bash
find . -type f -mtime -1
```

Команда:

```bash
find . -type f -name '*.log' -exec gzip {} \;
```

Удаление:

```bash
find /tmp -type f -name '*.tmp' -delete
```

Перед destructive operation полезно:

```bash
find /tmp -type f -name '*.tmp' -print
```

## `-print0` и `xargs -0`

Безопаснее для имён с пробелами, переводами строк и специальными символами:

```bash
find . -type f -print0 |
    xargs -0r command
```

---

# 27. Процессы

Просмотр:

```bash
ps aux
```

Поиск:

```bash
pgrep -a nginx
```

Завершение:

```bash
kill "$pid"
```

`SIGTERM` является предпочтительным первым сигналом для корректного завершения:

```bash
kill -TERM "$pid"
```

`SIGKILL`:

```bash
kill -KILL "$pid"
```

или:

```bash
kill -9 "$pid"
```

не позволяет процессу выполнить cleanup и поэтому является последним средством.

---

# 28. Права и безопасность

Unix-модель прав включает:

```text
user
group
other
```

Проверка:

```bash
ls -l file
```

Типичные права:

```bash
chmod 600 secret
chmod 644 document
chmod 755 executable
```

Не следует хранить секреты непосредственно в исходном коде:

```bash
API_TOKEN="real-secret"
```

Лучше использовать защищённое окружение, secret manager или другой механизм управления секретами.

Особенно опасны:

```bash
eval "$user_input"
```

и выполнение непроверенного ввода через shell.

---

# 29. SSH, rsync и tar

## SSH

```bash
ssh user@server 'hostname'
```

Передача команд:

```bash
ssh user@server 'df -h /'
```

Для сложных сценариев полезно явно контролировать quoting: локальный shell и удалённый shell выполняют свои собственные этапы разбора.

## rsync

```bash
rsync -av source/ destination/
```

Проверка без изменений:

```bash
rsync -av --dry-run source/ destination/
```

Зеркалирование:

```bash
rsync -av --delete source/ destination/
```

`--delete` требует особой осторожности.

## tar

Создание:

```bash
tar -czf archive.tar.gz directory/
```

Распаковка:

```bash
tar -xzf archive.tar.gz
```

Просмотр:

```bash
tar -tzf archive.tar.gz
```

---

# 30. HTTP, JSON и API

`curl`:

```bash
curl -fsS --max-time 10 'https://example.com'
```

POST JSON:

```bash
curl -fsS \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"name":"Alex"}' \
    'https://example.com/api'
```

Для переменных:

```bash
payload=$(jq -n --arg name "$name" '{name: $name}')

curl -fsS \
    -H 'Content-Type: application/json' \
    -d "$payload" \
    "$url"
```

`jq`:

```bash
jq -r '.name'
jq -r '.users[] | select(.active == true) | .name'
```

Не рекомендуется строить сложный JSON простой конкатенацией строк, если данные могут содержать кавычки или специальные символы.

---

# 31. Git из Bash

Проверка состояния:

```bash
git status --porcelain
```

Текущая ветка:

```bash
git branch --show-current
```

Проверка, есть ли изменения:

```bash
if [[ -n "$(git status --porcelain)" ]]; then
    printf '%s\n' "Working tree is not clean" >&2
    exit 1
fi
```

Git-операции в automation следует проверять по exit status, а не только по тексту вывода.

---

# 32. Cron, systemd и launchd

## Cron

Редактор:

```bash
crontab -e
```

Формат:

```text
minute hour day-of-month month day-of-week command
```

Пример:

```cron
0 3 * * * /opt/scripts/backup.sh
```

Для cron используйте абсолютные пути и учитывайте, что окружение cron отличается от интерактивного shell.

## systemd

Linux-система с systemd может использовать `.service` и `.timer`.

После изменения unit:

```bash
sudo systemctl daemon-reload
```

Запуск:

```bash
sudo systemctl enable --now example.service
```

Статус:

```bash
systemctl status example.service
```

Логи:

```bash
journalctl -u example.service
```

## macOS

macOS использует `launchd`, а не systemd.

Инструмент управления:

```bash
launchctl
```

Поэтому документация для Linux и macOS должна разделять systemd и launchd.

---

# 33. Временные файлы, блокировки и timeout

## `mktemp`

```bash
tmp="$(mktemp)"
```

Cleanup:

```bash
cleanup() {
    rm -f -- "$tmp"
}

trap cleanup EXIT
```

Для временного каталога:

```bash
tmpdir="$(mktemp -d)"
trap 'rm -rf -- "$tmpdir"' EXIT
```

При сложной логике лучше хранить cleanup в функции.

## `flock`

На Linux:

```bash
exec 9>/tmp/my-script.lock

if ! flock -n 9; then
    printf '%s\n' "Already running" >&2
    exit 1
fi
```

`flock` не является универсальной возможностью всех Unix/macOS-систем, поэтому переносимость необходимо учитывать.

## Timeout

Если доступна GNU `timeout`:

```bash
timeout 30s ./script.sh
```

Это также не универсальная POSIX-команда.

---

# 34. Логирование

Простой logger:

```bash
log() {
    printf '[%s] %s\n' "$(date '+%F %T')" "$*"
}

die() {
    printf '[%s] ERROR: %s\n' "$(date '+%F %T')" "$*" >&2
    exit 1
}
```

Использование:

```bash
log "Starting backup"
die "Backup failed"
```

Для сложных систем логирование может передаваться journald, syslog или внешнему logging stack.

---

# 35. Отладка и статический анализ

## Синтаксис

```bash
bash -n script.sh
```

## Трассировка

```bash
bash -x script.sh
```

или:

```bash
set -x
...
set +x
```

## ShellCheck

```bash
shellcheck script.sh
```

ShellCheck выявляет множество проблем:

- неправильное quoting;
- сомнительные конструкции;
- ошибки shell semantics;
- потенциально опасные расширения;
- некоторые проблемы переносимости.

Отладка и статический анализ дополняют, но не заменяют тестирование.

---

# 36. Переносимость Linux/macOS

Нужно различать:

### Bash portability

Разные версии Bash поддерживают разные возможности.

### Utility portability

GNU и BSD-утилиты могут отличаться.

Примеры потенциальных различий:

```bash
sed
date
grep
find
xargs
readlink
stat
```

Если скрипт должен работать на Linux и macOS, избегайте бездумного использования GNU-only options.

Проверяйте окружение:

```bash
uname -s
bash --version
command -v gsed
```

---

# 37. Производительность

Bash отлично подходит для orchestration, но не всегда эффективен для больших объёмов данных.

Неэффективный подход:

```bash
while read -r line; do
    some_command "$line"
done < huge_file
```

если `some_command` запускается сотни тысяч раз.

Иногда лучше:

- передать данные одной программе;
- использовать `awk`;
- использовать `sort`;
- использовать специализированный инструмент;
- перенести алгоритмически сложную часть в Python/Go/Rust и оставить Bash orchestration.

Также полезно избегать ненужных subprocess:

```bash
value="$(cat file)"
```

обычно можно заменить:

```bash
value="$(< file)"
```

---

# 38. Антипаттерны

## Парсить `ls`

Плохо:

```bash
for file in $(ls); do
    ...
done
```

Лучше glob или `find`.

## Ненужный `cat`

Плохо:

```bash
cat file | grep pattern
```

Лучше:

```bash
grep pattern file
```

Это не «запрещённая» конструкция, а обычно избыточная.

## Некавыченные переменные

Плохо:

```bash
rm $file
```

Лучше:

```bash
rm -- "$file"
```

## `eval`

Опасно:

```bash
eval "$user_input"
```

`eval` повторно интерпретирует строку как shell-код и требует исключительного контроля над входом.

## Парсинг `ps`/`df`/`ls` обычным текстовым split

По возможности используйте предназначенные для машинного использования интерфейсы, специальные флаги или системные API.

## Опасный `rm -rf`

Никогда не выполняйте destructive operation над непроверенным путём.

---

# 39. Production-шаблон

Базовый каркас:

```bash
#!/usr/bin/env bash

set -euo pipefail

readonly SCRIPT_NAME="$(basename "$0")"

log() {
    printf '[%s] %s\n' "$(date '+%F %T')" "$*"
}

die() {
    printf '[%s] ERROR: %s\n' "$(date '+%F %T')" "$*" >&2
    exit 1
}

usage() {
    printf 'Usage: %s ARG\n' "$SCRIPT_NAME" >&2
}

main() {
    if (( $# < 1 )); then
        usage
        exit 2
    fi

    local arg="$1"

    log "Starting"
    log "Argument: $arg"

    # Основная логика.

    log "Finished"
}

main "$@"
```

Почему `main "$@"` полезен:

- отделяет глобальную инициализацию от основной логики;
- локальные переменные функции не загрязняют глобальную область;
- упрощает тестирование;
- делает структуру скрипта очевидной.

---

# 40. Тестирование и идемпотентность

Проверяйте:

1. отсутствие аргументов;
2. неправильные аргументы;
3. отсутствующие файлы;
4. пробелы в именах;
5. пустые строки;
6. специальные символы;
7. недоступную сеть;
8. отсутствие зависимостей;
9. недостаточные права;
10. повторный запуск.

## Идемпотентность

Идемпотентная операция может быть безопасно выполнена повторно без накопления нежелательных побочных эффектов.

Например, вместо безусловного добавления строки:

```bash
printf '%s\n' "option=value" >> config
```

может потребоваться проверка существования записи.

Для systemd:

```bash
systemctl enable service
```

концептуально отличается от сценария, который каждый запуск вручную создаёт новый ресурс.

Тестируйте повторный запуск:

```bash
./script.sh
./script.sh
```

и проверяйте, что состояние системы осталось корректным.

---

# 41. Когда Bash перестаёт быть подходящим инструментом

Bash особенно хорош для:

- запуска и связывания программ;
- файловых операций;
- системного администрирования;
- CI/CD;
- orchestration;
- простых deployment scripts;
- автоматизации Unix.

Рассмотрите другой язык, когда появляются:

- сложные структуры данных;
- большие объёмы данных;
- сложные алгоритмы;
- сложный HTTP-клиент;
- многопоточность;
- сложное тестирование;
- большие объёмы бизнес-логики;
- необходимость строгой типизации.

Практическое правило:

> Bash должен преимущественно **оркестрировать** специализированные инструменты, а не превращаться в большой универсальный application runtime.

---

# 42. Краткий справочник

## Переменные

```bash
name="Alex"
printf '%s\n' "$name"
readonly NAME="value"
export PATH="$HOME/bin:$PATH"
```

## Аргументы

```bash
$0
$1
$#
"$@"
"$*"
$?
$!
$$
```

## Условия

```bash
[[ -f "$file" ]]
[[ "$a" == "$b" ]]
[[ "$text" == prefix* ]]
[[ "$value" =~ regex ]]
(( a < b ))
```

## Циклы

```bash
for x in "${array[@]}"; do
    ...
done

while condition; do
    ...
done

until condition; do
    ...
done
```

## Функции

```bash
function_name() {
    local value="$1"
    ...
}
```

## Файлы

```bash
[[ -e "$path" ]]
[[ -f "$path" ]]
[[ -d "$path" ]]
```

## Redirect

```bash
> file
>> file
2> file
> file 2>&1
&> file
< file
```

## Pipeline

```bash
command1 | command2
```

## Background

```bash
command &
pid=$!
wait "$pid"
```

## Temporary files

```bash
tmp="$(mktemp)"
trap 'rm -f -- "$tmp"' EXIT
```

## Strict-ish baseline

```bash
set -euo pipefail
```

Но `set -e` требует понимания его исключений и не заменяет явную обработку ошибок.

## Проверка

```bash
bash -n script.sh
shellcheck script.sh
bash -x script.sh
```

## Поиск

```bash
find . -type f -name '*.log'
```

## Text processing

```bash
grep
sed
awk
sort
uniq
cut
tr
wc
```

## JSON

```bash
jq
```

## Network

```bash
curl
ssh
rsync
```

## Archive

```bash
tar
```

---

# Заключение

Хороший Bash-скрипт обладает следующими свойствами:

1. Явно указывает используемый интерпретатор.
2. Корректно обрабатывает аргументы.
3. Соблюдает quoting.
4. Проверяет ошибки критических операций.
5. Не доверяет непроверенному пользовательскому вводу.
6. Безопасно работает с именами файлов.
7. Корректно обрабатывает временные ресурсы.
8. Имеет предсказуемый exit status.
9. Проверяется ShellCheck и `bash -n`.
10. Протестирован на ошибочных и граничных сценариях.
11. Учитывает целевую версию Bash.
12. Учитывает различия GNU/BSD и Linux/macOS.
13. Не содержит ненужной бизнес-логики, которую разумнее реализовать в Python, Go или другом подходящем языке.
14. По возможности является идемпотентным.
15. Явно документирует предположения об окружении.

Главная ментальная модель Bash:

```text
Входные данные
      ↓
Разбор shell
      ↓
Expansions
      ↓
Запуск команды
      ↓
stdin / stdout / stderr
      ↓
Exit status
      ↓
Следующая команда / условие / pipeline
```

Для уверенного владения Bash необходимо понимать не только синтаксис отдельных команд, но и **порядок разбора shell-команды, quoting, expansion, word splitting, globbing, процессы, файловые дескрипторы и exit status**. Именно эти механизмы объясняют большинство сложных и труднообнаружимых ошибок shell-скриптов.

