
Создаем файл в папке ~/.local/share/applications с расширением .desktop
например War Thunder.desktop
со следующим содержимым:
```sh
[Desktop Entry]
Name=War Thunder
Comment=Tactical MMO Game
Exec=/home/oleg/WarThunder/launcher
Icon=/home/oleg/WarThunder/launcher.ico
Terminal=false
Type=Application
Categories=Game;
```
где Name - это имя нашего Файла, Comment - Текст подсказка , Exec - путь до исполняемого файла либо команда, Icon - путь до файла значка.

Что бы наше приложение отображалось в лаунчере его нужно валидировать командой:
```sh
desktop-file-validate ~/.local/share/applications/War\ Thunder.desktop 
```
