
Обеспечение безопасной передачи данных является требованием многих производственных систем. PostgreSQL поддерживает TLS в качестве средства шифрования сетевой связи, проверки хостов и возможности аутентификации на основе сертификатов.

Функциональность TLS PostgreSQL может быть расширена в развертываниях Kubernetes. Оператор Crunchy Data Postgres обеспечивает поддержку TLS с версии 4.3, используя Kubernetes Secrets для безопасного монтирования компонентов TLS в каждый модуль. Оператор PostgreSQL не высказывает мнения о PKI, используемом для создания сертификата TLS, а скорее загружает пару ключей TLS и центр сертификации (CA) для сервера PostgreSQL.

Давайте проведем пример создания кластера PostgreSQL с TLS.

## Prerequisites

В этом примере предполагается, что вы развернули оператор Crunchy Data Postgres в Kubernetes с помощью быстрого запуска.

Для включения TLS в кластерах PostgreSQL необходимы три пункта:

* Сертификат CA

* Закрытый ключ TLS

* Сертификат TLS

Существует множество способов создания этих элементов. На самом деле, Kubernetes поставляется с собственной системой управления сертификатами! Документация PostgreSQL также предоставляет пример того, как создать сертификат TLS.

Вам решать, как вы хотите управлять этим для своего кластера, но давайте пройдемся по примеру ниже.

Сначала нам нужно создать CA. Используйте приведенную ниже команду для создания ECDSA CA:

```sh
openssl req \ 
-x509 \ 
-nodes \ 
-newkey ec \ 
-pkeyopt ec_paramgen_curve:prime256v1 \ 
-pkeyopt ec_param_enc:named_curve \ 
-sha384 \ 
-keyout ca.key \ 
-out ca.crt \ 
-days 3650 \ 
-subj "/CN=*"
```

Для производственной системы вы, скорее всего, также создадите промежуточный CA.

Теперь давайте сгенерируем ключ TLS и сертификат, которые будет использовать наш кластер PostgreSQL. В следующем разделе мы создадим кластер PostgreSQL с именем hippo в пространстве имен pgo (на основе быстрого запуска оператора Postgres). Зная это и в соответствии с тем, как работает DNS в Kubernetes, давайте сгенерируем сертификат с CN hippo.pgo:

```sh
openssl req \ 
-new \ 
-newkey ec \ 
-nodes \ 
-pkeyopt ec_paramgen_curve:prime256v1 \ 
-pkeyopt ec_param_enc:named_curve \ 
-sha384 \ 
-keyout server.key \ 
-out server.csr \ 
-days 365 \ 
-subj "/CN=hippo.pgo"
```

Наконец, примите запрос на подпись сертификата (server.csr), сгенерированный предыдущей командой, и попросите CA подписать его:

```sh
openssl x509 \ 
-req \ 
-in server.csr \ 
-days 365 \ 
-CA ca.crt \ 
-CAkey ca.key \ 
-CAcreateserial \ 
-sha384 \ 
-out server.crt
```

Теперь мы можем перейти к развертыванию кластера PostgreSQL с TLS в Kubernetes!

## Deploying a PostgreSQL Cluster with TLS

Чтобы настроить TLS для вашего кластера PostgreSQL, вы должны создать два секрета. Один, который содержит сертификат CA, а другой, который содержит пару ключей TLS сервера.

Сначала создайте секрет, содержащий ваш сертификат CA. Создайте Секрет как общий Секрет со следующими требованиями:

* Секрет должен находиться в том же пространстве имен, где вы развертываете свой кластер PostgreSQL

* Имя ключа, который удерживает ЦС, должно быть ca.crt

Например, чтобы создать секрет ЦС с доверенным ЦСД для использования для кластеров PostgreSQL в пространстве имен pgo, вы можете выполнить следующую команду:

```sh
kubectl create secret generic postgresql-ca -n pgo --from-file=ca.crt=ca.crt
```

Обратите внимание, что вы можете повторно использовать этот CA Secret для других кластеров PostgreSQL, развернутых оператором Postgres.

Затем создайте секрет, который содержит ваш ключ TLS и сертификат. Вы можете создать это как секрет Kubernetes TLS:

```sh
kubectl create secret tls hippo.tls -n pgo --cert=server.crt --key=server.key
```

С помощью этих секретов вы можете создать кластер PostgreSQL с с включенной TLS именем hippo с помощью следующей команды:

```sh
pgo create cluster hippo \ --server-ca-secret=postgresql-ca \ --server-tls-secret=hippo.tls
```

--Server-ca-secret и --server-tls-secret автоматически включат TLS-соединения в развернутом кластере PostgreSQL. Эти флаги должны ссылаться на CA Secret и TLS Key Pair Secret соответственно. Если вы хотите, чтобы все соединения были через TLS, вы можете добавить флаг --tls-only:

```sh
pgo create cluster hippo --tls-only \ --server-ca-secret=postgresql-ca \ --server-tls-secret=hippo.tls
```

Для остальных примеров я буду использовать кластер Postgres, развернутый с включенным флагом --tls-only.

## SSL/TLS Modes & Connecting to a PostgreSQL Cluster with TLS

Если вы успешно развернуты, при подключении к кластеру PostgreSQL, предполагая, что ваш PGSSLMODE установлен на предпочтительный или более высокий, вы увидите что-то вроде этого в вашем терминале psql:

```sh
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
```

Давайте посмотрим на это в действии и попробуем подключиться к кластеру PostgreSQL и поэкспериментировать с несколькими "режимами TLS", которые предоставляет PostgreSQL. Для простоты в отдельном окне терминала создайте порт-форвард Kubernetes с нашей хост-машины в кластер PostgreSQL:

```sh
kubectl -n pgo port-forward svc/hippo 5432:5432
```

Для целей этого упражнения мы войдем в систему в качестве суперпользователя postgres. Мы будем использовать трюк Kubernetes, чтобы динамически получить пароль из Secret, прежде чем войти в систему и заполнить его в переменной среды PGPASSWORD. Для нашего кластера Hippo это выглядит похоже на это:

```shell
PGPASSWORD=$(kubectl -n pgo get secrets hippo-postgres-secret -o jsonpath="{.data.password}" | base64 -d)
```

Во-первых, давайте посмотрим на sslmode disable - как следует из названия, это означает, что клиент PostgreSQL будет подключаться только к серверу без TLS и отвергать любые попытки подключения через TLS. Если я попытаюсь подключиться к своему кластеру "только TLS" в этом режиме, соединение будет отклонено:

```sh
PGSSLMODE=disable PGPASSWORD=$(kubectl -n pgo get secrets hippo-postgres-secret -o jsonpath="{.data.password}" | base64 -d) psql -h localhost -U postgres hippo 

psql: error: FATAL: no pg_hba.conf entry for host "127.0.0.1", user "postgres", database "hippo", SSL off
```

Следующим sslmode, который следует рассмотреть, является prefer, который является режимом по умолчанию, который использует Postgres. prefer сначала попытается подключиться к кластеру PostgreSQL через TLS, но если TLS недоступен, он вернется к использованию незашифрованного соединения.

Затем есть sslmode require, в котором клиент откажется подключиться к серверу PostgreSQL, если соединение не будет через TLS:

```sh
PGSSLMODE=require PGPASSWORD=$(kubectl -n pgo get secrets hippo-postgres-secret -o jsonpath="{.data.password}" | base64 -d) psql -h localhost -U postgres hippo 

psql (13.1) 

SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
```

Хотя require sslmode защитит ваши соединения от прослушивания, он не защитит от потенциальных атак MITM, так как не выполняет никакой проверки личности: все, что он делает, это проверяет, что он может подключиться к серверу через TLS. Для защиты от MITM вам нужно будет использовать verify-full sslmode (хотя в некоторых ограниченных случаях verify-ca может быть достаточно хорошим).

Давайте сначала посмотрим, что нужно для использования verify-ca sslmode. Давайте возьмем строку подключения выше и используем опцию verify-ca для sslmode:

```sh
PGSSLMODE=verify-ca PGPASSWORD=$(kubectl -n pgo get secrets hippo-postgres-secret -o jsonpath="{.data.password}" | base64 -d) psql -h localhost -U postgres hippo 

psql: error: root certificate file "~/.postgresql/root.crt" does not exist
```

Либо предоставьте файл, либо измените sslmode, чтобы отключить проверку сертификата сервера.

И verify-ca, и verify-full должны знать о пакете доверенного центра сертификации, чтобы они могли проверить подлинность сервера. Приведенная выше подсказка предполагает, что клиент PostgreSQL может использовать доверенный пакет CA, храня его в определенном месте за пределами каталога $HOME, но мы также можем использовать переменную среды PGSSLROOTCERT, чтобы указать на сертификат CA, который мы создали ранее:

```sh
PGSSLMODE=verify-ca PGSSLROOTCERT=ca.crt PGPASSWORD=$(kubectl -n pgo get secrets hippo-postgres-secret -o jsonpath="{.data.password}" | base64 -d) psql -h localhost -U postgres hippo 

psql (13.1) 

SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
```

Verify-full добавляет дополнительную защиту, выполняя проверку личности сервера, проверяя альтернативные имена субъекта или общее имя сертификата и гарантируя, что он соответствует значению хоста (-h). Используя приведенный выше пример, если бы вы переключили sslmode на verify-full, вы бы увидели следующее:

```sh
PGSSLMODE=verify-full PGSSLROOTCERT=ca.crt PGPASSWORD=$(kubectl -n pgo get secrets hippo-postgres-secret -o jsonpath="{.data.password}" | base64 -d) psql -h localhost -U postgres hippo 

psql: error: server certificate for "hippo.pgo" does not match host name "localhost"
```

Учитывая, что это пример использования стратегии переноса выше, мы не сможем подключиться через verify-full, если не добавим псевдоним в файл hosts. Например, добавьте следующее в файл хостов:

```sh
echo '127.0.0.1 hippo.pgo' | sudo tee -a /etc/hosts
```

И попробуйте подключиться снова:

```sh
PGSSLMODE=verify-full PGSSLROOTCERT=ca.crt PGPASSWORD=$(kubectl -n pgo get secrets hippo-postgres-secret -o jsonpath="{.data.password}" | base64 -d) psql -h localhost -U postgres hippo 

psql (13.1) 

SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off) Type "help" for help.
```

Для производственных кластеров, где вы хотите обеспечить надлежащую проверку TLS, вы захотите использовать sslmode verify-full.

## Next Steps

При развертывании кластеров PostgreSQL в производственных средах или в любом месте, где вы работаете в ненадежной сети, имеет первостепенное значение, чтобы вы развернули их с помощью TLS. Оператор PostgreSQL упрощает процесс создания кластеров Postgres с TLS, но вам придется решить, какой уровень проверки личности вы хотите обеспечить для ваших приложений, подключающихся к вашим базам данных.