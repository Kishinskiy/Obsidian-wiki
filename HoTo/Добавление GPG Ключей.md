
## Как добавлять ключи новым способом  

Новый «современный» вариант плохо документирован, попробуем восполнить этот пробел.

Теперь ключи надо добавлять командами следующего вида.

Если добавляется удалённый файл ключей:

```sh
curl -s URL | sudo gpg --no-default-keyring --keyring gnupg-ring:/etc/apt/trusted.gpg.d/ИМЯ.gpg --import
```

Если добавляется локальный файл ключей:

```sh
cat ФАЙЛ.pub | sudo gpg --no-default-keyring --keyring gnupg-ring:/etc/apt/trusted.gpg.d/ИМЯ.gpg --import
```

В этих командах нужно подставить:

- **URL** — адрес файла .pub
- **ИМЯ** — можно выбрать любое имя файла
- **ФАЙЛ** — имя файла .pub

Затем обязательно выполнить следующую команду для установки правильных прав доступа к файлу:

```sh
sudo chmod 644 /etc/apt/trusted.gpg.d/ИМЯ.gpg
```

Пример. Если вы уже знаете URL-адрес требуемого открытого ключа, используйте wget или curl для его загрузки и импорта. Не забудьте обновить права доступа к файлам с 600 до 644.

```sh
curl -s https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo gpg --no-default-keyring --keyring gnupg-ring:/etc/apt/trusted.gpg.d/earth.gpg --import

sudo chmod 644 /etc/apt/trusted.gpg.d/earth.gpg
```