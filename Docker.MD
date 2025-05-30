### Структура проекта
    ````
    ├── myproject/
    │   ├── manage.py
    │   ├── myproject/
    │   │   ├── __init__.py
    │   │   ├── asgi.py
    │   │   ├── settings.py
    │   │   ├── urls.py
    │   │   └── wsgi.py
    │   └── (другие приложения Django)
    ├── static_collected/ (будет создана Django)
    ├── media/ (для загружаемых файлов)
    ├── Dockerfile
    ├── requirements.txt
    ├── nginx.conf
    └── docker-compose.yml
    ````
### Dockerfile для Django-приложения
````
    # Используем официальный образ Python
    FROM python:3.9-slim-buster

    # Устанавливаем переменные окружения
    ENV PYTHONUNBUFFERED 1
    ENV DJANGO_SETTINGS_MODULE myproject.settings

    # Устанавливаем рабочую директорию
    WORKDIR /app

    # Копируем requirements.txt и устанавливаем зависимости
    COPY requirements.txt /app/
    RUN pip install --no-cache-dir -r requirements.txt

    # Копируем остальной код проекта
    COPY . /app/

    # Собираем статические файлы
    RUN python manage.py collectstatic --noinput

    # Открываем порт, на котором будет слушать Gunicorn
    EXPOSE 8000

    # Команда для запуска Gunicorn
    # 0.0.0.0:8000 - Gunicorn будет слушать на всех интерфейсах на порту 8000
    # myproject.wsgi:application - путь к WSGI-приложению Django
    CMD ["gunicorn", "--bind", "0.0.0.0:8000", "myproject.wsgi:application"]
````
### requirements.txt для Django
````
    Django==4.2
    gunicorn==20.1.0
    psycopg2-binary==2.9.3 # Если используете PostgreSQL
````
### nginx.conf (конфигурация Nginx)
````
    upstream django_app {
        # Имя сервиса, которое мы дадим Django-контейнеру в docker-compose.yml
        # и порт, который он EXPOSE'ит
        server django_app:8000;
    }

    server {
        listen 80;
        server_name localhost; # Можете изменить на ваш домен, например yourdomain.com

        # Для статических файлов
        location /static/ {
            # Путь внутри контейнера Nginx, куда будут смонтированы статические файлы
            alias /vol/web/static/;
        }

        # Для медиафайлов
        location /media/ {
            # Путь внутри контейнера Nginx, куда будут смонтированы медиафайлы
            alias /vol/web/media/;
        }

        # Все остальные запросы отправляем на Gunicorn
        location / {
            proxy_pass http://django_app;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $host;
            proxy_redirect off;
        }
    }
````
### docker-compose.yml
````
    version: '3.8'

    services:
    # Сервис для Django-приложения
    django_app:
        build:
        context: . # Контекст сборки - текущая директория (где Dockerfile)
        dockerfile: Dockerfile # Имя Dockerfile
        volumes:
        - ./myproject:/app # Монтируем код проекта, чтобы изменения были видны сразу
        - static_data:/vol/web/static # Том для статических файлов
        - media_data:/vol/web/media   # Том для медиафайлов
        expose:
        - 8000 # Открываем порт 8000 внутри Docker-сети
        env_file: # Если у вас есть переменные окружения, например, SECRET_KEY
        - .env # Например, файл .env в корне проекта
        depends_on:
        - db # Зависимость от сервиса базы данных

    # Сервис для Nginx
    nginx:
        image: nginx:1.23-alpine # Легковесный образ Nginx
        ports:
        - "80:80" # Пробрасываем порт 80 контейнера на порт 80 хоста
        volumes:
        - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro # Копируем нашу конфигурацию Nginx
        - static_data:/vol/web/static:ro # Монтируем статические файлы (только для чтения)
        - media_data:/vol/web/media:ro   # Монтируем медиафайлы (только для чтения)
        depends_on:
        - django_app # Зависимость от сервиса Django-приложения

    # Сервис для базы данных PostgreSQL
    db:
        image: postgres:15-alpine # Легковесный образ PostgreSQL
        environment:
        POSTGRES_DB: myprojectdb # Имя базы данных
        POSTGRES_USER: myuser # Имя пользователя
        POSTGRES_PASSWORD: mypassword # Пароль
        volumes:
        - pg_data:/var/lib/postgresql/data # Том для сохранения данных базы

    volumes:
    static_data:
    media_data:
    pg_data:
````
### Dockerfile для FastAPI-приложения
````
    FROM python:3.9-slim-buster

    WORKDIR /app

    COPY requirements.txt /app/
    RUN pip install --no-cache-dir -r requirements.txt

    COPY ./app /app/app # Копируем папку с FastAPI-приложением
    COPY ./static_files /app/static_files # Если есть статические файлы

    EXPOSE 8000

    # Команда для запуска Uvicorn
    # --host 0.0.0.0 - Uvicorn будет слушать на всех интерфейсах
    # --port 8000 - на порту 8000
    # app.main:app - путь к FastAPI-приложению
    CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
````
### requirements.txt для FastAPI
````
    fastapi==0.95.0
    uvicorn[standard]==0.22.0
    psycopg2-binary==2.9.3 # Если используете PostgreSQL
````
### docker-compose.yml для FastAPI
````
    version: '3.8'

    services:
    fastapi_app:
        build:
        context: .
        dockerfile: Dockerfile
        volumes:
        - ./app:/app/app # Монтируем код FastAPI
        - ./static_files:/app/static_files # Если есть статические файлы, монтируем их тоже
        - static_data:/vol/web/static # Том для статических файлов, если Nginx их раздает
        expose:
        - 8000
        # env_file:
        #  - .env
        depends_on:
        - db

    nginx:
        image: nginx:1.23-alpine
        ports:
        - "80:80"
        volumes:
        - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
        - static_data:/vol/web/static:ro # Монтируем статические файлы для Nginx
        depends_on:
        - fastapi_app

    db:
        image: postgres:15-alpine
        environment:
        POSTGRES_DB: myfastapidb
        POSTGRES_USER: myuser
        POSTGRES_PASSWORD: mypassword
        volumes:
        - pg_data:/var/lib/postgresql/data

    volumes:
    static_data:
    pg_data:
````
### Вопросы Docker
````
Что такое Docker и чем контейнеры отличаются от виртуальных машин?
Объясните разницу между Docker-образом и Docker-контейнером.
Как работает команда docker run? Какие основные параметры можно передать?
Что такое Dockerfile? Напишите минимальный пример Dockerfile для Python-приложения.
Как посмотреть список запущенных контейнеров и их статус? Какие команды вы используете?
Работа с образами
Как создать Docker-образ из Dockerfile? Объясните процесс сборки.
Что такое многоступенчатая сборка (multi-stage build)? Напишите пример Dockerfile с её использованием.
Как оптимизировать размер Docker-образа? Назовите несколько лучших практик.
Как работать с Docker Hub? Как пушить и пуллить образы?
Как удалить неиспользуемые образы и контейнеры? Какие команды для этого нужны?
Контейнеры
Как запустить контейнер в фоновом режиме? Как подключиться к уже запущенному контейнеру?
Как передать переменные окружения в контейнер? Приведите пример.
Как ограничить ресурсы контейнера (CPU, память)? Напишите пример команды.
Как просмотреть логи контейнера? Как следить за логами в реальном времени?
Как перезапустить контейнер с сохранением данных? Что произойдёт, если контейнер удалён?
Docker Compose
Что такое Docker Compose? Для чего он используется?
Напишите пример docker-compose.yml для запуска веб-приложения с базой данных PostgreSQL.
Как масштабировать сервис в Docker Compose? Приведите пример команды.
Как настроить зависимости между сервисами в Docker Compose (например, чтобы база данных запускалась раньше приложения)?
Как использовать переменные окружения в docker-compose.yml? Приведите пример.
Сети и тома
Как работают сети в Docker? Объясните разницу между bridge, host и overlay сетями.
Как настроить пользовательскую сеть в Docker? Напишите пример для Docker Compose.
Что такое Docker-тома (volumes)? Чем они отличаются от bind mounts?
Как создать и подключить том для хранения данных PostgreSQL? Приведите пример.
Как перенести данные между контейнерами с использованием томов или сетей?
Безопасность
Как защитить Docker-контейнеры от уязвимостей? Назовите несколько лучших практик.
Как настроить запуск контейнера от имени непривилегированного пользователя?
Как использовать секреты (secrets) в Docker или Docker Compose? Приведите пример.
Как ограничить доступ к Docker API? Какие настройки безопасности вы применяете?
Как сканировать Docker-образы на наличие уязвимостей (например, с помощью Trivy)?
Оркестрация и масштабирование
Что такое Docker Swarm? Чем он отличается от Kubernetes?
Как настроить кластер в Docker Swarm? Опишите основные шаги.
Как развернуть приложение с балансировкой нагрузки в Docker Swarm?
Как использовать Docker для работы с микросервисной архитектурой? Приведите пример.
Как настроить автоматическое масштабирование сервисов в Docker?
Мониторинг и логирование
Как настроить логирование в Docker? Какие драйверы логов поддерживает Docker?
Как интегрировать Docker с системами мониторинга, такими как Prometheus и Grafana?
Как просмотреть статистику использования ресурсов контейнером (CPU, память)?
Как настроить централизованное логирование для нескольких контейнеров (например, с ELK)?
Как отладить проблемы в запущенном контейнере? Какие команды и подходы вы используете?
Деплой и CI/CD
Как настроить деплой Docker-приложения на сервер с использованием SSH?
Как интегрировать Docker с CI/CD (например, GitHub Actions или Jenkins)? Опишите процесс.
Как настроить автоматический деплой Docker-образа после сборки в CI/CD?
Как использовать Docker для деплоя FastAPI-приложения с Gunicorn и Nginx?
Как откатить изменения при неудачном деплое Docker-приложения?
Продвинутые темы
Как настроить Docker для работы с многоуровневой архитектурой (например, фронтенд, бэкенд, база данных)?
Как использовать Docker для запуска тестов в изолированной среде?
Как настроить Docker для работы с WebSocket-приложениями?
Как минимизировать время сборки Docker-образов с помощью кэширования слоёв?
Как настроить Docker для работы в серверлесс-архитектуре (например, AWS ECS или Lambda)?
Практические задачи
Напишите Dockerfile для FastAPI-приложения с установкой зависимостей из requirements.txt.
Создайте docker-compose.yml для запуска приложения с Redis и PostgreSQL, настроив их взаимодействие.
Реализуйте скрипт для автоматической очистки старых Docker-образов и контейнеров.
Настройте Docker Compose для запуска приложения с балансировкой нагрузки между несколькими контейнерами.
Напишите инструкцию для деплоя Docker-приложения на AWS EC2 с использованием Docker Compose.
Эти вопросы позволяют оценить как теоретические знания Docker, так и практические навыки работы с контейнерами, образами, сетями и деплоем. Они ориентированы на уровень Middle, предполагая, что разработчик имеет опыт создания и деплоя контейнеризированных приложений. Для углублённой проверки можно попросить кандидата написать Dockerfile, настроить Docker Compose или отладить проблему в существующей конфигурации.
````