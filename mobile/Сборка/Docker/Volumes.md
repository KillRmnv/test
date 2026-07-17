## Volumes в Docker: Полное руководство

Volumes — это механизм Docker для хранения и управления данными, которые должны жить дольше, чем контейнеры. Это одна из самых важных тем для понимания при работе с Docker.

## Зачем нужны Volumes?

Контейнеры по умолчанию **эфемерны** (временные):
- Все данные, созданные внутри контейнера, существуют только пока жив контейнер
- При удалении контейнера (`docker rm`) все его данные теряются
- При пересоздании контейнера вы начинаете с "чистого листа"

**Volumes решают эту проблему** — они позволяют сохранять данные между запусками контейнеров и даже обмениваться данными между контейнерами.

## Три типа монтирования в Docker

В Docker есть три способа подключить данные к контейнеру:

### 1. **Bind Mounts** (как в твоем примере)
```yaml
volumes:
  - ./surreal_data:/mydata           # относительный путь
  - /home/user/data:/app/data        # абсолютный путь
```
Монтирует конкретную папку с **хоста** в контейнер. 
- **Плюсы**: Просто, удобно для разработки, файлы видны сразу на хосте
- **Минусы**: Зависит от файловой системы хоста, нужно управлять правами

### 2. **Named Volumes** (рекомендуется для продакшна)
```yaml
volumes:
  - db_data:/var/lib/postgresql/data  # имя тома

# Внизу файла нужно объявить том
volumes:
  db_data:  # Docker сам создаст и будет управлять
```
Docker создает специальную папку в своем управляемом пространстве (`/var/lib/docker/volumes/`).
- **Плюсы**: Полностью управляется Docker, безопаснее, переносимее
- **Минусы**: Сложнее получить прямой доступ к файлам с хоста

### 3. **tmpfs Mounts** (для временных данных)
```yaml
tmpfs: /app/temp  # монтируется в оперативную память
```
Данные хранятся только в RAM, никогда не записываются на диск.
- **Плюсы**: Максимальная скорость, данные исчезают при остановке
- **Минусы**: Ограничено размером RAM, только для временных данных

## Bind Mounts — твой случай

В твоем примере используются именно bind mounts:
```yaml
volumes:
  - ./surreal_data:/mydata           # для базы данных
  - ./notebook_data:/app/data        # для приложения
```

### Как это работает
1. `./surreal_data` — папка **на твоем компьютере** (относительно файла compose.yaml)
2. `:/mydata` — папка **внутри контейнера**
3. Docker делает так, что всё, что пишется в `/mydata` внутри контейнера, на самом деле сохраняется в `./surreal_data` на хосте

### Почему это удобно
```bash
# После запуска контейнера ты можешь зайти в папку на хосте
$ cd ./surreal_data
$ ls -la
-rw-r--r-- 1 root root 16384 database.db  # файл базы данных доступен напрямую!

# Можно сделать бэкап простым копированием
$ cp -r surreal_data/ backup/

# Можно посмотреть содержимое без входа в контейнер
$ cat surreal_data/logs.txt
```

## Named Volumes — более продвинутый подход

Давай перепишем твой пример с named volumes:

```yaml
services:
  surrealdb:
    image: surrealdb/surrealdb:v2
    command: start --log info --user root --pass root rocksdb:/mydata/mydatabase.db
    user: root
    ports:
      - "8000:8000"
    volumes:
      - surreal_data:/mydata  # теперь это имя тома, а не путь
    environment:
      - SURREAL_EXPERIMENTAL_GRAPHQL=true
    restart: always

  open_notebook:
    image: lfnovo/open_notebook:v1-latest
    ports:
      - "8502:8502"
      - "5055:5055"
    environment:
      - OPEN_NOTEBOOK_ENCRYPTION_KEY=romanoff
      - SURREAL_URL=ws://surrealdb:8000/rpc
      - SURREAL_USER=root
      - SURREAL_PASSWORD=root
      - SURREAL_NAMESPACE=open_notebook
      - SURREAL_DATABASE=open_notebook
    volumes:
      - notebook_data:/app/data
    depends_on:
      - surrealdb
    restart: always

# Объявляем тома внизу файла
volumes:
  surreal_data:    # Docker создаст их автоматически
  notebook_data:
```

### Преимущества named volumes

1. **Абстракция от хоста** — не важно, где физически лежат файлы
2. **Управление через Docker** — можно использовать специальные команды
3. **Драйверы** — можно использовать облачные драйверы (например, для AWS EBS)
4. **Безопаснее** — права доступа управляются Docker

### Команды для работы с named volumes

```bash
# Список всех томов
$ docker volume ls
DRIVER    VOLUME NAME
local     project_surreal_data
local     project_notebook_data

# Информация о томе
$ docker volume inspect project_surreal_data
[
    {
        "CreatedAt": "2024-01-15T10:00:00Z",
        "Driver": "local",
        "Labels": {
            "com.docker.compose.project": "project",
            "com.docker.compose.volume": "surreal_data"
        },
        "Mountpoint": "/var/lib/docker/volumes/project_surreal_data/_data",
        "Name": "project_surreal_data",
        "Options": null,
        "Scope": "local"
    }
]

# Удалить все неиспользуемые тома
$ docker volume prune

# Создать бэкап тома
$ docker run --rm -v project_surreal_data:/source -v $(pwd):/backup alpine tar czf /backup/surreal_data.tar.gz -C /source .
```

## Синтаксис volumes в Compose

### Короткий синтаксис (как у тебя)
```yaml
volumes:
  - ./host/path:/container/path           # bind mount
  - ./host/path:/container/path:ro        # read-only
  - data_volume:/container/path           # named volume
```

### Длинный синтаксис (больше контроля)
```yaml
volumes:
  - type: bind           # bind, volume, или tmpfs
    source: ./surreal_data
    target: /mydata
    read_only: false     # можно запретить запись
    bind:
      create_host_path: true  # создать папку на хосте, если нет
    
  - type: volume
    source: notebook_data
    target: /app/data
    volume:
      nocopy: true       # не копировать данные из образа при первом монтировании
```

## Важные нюансы

### 1. **Права доступа** (очень важно для bind mounts!)
```yaml
volumes:
  - ./surreal_data:/mydata
```

Если в контейнере процесс работает от root, а на хосте папка принадлежит пользователю:
```bash
$ ls -la
drwxr-xr-x 2 roman roman 4096 surreal_data  # владелец roman
```

Контейнер от root сможет писать в эту папку, но файлы будут создаваться от root:
```bash
$ ls -la surreal_data/
-rw-r--r-- 1 root root 16384 database.db  # файл принадлежит root!
$ rm database.db
rm: remove write-protected file? # просит sudo!
```

**Решение** — либо использовать named volumes, либо синхронизировать UID:
```yaml
services:
  surrealdb:
    user: "${UID}:${GID}"  # передать UID пользователя
```

### 2. **Read-only монтирование**
Если контейнеру нужно только читать данные:
```yaml
volumes:
  - ./config:/app/config:ro  # read-only
  - ./logs:/app/logs:rw      # read-write (по умолчанию)
```

### 3. **Перезапись данных из образа**
Если в образе уже есть данные в папке, а ты монтируешь пустой том:
- Для **bind mounts**: данные из образа **скрываются** (папка на хосте пустая, и в контейнере будет пусто)
- Для **named volumes**: Docker **копирует** данные из образа в том при первом использовании (флаг `nocopy: true` отключает это)

### 4. **Несколько контейнеров могут использовать один том**
```yaml
volumes:
  shared_data:

services:
  app1:
    volumes:
      - shared_data:/data
  app2:
    volumes:
      - shared_data:/data
```

## Практические советы

### Для разработки (как в твоем случае)
Bind mounts отлично подходят:
- Код меняется на хосте — сразу видно в контейнере
- Можно использовать свои любимые редакторы
- Локальные бэкапы простым копированием

### Для продакшна
Используй named volumes:
- Docker управляет данными
- Легче переносить между серверами
- Можно использовать драйверы для облачных хранилищ

### Для баз данных
```yaml
services:
  postgres:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data  # named volume
    environment:
      - POSTGRES_PASSWORD=secret
```

### Для статических файлов
```yaml
services:
  nginx:
    image: nginx
    volumes:
      - ./static_files:/usr/share/nginx/html:ro  # только чтение
```

## Команды для управления volumes

```bash
# Создать том
$ docker volume create my_data

# Скопировать данные в том
$ docker run --rm -v my_data:/data -v $(pwd):/backup alpine cp /backup/file.txt /data/

# Посмотреть содержимое тома
$ docker run --rm -it -v my_data:/data alpine ls -la /data/

# Удалить том
$ docker volume rm my_data

# Удалить все неиспользуемые тома
$ docker volume prune -f
```

