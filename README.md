# 🐳 Docker CLI Шпаргалка (без Docker Desktop)

**Полный flow работы с Docker в Ubuntu (и других Linux-дистрибутивах) через CLI.**

---

## 📌 Содержание
1. [Установка Docker](#установка-docker)
2. [Основные команды Docker](#основные-команды-docker)
3. [Оптимизация Docker](#оптимизация-docker)
4. [Сетевые настройки](#сетевые-настройки)
5. [Отладка и мониторинг](#отладка-и-мониторинг)
6. [Работа с Dockerfile](#работа-с-dockerfile)
7. [Полезные советы](#полезные-советы)
8. [Частые проблемы и их решения](#частые-проблемы-и-их-решения)
9. [Итоговый flow работы с Docker](#итоговый-flow-работы-с-docker)
10. [Полезные ссылки](#полезные-ссылки)

---

## 🔹 Установка Docker

### 1.1. Установка Docker Engine
```bash
# Обновляем пакеты
sudo apt update && sudo apt upgrade -y

# Устанавливаем зависимости
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Добавляем GPG-ключ Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Добавляем репозиторий Docker (для Ubuntu 20.04)
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Устанавливаем Docker Engine
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Проверяем установку
sudo docker run hello-world


### 1.2. Добавление пользователя в группу `docker`
```bash
# Создаём группу docker (если её нет)
sudo groupadd docker

# Добавляем текущего пользователя в группу
sudo usermod -aG docker $USER

# Применяем изменения (выход/вход или перезагрузка)
newgrp docker
```

### 1.3. Установка Docker Compose
```bash
# Скачиваем последнюю версию Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Делаем файл исполняемым
sudo chmod +x /usr/local/bin/docker-compose

# Проверяем версию
docker-compose --version
```

---

## 🔹 Основные команды Docker

---

### 📌 Работа с контейнерами

| Команда | Описание |
|---------|----------|
| `docker ps` | Показывает **запущенные** контейнеры |
| `docker ps -a` | Показывает **все** контейнеры (включая остановленные) |
| `docker ps -aq` | Выводит только **ID** всех контейнеров |
| `docker run [OPTIONS] IMAGE [COMMAND]` | Запускает контейнер из образа |
| `docker run -it ubuntu bash` | Запускает контейнер с **интерактивной сессией** (bash) |
| `docker run -d --name my_container nginx` | Запускает контейнер в **фоновом режиме** (`-d`) с именем `my_container` |
| `docker start <container_id>` | Запускает остановленный контейнер |
| `docker stop <container_id>` | Останавливает контейнер |
| `docker restart <container_id>` | Перезапускает контейнер |
| `docker rm <container_id>` | Удаляет контейнер |
| `docker rm -f <container_id>` | Удаляет **принудительно** работающий контейнер |
| `docker rm $(docker ps -aq)` | Удаляет **все** контейнеры |
| `docker exec -it <container_id> bash` | Заходит **внутрь** работающего контейнера |
| `docker logs <container_id>` | Показывает логи контейнера |
| `docker logs -f <container_id>` | Показывает логи в **режиме слежения** (`-f`) |
| `docker inspect <container_id>` | Показывает **детальную информацию** о контейнере (в JSON) |
| `docker stats` | Показывает **потребление ресурсов** контейнерами в реальном времени |

**Примеры:**
```bash
# Запуск контейнера с ограничением памяти и CPU
docker run --memory="512m" --cpus="1" -d nginx

# Запуск контейнера с пробросом портов (8080 на хосте → 80 в контейнере)
docker run -p 8080:80 -d nginx

# Запуск контейнера с подключением тома (папки на хосте)
docker run -v /path/on/host:/path/in/container -d nginx

# Запуск контейнера с переменными окружения
docker run -e "ENV_VAR=value" -d nginx
```

---

### 📌 Работа с образами

| Команда | Описание |
|---------|----------|
| `docker images` | Показывает **все локальные образы** |
| `docker pull <image>` | Скачивает образ с Docker Hub |
| `docker rmi <image_id>` | Удаляет образ |
| `docker rmi $(docker images -q)` | Удаляет **все** локальные образы |
| `docker build -t my_image .` | Собирает образ из **Dockerfile** в текущей папке |
| `docker tag <image_id> my_image:1.0` | Создаёт тег для образа |
| `docker push my_image:1.0` | Загружает образ в **Docker Hub** (требует авторизации) |
| `docker history <image>` | Показывает **историю слоёв** образа |

**Примеры:**
```bash
# Сборка образа из Dockerfile
docker build -t my_app:latest .

# Удаление всех неиспользуемых образов
docker image prune -a

# Поиск образа на Docker Hub
docker search nginx
```

---

### 📌 Работа с томами (volumes)

| Команда | Описание |
|---------|----------|
| `docker volume ls` | Показывает все тома |
| `docker volume create my_volume` | Создаёт новый том |
| `docker volume inspect my_volume` | Показывает информацию о томе |
| `docker volume rm my_volume` | Удаляет том |
| `docker volume prune` | Удаляет **все неиспользуемые** тома |

**Примеры:**
```bash
# Запуск контейнера с подключением тома
docker run -v my_volume:/data -d nginx

# Запуск контейнера с подключением **папки на хосте**
docker run -v /home/user/data:/data -d nginx
```

---

### 📌 Работа с сетями

| Команда | Описание |
|---------|----------|
| `docker network ls` | Показывает все сети |
| `docker network create my_network` | Создаёт новую сеть |
| `docker network inspect my_network` | Показывает информацию о сети |
| `docker network rm my_network` | Удаляет сеть |
| `docker network prune` | Удаляет **все неиспользуемые** сети |

**Примеры:**
```bash
# Создание пользовательской сети
docker network create my_app_network

# Запуск контейнера в пользовательской сети
docker run --network=my_app_network -d nginx
```

---

### 📌 Работа с Docker Compose

| Команда | Описание |
|---------|----------|
| `docker-compose up` | Запускает сервисы из `docker-compose.yml` |
| `docker-compose up -d` | Запускает сервисы в **фоновом режиме** |
| `docker-compose down` | Останавливает и удаляет контейнеры, сети и тома |
| `docker-compose ps` | Показывает статус сервисов |
| `docker-compose logs` | Показывает логи всех сервисов |
| `docker-compose logs -f` | Показывает логи в **режиме слежения** |
| `docker-compose build` | Собирает образы для сервисов |
| `docker-compose pull` | Скачивает образы для сервисов |
| `docker-compose exec <service> bash` | Заходит внутрь контейнера сервиса |

**Пример `docker-compose.yml`:**
```yaml
version: "3.8"
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: "0.5"
  db:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD: mypassword
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

---

## 🔹 Оптимизация Docker

---

### 📌 Ограничение ресурсов для контейнеров
```bash
# Ограничение памяти и CPU
docker run --memory="512m" --cpus="1" -d nginx
```

**В Docker Compose:**
```yaml
services:
  my_service:
    image: nginx
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: "0.5"
```

---

### 📌 Очистка неиспользуемых ресурсов
```bash
# Удаление остановленных контейнеров
docker container prune

# Удаление неиспользуемых образов
docker image prune -a

# Удаление неиспользуемых томов
docker volume prune

# Удаление неиспользуемых сетей
docker network prune

# Полная очистка (всё выше + кэш сборки)
docker system prune -a --volumes
```

---
### 📌 Настройка Docker Daemon
Файл конфигурации: `/etc/docker/daemon.json`

**Пример ограничения памяти для Docker Daemon:**
```json
{
  "default-ulimits": {
    "memlock": {
      "Name": "memlock",
      "Hard": -1,
      "Soft": -1
    }
  },
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

**Применение изменений:**
```bash
sudo systemctl restart docker
```

---

## 🔹 Сетевые настройки

---

### 📌 Проброс портов
```bash
# Проброс порта 8080 на хосте → 80 в контейнере
docker run -p 8080:80 -d nginx

# Проброс всех портов (не рекомендуется)
docker run -P -d nginx
```

---
### 📌 Подключение к сети хоста
```bash
# Запуск контейнера с сетью хоста
docker run --network=host -d nginx
```

---
### 📌 Создание пользовательской сети
```bash
# Создание сети
docker network create my_network

# Подключение контейнера к сети
docker run --network=my_network -d nginx
```

---

## 🔹 Отладка и мониторинг

---

### 📌 Просмотр логов
```bash
# Логи контейнера
docker logs <container_id>

# Логи в реальном времени
docker logs -f <container_id>

# Логи с временными метками
docker logs -t <container_id>
```

---
### 📌 Мониторинг ресурсов
```bash
# Потребление ресурсов контейнерами
docker stats

# Потребление ресурсов всеми процессами (включая Docker)
htop
```

---
### 📌 Вход в контейнер
```bash
# Вход в контейнер (если есть bash)
docker exec -it <container_id> bash

# Вход в контейнер (если нет bash, но есть sh)
docker exec -it <container_id> sh
```

---
### 📌 Копирование файлов
```bash
# Копирование файла из контейнера на хост
docker cp <container_id>:/path/in/container /path/on/host

# Копирование файла с хоста в контейнер
docker cp /path/on/host <container_id>:/path/in/container
```

---

## 🔹 Работа с Dockerfile

---

### 📌 Пример Dockerfile
```dockerfile
# Базовый образ
FROM ubuntu:20.04

# Устанавливаем зависимости
RUN apt update && apt install -y curl

# Создаём рабочую директорию
WORKDIR /app

# Копируем файлы из хоста в контейнер
COPY . .

# Устанавливаем переменные окружения
ENV MY_VAR=value

# Запускаем команду при старте контейнера
CMD ["bash"]
```

---
### 📌 Сборка образа
```bash
docker build -t my_image:latest .
```

---
### 📌 Запуск контейнера из образа
```bash
docker run -d my_image:latest
```

---

## 🔹 Полезные советы

---

### 📌 Как уменьшить размер образов?
- Используйте **многоступенчатую сборку** (multi-stage builds):
  ```dockerfile
  FROM node:14 as builder
  WORKDIR /app
  COPY . .
  RUN npm install && npm run build

  FROM nginx:alpine
  COPY --from=builder /app/dist /usr/share/nginx/html
  ```

- Удаляйте кэш и временные файлы в одном слое:
  ```dockerfile
  RUN apt update && apt install -y curl && rm -rf /var/lib/apt/lists/*
  ```

---
### 📌 Как ускорить сборку?
- Используйте **кэш Docker**: порядок команд в Dockerfile важен! Ставьте часто меняющиеся команды (например, `COPY . .`) **после** установки зависимостей.
- Используйте **`.dockerignore`**, чтобы исключить ненужные файлы из контекста сборки.

---
### 📌 Как работать с Docker без root?
- Добавьте пользователя в группу `docker` (как описано выше).
- Если всё равно требуется `sudo`, проверьте права на `/var/run/docker.sock`:
  ```bash
  sudo chmod 666 /var/run/docker.sock
  ```
  *(Не рекомендуется для продакшн-систем из-за безопасности!)*

---

## 🔹 Частые проблемы и их решения

| Проблема | Решение |
|----------|---------|
| **`Cannot connect to the Docker daemon`** | Проверьте, запущен ли Docker: `sudo systemctl start docker` |
| **`Permission denied`** | Добавьте пользователя в группу `docker` и перезайдите |
| **Контейнер не запускается** | Проверьте логи: `docker logs <container_id>` |
| **Не хватает памяти** | Ограничьте память для контейнера: `--memory="512m"` |
| **Docker занимает много места** | Очистите неиспользуемые ресурсы: `docker system prune -a --volumes` |
| **Ошибка при сборке образа** | Проверьте Dockerfile на синтаксические ошибки |
| **Порт занят** | Проверьте, какой процесс занимает порт: `sudo lsof -i :8080` |

---

## 🔹 Итоговый flow работы с Docker

1. **Установите Docker** (`docker-ce`, `docker-compose`).
2. **Добавьте пользователя в группу `docker`**.
3. **Создайте Dockerfile** для своего приложения.
4. **Соберите образ**: `docker build -t my_app .`.
5. **Запустите контейнер**:
   - В фоновом режиме: `docker run -d -p 8080:80 my_app`.
   - С ограничением ресурсов: `docker run --memory="512m" --cpus="1" -d my_app`.
6. **Проверьте работу**: `docker ps`, `docker logs`.
7. **Оптимизируйте**:
   - Очищайте неиспользуемые ресурсы (`docker system prune`).
   - Используйте Docker Compose для сложных приложений.
8. **Отладьте** при необходимости (`docker exec`, `docker inspect`).

---

## 🔹 Полезные ссылки
- [Официальная документация Docker](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Docker Hub](https://hub.docker.com/) (репозиторий образов)
- [Best Practices for Writing Dockerfiles](https://docs.docker.com/develop/dev-best-practices/)

---
**💡 Совет:** Сохраните этот файл как `docker-cheatsheet.md` и используйте его как справочник!
```
