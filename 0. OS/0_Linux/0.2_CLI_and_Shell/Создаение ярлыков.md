
| `~/.local/share/applications/`  | Только для текущего пользователя (рекомендуется) |
| ------------------------------- | ------------------------------------------------ |
| <br> `/usr/share/applications/` | Для всех пользователей (нужен `sudo`)            |
Например:
создал каталог bin у себя hома или `любой другой на твой вкус` 
скачал с инэта  `myapp`  , `ну каталожек я думаю ты уже создал(а) там`
 в  `~/bin/myapp`
ну или просто скачал,  в `~/Download/`  `там в архиве не архив архиве  не архивет типо умеет 
и создаем .desktop файл
```bash
nano ~/.local/share/applications/myapp.desktop
```
Содержаем / сохраняем
```bash
[Desktop Entry]
Name=Obsidian
Comment=Knowledge base in Markdown
Exec=/home/твой_пользователь/Applications/Obsidian.AppImage
Icon=myapp
Terminal=false
Type=Application
Categories=Office;Notes;
StartupNotify=true
```
`1-#можешь поставить октоторп, перед Icon и система выставить icon как unk.icon или  можно свою иконку поставить ( православную), просто скачай  в .`png или  .svg` и укажиии путь`


