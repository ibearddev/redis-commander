# Redis Commander

Redis UI (web Ui) написан на node.js.
Инструмент позволяет просматривать ключи в интерфейсе.
Отлично подойдет для мониторинга ключей в кубике (используйте в dev версии).
Данный инструмент отлично подойдет для QA команды, которой не дают доступ в redis-cli, но проверять существование ключей необходимо)

# Установка

```bash
$ npm install -g redis-commander
$ redis-commander
```

Установка через **yarn** в настоящее время не поддерживается. Только менеджер пакетов **npm**.

Лучшей практикой будет запуск Redis Commander с помощью Docker rediscommander/redis-commander(инструкции см. Ниже).

# Функционал

Веб-интерфейс для отображения и редактирования данных на нескольких разных серверах Redis.

Поддерживает следующие типы данных для просмотра, добавления, обновления и удаления данных:
* Strings
* Lists
* Sets
* Sorted Set
* Streams (базовая поддержка на основе проекта HFXBus из https://github.com/exocet-engineering/hfx-bus , только просмотр / добавление / удаление данных)
* ReJSON (базовая поддержка, только для просмотра значений ключей типа ReJSON)

# Usage

```
$ redis-commander --help
Options:
  --redis-port                         Указать порт для Redis.                                [string]
  --redis-host                         Указать хост для Redis.                                [string]
  --redis-socket                       Указать unix-socket для Redis.                         [string]
  --redis-password                     Указать пароль для Redis.                              [string]
  --redis-db                           Указать бд для Redis.                                  [string]
  --redis-label                        Указать метку для соединения.                          [string]
  --redis-tls                          Использовать TLS для подключения к серверу redis или sentinel. [boolean] [по умолчанию: false]
  --redis-optional                     Установить значение true, если автопереподключение не должно выполняться, если сервер не работает [boolean] [по умолчанию: false]
  --sentinel-port                      Указать порт Redis для подключения к sentinel.         [string]
  --sentinel-host                      Указать хост Redis для подключения к sentinel.         [string]
  --sentinels                          Список sentinel разделенный запятыми host:port.        [string]
  --sentinel-name                      Указать группу sentinel.                               [string]  [по умолчанию: mymaster]
  --sentinel-password                  Указать пароль sentinel инстанса.                      [string]
  --http-auth-username, --http-u       Указать пользователя (username) для http authorisation.[string]
  --http-auth-password, --http-p       Указать пароль для http authorisation.                 [string]
  --http-auth-password-hash, --http-h  Указать хеш пароля для http authorisation.             [string]
  --address, -a                        Указать хост для запуска сервера.                      [string]  [по умолчанию: 0.0.0.0]
  --port, -p                           Указать порт для запуска сервера.                      [string]  [по умолчанию: 8081]
  --url-prefix, -u                     Указать префикс URL-адреса куда слать ответы.          [string]  [по умолчанию: ""]
  --root-pattern, --rp                 Указать корневой шаблон ключей Redis.                  [string]  [по умолчанию: "*"]
  --read-only                          Указать нужно ли запускать в режиме только чтение.     [boolean] [по умолчанию: false]
  --trust-proxy                        Указать нужно ли запускать за прокси (включите Express «доверенные прокси») [boolean|string] [по умолчанию: false]
  --nosave, --ns                       Указать можно ли не сохранять новые подключения в файл конфигурации. [boolean] [по умолчанию: true]
  --noload, --nl                       Указать можно ли не использовать соединения из конфигурации.         [boolean] [по умолчанию: false]
  --use-scan, --sc                     Указать командеру использовать команду scan вместо keys.             [boolean] [по умолчанию: false]
  --clear-config, --cc                 Очистить файл конфигурации.
  --migrate-config                     Перенести старый файл конфигурации из $HOME в новый стиль.
  --scan-count, --sc                   Указать размер каждого отдельного scan.     [integer] [по умолчанию: 100]
  --no-log-data                        Указать можно ли не записывать значения данных из хранилища Redis.    [boolean] [по умолчанию: false]
  --open                               Открыть веб-браузер с помощью Redis-Commander.      [boolean] [по умолчанию: false]
  --folding-char, --fc                 Указать символ для складывания ключей в дереве.     [character] [по умолчанию: ":"]
  --test, -t                           Протестировать конфигурацию, провалидировать (file, env-vars, command line)
```

Соединение может быть установлено либо через прямое соединение с сервером redis, либо косвенно через экземпляр дозорного.

## Конфигурация

Redis Commander можно настроить с помощью файлов конфигурации, переменных среды или параметров командной строки. Различные типы значений конфигурации перезаписывают друг друга, используется только последнее (наиболее важное) значение.

Для файлов конфигурации используется node-config модуль ( https://github.com/lorenwest/node-config ) с синтаксисом по умолчанию json.

Порядок приоритета для всех значений конфигурации (от наименьшего к наиболее важному):
- Файлы конфигурации

  `default.json` - этот файл содержит все значения по умолчанию и НЕ ДОЛЖЕН изменяться

  `local.json` - необязательный файл, здесь должны быть размещены все локальные переопределения значений `default.json`, а также список подключений redis, которые используются при запуске

  `local-<NODE_ENV>.json` - Не добавляйте в этот файл ничего, кроме подключений! Redis Commander перезапишет этот файл каждый раз, когда соединение добавляется или удаляется через пользовательский интерфейс. Внутри контейнера Docker этот файл используется для хранения всех подключений, проанализированных из REDIS_HOSTS env var. Этот файл перезаписывает все соединения, определенные внутри `local.json`

  Есть еще несколько возможных файлов, доступных для использования - пожалуйста, проверьте wiki node-config, чтобы получить полный список всех возможных [файлов](https://github.com/lorenwest/node-config/wiki/Configuration-Files)

- Переменные среды - полный список возможных переменных окружения (кроме специфичных для Docker) можно получить из файла `config/custom-environment-variables.json` вместе с их сопоставлением с соответствующим ключом конфигурации.

- Параметры командной строки - перезаписывает все

Чтобы проверить окончательную конфигурацию, созданную из файлов, набор env-vars и параметры командной строки перезаписывают start redis commander с дополнительным параметром "--test". Все недопустимые ключи конфигурации будут перечислены в выводе. Тест конфигурации не проверяет, можно ли разрешить имена хостов или IP-адреса.

Дополнительную информацию можно найти в документации на [docs/configuration.md](docs/configuration.md)
и [docs/connections.md](docs/connections.md).

## Переменные среды (ENV)

Эти переменные среды можно использовать для запуска Redis Commander как обычного приложения или внутри контейнера докеров (определенного внутри файла `config/custom-environment-variables.json`):

```
HTTP_USER
HTTP_PASSWORD
HTTP_PASSWORD_HASH
ADDRESS
PORT
READ_ONLY
URL_PREFIX
ROOT_PATTERN
NOSAVE
NO_LOG_DATA
FOLDING_CHAR
VIEW_JSON_DEFAULT
USE_SCAN
SCAN_COUNT
FLUSH_ON_IMPORT
REDIS_CONNECTION_NAME
REDIS_LABEL
CLIENT_MAX_BODY_SIZE
BINARY_AS_HEX
```

## Docker

Все переменные среды, перечисленные в разделе «Переменные среды», можно использовать при запуске образа с Docker. Также доступны следующие дополнительные переменные среды (определенные внутри скрипта запуска докера):

```
REDIS_PORT
REDIS_HOST
REDIS_SOCKET
REDIS_TLS
REDIS_PASSWORD
REDIS_DB
REDIS_HOSTS
REDIS_OPTIONAL
SENTINEL_PORT
SENTINEL_HOST
SENTINELS
SENTINEL_NAME
SENTINEL_PASSWORD
HTTP_PASSWORD_FILE
REDIS_PASSWORD_FILE
SENTINEL_PASSWORD_FILE
K8S_SIGTERM
```

Переменная `K8S_SIGTERM` (по умолчанию "0") может быть установлено в "1" для работы c kubernetes specificas, чтобы обеспечит возможность замены pod с нулевым временем простоя. Более подробную информацию о том, как kubernetes обрабатывает завершение работы старых pod и настройку новых, можно найти в ветке [https://github.com/kubernetes/contrib/issues/1140#issuecomment-290836405]

Хосты можно дополнительно указать с помощью строки, разделенной запятыми, путем установки `REDIS_HOSTS` в переменных среды.

После запуска контейнера, `redis-commander` будет доступен по адресу [localhost:8081](http://localhost:8081).

### Valid host

`REDIS_HOSTS` переменная окружения представляет собой разделенный запятыми список определений хостов, где каждый хост должен следовать один из этих шаблонов:

`hostname`

`label:hostname`

`label:hostname:port`

`label:hostname:port:dbIndex`

`label:hostname:port:dbIndex:password`

Строки подключения, определенные с помощью `REDIS_HOSTS` переменной, не поддерживают подключения TLS. Если удаленному серверу Redis требуется TLS, запишите все подключения в файл конфигурации вместо использования REDIS_HOSTS(см. [docs/connections.md](docs/connections.md) в конце более сложные примеры).

### С помощью docker-compose

```yml
version: '3'
services:
  redis:
    container_name: redis
    hostname: redis
    image: redis

  redis-commander:
    container_name: redis-commander
    hostname: redis-commander
    image: rediscommander/redis-commander:latest
    restart: always
    environment:
    - REDIS_HOSTS=local:redis:6379
    ports:
    - "8081:8081"
```

### Без docker-compose

#### Простой

Если вы используете Redis по пути:порту `localhost:6379`, тогда выполните команду и этого будет достаточно.

```bash
docker run --rm --name redis-commander -d \
  -p 8081:8081 \
  rediscommander/redis-commander:latest
```

#### Если нужно указать хост

```bash
docker run --rm --name redis-commander -d \
  --env REDIS_HOSTS=10.10.20.30 \
  -p 8081:8081 \
  rediscommander/redis-commander:latest
```

#### Если нужно указать несколько хостов с метками

```bash
docker run --rm --name redis-commander -d \
  --env REDIS_HOSTS=local:localhost:6379,myredis:10.10.20.30 \
  -p 8081:8081 \
  rediscommander/redis-commander:latest
```

## Kubernetes

Пример развертывания можно найти на [k8s/redis-commander/deployment.yaml](k8s/redis-commander/deployment.yaml).

Если у вас уже есть кластер, работающий с `redis` iв пространстве имен по умолчанию, разверните `redis-commander` с помощью `kubectl apply -f k8s/redis-commander`. Если у вас еще нет `redis` запущенной версии, вы можете развернуть простой модуль с помощью `kubectl apply -f k8s/redis`.

В качестве альтернативы вы можете добавить контейнер в спецификацию развертывания следующим образом:

```
containers:
- name: redis-commander
  image: rediscommander/redis-commander
  env:
  - name: REDIS_HOSTS
    value: instance1:redis:6379
  ports:
  - name: redis-commander
    containerPort: 8081
```

известные проблемы с Kubernetes:

* использование REDIS_HOSTS работает только с Redis db без пароля. Вы должны указать REDIS_HOST на Redis db, защищенном паролем


## Helm chart

Вы можете установить приложение на любой кластер Kubernetes с помощью Helm. В настоящее время нет доступного репозитория helm, поэтому требуется локальная проверка источников helm внутри этой репы:

```sh
helm -n myspace install redis-web-ui ./k8s/helm-chart/redis-commander
```

Дополнительно [документация](k8s/helm-chart/README.md) об Helm chart и ее значениях.

## OpenShift V3

Чтобы использовать Node.js образ, используйте следующее.

1. Откройте Каталог и выберите шаблон Node.js
1. Укажите имя приложения и URL-адрес [redis-command github repository](https://github.com/joeferner/redis-commander.git)
1. Клик на ссылку ```advanced options```
1. (optional) укажите имя хоста для маршрута - _если оно не указано, оно будет сгенерировано_
1. В разделе "Конфигурация развертывания"
   * Добавьте переменную среды ```REDIS_HOST``` , значение которой является именем службы Redis, например ```redis```
   * Добавьте переменную среды ```REDIS_PORT```, значение которой представляет собой порт, доступный для службы Redis, например ```6379```
   * Добавьте пароль, сгенерированного в [redis шаблон](https://github.com/sclorg/redis-container/blob/master/examples/redis-persistent-template.json):
     * name: ```REDIS_PASSWORD```
     * resource: ```redis```
     * key: ```database-password```
1. (optional) укажите метку, например ```appl=redis-commander-dev1```
   * _эта метка будет применена ко всем созданным объектам, что позволит легко удалить их позже с помощью:_
   ```bash
   oc delete all --selector appl=redis-commander-dev1
   ```

## Создание своего образа Docker на основе данного

Чтобы использовать эти образы в качестве базового образа для других образов, вам необходимо вызвать "apk update" внутри вашего Dockerfile перед добавлением других пакетов apk с помощью "apk add foo". 
Впоследствии, чтобы уменьшить размер изображения, вы можете снова удалить все временные конфигурации apk, как это делает этот файл Docker.
