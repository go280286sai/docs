### Fastapi
Uvicorn — это высокопроизводительный ASGI-сервер,  для работы с асинхронными веб-фреймворками. Он поддерживает HTTP/1.1
````
    pip install fastapi
    pip install uvicorn[standard]
````
Hypercorn - сервер поддерживает HTTP/2 и может работать с SSL/TLS, что необходимо для полноценного использования HTTP/2.
#### Пример
````
    from fastapi import FastAPI
    
    app = FastAPI()
    
    # Обработчик для GET-запроса к корневому URL
    @app.get("/")
    async def read_root():
        return {"message": "Hello, FastAPI!"}
    
    # Обработчик для GET-запроса по пути /items/
    @app.get("/items/")
    async def read_items():
        return [{"item_name": "Foo"}, {"item_name": "Bar"}]
    
    # Обработчик для POST-запроса по пути /create_item/
    @app.post("/create_item/")
    async def create_item():
        return {"status": "Item created successfully"}
    
    # Обработчик для PUT-запроса по пути /update_item/
    @app.put("/update_item/")
    async def update_item():
        return {"status": "Item updated successfully"}
    
    # Обработчик для DELETE-запроса по пути /delete_item/
    @app.delete("/delete_item/")
    async def delete_item():
        return {"status": "Item deleted successfully"}
````
#### Запуск
````
    uvicorn main:app --reload
    # http://127.0.0.1:8000/
    # http://127.0.0.1:8000/docs
````
#### Тело запроса (Request Body) с Pydantic
````
    from fastapi import FastAPI
    from pydantic import BaseModel
    
    app = FastAPI()
    
    # Определяем Pydantic модель для элемента
    class Item(BaseModel):
        name: str
        description: str | None = None  # Опциональное поле
        price: float
        tax: float | None = None       # Опциональное поле
    
    # Обработчик для создания элемента с телом запроса
    @app.post("/items_body/")
    async def create_item_body(item: Item):
        return item
    
    # Обработчик для обновления элемента с телом запроса и параметром пути
    @app.put("/items_body/{item_id}")
    async def update_item_body(item_id: int, item: Item):
        return {"item_id": item_id, **item.model_dump()} # .model_dump() для Pydantic v2
````
####  Валидация и метаданные с Query, Path, Body, Form, File, Cookie, Header
````
    from fastapi import FastAPI, Query, Path, Body
    from pydantic import BaseModel
    
    app = FastAPI()
    
    class Item(BaseModel):
        name: str
        description: str | None = None
        price: float
        tax: float | None = None
    
    @app.get("/items_advanced/{item_id}")
    async def read_items_advanced(
        item_id: int = Path(..., title="ID элемента", gt=0), # Обязательный параметр пути, больше 0
        q: str | None = Query(None, min_length=3, max_length=50, description="Параметр поиска") # Опциональный параметр запроса, с мин/макс длиной и описанием
    ):
        results = {"item_id": item_id}
        if q:
            results.update({"q": q})
        return results
    
    @app.post("/items_advanced_body/")
    async def create_item_advanced_body(
        item: Item = Body(..., example={"name": "Foo", "description": "A nice item", "price": 35.4, "tax": 3.2}), # Пример для документации
        importance: int = Body(..., gt=0, lt=10, description="Важность элемента") # Дополнительное поле в теле запроса
    ):
        return {"item": item, "importance": importance}
````
#### Ответы (Responses)
````
    from fastapi import FastAPI, Response, status
    from fastapi.responses import HTMLResponse, JSONResponse
    
    app = FastAPI()
    
    @app.get("/plain_text/", response_class=Response)
    async def get_plain_text():
        return Response(content="Привет, это простой текст!", media_type="text/plain")
    
    @app.get("/html_content/", response_class=HTMLResponse)
    async def get_html_content():
        return """
        <html>
            <head>
                <title>HTML Страница</title>
            </head>
            <body>
                <h1>Привет из FastAPI!</h1>
                <p>Это HTML контент.</p>
            </body>
        </html>
        """
    
    @app.get("/json_response/", response_class=JSONResponse)
    async def get_json_response():
        return JSONResponse(content={"message": "Это кастомный JSON ответ"})
    
    # Установка статуса ответа
    @app.post("/create_resource/", status_code=status.HTTP_201_CREATED)
    async def create_resource():
        return {"message": "Ресурс успешно создан"}
````
#### Настройка CORS в FastAPI
````
   pip install fastapi "uvicorn[standard]" python-multipart
````
````
    # my-app-backend/main.py
    from fastapi import FastAPI
    from fastapi.middleware.cors import CORSMiddleware
    from fastapi.staticfiles import StaticFiles # Импортируем StaticFiles
    from starlette.responses import FileResponse # Для index.html
    
    app = FastAPI()
    
    # Настройка CORS (если все еще нужно, для API)
    app.add_middleware(
        CORSMiddleware,
        allow_origins=["*"], # В продакшене укажите конкретные домены!
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )
    
    @app.get("/api/") # Ваши API маршруты
    async def read_root():
        return {"message": "Hello from FastAPI API!"}
    
    # ... (другие ваши API маршруты)
    
    # Важно: монтируйте StaticFiles ПОСЛЕ всех ваших API маршрутов
    # Иначе FastAPI может перехватывать запросы, предназначенные для API
    app.mount("/static", StaticFiles(directory="static"), name="static")
    
    # Маршрут для обработки SPA (Single Page Application)
    # Это позволяет React Router обрабатывать маршруты на клиенте
    @app.get("/{full_path:path}")
    async def serve_react_app(full_path: str):
        # Проверяем, существует ли запрошенный путь как статический файл
        # (например, для css/js файлов, которые не находятся внутри /static)
        # В большинстве случаев, для React SPA достаточно просто отдать index.html
        return FileResponse("static/index.html")
````
#### Маршрутизаторы (Routers)
````
    # main.py
    from fastapi import FastAPI
    from routers import items, users # Импортируем роутеры из других файлов
    
    app = FastAPI()
    
    app.include_router(items.router, prefix="/items", tags=["items"])
    app.include_router(users.router, prefix="/users", tags=["users"])
    
    @app.get("/")
    async def root():
        return {"message": "Welcome to the main API!"}
    
    # routers/items.py
    from fastapi import APIRouter
    
    router = APIRouter()
    
    @router.get("/")
    async def read_items_router():
        return [{"name": "Item A"}, {"name": "Item B"}]
    
    @router.get("/{item_id}")
    async def read_item_router(item_id: int):
        return {"item_id": item_id, "name": f"Item {item_id}"}
    
    # routers/users.py
    from fastapi import APIRouter
    
    router = APIRouter()
    
    @router.get("/")
    async def read_users_router():
        return [{"name": "John Doe"}, {"name": "Jane Smith"}]
    
    @router.get("/{user_id}")
    async def read_user_router(user_id: int):
        return {"user_id": user_id, "name": f"User {user_id}"}
````
### Работа с куками и сессиями
````
    from fastapi import FastAPI, Response, Request, Cookie
    from fastapi.responses import JSONResponse
    
    app = FastAPI()
    
    @app.post("/set_cookie/")
    async def set_cookie(response: Response):
        response.set_cookie(key="username", value="Alex", httponly=True)
        return {"message": "Куки установлены!"}
    
    @app.get("/get_cookie/")
    async def get_cookie(username: str | None = Cookie(None)):
        return {"username": username} if username else {"message": "Куки не найдены"}
    
    @app.post("/session/")
    async def session_example(request: Request):
        session_data = await request.json()
        request.session["data"] = session_data
        return JSONResponse({"message": "Сессия обновлена!", "session": session_data})
````
#### Загрузка изображений через форму
````
    from fastapi import FastAPI, File, UploadFile
    import shutil
    
    app = FastAPI()
    
    @app.post("/upload-image/")
    async def upload_image(file: UploadFile = File(...)):
        with open(f"uploads/{file.filename}", "wb") as buffer:
            shutil.copyfileobj(file.file, buffer)
        
        return {"filename": file.filename, "message": "Файл загружен успешно!"}
````
Отправка формы (multipart)
````
    from fastapi import FastAPI, Form
    
    app = FastAPI()
    
    @app.post("/submit-form/")
    async def submit_form(name: str = Form(...), age: int = Form(...)):
        return {"name": name, "age": age}
````

### Вопросы
````
Что такое FastAPI и чем он отличается от других Python-фреймворков, таких как Flask или Django?
Объясните, почему FastAPI использует асинхронное программирование. Какие преимущества это даёт?
Как создать базовое приложение на FastAPI? Напишите минимальный пример с одним эндпоинтом.
Что такое APIRouter и как его использовать для организации маршрутов?
Как FastAPI генерирует интерактивную документацию (Swagger/ReDoc)? Как её настроить или отключить?
Маршрутизация и обработка запросов
Как обрабатывать GET, POST, PUT, DELETE запросы в FastAPI? Приведите пример эндпоинта для каждого метода.
Как передавать параметры в эндпоинты (path parameters, query parameters)? В чём их различия?
Как обрабатывать данные из тела запроса? Напишите пример эндпоинта с JSON-телом.
Как реализовать валидацию параметров запроса с помощью Pydantic? Приведите пример модели.
Как обрабатывать файлы, загруженные через POST-запрос? Напишите пример загрузки изображения.
Pydantic и валидация
Что такое Pydantic и как он интегрируется с FastAPI? Приведите пример модели с валидацией.
Как создать кастомные валидаторы в Pydantic? Напишите пример проверки email.
Как использовать Pydantic для обработки вложенных данных (например, списка объектов)?
Как настроить сериализацию и десериализацию данных в Pydantic (например, исключить поля)?
Как обрабатывать опциональные поля в Pydantic-моделях?
Асинхронное программирование
Чем отличаются синхронные (def) и асинхронные (async def) эндпоинты в FastAPI? Когда использовать каждую?
Как использовать asyncio в FastAPI для выполнения длительных операций? Приведите пример.
Как интегрировать FastAPI с асинхронными библиотеками, такими как aiohttp или aiomysql?
Как обрабатывать конкурентные запросы в FastAPI? Какие проблемы могут возникнуть?
Как реализовать фоновые задачи (Background Tasks) в FastAPI? Напишите пример.
Зависимости (Dependencies)
Что такое система зависимостей в FastAPI? Как создать и использовать зависимость?
Как реализовать авторизацию через зависимости? Приведите пример проверки токена.
Как использовать зависимости для подключения к базе данных в каждом запросе?
Как кэшировать результат зависимости для повторного использования в запросе?
Как обрабатывать ошибки, возникающие в зависимостях?
Работа с базами данных
Как интегрировать FastAPI с SQLAlchemy для работы с реляционными базами данных? Приведите пример.
Как настроить асинхронное подключение к базе данных (например, с использованием databases или SQLAlchemy 2.0)?
Как использовать ORM (например, SQLAlchemy или Tortoise ORM) с FastAPI? Напишите пример CRUD-операций.
Как реализовать миграции базы данных в FastAPI-проекте (например, с Alembic)?
Как оптимизировать запросы к базе данных в FastAPI для повышения производительности?
Аутентификация и авторизация
Как реализовать аутентификацию по JWT в FastAPI? Напишите пример.
Как настроить OAuth2 с использованием FastAPI? Какие схемы поддерживает фреймворк?
Как защитить эндпоинты от несанкционированного доступа? Приведите пример роли-based авторизации.
Как интегрировать FastAPI с внешними провайдерами авторизации, такими как Auth0?
Как безопасно хранить и обрабатывать пароли в FastAPI-приложении?
Тестирование
Как тестировать FastAPI-приложение с использованием pytest? Напишите пример теста для эндпоинта.
Как использовать TestClient из fastapi.testclient для тестирования API?
Как мокать зависимости или внешние API при тестировании FastAPI-приложения?
Как тестировать асинхронные эндпоинты и фоновые задачи?
Как проверить корректность валидации Pydantic-моделей в тестах?
Производительность и масштабирование
Как оптимизировать производительность FastAPI-приложения? Какие инструменты или подходы вы используете?
Как настроить кэширование в FastAPI (например, с использованием Redis)?
Как масштабировать FastAPI-приложение для обработки большого количества запросов?
Как настроить FastAPI для работы с WebSocket? Напишите пример эндпоинта WebSocket.
Как использовать FastAPI с асинхронными очередями задач (например, Celery)?
Безопасность
Как защитить FastAPI-приложение от распространённых уязвимостей (например, SQL-инъекций, XSS, CSRF)?
Как настроить CORS в FastAPI? Когда это необходимо?
Как ограничить частоту запросов (rate limiting) в FastAPI? Какие библиотеки или подходы вы используете?
Как настроить HTTPS для FastAPI-приложения?
Как безопасно обрабатывать конфиденциальные данные, такие как API-ключи или токены?
Деплой и инфраструктура
Как настроить деплой FastAPI-приложения с использованием Gunicorn и Uvicorn?
Как контейнеризировать FastAPI-приложение с помощью Docker? Напишите пример Dockerfile.
Как настроить мониторинг и логирование в FastAPI-приложении (например, с использованием logging или Prometheus)?
Как интегрировать FastAPI с CI/CD для автоматического деплоя (например, GitHub Actions)?
Как настроить FastAPI для работы в серверлесс-архитектуре (например, AWS Lambda)?
Практические задачи
Напишите FastAPI-приложение с эндпоинтом, который принимает JSON с данными пользователя и сохраняет их в базу данных.
Реализуйте эндпоинт для загрузки CSV-файла и обработки его данных с возвратом результата в JSON.
Создайте API с пагинацией, фильтрацией и сортировкой для списка записей из базы данных.
Реализуйте систему авторизации с использованием JWT и ролями (например, admin и user).
Напишите WebSocket-эндпоинт, который отправляет обновления в реальном времени о статусе задачи.
````