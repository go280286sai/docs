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