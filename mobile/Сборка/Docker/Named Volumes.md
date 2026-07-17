## Named Volumes: Где физически хранятся данные?

Когда ты объявляешь:
```yaml
volumes:
  - pg_data:/var/lib/postgresql/data
```

Docker создает папку на **хост-машине** по пути:
```
/var/lib/docker/volumes/имя_проекта_pg_data/_data
```

### Как узнать точный путь?

```bash
# 1. Запусти проект с postgres
$ docker compose up -d

# 2. Посмотри информацию о томе
$ docker volume ls
DRIVER    VOLUME NAME
local     myproject_pg_data  # Docker добавил префикс с именем проекта

# 3. Инспектируй том
$ docker volume inspect myproject_pg_data
[
    {
        "CreatedAt": "2024-01-15T10:00:00Z",
        "Driver": "local",
        "Labels": {
            "com.docker.compose.project": "myproject",
            "com.docker.compose.volume": "pg_data"
        },
        "Mountpoint": "/var/lib/docker/volumes/myproject_pg_data/_data",  # Вот он!
        "Name": "myproject_pg_data",
        "Options": null,
        "Scope": "local"
    }
]

# 4. Теперь можно зайти и посмотреть файлы (нужен sudo)
$ sudo ls -la /var/lib/docker/volumes/myproject_pg_data/_data
total 88
drwx------ 19 999 999  4096 Jan 15 10:00 base
drwx------  2 999 999  4096 Jan 15 10:00 global
-rw-------  1 999 999    88 Jan 15 10:00 postgresql.conf
# ... все файлы PostgreSQL
```

## Почему Docker прячет named volumes?

### 1. **Абстракция и переносимость**
Docker не хочет, чтобы ты зависел от конкретной файловой системы. На разных ОС пути могут отличаться:
- Linux: `/var/lib/docker/volumes/...`
- Mac (Docker Desktop): внутри виртуальной машины
- Windows: внутри WSL2 или виртуалки

### 2. **Безопасность**
Обычный пользователь не имеет доступа к `/var/lib/docker/`. Это защищает данные от случайного изменения или удаления.

### 3. **Управление через Docker**
Docker хочет, чтобы ты работал с томами через его API, а не через файловую систему напрямую.

## Полный путь с учетом проекта

Docker Compose добавляет префикс с именем проекта к именам томов:

```yaml
# docker-compose.yml в папке myapp
volumes:
  - pg_data:/var/lib/postgresql/data
```

Реальный путь будет:
```
/var/lib/docker/volumes/myapp_pg_data/_data
#                        ^^^^^ имя папки = имя_проекта + _ + имя_тома
```

## Как это работает под капотом

Давай проследим весь путь:

```bash
# 1. Контейнер PostgreSQL думает, что пишет в /var/lib/postgresql/data
$ docker exec postgres_container ls -la /var/lib/postgresql/data/
total 88
drwx------ 19 999 999  4096 Jan 15 10:00 base
...

# 2. На самом деле Docker перенаправляет все операции
#    в /var/lib/docker/volumes/myapp_pg_data/_data
$ sudo ls -la /var/lib/docker/volumes/myapp_pg_data/_data
total 88
drwx------ 19 999 999  4096 Jan 15 10:00 base  # Те же файлы!
...

# 3. Docker использует механизм mount (как в Linux)
$ mount | grep pg_data
/dev/sda1 on /var/lib/docker/volumes/myapp_pg_data/_data type ext4 ...
```

## Как получить доступ к файлам named volume?

### Способ 1: Через временный контейнер (рекомендуется)
```bash
# Запустить alpine с примонтированным томом
$ docker run -it --rm -v myapp_pg_data:/data alpine sh

# Теперь внутри контейнера можно работать с файлами
/ # ls -la /data/
total 88
drwx------ 19 999 999  4096 Jan 15 10:00 base
/ # cp /data/postgresql.conf /backup/  # и т.д.
```

### Способ 2: Напрямую через sudo (если очень нужно)
```bash
# Только на Linux, где есть доступ к /var/lib/docker
$ sudo -s
# cd /var/lib/docker/volumes/myapp_pg_data/_data
# ls -la
```

### Способ 3: Создать бэкап
```bash
# Создать архив тома
$ docker run --rm -v myapp_pg_data:/source -v $(pwd):/backup alpine \
  tar czf /backup/postgres_backup.tar.gz -C /source .

# Восстановить из архива
$ docker run --rm -v myapp_pg_data:/target -v $(pwd):/backup alpine \
  tar xzf /backup/postgres_backup.tar.gz -C /target
```

## Сравнение bind mount и named volume

### Bind Mount (твой случай)
```yaml
volumes:
  - ./surreal_data:/mydata
```
- **Где файлы**: `./surreal_data` (рядом с compose файлом)
- **Видимость**: Сразу видно в файловом менеджере
- **Права**: Нужно следить за UID/GID
- **Бэкап**: Просто скопировать папку
- **Перенос**: Нужно копировать папку вручную

### Named Volume
```yaml
volumes:
  - surreal_data:/mydata

volumes:
  surreal_data:
```
- **Где файлы**: `/var/lib/docker/volumes/.../_data`
- **Видимость**: Не видно без специальных команд
- **Права**: Docker управляет автоматически
- **Бэкап**: Через `docker run` с томом
- **Перенос**: Через `docker volume create --driver`

## Дополнительные возможности named volumes

### 1. **Драйверы для разных хранилищ**
```yaml
volumes:
  postgres_data:
    driver: cloudstor:aws  # использовать AWS EBS
    driver_opts:
      size: "100"
      type: gp3
```

### 2. **Лейблы для организации**
```yaml
volumes:
  pg_data:
    labels:
      environment: production
      backup: daily
      owner: database-team
```

### 3. **Внешние тома**
```yaml
volumes:
  pg_data:
    external: true  # использовать уже существующий том
    name: production_postgres_data  # имя внешнего тома
```

## Типичные проблемы и решения

### Проблема: "Я не могу найти файлы своего тома!"
```bash
# Решение: используй inspect
$ docker volume ls | grep myapp
$ docker volume inspect myapp_pg_data
```

### Проблема: "Том занимает много места, где он?"
```bash
# Найти размер всех томов
$ docker system df -v | grep -A5 "Local Volumes"
```

### Проблема: "Нужно перенести данные на другой сервер"
```bash
# Экспорт
$ docker run --rm -v myapp_pg_data:/source -v $(pwd):/backup alpine \
  tar czf /backup/pg_data.tar.gz -C /source .

# Импорт на новом сервере
$ docker run --rm -v myapp_pg_data:/target -v $(pwd):/backup alpine \
  tar xzf /backup/pg_data.tar.gz -C /target
```

## Резюме

1. **Named volumes физически находятся** в `/var/lib/docker/volumes/имя_тома/_data`
2. Docker Compose добавляет **префикс с именем проекта** к имени тома
3. **Не работай напрямую** с файлами через sudo — используй временные контейнеры
4. Named volumes **безопаснее и переносимее** bind mounts
5. Для бэкапов и восстановления используй **docker run** с монтированием томов

