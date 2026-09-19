# yt-dlp — скачивание видео, плейлистов и конвертация в MP3

Практическая шпаргалка для Arch Linux. Предполагается, что `yt-dlp` и `ffmpeg` уже установлены.

> Используй скачивание с учётом авторских прав и правил YouTube. Для своих видео, материалов с разрешением и контента, который можно сохранять, `yt-dlp` подходит отлично.

---

## 1. Скачать одно видео по ссылке

Самый простой вариант:

```bash
yt-dlp "https://www.youtube.com/watch?v=VIDEO_ID"
```

По умолчанию `yt-dlp` выбирает доступное лучшее качество. Если видео и аудио идут отдельными потоками, FFmpeg объединит их в один файл.

### Сохранить в MP4

```bash
yt-dlp -f "bv*+ba/b" --merge-output-format mp4 "VIDEO_URL"
```

### Предпочесть MP4-видео и M4A-аудио

```bash
yt-dlp -f "bv*[ext=mp4]+ba[ext=m4a]/b[ext=mp4]/b" "VIDEO_URL"
```

### Скачать не выше 1080p

```bash
yt-dlp -f "bv*[height<=1080]+ba/b[height<=1080]" "VIDEO_URL"
```

### Скачать не выше 720p

```bash
yt-dlp -f "bv*[height<=720]+ba/b[height<=720]" "VIDEO_URL"
```

### Посмотреть доступные форматы

```bash
yt-dlp -F "VIDEO_URL"
```

После этого можно выбрать конкретные ID форматов:

```bash
yt-dlp -f VIDEO_FORMAT_ID+AUDIO_FORMAT_ID "VIDEO_URL"
```

Например:

```bash
yt-dlp -f 137+140 "VIDEO_URL"
```

> ID форматов индивидуальны для конкретного видео.

---

## 2. Скачать весь плейлист

```bash
yt-dlp "https://www.youtube.com/playlist?list=PLAYLIST_ID"
```

### Сохранять видео в отдельную папку

```bash
yt-dlp \
  -o "playlist/%(playlist_index)s - %(title)s.%(ext)s" \
  "PLAYLIST_URL"
```

Пример структуры:

```text
playlist/
├── 1 - Первое видео.mp4
├── 2 - Второе видео.mp4
└── 3 - Третье видео.mp4
```

### Папка с названием плейлиста

```bash
yt-dlp \
  -o "%(playlist_title)s/%(playlist_index)s - %(title)s.%(ext)s" \
  "PLAYLIST_URL"
```

### Не скачивать уже загруженные видео повторно

```bash
yt-dlp \
  --download-archive archive.txt \
  "PLAYLIST_URL"
```

`archive.txt` хранит идентификаторы успешно скачанных видео. При следующем запуске уже загруженные элементы будут пропускаться.

---

## 3. Скачать только определённые видео из плейлиста

Для этого используется параметр:

```bash
-I
```

или полная форма:

```bash
--playlist-items
```

### Видео с 1-го по 8-е

```bash
yt-dlp -I 1:8 "PLAYLIST_URL"
```

### Только 1-е и 8-е

```bash
yt-dlp -I 1,8 "PLAYLIST_URL"
```

### Видео 1, 2, 3, 7 и 8

```bash
yt-dlp -I 1:3,7:8 "PLAYLIST_URL"
```

### С 5-го до конца

```bash
yt-dlp -I 5: "PLAYLIST_URL"
```

### Последние 5 видео

```bash
yt-dlp -I -5: "PLAYLIST_URL"
```

### Каждое второе видео

```bash
yt-dlp -I ::2 "PLAYLIST_URL"
```

### Видео с 1-го по 8-е, но только каждое второе

```bash
yt-dlp -I 1:8:2 "PLAYLIST_URL"
```

### Пример с нумерацией файлов

```bash
yt-dlp \
  -I 1:8 \
  -o "%(playlist_index)s - %(title)s.%(ext)s" \
  "PLAYLIST_URL"
```

> Синтаксис диапазона:
> 
> ```text
> START:END:STEP
> ```
> 
> - `1:8` — с 1-го по 8-е
>     
> - `1:8:2` — с 1-го по 8-е через один
>     
> - `5:` — с 5-го до конца
>     
> - `::2` — каждый второй элемент
>     

---

## 4. Если ссылка содержит и видео, и плейлист

Например:

```text
https://www.youtube.com/watch?v=VIDEO_ID&list=PLAYLIST_ID
```

Если нужно скачать **только это видео**, а не весь плейлист:

```bash
yt-dlp --no-playlist "VIDEO_URL"
```

Это полезно, когда копируешь ссылку на видео из плейлиста.

---

## 5. Скачать видео сразу в MP3

Для извлечения аудио используется параметр `-x`:

```bash
yt-dlp -x --audio-format mp3 "VIDEO_URL"
```

### Лучшее доступное качество MP3

```bash
yt-dlp \
  -x \
  --audio-format mp3 \
  --audio-quality 0 \
  "VIDEO_URL"
```

### MP3 с битрейтом 320 kbps

```bash
yt-dlp \
  -x \
  --audio-format mp3 \
  --audio-quality 320K \
  "VIDEO_URL"
```

> MP3 — формат с потерями. Если исходное аудио уже сжато, конвертация в MP3 не улучшит качество. Для сохранения исходного аудиопотока лучше использовать M4A или Opus, если они доступны.

### MP3 с названием видео

```bash
yt-dlp \
  -x \
  --audio-format mp3 \
  -o "%(title)s.%(ext)s" \
  "VIDEO_URL"
```

### MP3 с обложкой и метаданными

```bash
yt-dlp \
  -x \
  --audio-format mp3 \
  --embed-thumbnail \
  --embed-metadata \
  "VIDEO_URL"
```

---

## 6. Весь плейлист в MP3

```bash
yt-dlp \
  -x \
  --audio-format mp3 \
  -o "%(playlist_index)s - %(title)s.%(ext)s" \
  "PLAYLIST_URL"
```

### В отдельную папку с названием плейлиста

```bash
yt-dlp \
  -x \
  --audio-format mp3 \
  -o "%(playlist_title)s/%(playlist_index)s - %(title)s.%(ext)s" \
  "PLAYLIST_URL"
```

### Только первые 8 видео

```bash
yt-dlp \
  -I 1:8 \
  -x \
  --audio-format mp3 \
  -o "%(playlist_index)s - %(title)s.%(ext)s" \
  "PLAYLIST_URL"
```

### Только 1-е, 3-е и 8-е

```bash
yt-dlp \
  -I 1,3,8 \
  -x \
  --audio-format mp3 \
  -o "%(playlist_index)s - %(title)s.%(ext)s" \
  "PLAYLIST_URL"
```

### MP3 с обложкой и метаданными для плейлиста

```bash
yt-dlp \
  -x \
  --audio-format mp3 \
  --audio-quality 0 \
  --embed-thumbnail \
  --embed-metadata \
  -o "%(playlist_index)s - %(title)s.%(ext)s" \
  "PLAYLIST_URL"
```

---

## 7. Полезные шаблоны имён файлов

`yt-dlp` позволяет использовать метаданные видео.

|Шаблон|Что означает|
|---|---|
|`%(title)s`|Название видео|
|`%(id)s`|ID видео|
|`%(ext)s`|Расширение|
|`%(playlist_index)s`|Номер в плейлисте|
|`%(playlist_title)s`|Название плейлиста|
|`%(uploader)s`|Автор / канал|
|`%(upload_date)s`|Дата публикации|

### Пример: автор, дата, название и ID

```bash
yt-dlp \
  -o "%(uploader)s/%(upload_date)s - %(title)s [%(id)s].%(ext)s" \
  "VIDEO_URL"
```

### Пример: плейлист с двузначной нумерацией

```bash
yt-dlp \
  -o "%(playlist_title)s/%(playlist_index)02d - %(title)s.%(ext)s" \
  "PLAYLIST_URL"
```

Результат:

```text
Название плейлиста/
├── 01 - Первое видео.mp4
├── 02 - Второе видео.mp4
└── 03 - Третье видео.mp4
```

---

## 8. Полезные дополнительные параметры

### Указать папку для загрузки

```bash
yt-dlp \
  -P ~/Downloads/youtube \
  "VIDEO_URL"
```

### Скачать и сохранить исходный файл

```bash
yt-dlp \
  -P ~/Downloads/youtube \
  -o "%(title)s.%(ext)s" \
  "VIDEO_URL"
```

### Не перезаписывать существующие файлы

```bash
yt-dlp \
  --no-overwrites \
  "VIDEO_URL"
```

### Продолжить незавершённую загрузку

```bash
yt-dlp \
  -c \
  "VIDEO_URL"
```

### Ограничить скорость загрузки

```bash
yt-dlp \
  -r 5M \
  "VIDEO_URL"
```

### Скачать только аудио без конвертации

```bash
yt-dlp -f ba "VIDEO_URL"
```

### Скачать субтитры

```bash
yt-dlp \
  --write-subs \
  --sub-langs "ru,en" \
  "VIDEO_URL"
```

### Скачать автоматически сгенерированные субтитры

```bash
yt-dlp \
  --write-auto-subs \
  --sub-langs "ru,en" \
  "VIDEO_URL"
```

### Скачать субтитры и встроить их в видео

```bash
yt-dlp \
  --write-subs \
  --sub-langs "ru,en" \
  --embed-subs \
  "VIDEO_URL"
```

---

## 9. Часто используемые команды — кратко

|Задача|Команда|
|---|---|
|Одно видео|`yt-dlp "URL"`|
|Только одно видео из ссылки плейлиста|`yt-dlp --no-playlist "URL"`|
|Весь плейлист|`yt-dlp "PLAYLIST_URL"`|
|Видео 1–8|`yt-dlp -I 1:8 "PLAYLIST_URL"`|
|Видео 1, 3, 8|`yt-dlp -I 1,3,8 "PLAYLIST_URL"`|
|С 5-го до конца|`yt-dlp -I 5: "PLAYLIST_URL"`|
|MP3|`yt-dlp -x --audio-format mp3 "URL"`|
|MP3 320 kbps|`yt-dlp -x --audio-format mp3 --audio-quality 320K "URL"`|
|Список форматов|`yt-dlp -F "URL"`|
|Не выше 1080p|`yt-dlp -f "bv*[height<=1080]+ba/b[height<=1080]" "URL"`|
|Не скачивать повторно|`yt-dlp --download-archive archive.txt "URL"`|
|Сохранить в папку|`yt-dlp -P ~/Downloads/youtube "URL"`|
|Скачать только аудио|`yt-dlp -f ba "URL"`|

---

## 10. Если YouTube выдаёт ошибку

### Обновить yt-dlp

```bash
yt-dlp -U
```

Если установлен через `pip`:

```bash
python3 -m pip install -U "yt-dlp[default]"
```

### Использовать cookies браузера

Иногда видео требует авторизацию или YouTube иначе обрабатывает запросы.

Для Chromium / Chrome:

```bash
yt-dlp --cookies-from-browser chromium "VIDEO_URL"
```

Для Firefox:

```bash
yt-dlp --cookies-from-browser firefox "VIDEO_URL"
```

Для Google Chrome:

```bash
yt-dlp --cookies-from-browser chrome "VIDEO_URL"
```

> Не передавай cookies посторонним людям и не публикуй файл cookies. Это данные сессии браузера.

---

## 11. Универсальные команды

### Скачать видео в MP4

```bash
yt-dlp \
  -f "bv*+ba/b" \
  --merge-output-format mp4 \
  "URL"
```

### Скачать MP3

```bash
yt-dlp \
  -x \
  --audio-format mp3 \
  --audio-quality 0 \
  "URL"
```

### Первые 8 видео плейлиста в MP3

```bash
yt-dlp \
  -I 1:8 \
  -x \
  --audio-format mp3 \
  -o "%(playlist_index)s - %(title)s.%(ext)s" \
  "PLAYLIST_URL"
```

### Первые 8 видео плейлиста в MP4

```bash
yt-dlp \
  -I 1:8 \
  -f "bv*+ba/b" \
  --merge-output-format mp4 \
  -o "%(playlist_index)s - %(title)s.%(ext)s" \
  "PLAYLIST_URL"
```

### Скачать плейлист, пропуская уже загруженные видео

```bash
yt-dlp \
  --download-archive archive.txt \
  -o "%(playlist_index)s - %(title)s.%(ext)s" \
  "PLAYLIST_URL"
```

---

## 12. Ссылки на документацию

- [Официальный репозиторий yt-dlp](https://github.com/yt-dlp/yt-dlp)
    
- [README с параметрами командной строки](https://github.com/yt-dlp/yt-dlp#usage-and-options)
    
- [Wiki / FAQ](https://github.com/yt-dlp/yt-dlp/wiki/FAQ)
    
- [Документация FFmpeg](https://ffmpeg.org/documentation.html)