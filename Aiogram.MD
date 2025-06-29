### Aiogram
````
    pip install aiogram
````
````
    import asyncio
    from aiogram import Bot, Dispatcher, types
    from aiogram.filters import CommandStart
    from aiogram.types import InlineKeyboardMarkup, InlineKeyboardButton
    from aiogram.utils.keyboard import InlineKeyboardBuilder # Удобный инструмент для построения клавиатур
    
    TOKEN = "YOUR_BOT_TOKEN"
    
    bot = Bot(token=TOKEN)
    dp = Dispatcher()
    
    @dp.message(CommandStart())
    async def cmd_start(message: types.Message):
        # Создаем билдер для inline-клавиатуры
        builder = InlineKeyboardBuilder()
        builder.add(InlineKeyboardButton(text="Красный", callback_data="color_red"))
        builder.add(InlineKeyboardButton(text="Синий", callback_data="color_blue"))
        builder.add(InlineKeyboardButton(text="Зеленый", callback_data="color_green"))
    
        await message.answer(
            "Выберите ваш любимый цвет:",
            reply_markup=builder.as_markup() # Отправляем клавиатуру
        )
    
    # Обработчик для коллбэк-запросов от inline-кнопок
    @dp.callback_query()
    async def process_callback_button(callback_query: types.CallbackQuery):
        selected_color = callback_query.data.replace("color_", "")
        await callback_query.message.answer(f"Вы выбрали: {selected_color.capitalize()} цвет.")
        await callback_query.answer(text=f"Выбран: {selected_color.capitalize()}", show_alert=False) # Оповещение пользователю
    
    async def main():
        await dp.start_polling(bot)
    
    if __name__ == "__main__":
        asyncio.run(main())
````

### Работа с состояниями (FSM - Finite State Machine)
````
import asyncio
from aiogram import Bot, Dispatcher, types, F
from aiogram.filters import CommandStart
from aiogram.fsm.context import FSMContext
from aiogram.fsm.state import State, StatesGroup

TOKEN = "YOUR_BOT_TOKEN"

bot = Bot(token=TOKEN)
dp = Dispatcher()

# Определяем состояния для нашей FSM
class Form(StatesGroup):
    name = State()  # Состояние ожидания имени
    age = State()   # Состояние ожидания возраста

@dp.message(CommandStart())
async def cmd_start(message: types.Message, state: FSMContext):
    await state.set_state(Form.name) # Устанавливаем текущее состояние в Form.name
    await message.answer("Привет! Как тебя зовут?")

@dp.message(Form.name) # Обработчик для состояния Form.name
async def process_name(message: types.Message, state: FSMContext):
    await state.update_data(name=message.text) # Сохраняем имя в контекст состояния
    await state.set_state(Form.age) # Переходим в состояние Form.age
    await message.answer("Сколько тебе лет?")

@dp.message(Form.age, F.text.as_int()) # Обработчик для состояния Form.age, ожидаем число
async def process_age(message: types.Message, state: FSMContext):
    user_data = await state.get_data() # Получаем все данные из контекста
    user_name = user_data.get("name")
    user_age = message.text # Возраст уже проверен фильтром F.text.as_int()

    await message.answer(f"Приятно познакомиться, {user_name}! Тебе {user_age} лет.")
    await state.clear() # Очищаем состояние после завершения диалога

@dp.message(Form.age) # Если в Form.age пришло не число
async def process_age_invalid(message: types.Message):
    await message.answer("Пожалуйста, введите возраст числом.")

async def main():
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
````
### Создать меню команд
````
from aiogram import Bot, Dispatcher, types
from aiogram.filters import CommandStart, Command
import asyncio

bot = Bot("YOUR_BOT_TOKEN")
dp = Dispatcher()

async def set_commands(bot: Bot):
    commands = [
        types.BotCommand(command="start", description="Запустить бота"),
        types.BotCommand(command="help", description="Получить помощь"),
        types.BotCommand(command="menu", description="Показать меню")
    ]
    await bot.set_my_commands(commands)

@dp.message(CommandStart())
async def start(message: types.Message):
    await message.answer("Привет! Используйте /menu для открытия меню.")

@dp.message(Command("menu"))
async def menu(message: types.Message):
    keyboard = types.ReplyKeyboardMarkup(resize_keyboard=True)
    buttons = ["Профиль", "Настройки", "Помощь"]
    keyboard.add(*buttons)
    await message.answer("Выберите действие:", reply_markup=keyboard)

async def main():
    await set_commands(bot)  # Устанавливаем команды при запуске
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
````

#### MediaGroup
````
from aiogram.utils.media_group import MediaGroupBuilder
from aiogram.types import Message, FSInputFile

@dp.message(CommandStart())
async def echo(message: Message, state: FSMContext):
    media = MediaGroupBuilder()
    media.add_document(FSInputFile("read.txt"))
    media.add_document(FSInputFile("read2.txt"))
    await message.answer_media_group(media=media.build())
````

#### Download photo
````
@dp.message(F.photo)
async def download_photo(message: Message, bot: Bot):
    await bot.download(message.photo[-1], destination=f"{message.photo[-1].file_id}.jpg")
````
### Excel
````
    pip install openpyxl
````
#### Создание нового файла .xlsx и запись данных
````
from openpyxl import Workbook

# Создаем новую рабочую книгу (Workbook)
wb = Workbook()

# Получаем активный лист (по умолчанию это первый лист)
ws = wb.active

# Устанавливаем заголовок листа
ws.title = "МойПервыйЛист"

# Запись данных в ячейки
ws['A1'] = "Привет"
ws['B1'] = "Мир!"
ws['A2'] = 123
ws['B2'] = 45.67

# Добавление строк данных (сразу несколько ячеек в строке)
ws.append(["Ячейка 3A", "Ячейка 3B", "Ячейка 3C"])
ws.append([1, 2, 3, 4, 5]) # Можно добавлять больше столбцов, чем было в предыдущей строке

# Доступ к ячейкам по номеру строки и столбца (1-индексирование)
ws.cell(row=5, column=1, value="Пятая строка")

# Сохраняем книгу
wb.save("example_openpyxl.xlsx")
print("Файл 'example_openpyxl.xlsx' создан успешно.")
````
#### Чтение данных из файла .xlsx
````
from openpyxl import load_workbook

try:
    # Загружаем рабочую книгу
    wb = load_workbook("example_openpyxl.xlsx")

    # Получаем активный лист или лист по имени
    # ws = wb.active
    ws = wb["МойПервыйЛист"]

    print(f"Имя активного листа: {ws.title}")

    # Чтение данных из отдельных ячеек
    print(f"Значение A1: {ws['A1'].value}")
    print(f"Значение B1: {ws['B1'].value}")

    # Итерация по строкам и столбцам
    print("\nДанные по строкам:")
    for row in ws.iter_rows(min_row=1, max_row=ws.max_row, min_col=1, max_col=ws.max_column):
        row_values = [cell.value for cell in row]
        print(row_values)

    print("\nДанные по столбцам:")
    for col in ws.iter_cols(min_row=1, max_row=ws.max_row, min_col=1, max_col=ws.max_column):
        col_values = [cell.value for cell in col]
        print(col_values)

    # Доступ к определенным диапазонам
    print("\nДанные из диапазона A1:B2:")
    for row in ws['A1':'B2']:
        for cell in row:
            print(cell.value, end="\t")
        print()

except FileNotFoundError:
    print("Файл 'example_openpyxl.xlsx' не найден. Сначала запустите пример создания файла.")
````
#### Изменение существующего файла .xlsx
````
from openpyxl import load_workbook

try:
    wb = load_workbook("example_openpyxl.xlsx")
    ws = wb["МойПервыйЛист"]

    # Изменяем значение ячейки
    ws['C1'] = "Новое значение"
    ws.cell(row=2, column=3, value="Еще одно новое")

    # Добавляем новый лист
    new_sheet = wb.create_sheet("НовыйЛист")
    new_sheet['A1'] = "Это новый лист"

    # Удаляем лист по имени
    # wb.remove(wb["МойПервыйЛист"]) # Осторожно, это удалит лист!

    # Сохраняем изменения
    wb.save("example_openpyxl_modified.xlsx")
    print("Файл 'example_openpyxl.xlsx' изменен и сохранен как 'example_openpyxl_modified.xlsx'.")

except FileNotFoundError:
    print("Файл 'example_openpyxl.xlsx' не найден. Сначала запустите пример создания файла.")
````
### Word
````
    pip install python-docx
````
#### Создание нового файла .docx
````
from docx import Document
from docx.shared import Inches

# Создаем новый документ
document = Document()

# Добавляем заголовок
document.add_heading('Мой первый документ на Python', level=1)

# Добавляем абзац
document.add_paragraph('Это обычный абзац текста. '
                       'Он содержит несколько предложений, '
                       'демонстрируя базовую возможность добавления текста.')

# Добавляем абзац с жирным текстом
p = document.add_paragraph('А вот абзац с ')
p.add_run('жирным текстом').bold = True
p.add_run(' и ')
p.add_run('курсивом').italic = True
p.add_run('.')

# Добавляем цитату
document.add_paragraph('Это блок цитаты:', style='Intense Quote')

# Добавляем список
document.add_heading('Список элементов:', level=2)
document.add_paragraph('Первый элемент списка', style='List Bullet')
document.add_paragraph('Второй элемент списка', style='List Bullet')
document.add_paragraph('Вложенный элемент списка', style='List Bullet 2')
document.add_paragraph('Третий элемент списка', style='List Number')

# Добавляем изображение (убедитесь, что у вас есть 'image.png' в той же директории)
# Если изображения нет, закомментируйте эту строку или используйте свой путь к изображению
try:
    document.add_picture('image.png', width=Inches(1.25))
    print("Изображение 'image.png' добавлено.")
except FileNotFoundError:
    print("Изображение 'image.png' не найдено. Пропустили добавление изображения.")


# Сохраняем документ
document.save('example_document.docx')
print("Документ 'example_document.docx' создан успешно.")
````
#### Изменение существующего файла .docx
````
from docx import Document
from docx.enum.text import WD_ALIGN_PARAGRAPH

try:
    document = Document('example_document.docx')

    # Добавляем новый абзац в конце
    document.add_paragraph('Это новый абзац, добавленный в существующий документ.')

    # Изменяем текст в первом абзаце (найдем его по части текста)
    for paragraph in document.paragraphs:
        if 'обычный абзац текста' in paragraph.text:
            paragraph.text = 'Этот абзац был изменен программой Python.'
            break # Прерываем цикл после нахождения и изменения первого совпадения

    # Выравнивание абзаца
    for paragraph in document.paragraphs:
        if 'Этот абзац был изменен' in paragraph.text:
            paragraph.alignment = WD_ALIGN_PARAGRAPH.CENTER
            break

    # Сохраняем изменения в новом файле, чтобы не перезаписывать оригинал
    document.save('example_document_modified.docx')
    print("Документ 'example_document.docx' изменен и сохранен как 'example_document_modified.docx'.")

except FileNotFoundError:
    print("Файл 'example_document.docx' не найден. Сначала запустите пример создания документа.")
except Exception as e:
    print(f"Произошла ошибка при изменении документа: {e}")
````
#### Чтение данных из существующего документа Word
````
from docx import Document

try:
    # Загружаем существующий документ
    document = Document('example_document.docx')

    print("=== Чтение документа ===")

    # Чтение заголовков и абзацев
    for paragraph in document.paragraphs:
        # Проверяем, является ли абзац заголовком
        if paragraph.style.name.startswith('Heading'):
            print(f"Заголовок ({paragraph.style.name}): {paragraph.text}")
        else:
            print(f"Абзац: {paragraph.text}")

    # Чтение данных из таблиц (если есть)
    print("\n=== Чтение таблиц ===")
    if document.tables:
        for i, table in enumerate(document.tables):
            print(f"\nТаблица {i+1}:")
            for row in table.rows:
                row_text = [cell.text for cell in row.cells]
                print("\t".join(row_text))
    else:
        print("В документе нет таблиц.")

except FileNotFoundError:
    print("Файл 'example_document.docx' не найден. Сначала запустите пример создания документа.")
except Exception as e:
    print(f"Произошла ошибка при чтении документа: {e}")
````
#### Работа с таблицами
````
from docx import Document
from docx.shared import Cm

# Создаем новый документ
document = Document()
document.add_heading('Таблица данных', level=1)

# Добавляем таблицу 3x3
# Параметры: rows, cols
table = document.add_table(rows=3, cols=3, style='Table Grid') # 'Table Grid' - стандартный стиль

# Заголовки таблицы
# cells[0] - первая строка, cells[0] - первая ячейка и т.д.
hdr_cells = table.rows[0].cells
hdr_cells[0].text = 'ID'
hdr_cells[1].text = 'Имя'
hdr_cells[2].text = 'Возраст'

# Добавление данных в строки
# Строка 1 (индекс 1, т.к. 0-я строка - заголовки)
row_cells = table.rows[1].cells
row_cells[0].text = '1'
row_cells[1].text = 'Алиса'
row_cells[2].text = '30'

# Строка 2 (индекс 2)
row_cells = table.rows[2].cells
row_cells[0].text = '2'
row_cells[1].text = 'Боб'
row_cells[2].text = '24'

# Добавление новой строки в конце таблицы
row_data = ('3', 'Чарли', '35')
row = table.add_row()
for i, text in enumerate(row_data):
    row.cells[i].text = text

# Изменение ширины столбцов (не всегда работает идеально, зависит от Word)
# секция = table.columns[0].width = Cm(3)
# table.columns[1].width = Cm(5)

document.save('table_document.docx')
print("Документ 'table_document.docx' с таблицей создан успешно.")
````
### Вопросы
````
Что такое aiogram и чем он отличается от других библиотек для создания Telegram-ботов, например, python-telegram-bot?
Объясните, как работает асинхронное программирование в aiogram. Почему важно использовать asyncio и aiohttp?
Как настроить базовый Telegram-бот с использованием aiogram? Напишите минимальный пример кода для обработки команды /start.
Что такое Dispatcher и Bot в aiogram? Как они взаимодействуют?
Как правильно настроить токен бота и запустить его с использованием polling? Какие есть альтернативы polling (например, webhooks)?
Обработка сообщений и событий
Как обрабатывать разные типы сообщений (текст, фото, аудио, документы) в aiogram? Приведите пример обработки фото.
Как использовать фильтры в aiogram (например, Command, Text, Regexp)? Напишите пример фильтра для обработки сообщений, содержащих определённое слово.
Как реализовать обработку callback-запросов от инлайн-кнопок? Приведите пример создания и обработки инлайн-клавиатуры.
Как получить информацию о пользователе, отправившем сообщение (например, ID, имя)?
Как отправить сообщение в определённый чат или пользователю по его ID?
Конечные автоматы (FSM)
Что такое Finite State Machine (FSM) в контексте aiogram? Как настроить состояния с использованием StatesGroup?
Напишите пример кода для реализации формы, которая запрашивает у пользователя имя и возраст с использованием FSM.
Как реализовать кнопку "Назад" для возврата к предыдущему состоянию в FSM?
Как сохранить данные в FSM и получить их позже? Приведите пример использования FSMContext.
Как обрабатывать отмену действия в FSM (например, команда /cancel)?
Клавиатуры
Чем отличаются ReplyKeyboardMarkup и InlineKeyboardMarkup? Когда использовать каждую из них?
Как создать динамическую инлайн-клавиатуру, количество кнопок в которой зависит от данных?
Как удалить клавиатуру после отправки сообщения? Приведите пример.
Как реализовать клавиатуру с пагинацией для отображения списка элементов?
Как отправить сообщение с клавиатурой, содержащей кнопку для перехода в веб-приложение Telegram?
Работа с файлами
Как скачать файл, отправленный пользователем (например, фото или документ), и сохранить его локально?
Как отправить группу медиафайлов (например, несколько фото) в одном сообщении?
Как обработать голосовое сообщение и передать его в API для транскрипции (например, OpenAI Whisper)?
Как отправить файл в формате BytesIO через aiogram в REST API?
Как проверить, что файл, отправленный пользователем, имеет допустимый формат или размер?
Вебхуки и масштабирование
Как настроить вебхуки в aiogram вместо polling? Какие преимущества и сложности возникают при использовании вебхуков?
Как масштабировать Telegram-бот, созданный на aiogram, для обработки большого количества пользователей?
Как избежать ошибки TerminatedByOtherGetUpdates при запуске нескольких экземпляров бота?
Как настроить aiogram для работы с несколькими ботами в одном проекте?
Как использовать Redis или другую базу данных для хранения состояний FSM в масштабируемом приложении?
Асинхронные задачи и интеграции
Как использовать aioschedule для отправки сообщений по расписанию в aiogram?
Как интегрировать aiogram с FastAPI для обработки вебхуков или других API-запросов?
Как запустить длительную задачу (например, парсинг данных) в фоновом режиме, не блокируя бота?
Как интегрировать aiogram с Django для создания бота, связанного с веб-приложением?
Как использовать aiohttp для отправки запросов к внешним API из aiogram? Приведите пример.
Безопасность и обработка ошибок
Как обрабатывать ошибки Telegram API (например, TelegramBadRequest)?
Как защитить бота от спама или злоупотреблений (например, ограничение частоты запросов)?
Как настроить логирование в aiogram для отладки и мониторинга?
Как безопасно хранить токен бота? Какие подходы вы используете для защиты конфиденциальных данных?
Как обработать ситуацию, когда бот не может подключиться к api.telegram.org?
Продвинутые возможности
Как реализовать диалоговую систему с использованием aiogram-dialog? Чем она отличается от стандартного FSM?
Как создать кастомный фильтр в aiogram для обработки специфических сообщений?
Как настроить middleware в aiogram для обработки всех входящих обновлений? Приведите пример.
Как реализовать отправку "typing" действия во время обработки сообщения?
Как получить количество участников в группе или проверить, переслал ли пользователь сообщение?
Тестирование и деплой
Как тестировать бота, созданного на aiogram? Какие инструменты или подходы вы используете?
Как настроить деплой aiogram-бота на сервере (например, с использованием Docker или PythonAnywhere)?
Как использовать pytest для тестирования обработчиков сообщений в aiogram?
Как минимизировать вывод логов от aiogram (например, сообщения вроде Updates were skipped successfully)?
Как настроить CI/CD для aiogram-бота, чтобы автоматизировать деплой и тестирование?
Практические задачи
Напишите код для бота, который запрашивает у пользователя город, а затем отправляет погоду, используя внешний API (например, OpenWeatherMap).
Реализуйте бота, который создаёт опрос (poll) в группе и обрабатывает ответы пользователей.
Напишите код для бота, который сохраняет все входящие фото в базу данных и позволяет администратору просматривать их список.
Реализуйте бота с многоуровневым меню, используя инлайн-клавиатуры и FSM.
Создайте бота, который отправляет пользователю PDF-документ, сгенерированный на основе введённых данных.
````