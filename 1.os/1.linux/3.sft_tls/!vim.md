# Vim: академическое практическое руководство

> **Версия документа:** 1.0  
> **Назначение:** систематическое изучение Vim от базовой навигации до продвинутого редактирования, поиска, регистров, макросов, окон, вкладок, командной строки, настроек и автоматизации.  
> **Основной ориентир:** современный Vim 9.x. Большинство базовых возможностей применимо и к Vim 8.x.  
> **Примечание:** Vim и Neovim имеют большое общее ядро команд и концепций, но их конфигурация, расширения и экосистема различаются.

---

## Содержание

1. [Что такое Vim](#1-что-такое-vim)
2. [Установка и проверка](#2-установка-и-проверка)
3. [Ментальная модель Vim](#3-ментальная-модель-vim)
4. [Режимы Vim](#4-режимы-vim)
5. [Запуск и открытие файлов](#5-запуск-и-открытие-файлов)
6. [Выход и сохранение](#6-выход-и-сохранение)
7. [Навигация](#7-навигация)
8. [Основные операции редактирования](#8-основные-операции-редактирования)
9. [Операторы и motions](#9-операторы-и-motions)
10. [Удаление, копирование и вставка](#10-удаление-копирование-и-вставка)
11. [Visual mode](#11-visual-mode)
12. [Отмена и повтор](#12-отмена-и-повтор)
13. [Поиск](#13-поиск)
14. [Замена](#14-замена)
15. [Регистры](#15-регистры)
16. [Макросы](#16-макросы)
17. [Текстовые объекты](#17-текстовые-объекты)
18. [Повторение команд и автоматизация](#18-повторение-команд-и-автоматизация)
19. [Работа с несколькими файлами](#19-работа-с-несколькими-файлами)
20. [Окна и split](#20-окна-и-split)
21. [Табы](#21-табы)
22. [Buffers, windows и tab pages](#22-buffers-windows-и-tab-pages)
23. [Файловая система и рабочий каталог](#23-файловая-система-и-рабочий-каталог)
24. [Командная строка Vim](#24-командная-строка-vim)
25. [Ex-команды](#25-ex-команды)
26. [Флаги и диапазоны команд](#26-флаги-и-диапазоны-команд)
27. [Quickfix и location list](#27-quickfix-и-location-list)
28. [Tags и навигация по коду](#28-tags-и-навигация-по-коду)
29. [Fold](#29-fold)
30. [Indentation](#30-indentation)
31. [Форматирование текста](#31-форматирование-текста)
32. [Spell checking](#32-spell-checking)
33. [Сессии](#33-сессии)
34. [Swap, backup и recovery](#34-swap-backup-и-recovery)
35. [Конфигурация `.vimrc`](#35-конфигурация-vimrc)
36. [Опции Vim](#36-опции-vim)
37. [Mappings](#37-mappings)
38. [Autocommands](#38-autocommands)
39. [Vimscript и Vim 9 Script](#39-vimscript-и-vim-9-script)
40. [Плагины](#40-плагины)
41. [Диагностика и справка](#41-диагностика-и-справка)
42. [Производительность](#42-производительность)
43. [Безопасность](#43-безопасность)
44. [Практический workflow](#44-практический-workflow)
45. [Антипаттерны](#45-антипаттерны)
46. [Краткий cheat sheet](#46-краткий-cheat-sheet)

---

# 1. Что такое Vim

**Vim (Vi IMproved)** — modal text editor, основанный на концепциях `vi`.

Ключевая идея Vim:

> Клавиши описывают **операции над текстом**, а не только непосредственное изменение символов.

Поэтому Vim следует изучать не как набор случайных горячих клавиш, а как композицию:

```text
operator + motion
```

Например:

```text
dw
```

означает:

```text
d = delete
w = движение до следующего слова
```

А:

```text
ci"
```

означает:

```text
c  = change
i" = inner quoted string
```

Это центральная концепция Vim.

---

# 2. Установка и проверка

Проверка:

```bash
vim --version
```

Минимальная информация:

```bash
vim --clean
```

`--clean` запускает Vim без пользовательских настроек и plugins, что удобно для диагностики.

Проверка runtime:

```vim
:echo $VIMRUNTIME
```

Проверка возможностей:

```vim
:version
```

---

# 3. Ментальная модель Vim

Вместо запоминания сотен независимых команд полезно мыслить четырьмя слоями:

```text
MODE
  ↓
OPERATOR
  ↓
MOTION / TEXT OBJECT
  ↓
COUNT
```

Например:

```text
3dw
```

можно понимать как:

```text
3 × delete × word-motion
```

Другие примеры:

```text
d$
```

удалить до конца строки.

```text
ci(
```

изменить содержимое внутри `(...)`.

```text
yap
```

скопировать абзац.

```text
2dd
```

удалить две строки.

---

# 4. Режимы Vim

Основные режимы:

| Режим | Назначение |
|---|---|
| Normal | навигация и операции |
| Insert | ввод текста |
| Visual | выделение |
| Command-line | `:`-команды и поиск |
| Replace | замена текста |

## Normal mode

Основной режим Vim.

Из Insert mode:

```text
Esc
```

## Insert mode

Ввод текста:

```text
i
a
o
O
I
A
```

## Visual mode

```text
v
V
Ctrl-v
```

соответственно:

- characterwise;
- linewise;
- blockwise.

## Command-line mode

```text
:
```

Например:

```vim
:w
:q
:%s/foo/bar/g
```

---

# 5. Запуск и открытие файлов

Открыть файл:

```bash
vim file.txt
```

Несколько файлов:

```bash
vim file1.txt file2.txt
```

Открыть в read-only:

```bash
vim -R file.txt
```

Открыть без пользовательского config/plugins:

```bash
vim --clean file.txt
```

Создать новый файл:

```bash
vim new.txt
```

---

# 6. Выход и сохранение

Сохранить:

```vim
:w
```

Выйти:

```vim
:q
```

Сохранить и выйти:

```vim
:wq
```

Альтернативно:

```vim
:x
```

Выйти без сохранения:

```vim
:q!
```

Записать, даже если файл read-only при наличии необходимых прав:

```vim
:w!
```

Записать под другим именем:

```vim
:w new-name.txt
```

---

# 7. Навигация

Базовые движения:

```text
h  ←
j  ↓
k  ↑
l  →
```

Хотя стрелки работают, обучение `hjkl` полезно, поскольку руки остаются на основной позиции.

## В пределах строки

```text
0   начало строки
^   первый непробельный символ
$   конец строки
```

## По словам

```text
w   следующее слово
W   следующее WORD
b   предыдущее слово
B   предыдущее WORD
e   конец слова
E   конец WORD
```

`w` и `W` отличаются определением слова: `W` работает с whitespace-separated WORD.

## По экрану

```text
H   верх экрана
M   середина
L   низ
```

## По файлу

```text
gg      начало файла
G       конец файла
42G     строка 42
:42     строка 42
```

Также:

```text
Ctrl-f  страница вперёд
Ctrl-b  страница назад
Ctrl-d  вниз примерно на пол-экрана
Ctrl-u  вверх примерно на пол-экрана
```

---

# 8. Основные операции редактирования

Вход в Insert mode:

```text
i   перед курсором
a   после курсора
I   начало строки
A   конец строки
o   новая строка ниже
O   новая строка выше
```

Замена одного символа:

```text
r
```

Замена текущего символа с переходом в Insert:

```text
s
```

Изменение:

```text
c
```

Удаление:

```text
d
```

Копирование:

```text
y
```

Вставка:

```text
p
P
```

---

# 9. Операторы и motions

Главные операторы:

```text
d   delete
c   change
y   yank
>   indent
<   unindent
=   auto-indent
g~  toggle case
gu  lowercase
gU  uppercase
```

Motion:

```text
w
b
e
$
0
gg
G
```

Композиция:

```text
dw
d$
dgg
yG
caw
```

## Повтор оператора

```text
dd
cc
yy
```

означает операцию над текущей строкой.

С числом:

```text
3dd
2yy
5cc
```

---

# 10. Удаление, копирование и вставка

Удалить символ:

```text
x
```

Удалить предыдущий символ:

```text
X
```

Удалить строку:

```text
dd
```

Удалить несколько строк:

```text
3dd
```

Скопировать строку:

```text
yy
```

Вставить после:

```text
p
```

Вставить перед:

```text
P
```

Удалить до конца строки:

```text
d$
```

Удалить слово:

```text
dw
```

Удалить внутри кавычек:

```text
di"
```

---

# 11. Visual mode

Characterwise:

```text
v
```

Linewise:

```text
V
```

Blockwise:

```text
Ctrl-v
```

После выделения можно:

```text
d   удалить
y   скопировать
c   заменить
>   увеличить отступ
<   уменьшить отступ
~   изменить регистр
```

Block mode особенно полезен для колонок:

```text
Ctrl-v
```

затем движение, после чего:

```text
I
```

может вставить текст в начало выделенных строк.

---

# 12. Отмена и повтор

Отмена:

```text
u
```

Повтор отменённого изменения:

```text
Ctrl-r
```

Повтор последней операции:

```text
.
```

`.` — одна из самых мощных возможностей Vim.

Пример:

```text
ciwnew<Esc>
```

затем перемещение к следующему слову и:

```text
.
```

---

# 13. Поиск

Поиск вперёд:

```text
/pattern
```

Поиск назад:

```text
?pattern
```

Следующее совпадение:

```text
n
```

Предыдущее:

```text
N
```

Поиск слова под курсором:

```text
*
```

Поиск предыдущего совпадения слова:

```text
#
```

Поиск с учётом регистра зависит от настроек:

```vim
:set ignorecase
:set smartcase
```

Очистить подсветку:

```vim
:nohlsearch
```

---

# 14. Замена

Основная форма:

```vim
:[range]s/{pattern}/{replacement}/[flags]
```

Только текущая строка:

```vim
:s/foo/bar/
```

Все совпадения текущей строки:

```vim
:s/foo/bar/g
```

Во всём файле:

```vim
:%s/foo/bar/g
```

С подтверждением:

```vim
:%s/foo/bar/gc
```

Без учёта регистра:

```vim
:%s/foo/bar/gi
```

Только диапазон:

```vim
:10,20s/foo/bar/g
```

## Экранирование

Для сложных regex учитывайте Vim's regex syntax и необходимость экранирования `/`.

Можно использовать другой разделитель:

```vim
:%s#old/path#new/path#g
```

---

# 15. Регистры

Регистры — хранилища текста внутри Vim.

Просмотр:

```vim
:registers
```

Без имени используется unnamed register.

Именованные регистры:

```text
"a
"b
"c
...
"z
```

Записать в регистр:

```text
"ayy
```

Вставить из регистра:

```text
"ap
```

## Черный hole register

Удалить, не затирая основной yank:

```text
"_dd
```

Это очень полезно.

## Системный clipboard

При наличии clipboard support:

```text
"+y
"+p
```

Также часто:

```text
"*y
"*p
```

Поведение `+` и `*` зависит от платформы и clipboard backend.

Проверить:

```vim
:version
```

и искать `+clipboard`.

---

# 16. Макросы

Запись макроса:

```text
qa
```

затем действия, затем:

```text
q
```

Воспроизведение:

```text
@a
```

Повтор последнего macro:

```text
@@
```

С числом:

```text
10@a
```

Макрос следует строить как обычную последовательность нормальных Vim-команд.

Пример задачи:

1. перейти к строке;
2. изменить формат;
3. записать macro;
4. повторить его на остальных строках.

---

# 17. Текстовые объекты

Text objects позволяют адресовать структурированные части текста.

Основные:

```text
iw  inner word
aw  a word

i"  inside quotes
a"  quotes + surrounding whitespace

i'  inside single quotes
a'  single quotes

i(  inside parentheses
a(  parentheses

i[  inside brackets
a[
i{
a{
```

Примеры:

```text
ciw
di"
ya(
```

Паттерн:

```text
operator + text-object
```

Очень важная группа:

```text
ip  paragraph
ap  paragraph + surrounding whitespace
is  sentence
as  sentence
it  HTML/XML tag
at  tag + surrounding structure
```

Поддержка конкретных text objects может зависеть от Vim и plugins.

---

# 18. Повторение команд и автоматизация

Повтор:

```text
.
```

Повтор последнего Ex-команды:

```text
:@:
```

Повтор диапазона:

```text
:10,20normal A;
```

Это применяет Normal-mode команду к каждой строке диапазона.

Пример:

```vim
:10,20normal I// 
```

Добавит `// ` в начало каждой из строк.

---

# 19. Работа с несколькими файлами

Открытие:

```vim
:e file.txt
```

Следующий файл из аргумент-листа:

```vim
:next
```

Предыдущий:

```vim
:prev
```

Список:

```vim
:args
```

Открыть несколько файлов из shell:

```bash
vim *.txt
```

Переключение между buffers:

```vim
:buffer 3
:bnext
:bprevious
```

---

# 20. Окна и split

Горизонтальный split:

```vim
:split file.txt
```

или:

```text
Ctrl-w s
```

Вертикальный:

```vim
:vsplit file.txt
```

или:

```text
Ctrl-w v
```

Перемещение между окнами:

```text
Ctrl-w h
Ctrl-w j
Ctrl-w k
Ctrl-w l
```

Следующее окно:

```text
Ctrl-w w
```

Закрыть текущее:

```vim
:q
```

Закрыть остальные окна:

```vim
:only
```

Изменение размера:

```text
Ctrl-w =
Ctrl-w +
Ctrl-w -
Ctrl-w >
Ctrl-w <
```

---

# 21. Табы

Новая tab page:

```vim
:tabnew
```

Следующая:

```text
gt
```

Предыдущая:

```text
gT
```

Создать tab с файлом:

```vim
:tabedit file.txt
```

Список:

```vim
:tabs
```

Закрыть:

```vim
:tabclose
```

Важно: **tab page в Vim — не просто «файл»**. Она содержит набор windows.

---

# 22. Buffers, windows и tab pages

Это одна из самых важных концепций Vim.

```text
Buffer
  = содержимое/текстовый объект

Window
  = viewport, отображающий buffer

Tab page
  = набор windows
```

Один buffer может отображаться в нескольких windows.

Проверка:

```vim
:ls
```

Список windows:

```vim
:windows
```

Список tabs:

```vim
:tabs
```

Следующий buffer:

```vim
:bnext
```

Предыдущий:

```vim
:bprevious
```

Закрыть buffer:

```vim
:bdelete
```

---

# 23. Файловая система и рабочий каталог

Текущий рабочий каталог:

```vim
:pwd
```

Изменить:

```vim
:cd /path/to/project
```

Каталог файла:

```vim
:lcd %:p:h
```

Здесь:

```text
%    текущий файл
:p   полный путь
:h   head — каталог
```

Открыть file explorer:

```vim
:Explore
```

В зависимости от конфигурации может использоваться netrw или plugin-based file manager.

---

# 24. Командная строка Vim

В Normal mode:

```text
:
```

После этого вводятся Ex-команды.

Поиск:

```text
/
?
```

Автодополнение:

```text
Tab
Ctrl-d
```

История:

```text
Up
Down
```

Примеры:

```vim
:w
:e file.txt
:set number
:help
```

---

# 25. Ex-команды

Сохранение:

```vim
:w
```

Открытие:

```vim
:e file
```

Выход:

```vim
:q
```

Удаление строк:

```vim
:10,20d
```

Копирование:

```vim
:10,20t30
```

Перемещение:

```vim
:10,20m30
```

Запуск shell-команды:

```vim
:!ls
```

Запуск команды с текущим файлом:

```vim
:!python3 %
```

Повтор последней shell-команды:

```vim
:!!
```

---

# 26. Флаги и диапазоны команд

Специальные диапазоны:

```vim
:.      текущая строка
:$      последняя строка
:%      весь файл
:'<,'>  visual selection
```

Примеры:

```vim
:.,$d
```

Удалить от текущей строки до конца.

```vim
:%d
```

Удалить весь buffer.

```vim
:10,20d
```

Удалить строки 10–20.

---

# 27. Quickfix и location list

Quickfix — структурированный список результатов, ошибок или переходов.

Открыть:

```vim
:copen
```

Следующий элемент:

```vim
:cnext
```

Предыдущий:

```vim
:cprevious
```

Закрыть:

```vim
:cclose
```

Полезно для компиляторов, grep-поиска и тестов.

Location list похож на quickfix, но привязан к конкретному window:

```vim
:lopen
:lclose
:lnext
:lprevious
```

---

# 28. Tags и навигация по коду

Если создан tags-файл:

```vim
:tag function_name
```

или:

```text
Ctrl-]
```

по идентификатору под курсором.

Вернуться:

```text
Ctrl-t
```

Список tags:

```vim
:tags
```

Создание с помощью `ctags`:

```bash
ctags -R .
```

Точный формат и возможности зависят от установленной реализации ctags.

---

# 29. Fold

Folding позволяет скрывать блоки текста.

Ручные fold-команды:

```text
zf   создать fold
zo   открыть
zc   закрыть
za   toggle
zR   открыть все
zM   закрыть все
```

Включить folding:

```vim
:set foldmethod=indent
```

Другие методы:

```vim
:set foldmethod=manual
:set foldmethod=syntax
:set foldmethod=marker
```

---

# 30. Indentation

Автоотступ:

```text
==
```

Отступ строки:

```text
>>
```

Убрать:

```text
<<
```

Для выделения:

```text
>
<
=
```

Настройки:

```vim
:set shiftwidth=4
:set tabstop=4
:set softtabstop=4
:set expandtab
```

Частая конфигурация:

```vim
set expandtab
set shiftwidth=4
set softtabstop=4
```

`tabstop` определяет ширину tab character при отображении.

`shiftwidth` определяет размер структурного отступа.

`expandtab` заставляет Insert mode вставлять пробелы вместо tab characters.

---

# 31. Форматирование текста

Оператор форматирования:

```text
gq
```

Например:

```text
gqap
```

форматирует абзац.

Перенос строк:

```vim
:set textwidth=80
```

Форматирование комментариев и кода может зависеть от `formatoptions`:

```vim
:set formatoptions?
```

---

# 32. Spell checking

Включить:

```vim
:set spell
```

Отключить:

```vim
:set nospell
```

Выбрать язык:

```vim
:set spelllang=en_us
```

Следующая ошибка:

```text
]s
```

Предыдущая:

```text
[s
```

Предложения:

```text
z=
```

Добавить слово в словарь:

```text
zg
```

---

# 33. Сессии

Сохранить:

```vim
:mksession! session.vim
```

Загрузить:

```vim
:source session.vim
```

Сессия может хранить:

- открытые windows;
- buffers;
- tab pages;
- layout;
- некоторые настройки.

Не следует считать session-файл универсальным snapshot всего состояния Vim.

---

# 34. Swap, backup и recovery

Vim может создавать swap-файл для восстановления после аварии.

При обнаружении swap Vim может показать предупреждение.

Восстановление:

```vim
:recover file
```

или:

```bash
vim -r file
```

Проверка swap:

```bash
vim -r
```

Backup:

```vim
:set backup
```

Write backup:

```vim
:set writebackup
```

Persistent undo:

```vim
:set undofile
```

Путь хранения задаётся через:

```vim
:set undodir?
```

Рекомендуется понимать различия между:

```text
swap
backup
writebackup
undo file
```

Они решают разные задачи.

---

# 35. Конфигурация `.vimrc`

Обычно пользовательский config находится в:

```text
~/.vimrc
```

Для Vim 9 можно также использовать конфигурацию в:

```text
~/.vim/
```

Минимальный пример:

```vim
set number
set relativenumber
set ignorecase
set smartcase
set expandtab
set shiftwidth=4
set softtabstop=4
set mouse=a
set hidden
set incsearch
set hlsearch
set wildmenu
```

Для диагностики расположения:

```vim
:echo $MYVIMRC
```

---

# 36. Опции Vim

Посмотреть значение:

```vim
:set number?
```

Установить:

```vim
:set number
```

Отключить:

```vim
:set nonumber
```

Переключить:

```vim
:set invnumber
```

Посмотреть все:

```vim
:set all
```

Полезные options:

```vim
number
relativenumber
cursorline
ignorecase
smartcase
incsearch
hlsearch
expandtab
shiftwidth
tabstop
softtabstop
hidden
splitright
splitbelow
wildmenu
clipboard
undofile
```

---

# 37. Mappings

Mapping связывает клавиши с последовательностью действий.

Normal mode:

```vim
nnoremap <leader>w :write<CR>
```

Leader:

```vim
let mapleader = " "
```

После этого:

```text
Space + w
```

сохраняет файл.

Insert mode:

```vim
inoremap jj <Esc>
```

Visual mode:

```vim
vnoremap <leader>y "+y
```

Основные команды:

```text
:noremap
:nnoremap
:inoremap
:vnoremap
:xnoremap
:tnoremap
```

Для новых mappings предпочтительнее использовать `noremap`-семейство, если рекурсивное расширение не требуется.

---

# 38. Autocommands

Autocommand выполняет действие при событии.

Пример:

```vim
augroup my_settings
    autocmd!
    autocmd BufWritePre *.sh :%s/\s\+$//e
augroup END
```

`augroup` нужен для удобного повторного определения правил без накопления дубликатов.

Примеры событий:

```text
BufRead
BufNewFile
BufWritePre
BufWritePost
FileType
VimEnter
VimLeave
```

Проверить:

```vim
:autocmd
```

---

# 39. Vimscript и Vim 9 Script

Классический Vimscript широко используется в конфигурациях.

Пример:

```vim
function! ToggleNumber()
    set number!
endfunction
```

Современный Vim 9 имеет более структурированный Vim9 script.

Пример:

```vim
vim9script

def Greet(name: string): string
    return 'Hello, ' .. name
enddef
```

Не следует смешивать Vimscript и Vim9 script без понимания различий синтаксиса и execution model.

Для простой `.vimrc` обычно достаточно хорошо организованного Vimscript.

---

# 40. Плагины

Vim имеет встроенную plugin architecture.

Типичная структура:

```text
~/.vim/
├── autoload/
├── colors/
├── ftplugin/
├── plugin/
├── syntax/
└── pack/
```

Современный Vim поддерживает native package mechanism:

```text
~/.vim/pack/{name}/start/{plugin}
~/.vim/pack/{name}/opt/{plugin}
```

После установки plugins полезно проверить:

```vim
:scriptnames
```

Это показывает загруженные Vimscript-файлы и помогает искать проблемы с конфигурацией.

Не следует добавлять большое количество plugins только ради замены нескольких базовых Vim-команд. Сначала следует освоить native capabilities.

---

# 41. Диагностика и справка

Главный источник документации:

```vim
:help
```

По теме:

```vim
:help motion
:help operator
:help text-objects
:help registers
:help visual-mode
:help windows
:help tab-page
```

По конкретной команде:

```vim
:help :split
:help :substitute
```

Проверить mapping:

```vim
:verbose map <leader>w
```

Проверить option:

```vim
:set number?
```

Проверить source:

```vim
:scriptnames
```

Проверить messages:

```vim
:messages
```

В режиме диагностики часто полезен запуск:

```bash
vim --clean
```

Если проблема исчезает, вероятно, причина в пользовательской конфигурации или plugins.

---

# 42. Производительность

Основные источники проблем:

- слишком много plugins;
- тяжёлые autocommands;
- дорогие regex;
- выполнение внешних процессов на каждый keystroke;
- сложные mappings;
- огромные файлы;
- синтаксическая подсветка для неподходящего типа файла.

Диагностика:

```vim
:profile start profile.log
:profile func *
:profile file *
```

После работы:

```vim
:profile pause
:profile dump
```

Для больших файлов иногда следует отключать дорогостоящие features.

---

# 43. Безопасность

Vim способен выполнять команды shell:

```vim
:!command
```

и запускать Vimscript.

Поэтому нельзя бездумно открывать неизвестные проекты и выполнять содержащиеся в них скрипты/plugins.

Особенно осторожно относитесь к:

```text
.vimrc
.vim/
vimrc-local
plugins
modelines
```

Modelines можно отключить:

```vim
:set nomodeline
```

Если вы работаете с недоверенным содержимым, используйте:

```bash
vim --clean
```

и осознанно контролируйте plugins/configuration.

---

# 44. Практический workflow

## Открытие проекта

```bash
cd ~/project
vim .
```

или:

```bash
vim src/main.c
```

## Навигация

Используйте:

```text
gg
G
/pattern
n
N
*
#
```

## Редактирование

Основа:

```text
i
a
o
c
d
y
p
u
Ctrl-r
.
```

## Структурированное редактирование

Используйте:

```text
ciw
ci"
ci(
di{
yi[
```

## Массовое изменение

Сначала:

```text
:%s/old/new/gc
```

с подтверждением, а затем при уверенности:

```text
:%s/old/new/g
```

## Проверка кода

Используйте:

```vim
:make
```

или внешние инструменты через quickfix/location list.

---

# 45. Антипаттерны

## Изучать Vim как список горячих клавиш

Непродуктивно:

```text
dd = удалить
yy = копировать
p = вставить
...
```

Правильнее изучать композицию:

```text
operator + motion
operator + text object
count + command
```

## Использовать мышь для всего

Мышь полезна, но постоянное переключение между keyboard и mouse разрушает эффективность modal workflow.

## Делать сложные изменения вручную

Если операция повторяется, рассмотрите:

```text
.
macro
:substitute
:normal
Visual block
```

## Сразу устанавливать десятки plugins

Сначала определите задачу. Затем проверьте, есть ли native Vim solution.

## Не понимать buffers/windows/tabs

Это приводит к неправильной модели Vim и путанице при работе с несколькими файлами.

## Использовать `:q!` для решения любой проблемы

`q!` просто отбрасывает несохранённые изменения. Он не является способом «исправить Vim».

---

# 46. Краткий cheat sheet

## Режимы

```text
Esc       Normal
i         Insert
v         Visual
V         Visual line
Ctrl-v    Visual block
:         Command-line
/         Search forward
?         Search backward
```

## Движение

```text
h j k l
w b e
0 ^ $
gg G
Ctrl-f Ctrl-b
```

## Редактирование

```text
i a o O
x X
r
d
c
y
p P
u
Ctrl-r
.
```

## Строки

```text
dd
yy
cc
D
C
S
```

## Text objects

```text
iw aw
i" a"
i' a'
i( a(
i[ a[
i{ a{
ip ap
is as
```

## Поиск

```text
/pattern
?pattern
n
N
*
#
:nohlsearch
```

## Замена

```vim
:s/old/new/
:s/old/new/g
:%s/old/new/g
:%s/old/new/gc
```

## Registers

```text
:registers
"ayy
"ap
"_dd
"+y
"+p
```

## Macros

```text
qa ... q
@a
@@
10@a
```

## Окна

```text
Ctrl-w s
Ctrl-w v
Ctrl-w h
Ctrl-w j
Ctrl-w k
Ctrl-w l
Ctrl-w =
```

## Buffers

```vim
:ls
:bnext
:bprevious
:bdelete
:buffer N
```

## Tabs

```vim
:tabnew
:tabnext
:tabprevious
:tabclose
:tabs
```

## Файлы

```vim
:e file
:w
:q
:wq
:q!
```

## Диагностика

```vim
:help
:messages
:scriptnames
:set option?
:verbose map ...
```

## Shell

```vim
:!command
```

---

# Заключение

Vim следует изучать не как обычный текстовый редактор, а как **язык преобразования текста**.

Базовая модель:

```text
COUNT
  +
OPERATOR
  +
MOTION / TEXT OBJECT
```

Например:

```text
3dw
```

означает три операции удаления слова.

```text
ci"
```

означает изменение текста внутри кавычек.

```text
yap
```

означает копирование абзаца.

Когда эта модель становится естественной, большая часть Vim перестаёт выглядеть как набор случайных комбинаций.

Следующий уровень — научиться выбирать правильный механизм для задачи:

```text
Одна правка       → operator + motion
Структурная правка → text object
Повтор             → .
Много одинаковых   → macro
Регулярная замена  → :substitute
Колонка            → Visual block
Диапазон строк     → :range + command
Несколько файлов   → buffers
Несколько layout   → windows
Рабочие контексты  → tab pages
Автоматизация      → mappings/autocmds
Сложная интеграция → plugins / Vimscript / Vim9
```

Главный принцип:

> **Сначала понять структуру текста и выбрать правильную абстракцию операции; затем выполнять минимальное количество действий.**

Это и есть основа эффективной работы в Vim.
