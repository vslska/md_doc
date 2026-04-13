### 🚀 Настройка PowerShell в стиле Unix (Emacs + Fish-style)

Этот мануал поможет настроить PowerShell так, чтобы он работал с привычными горячими клавишами Emacs и выдавал умные подсказки из истории, как в оболочке Fish.

### 1. Быстрая настройка (текущая сессия)

Чтобы мгновенно активировать режим управления и подсказки, выполни эти три команды:

```powershell
# Включить горячие клавиши Emacs (Ctrl+A, Ctrl+E, Ctrl+K и т.д.)
Set-PSReadLineOption -EditMode Emacs

# Включить «теневые» подсказки из истории (как в Fish)
Set-PSReadLineOption -PredictionSource History

# Включить удобное меню выбора при нажатии Tab (вместо перебора)
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete
```

### 2. Сохранение настроек навсегда

Чтобы настройки применялись автоматически при каждом запуске терминала, их нужно добавить в профиль пользователя.

1. Открой файл профиля:
```powershell
notepad $PROFILE
```

2.  Если файл не существует, создай его командой: `New-Item -Path $PROFILE -Type File -Force`, а затем открой.
3. Вставь в него следующий блок:

```powershell
# --- PSReadLine Configuration ---

# Режим Emacs (Ctrl+A, B, E, F, P, N, K, Y)
Set-PSReadLineOption -EditMode Emacs

# История команд как в Fish
Set-PSReadLineOption -PredictionSource History

# Выбор вариантов по Tab через меню (стрелками)
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete

# Цвет подсказки (светло-серый, чтобы не путать с вводом)
Set-PSReadLineOption -Colors @{ InlinePrediction = '#909090' }

# Чтобы поиск по истории работал по стрелкам вверх/вниз (учитывая начало команды)
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward
```
.

3. Шпаргалка по горячим клавишам (Emacs Mode)

|Клавиша|Действие|
|---|---|
|**`Ctrl + A`**|В начало строки|
|**`Ctrl + E`**|В конец строки|
|**`Alt + B / F`**|На слово назад / вперед|
|**`Ctrl + K`**|Удалить всё ПОСЛЕ курсора|
|**`Ctrl + U`**|Удалить ВСЮ строку|
|**`Ctrl + Y`**|Вставить удаленный текст (Yank)|
|**`Ctrl + R`**|Интерактивный поиск по всей истории|
|**`Стрелка вправо`**|Принять подсказку (Predictive text)|
|**`F2`**|Переключить вид подсказок (строка / список)|

---

**Мануал готов!** Если захочешь добавить в него раздел про **кастомные алиасы** или **красивую тему оформления (Oh My Posh)**, просто скажи.