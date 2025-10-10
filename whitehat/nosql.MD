### NoSQL-инъекция — это уязвимость, позволяющая атакующему вмешиваться в запросы к NoSQL-базе данных, обходить аутентификацию, извлекать данные и даже выполнять код.

#### Типы NoSQL-инъекций
- Синтаксическая инъекция — нарушение структуры запроса, вставка произвольного кода.
- Инъекция операторов — внедрение специальных операторов ($ne, $in, $regex, $where) для изменения логики запроса.

#### Типичные места для атак:
- Формы входа (логин/пароль)
- Формы поиска и фильтры
- API-эндпоинты, принимающие параметры запроса (REST, GraphQL)
- Любые входные данные, которые напрямую используются в запросе к БД.

Использование fuzz-строк и специальных символов для выявления уязвимостей.

Проверка поведения приложения при ложных и истинных условиях — например, '||'1'=='1' для обхода фильтрации.

Null-символы могут обрезать запрос, игнорируя последующие условия (например, released == 1).

#### Основные цели — это запросы, которые используют операторы запросов MongoDB, такие как $ne, $eq, $gt, $where, $regex и, что самое главное, $ne (not equal) и $gt (greater than) для обхода аутентификации.

Методы тестирования (с примерами)

Представьте, что у вас есть форма входа, которая отправляет POST-запрос с JSON-телом:
````json
{
  "username": "user123",
  "password": "pass123"
}
````
Стандартный запрос в коде на Node.js (уязвимый):
````javascript
const user = await db.collection('users').findOne({
  username: req.body.username,
  password: req.body.password
});
if (user) {
  // Успешный вход
}
````

Обход аутентификации

Внедрение операторов $ne и $in позволяет пройти проверку логина/пароля:
````json
{"username":{"$ne":"invalid"}, "password":{"$ne":"invalid"}}
````
Можно нацелиться на конкретные аккаунты, используя $in с предполагаемыми именами.

Извлечение данных

Использование $where и JavaScript-функций (match(), sleep()) для поэтапного извлечения паролей или других полей.

Методика: проверка символов по одному, например this.password[0] == 'a'.

Если у вас есть запрос вроде `db.users.find({username: input})`, попробуйте отправить `{"username": {"$ne": null}}`. 

Атаки, которые нужно попробовать:

1.	Обход аутентификации с помощью $ne
Вместо пароля передайте оператор запроса.
Тело POST-запроса:
````json
{
  "username": "admin",
  "password": {"$ne": ""}
}
````
Что это делает? Запрос к БД превратится в: `findOne({ username: "admin", password: { $ne: "" } })`. Это означает: "найди пользователя с логином admin и паролем, который НЕ РАВЕН пустой строке". Если пользователь admin существует, этот запрос почти наверняка вернет его запись, и вы войдете в систему без пароля.
2.	Обход аутентификации с помощью $regex (или логического OR)
Тело POST-запроса:
````json
{
  "username": {"$regex": ".*"},
  "password": {"$regex": ".*"}
}
````
Что это делает? Запрос найдет первого пользователя, у которого есть любое значение в username и любое значение в password.
3.	Использование логического OR ($or)
Тело POST-запроса:
````json
{
  "$or": [
    {"username": "admin"},
    {"username": "user"}
  ],
  "password": {"$ne": ""}
}
````
Что это делает? Найдет пользователя с логином admin ИЛИ user, у которого пароль не пустой.
4.	Атаки через URL-параметры (GET-запросы)
Если у вас есть поиск, например: `/api/users?search=John` 

Попробуйте: `/api/users?search[$regex]=a.*` - это вернет всех пользователей, у которых в имени есть буква 'a'.

Попробуйте: `/api/users?role[$ne]=user` - это может вернуть всех пользователей, у которых роль не user (например, администраторов).

#### Инструменты:
Burp Suite: Самый мощный инструмент.
- Перехватите запрос к вашему приложению.
- Отправьте его в инструмент Repeater.
- Измените Content-Type на application/json и попробуйте все примеры выше, меняя тело запроса.

NoSQLmap: Специализированный инструмент, похожий на sqlmap, но для NoSQL баз данных.

#### Убедитесь, что база данных не доступна публично:

Проверьте, открыт ли порт MongoDB (по умолчанию 27017) с помощью инструмента Nmap:
````bash
nmap -p 27017 <your-server-ip>
````
- Если порт открыт для внешнего доступа, это серьёзная уязвимость.
- Проверьте, включена ли аутентификация:
````bash
mongo <your-server-ip>:27017
````
- Если подключение удалось, база не защищена.
- Используйте Metasploit или Shodan для поиска открытых экземпляров MongoDB.

#### Тестирование API
- Если ваше приложение использует REST API для взаимодействия с MongoDB, проверьте:
- Возможность отправки некорректных JSON-запросов.
- Наличие уязвимостей, таких как IDOR (Insecure Direct Object References), позволяющих получить доступ к данным других пользователей.
- Используйте OWASP ZAP или Burp Suite для анализа API. 
#### Проверка на утечку данных
- Попробуйте получить доступ к конфиденциальным данным через некорректные запросы.
- Проверьте, возвращает ли сервер избыточные данные (например, полные документы вместо фильтрованных полей).
- Используйте инструменты, такие как MongoDB Compass, для анализа структуры базы и поиска слабых мест.
#### Автоматизированные инструменты
- NoSQLMap: Автоматизированный инструмент для тестирования NoSQL-инъекций.
- SQLMap: Хотя он ориентирован на SQL, некоторые функции работают с NoSQL.
- OWASP ZAP или Burp Suite: Для анализа веб-приложения и API.
- Metasploit: Для проверки конфигурации сервера.

### Как противодействовать NoSQL-инъекциям
#### Защита строится на двух принципах: Валидация и Санация (очистка) входных данных.
1. Используйте строгую валидацию всех входных данных
Никогда не доверяйте данным от клиента. Все, что приходит в req.body, req.query или req.params, должно проверяться.

Для Node.js используйте библиотеки валидации:
- Joi: Очень мощная и популярная.
- Yup: Часто используется с React, но отлично работает и в Node.js.
- Express-validator: Мидлваря для Express.js.
Пример с Joi:
````
const Joi = require('joi');

const loginSchema = Joi.object({
  username: Joi.string().alphanum().min(3).max(30).required(),
  password: Joi.string().pattern(new RegExp('^[a-zA-Z0-9]{3,30}$')).required()
  // Обратите внимание: здесь НЕТ полей типа object, $ne, $regex и т.д.
});

app.post('/login', (req, res) => {
  const { error, value } = loginSchema.validate(req.body);
  if (error) {
    return res.status(400).send('Invalid request');
  }

  // Теперь `value.username` и `value.password` гарантированно строки.
  const user = await db.collection('users').findOne({
    username: value.username,
    password: value.password // В реальном приложении пароль должен быть захэширован!
  });
  // ... остальной код
});
````
Что это дает? Если злоумышленник отправит {"$ne": ""}, Joi выдаст ошибку валидации, так как ожидает строку, а не объект.
2. Явное преобразование типов
Если вы ожидаете число, явно преобразуйте строку в число.
````javascript
const productId = parseInt(req.params.id, 10);
// Или с Joi: Joi.number().integer()
````
3. Используйте ORM/ODM с встроенной защитой
Библиотеки, такие как Mongoose для MongoDB, по умолчанию защищают от многих видов инъекций.
- Mongoose использует схему, которая определяет типы полей.
- Запросы в Mongoose обрабатываются так, что вложенные объекты рассматриваются как условия поиска, только если ключ является допустимым оператором для пути в схеме.

Пример с Mongoose:
````javascript
const userSchema = new mongoose.Schema({
  username: String,
  password: String
});

const User = mongoose.model('User', userSchema);

// Эта попытка атаки НЕ СРАБОТАЕТ с Mongongoose по умолчанию.
// Mongoose преобразует `password` в строку, и запрос будет искать
// пользователя с паролем, равным строке "{$ne: \"\"}", а не использующим оператор $ne.
User.findOne({ username: req.body.username, password: req.body.password });
````
Важно: Mongoose не является панацеей. Все равно нужно валидировать входные данные, чтобы соответствовать бизнес-логике (например, проверять длину строки, формат email).
4. Принцип наименьших привилегий для базы данных
Создайте для своего приложения пользователя БД с минимально необходимыми правами. Обычно приложению не нужны права на выполнение произвольных команд или администрирование БД.
5. Регулярное тестирование безопасности

Внедрите практику регулярного пентеста (тестирования на проникновение), включая проверки на NoSQL-инъекции, как часть процесса разработки.

Чтобы защитить ваш сайт и MongoDB от взлома, внедрите следующие меры:
1. Защита от NoSQL-инъекций
•	Санитизация и валидация ввода:
•	Проверяйте и фильтруйте пользовательский ввод на стороне сервера.
•	Используйте библиотеки, такие как express-validator (Node.js) или аналогичные, для валидации данных.
•	Избегайте использования пользовательского ввода напрямую в запросах к MongoDB.
•	Используйте ORM/ODM:
•	Применяйте библиотеки, такие как Mongoose (для Node.js), которые автоматически экранируют опасные операторы MongoDB (например, $where, $ne).
•	Пример безопасного запроса в Mongoose:
````javascript
const user = await User.findOne({ username: sanitizedInput });
````
•	Ограничьте использование опасных операторов:
•	Избегайте использования $where, $regex или других операторов, если они не нужны.
•	Если требуется $where, убедитесь, что входные данные строго валидируются.
2. Безопасная конфигурация MongoDB
•	Включите аутентификацию
•	Настройте пользователей с ролями в MongoDB
````javascript
//use admin
db.createUser({
  user: "admin",
  pwd: "strongpassword",
  roles: [{ role: "root", db: "admin" }]
})
````
- Убедитесь, что параметр auth=true установлен в конфигурационном файле MongoDB (mongod.conf).
- Ограничьте сетевой доступ:
- Настройте брандмауэр (например, iptables или облачный фаервол) для ограничения доступа к порту 27017 только с доверенных IP-адресов.
Используйте параметр bind_ip в конфигурации MongoDB:
````yaml
net:
  bindIp: 127.0.0.1, <your-app-server-ip>
  ````
- Шифрование соединений:
- Включите TLS/SSL для шифрования данных в транзите:
````yaml
net:
  ssl:
    mode: requireSSL
    PEMKeyFile: /path/to/your/certificate.pem
````
- Используйте защищённые протоколы для подключения (например, mongodb:// замените на mongodb+srv://).
3. Управление доступом
•	Принцип наименьших привилегий:
•	Создавайте роли в MongoDB с минимальными правами, необходимыми для работы приложения.

Пример:
````javascript
db.createRole({
  role: "appUser",
  privileges: [
    { resource: { db: "mydb", collection: "users" }, actions: ["find", "update"] }
  ],
  roles: []
})
````
•	Ограничьте доступ к коллекциям:
•	Используйте фильтрацию на уровне приложения, чтобы возвращать только необходимые поля:
````javascript
db.users.find({}, { password: 0, sensitiveField: 0 });
````
4. Мониторинг и логирование
- Включите журналирование в MongoDB для отслеживания подозрительных запросов:
````yaml
systemLog:
  destination: file
  path: /var/log/mongodb/mongod.log
  logAppend: true
````
- Используйте инструменты мониторинга, такие как MongoDB Atlas или Prometheus, для отслеживания активности.
5. Регулярные обновления
- Обновляйте MongoDB до последней стабильной версии, чтобы устранить известные уязвимости.
- Следите за бюллетенями безопасности MongoDB: [https://www.mongodb.com/security]().
6. Защита API
- Используйте JSON Schema Validation в MongoDB для проверки структуры входящих данных.
- Настройте CORS и ограничьте доступ к API с помощью заголовков.
- Применяйте аутентификацию (например, JWT) и авторизацию для API.
7. Резервное копирование
- Регулярно создавайте резервные копии базы данных:
````bash
mongodump --uri="mongodb://<user>:<password>@<host>:<port>/<db>" --out=/backup
````
- Храните бекап в безопасном месте, желательно зашифрованными.

Дополнительные рекомендации
- Пентест: Периодически проводите тестирование на проникновение с помощью профессиональных инструментов или услуг.
- Обучение команды: Убедитесь, что разработчики понимают риски NoSQL-инъекций и безопасные методы работы с MongoDB.
- Используйте WAF: Настройте веб-файрвол (например, Cloudflare или AWS WAF) для фильтрации вредоносных запросов.

#### Противодействие NoSQL-инъекциям
1. Валидация и санитизация данных
````javascript
// Плохо - уязвимый код
const user = await User.findOne({
  username: req.body.username,
  password: req.body.password
});

// Хорошо - с валидацией
const username = String(req.body.username);
const password = String(req.body.password);

// Убедитесь, что это строки, а не объекты
if (typeof req.body.username !== 'string' || 
    typeof req.body.password !== 'string') {
  return res.status(400).send('Invalid input');
}

const user = await User.findOne({ username, password });
````
2. Используйте схемы и ORM/ODM
````javascript
// Mongoose автоматически проверяет типы
const userSchema = new mongoose.Schema({
  username: { type: String, required: true },
  password: { type: String, required: true },
  email: { type: String, required: true }
});

const User = mongoose.model('User', userSchema);
````
3. Отключите $where и mapReduce
В конфигурации MongoDB:
````
// В настройках безопасности
mongod --noscripting
````
4. Белый список разрешенных полей
````javascript
const allowedFields = ['username', 'email', 'age'];
const query = {};

Object.keys(req.body).forEach(key => {
  if (allowedFields.includes(key)) {
    query[key] = String(req.body[key]);
  }
});
````
5. Используйте библиотеки для санитизации
````javascript
const mongoSanitize = require('express-mongo-sanitize');

app.use(mongoSanitize({
  replaceWith: '_',
  onSanitize: ({ req, key }) => {
    console.warn(`Sanitized key: ${key}`);
  }
}));
````
6. Минимум привилегий для БД
Создайте пользователя MongoDB с минимальными правами:
````javascript
db.createUser({
  user: "appUser",
  pwd: "securePassword",
  roles: [
    { role: "readWrite", db: "myDatabase" }
  ]
});
````
7. Дополнительные меры безопасности
- Хешируйте пароли (bcrypt, argon2)
- Используйте rate limiting
- Логируйте подозрительные запросы
- Регулярно обновляйте зависимости
- Используйте Content-Type validation

Пример комплексной защиты:
````javascript
const express = require('express');
const mongoSanitize = require('express-mongo-sanitize');
const validator = require('validator');

app.use(express.json());
app.use(mongoSanitize());

app.post('/login', async (req, res) => {
  // Проверка типов
  if (typeof req.body.username !== 'string' || 
      typeof req.body.password !== 'string') {
    return res.status(400).json({ error: 'Invalid input type' });
  }

  // Валидация формата
  const username = validator.escape(req.body.username.trim());
  const password = req.body.password;

  // Ограничение длины
  if (username.length > 50 || password.length > 100) {
    return res.status(400).json({ error: 'Input too long' });
  }

  try {
    const user = await User.findOne({ username });
    if (!user || !(await bcrypt.compare(password, user.password))) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    // Успешная авторизация
  } catch (error) {
    console.error('Login error:', error);
    res.status(500).json({ error: 'Server error' });
  }
});
````
