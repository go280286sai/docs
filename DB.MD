### SQLite
 
``` 
    import sqlite3
    conn = sqlite3.connect("data.db)  # Открывает существующий или новый файл. Также ':memory:'.
    conn.close()                      # Закрывает соединение. Отбрасывает неподтвержденные данные.
```

#### Read
``` 
    <cursor> = <conn>.execute('<query>') # Может вызвать подкласс sqlite3.Error.
    <tuple> = <cursor>.fetchone()        # Возвращает следующую строку. Также next(<cursor>).
    <list> = <cursor>.fetchall()         # Возвращает оставшиеся строки. Также list(<cursor>).
```

#### Write
``` 
    <conn>.execute('<query>')         # Может вызвать подкласс sqlite3.Error.
    <conn>.commit()                   # Сохраняет все изменения с момента последнего коммита.
    <conn>.rollback()                 # Отменяет все изменения с момента последнего коммита.
```

#### Or:
``` 
    with <conn>:                                   # Выходит из блока с помощью commit() или rollback(),
        <conn>.execute('<query>')                  # в зависимости от того, произошло ли какое-либо исключение.
```

### Placeholders
``` 
    <conn>.execute('<query>', <list/tuple>)        # Заменяет каждый вопросительный знак на элемент.
    <conn>.execute('<query>', <dict/namedtuple>)   # Заменяет каждый :<key> на значение.
    <conn>.executemany('<query>', <coll_of_coll>)  # Запускает execute() несколько раз.
```

#### Example
**Values are not actually saved in this example because `'conn.commit()'` is omitted!**
``` 
    >>> conn = sqlite3.connect('test.db')
    >>> conn.execute('CREATE TABLE IF NOT EXISTS person (person_id INTEGER PRIMARY KEY, name, height)')
    >>> conn.execute('INSERT INTO person VALUES (NULL, ?, ?)', ('Jean-Luc', 187)).lastrowid
    1
    >>> conn.execute('SELECT * FROM person').fetchall()
    [(1, 'Jean-Luc', 187)]
```
### Using aiosqlite (Recommended for Async SQLite)
````
    pip install aiosqlite
````
````
    import aiosqlite
    import asyncio
    
    async def async_sqlite_example():
        async with aiosqlite.connect("example.db") as db:
            await db.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)")
            await db.execute("INSERT INTO users (name) VALUES (?)", ("Alice",))
            await db.commit()
    
            async with db.execute("SELECT * FROM users") as cursor:
                async for row in cursor:
                    print(row)
    
    asyncio.run(async_sqlite_example())
````
### Using sqlite3 with concurrent.futures
````
    import sqlite3
    import asyncio
    
    def sync_db_operations():
        conn = sqlite3.connect("example.db")
        cursor = conn.cursor()
        cursor.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)")
        cursor.execute("INSERT INTO users (name) VALUES (?)", ("Bob",))
        conn.commit()
        cursor.execute("SELECT * FROM users")
        rows = cursor.fetchall()
        conn.close()
        return rows
    
    async def async_sqlite_example():
        rows = await asyncio.to_thread(sync_db_operations)
        for row in rows:
            print(row)
    
    asyncio.run(async_sqlite_example())
````
### Using Databases Library
````
    from databases import Database
    import asyncio
    
    async def async_sqlite_example():
        database = Database("sqlite:///example.db")
        await database.connect()
    
        await database.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)")
        await database.execute("INSERT INTO users (name) VALUES (:name)", {"name": "Charlie"})
    
        rows = await database.fetch_all("SELECT * FROM users")
        for row in rows:
            print(row)
    
        await database.disconnect()
    
    asyncio.run(async_sqlite_example())
````
### SQLAlchemy
**Library for interacting with various DB systems via SQL, method chaining, or ORM.**
```
    pip3 install sqlalchemy
```
``` 
    from sqlalchemy import create_engine, Column, Integer, String
    from sqlalchemy.orm import declarative_base, sessionmaker
    
    # Создание подключения к SQLite
    engine = create_engine("sqlite:///example.db", echo=True)
    
    # Определение базы данных
    Base = declarative_base()
    
    class User(Base):
        __tablename__ = "users"
    
        id = Column(Integer, primary_key=True)
        name = Column(String)
        age = Column(Integer)
    
    # Создание таблицы
    Base.metadata.create_all(engine)
    
    # Создание сессии
    Session = sessionmaker(bind=engine)
    session = Session()
    
    # Добавление данных
    new_user = User(name="Alex", age=30)
    session.add(new_user)
    session.commit()
    
    # Запрос данных
    users = session.query(User).all()
    for user in users:
        print(user.name, user.age)
                 
```
#### Асинхронное использование
```
    import asyncio
    from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
    from sqlalchemy.orm import sessionmaker
    from sqlalchemy import text
    
    # Создание асинхронного подключения к SQLite
    engine = create_async_engine("sqlite+aiosqlite:///example.db", echo=True)
    AsyncSessionLocal = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)
    
    async def async_main():
        async with AsyncSessionLocal() as session:
            # Здесь можно выполнять асинхронные запросы
            result = await session.execute(text("SELECT * from users"))
            for item in result.all():
                print(item)
    
    asyncio.run(async_main())
```

### Redis
```
    pip3 install redis
```
```
    import redis
    
    # Создание подключения к Redis
    r = redis.Redis(host='localhost', port=6379, db=0)
    
    # Проверка соединения
    print(r.ping())  # Должно вывести True
    
    # Установка значения
    r.set('my_key', 'Hello, Redis!')
    
    # Получение значения
    value = r.get('my_key')
    print(value.decode())  # Декодируем байты в строку
```
#### Асинхронное использование
````
    import asyncio
    from redis.asyncio import Redis
    
    async def main():
        try:
            conn = Redis.from_url("redis://localhost")
            await conn.set("key", "Value 12")
            res = await conn.get("key")
            print(res)
            await conn.aclose()  # Use aclose() for async Redis
        except Exception as e:
            print(e)
        finally:
            await conn.aclose()  # Ensure connection is closed
    
    if __name__ == "__main__":
        asyncio.run(main())
````
### Celery
```
    pip3 install celery
```
```
from celery import Celery 
from celery.exceptions import TimeoutError as CeleryTimeoutError # Инициализация приложения 

Celery app = Celery( 
    "main", 
    broker="redis://localhost:6379", 
    backend="redis://localhost:6379" ) # Настройка дополнительных параметров 

app.conf.update( 
    task_track_started=True, 
    task_time_limit=30, 
    task_soft_time_limit=30 ) # Определение синхронной задачи 

@app.task(name="do-something") 
def foo(x): 
    import time 
    time.sleep(5) # Используем синхронный sleep 
    return f"Задача {x} выполнена" 
    
def main(i: int): 
    try: 
        # Отправка задачи 
        result = foo.delay(i) # Правильный вызов delay 
        try: 
            # Ожидание результата с таймаутом 
            value = result.get(timeout=10) 
            print("Результат:", value) 
        except CeleryTimeoutError: 
            print("Результат не готов после 10 секунд") 
        except Exception as e: 
            print(f"Произошла ошибка: {e}") 
        except Exception as e: 
            print(f"Ошибка при отправке задачи: {e}") 
if __name__ == "__main__": 
    for i in range(10): 
        main(i)
````
````
    celery -A main worker --loglevel=info
````
### PostgresQL
```
    pip3 install psycopg['binary', 'pool']
```
```
    import psycopg
    
    # Подключение к базе данных
    conn = psycopg.connect("dbname=postgres user=postgres host=localhost password=masterkey")
    
    # Создание курсора
    cur = conn.cursor()
    
    # Выполнение запроса
    cur.execute("SELECT * FROM user")
    
    # Получение результатов
    rows = cur.fetchall()
    for row in rows:
        print(row)
    
    # Закрытие соединения
    cur.close()
    conn.close()
```
#### Асинхронное использование
````
    import asyncio
    import psycopg
    
    async def main():
        async with await psycopg.AsyncConnection.connect("dbname=postgres user=postgres host=localhost password=masterkey") as conn:
            async with conn.cursor() as cur:
                await cur.execute("SELECT * FROM user")
                rows = await cur.fetchall()
                for row in rows:
                    print(row)
    
    asyncio.run(main())
````
### MongoDB
```
    pip3 install pymongo
```
```
    import pymongo
    
    # Подключение к базе данных
    client = pymongo.MongoClient("mongodb://root:masterkey@localhost:27017")
    db = client["mydatabase"]
    collection = db["mycollection"]
    
    # Добавление документа
    document = {"name": "John", "age": 30}
    collection.insert_one(document)
    
    # Обновление документа
    collection.update_one({"name": "John"}, {"$set": {"age": 31}})
    
    # Удаление документа
    collection.delete_one({"name": "John"})
```
#### Асинхронное использование
````
    import asyncio
    from motor.motor_asyncio import AsyncIOMotorClient
    
    async def main():
        async with AsyncIOMotorClient("mongodb://root:masterkey@localhost:27017") as client:
            db = client["mydatabase"]
            collection = db["mycollection"]
            document = {"name": "John", "age": 30}
            await collection.insert_one(document)
            await collection.update_one({"name": "John"}, {"$set": {"age": 31}})
            await collection.delete_one({"name": "John"})
    
    asyncio.run(main())
````

### Mysql
```
    pip3 install mysql-connector-python
```
```
    import mysql.connector
    
    # Подключение к базе данных
    conn = mysql.connector.connect(
        host="localhost",
        port=3282,
        user="root",
        password="masterkey",
        database="learn_mysql"
    )
    
    # Создание курсора
    cur = conn.cursor()
    
    # Выполнение запроса
    cur.execute("SELECT * FROM users")
    
    # Получение результатов
    rows = cur.fetchall()
    for row in rows:
        print(row)
    
    # Закрытие соединения
    cur.close()
    conn.close()
```
#### Асинхронное использование
````
    import asyncio
    import mysql.connector
    
    async def main():
        conn = await mysql.connector.connect(
            host="localhost",
            user="root",
            port=3282,
            password="masterkey",
            database="learn_mysql"
        )
        async with conn.cursor() as cur:
            await cur.execute("SELECT * FROM users")
            rows = await cur.fetchall()
            for row in rows:
                print(row)
        await conn.close()
    
    asyncio.run(main())
````



