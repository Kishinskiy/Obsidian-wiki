- Генерация SSL-сертификатов
- [Настройка PostgreSQL для использования SSL](https://byzoni.org/posts/setting-up-ssl-tls-encryption-postgresql-connections/#%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-postgresql-%D0%B4%D0%BB%D1%8F-%D0%B8%D1%81%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F-ssl)
- [Настройка аутентификации клиента](https://byzoni.org/posts/setting-up-ssl-tls-encryption-postgresql-connections/#%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-%D0%B0%D1%83%D1%82%D0%B5%D0%BD%D1%82%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%86%D0%B8%D0%B8-%D0%BA%D0%BB%D0%B8%D0%B5%D0%BD%D1%82%D0%B0)
- [Тестирование вашей конфигурации](https://byzoni.org/posts/setting-up-ssl-tls-encryption-postgresql-connections/#%D1%82%D0%B5%D1%81%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%B2%D0%B0%D1%88%D0%B5%D0%B9-%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3%D1%83%D1%80%D0%B0%D1%86%D0%B8%D0%B8)
- [Завершение](https://byzoni.org/posts/setting-up-ssl-tls-encryption-postgresql-connections/#%D0%B7%D0%B0%D0%B2%D0%B5%D1%80%D1%88%D0%B5%D0%BD%D0%B8%D0%B5)


Обеспечение безопасности передаваемых данных является краеугольным камнем современной веб-разработки. Когда дело доходит до баз данных, шифрование соединений имеет жизненно важное значение для защиты конфиденциальной информации от перехвата или подделки. PostgreSQL, как ведущая реляционная база данных с открытым исходным кодом, поддерживает шифрование SSL/TLS для соединений, обеспечивая необходимый уровень безопасности. В этой статье мы покажем вам шаги, необходимые для настройки шифрования SSL/TLS для ваших соединений PostgreSQL.

## Генерация SSL-сертификатов

Первым шагом на пути к защите ваших соединений PostgreSQL с помощью SSL/TLS является создание необходимых сертификатов. Вам понадобится сертификат сервера и закрытый ключ для вашего сервера PostgreSQL. При желании вы можете создать клиентские сертификаты для дополнительного уровня безопасности.

Генерируем закрытый ключ
```sh
openssl genrsa -des3 -out server.key 4096
```

Удалить парольную фразу из ключа
```sh
openssl rsa -in server.key -out server.key
```

Создайте запрос на подпись сертификата (CSR)
```sh
openssl req -new -key server.key -out server.csr
```

Подпишите CSR своим закрытым ключом, чтобы создать сертификат сервера.
```sh
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

## Настройка PostgreSQL для использования SSL

Когда ваши сертификаты готовы, вам нужно настроить PostgreSQL для их использования. Поместите файлы `server.crt` и `server.key` в каталог данных вашей установки PostgreSQL. Этот каталог обычно находится по адресу `/var/lib/postgresql/{version}/data/`.

Установите правильные разрешения для закрытого ключа
```sh
chmod 400 server.key
```

Перемещаем сертификаты в каталог данных PostgreSQL
```sh
cp server.crt /var/lib/postgresql/{version}/data/
cp server.key /var/lib/postgresql/{version}/data/
```

Затем отредактируйте файл `postgresql.conf`, чтобы включить SSL:
```sh
ssl = on
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
```

## Настройка аутентификации клиента

PostgreSQL использует файл с именем `pg_hba.conf` для настройки аутентификации на основе хоста. Чтобы требовать SSL для определенных соединений, добавьте или измените соответствующие строки в этом файле:

```sh
# Require SSL for all users connecting from all hosts	hostssl all all 0.0.0.0/0 md5
```

При использовании клиентских сертификатов вам необходимо настроить правильную аутентификацию сертификата, добавив такие строки, как:
```sh
# Require certificate authentication for user 'myuser'	hostssl all myuser 0.0.0.0/0 cert clientcert=
```

## Тестирование вашей конфигурации

Чтобы убедиться, что SSL работает правильно, перезапустите сервер PostgreSQL и попытайтесь подключиться к клиенту с поддержкой SSL:

Перезапускаем PostgreSQL
```sh
sudo systemctl restart postgresql.service
```

Подключаемся с помощью psql с SSL
```sh
psql "host=localhost dbname=mydb user=myuser sslmode=require"
```

Если все настроено правильно, вы сможете подключиться без каких-либо проблем.

## Завершение

Настройка SSL/TLS-шифрования для ваших соединений PostgreSQL — это важный шаг в защите данных вашего приложения. Выполнив шаги, описанные выше, вы можете защитить передачу данных от несанкционированного доступа и гарантировать, что конфиденциальная информация останется конфиденциальной.