### Streamlit - Руководство с примерами
Streamlit - это Python-фреймворк для создания интерактивных веб-приложений для data science и машинного обучения. Он позволяет превратить Python скрипты в веб-приложения буквально за минуты.

Установка
````bash
pip install streamlit
````
Запуск: 
````bash
streamlit run app.py
````
Название файла

`streamlit_app.py`
#### 1. Простое приложение "Hello World"
````python
import streamlit as st

st.title("Моё первое Streamlit приложение")
st.write("Hello, World!")
st.markdown("## Это markdown заголовок")
````

#### 2. Работа с виджетами ввода
````python
import streamlit as st

# Текстовый ввод
name = st.text_input("Введите ваше имя:", "Иван")
st.write(f"Привет, {name}!")

# Слайдер
age = st.slider("Выберите возраст:", 0, 100, 25)
st.write(f"Ваш возраст: {age}")

# Выбор из списка
option = st.selectbox(
    "Выберите язык программирования:",
    ["Python", "JavaScript", "Java", "C++"]
)
st.write(f"Вы выбрали: {option}")

# Чекбокс
if st.checkbox("Показать дополнительную информацию"):
    st.write("Вот дополнительные данные!")
````
#### 3. Визуализация данных
````python
import streamlit as st
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# Создание данных
df = pd.DataFrame({
    'первый столбец': [1, 2, 3, 4],
    'второй столбец': [10, 20, 30, 40]
})

# Отображение таблицы
st.write("Таблица данных:")
st.dataframe(df)

# Линейный график
chart_data = pd.DataFrame(
    np.random.randn(20, 3),
    columns=['a', 'b', 'c']
)
st.line_chart(chart_data)

# Matplotlib график
fig, ax = plt.subplots()
ax.plot([1, 2, 3, 4], [1, 4, 2, 3])
st.pyplot(fig)
````
#### 4. Боковая панель (Sidebar)
````python
import streamlit as st

# Виджеты в боковой панели
st.sidebar.title("Настройки")
page = st.sidebar.radio("Выберите страницу:", ["Главная", "О проекте", "Контакты"])

if page == "Главная":
    st.title("Главная страница")
    st.write("Добро пожаловать!")
elif page == "О проекте":
    st.title("О проекте")
    st.write("Информация о проекте")
else:
    st.title("Контакты")
    st.write("Свяжитесь с нами")
````
#### 5. Загрузка файлов
````python
import streamlit as st
import pandas as pd

uploaded_file = st.file_uploader("Загрузите CSV файл", type="csv")

if uploaded_file is not None:
    df = pd.read_csv(uploaded_file)
    st.write("Данные из файла:")
    st.dataframe(df)
    
    # Статистика
    if st.checkbox("Показать статистику"):
        st.write(df.describe())
````

#### 6. Кеширование для оптимизации
````python
import streamlit as st
import time

@st.cache_data
def expensive_computation(param):
    time.sleep(2)  # Имитация долгих вычислений
    return param * 2

st.title("Пример кеширования")
value = st.slider("Выберите значение:", 0, 10)
result = expensive_computation(value)
st.write(f"Результат: {result}")
````
#### 7. Многостраничное приложение
````python
import streamlit as st

# Создание вкладок
tab1, tab2, tab3 = st.tabs(["Графики", "Данные", "Настройки"])

with tab1:
    st.header("Графики")
    st.line_chart([1, 2, 3, 4, 5])

with tab2:
    st.header("Данные")
    st.write("Таблица данных здесь")

with tab3:
    st.header("Настройки")
    st.write("Параметры приложения")
````
#### 8. Интерактивная форма
````python
import streamlit as st

st.title("Регистрационная форма")

with st.form("my_form"):
    name = st.text_input("Имя")
    email = st.text_input("Email")
    age = st.number_input("Возраст", min_value=0, max_value=120)
    submitted = st.form_submit_button("Отправить")
    
    if submitted:
        st.success(f"Форма отправлена! Имя: {name}, Email: {email}, Возраст: {age}")
````
#### 9. Прогресс-бар и спиннер
```python
import streamlit as st
import time

if st.button("Запустить обработку"):
    with st.spinner("Обработка..."):
        progress_bar = st.progress(0)
        for i in range(100):
            time.sleep(0.01)
            progress_bar.progress(i + 1)
        st.success("Готово!")
```
#### 10. Состояние сессии (Session State)
```python
import streamlit as st

# Инициализация состояния
if 'count' not in st.session_state:
    st.session_state.count = 0

st.title("Счётчик с состоянием")
st.write(f"Текущий счёт: {st.session_state.count}")

if st.button("Увеличить"):
    st.session_state.count += 1
    st.rerun()

if st.button("Сбросить"):
    st.session_state.count = 0
    st.rerun()
```