
1. в домашнем каталоге создадим папку debpkgs/ИмяПрограммы_версия_архитектура
		например:
```sh
mkdir ~/debpkgs/example_1.0.0_x86
```
| варианты архитектур: |
| -------------------- |
| x86                  |
| x64                  |
| all                     |

2. в созданой папке debpkgs/example_1.0.0_x86 создадим папку DEBIAN
все имя папки заглавными, это важно.

3. в  папке example_1.0.0_x86 размещаем наш исполняемый фаил сохроняя путь до него относительно корня.
   
   например если найш фаил exemple.sh должен лежать в папке /opt/myall/example.sh то полный путь до него будет такой
   /home/USER/debpkgs/opt/myall/example.sh

4.  В папке DEBIAN создадим файл control следующего содержания
```
	Package: my-program 
	Version: 1.0 
	Architecture: all 
	Essential: no 
	Priority: optional 
	Depends: packages;my-program;needs;to;run 
	Maintainer: Your Name 
	Description: A short description of my-program that will be displayed when the package is being selected for installation.
```

подробнее о формате файла control читайте в документации https://manpages.ubuntu.com/manpages/noble/en/man5/deb-control.5.html

5. Если требуется выполнить какой либо скрипт до или после установки пакета, но в папке DEBIAN необходимо создать файлы с именами preinst и postinst соответственно, так же не забудьте дать им права 755.

готово, осталось выполнить команду
dpkg-deb --build /home/USER/debpkgs/example_1.0.0_x86 и пакет будет собран
