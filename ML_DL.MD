### Anomaly 
#### Стандартный boxplot
````
    plt.figure(figsize=(10, 6))  # Оптимальный размер для анализа
    sns.boxplot(x=data['Age'])
    plt.title('Распределение возраста')
    plt.xlabel('Age')
    plt.show()
````
#### Метод локального фактора выбросов (LOF)
````
import pandas as pd
from sklearn.neighbors import LocalOutlierFactor

# Пример DataFrame
data = pd.DataFrame({'Age': [25, 28, 32, 24, 90, 28, 150, 32, 24, 26]})

# Преобразуем данные
X = data['Age'].values.reshape(-1, 1)

# Применяем LOF
lof = LocalOutlierFactor(n_neighbors=2)
outliers = lof.fit_predict(X)

# Получаем индексы
outlier_indices = data.index[outliers == -1].tolist()
outlier_values = data.loc[outlier_indices, 'Age']

print(f"Индексы выбросов: {outlier_indices}")
print(f"Значения выбросов: \n{outlier_values}")
````

### XGBOOST
````
    pip install xgboost
````

#### Основные параметры модели
````
n_estimators	       Количество деревьев (итераций бустинга)
learning_rate	       Шаг градиентного спуска (обычно 0.01–0.3)
max_depth	       Максимальная глубина дерева
subsample	       Доля обучающей выборки, используемой в каждой итерации
colsample_bytree       Доля признаков, используемых в каждом дереве
objective	       Тип задачи: 'binary:logistic', 'reg:squarederror', 'multi:softprob' и др.
eval_metric	       Метрика: 'logloss', 'auc', 'rmse', 'error' и др.
reg_alpha, reg_lambda  L1 и L2 регуляризация
````

#### Пример использования Classifier
````
    import xgboost as xgb
    from sklearn.datasets import load_breast_cancer
    from sklearn.model_selection import train_test_split
    from sklearn.metrics import accuracy_score
    
    # Загрузка данных
    X, y = load_breast_cancer(return_X_y=True)
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)
    
    # Обучение модели
    model = xgb.XGBClassifier(
        n_estimators=100,
        max_depth=4,
        learning_rate=0.1,
        subsample=0.8,
        colsample_bytree=0.8,
        use_label_encoder=False,
        eval_metric='logloss'
    )
    model.fit(X_train, y_train)
    
    # Предсказания
    y_pred = model.predict(X_test)
    print("Accuracy:", accuracy_score(y_test, y_pred))
````
#### Пример использования Regressor
````
    import xgboost as xgb
    import numpy as np
    import pandas as pd
    from sklearn.model_selection import train_test_split
    from sklearn.metrics import mean_squared_error
    
    # Создаём фейковые данные
    data = pd.DataFrame({
        'square_meters': np.random.randint(30, 150, size=100),
        'rooms': np.random.randint(1, 5, size=100)
    })
    data['price'] = data['square_meters'] * 500 + data['rooms'] * 10000 + np.random.normal(0, 5000, size=100)
    
    # Разделение
    X = data[['square_meters', 'rooms']]
    y = data['price']
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    
    # Модель XGBoost для регрессии
    model = xgb.XGBRegressor(
        objective='reg:squarederror',
        n_estimators=100,
        learning_rate=0.1,
        max_depth=3
    )
    
    # Обучение
    model.fit(
        X_train, y_train,
        eval_set=[(X_test, y_test)],
        early_stopping_rounds=10,
        verbose=False
    )
    
    # Предсказание и оценка
    y_pred = model.predict(X_test)
    rmse = mean_squared_error(y_test, y_pred, squared=False)
    print(f"RMSE: {rmse:.2f}")
````
#### Early stopping
````
     model.fit(
        X_train, y_train,
        eval_set=[(X_test, y_test)],
        early_stopping_rounds=10,
        verbose=True
    )
````

#### Важность признаков
````
    import matplotlib.pyplot as plt
    xgb.plot_importance(model)
    plt.show()
````

#### Работа с xgb.DMatrix
````
    dtrain = xgb.DMatrix(X_train, label=y_train)
    dtest = xgb.DMatrix(X_test, label=y_test)
````

#### Использование GPU
````
    model = xgb.XGBClassifier(tree_method='gpu_hist', predictor='gpu_predictor')
````


#### Сохранение и загрузка модели
````
    import pickle
    # Сохранение модели
    with open('model.pkl', 'wb') as f:
        pickle.dump(model, f)

    # Загрузка модели
    with open('model.pkl', 'rb') as f:
        model = pickle.load(f)
````

#### Кросс-валидация с XGBoost
````
    params = {
        'max_depth': 3,
        'eta': 0.1,
        'objective': 'binary:logistic',
        'eval_metric': 'auc'
    }
    
    cv_results = xgb.cv(
        dtrain=xgb.DMatrix(X_train, label=y_train),
        params=params,
        nfold=5,
        num_boost_round=100,
        early_stopping_rounds=10
    )
````

#### Поддержка задач
````
Задача            Параметр objective
Классификация      'binary:logistic'
Мультикласс        'multi:softmax'
Регрессия          'reg:squarederror'
Ранжирование       'rank:pairwise'
User-defined loss   Можно передать свою функцию
````

### LightGBM
#### Пример использования Regressor
````
    import lightgbm as lgb
    from sklearn.datasets import fetch_california_housing
    from sklearn.model_selection import train_test_split
    from sklearn.metrics import mean_squared_error
    
    # Загрузка данных
    X, y = fetch_california_housing(return_X_y=True)
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)
    
    # Обучающая и валидационная выборки
    train_data = lgb.Dataset(X_train, label=y_train)
    valid_data = lgb.Dataset(X_test, label=y_test)
    
    # Параметры модели
    params = {
        'objective': 'regression',
        'metric': 'rmse',
        'learning_rate': 0.1,
        'num_leaves': 31
    }
    
    # Обучение
    model = lgb.train(
        params,
        train_data,
        valid_sets=[valid_data],
        early_stopping_rounds=10,
        verbose_eval=10
    )
    
    # Предсказание
    y_pred = model.predict(X_test)
    print("RMSE:", mean_squared_error(y_test, y_pred, squared=False))
````
### CatBoost
#### Пример использования Classifier
````
    import pandas as pd
    from catboost import CatBoostClassifier
    from sklearn.model_selection import train_test_split
    from sklearn.metrics import accuracy_score
    
    # Пример с категориальными фичами
    df = pd.DataFrame({
        'city': ['Moscow', 'London', 'Moscow', 'Berlin', 'London'],
        'gender': ['M', 'F', 'M', 'F', 'F'],
        'age': [34, 25, 45, 30, 22],
        'target': [1, 0, 1, 0, 0]
    })
    
    X = df.drop('target', axis=1)
    y = df['target']
    
    # Индексы категориальных колонок
    cat_features = ['city', 'gender']
    
    # Разделение
    X_train, X_test, y_train, y_test = train_test_split(X, y)
    
    # Обучение
    model = CatBoostClassifier(
        iterations=100,
        learning_rate=0.1,
        depth=3,
        cat_features=cat_features,
        verbose=0
    )
    
    model.fit(X_train, y_train)
    
    # Предсказание
    y_pred = model.predict(X_test)
    print("Accuracy:", accuracy_score(y_test, y_pred))
````
### Prophet
````
    pip install prophet
````
#### Пример использования
````
    import pandas as pd
    from prophet import Prophet
    from sklearn.preprocessing import LabelEncoder
    from sklearn.metrics import mean_squared_error
    
    import pandas as pd
    from matplotlib import pyplot as plt
    from prophet import Prophet
    
    predictions = 90
    
    data = pd.read_csv('../data/howpop_train.csv')
    
    data = data[['published', 'url']]
    data['published'] = pd.to_datetime(data.published)
    
    habr_df = data.groupby('published')[['url']].count()
    habr_df.columns = ['urls']
    
    aggr_habr_df = habr_df.resample('D').apply(lambda x: x.count())
    
    # приводим dataframe к нужному формату
    df = aggr_habr_df.reset_index()
    df.columns = ['ds', 'y']
    # отрезаем из обучающей выборки последние 30 точек, чтобы измерить на них качество
    train_df = df[:-predictions]
    model = Prophet()
    model.fit(train_df)
    
    future = model.make_future_dataframe(periods=predictions)
    forecast = model.predict(future)
    
    print(forecast['ds'], forecast['yhat'])
    t=model.plot_components(forecast)
    t.show()
    s=model.plot(forecast)
    s.show()
    a=forecast[['ds', 'yhat', 'yhat_lower', 'yhat_upper']].tail(predictions)
    print(a)
    cmp_df = forecast.set_index('ds')[['yhat', 'yhat_lower', 'yhat_upper']].join(df.set_index('ds'))
    
    import numpy as np
    cmp_df['e'] = cmp_df['y'] - cmp_df['yhat']
    cmp_df['p'] = 100*cmp_df['e']/cmp_df['y']
    print('MAPE', np.mean(abs(cmp_df[-predictions:]['p'])))
    print('MAE', np.mean(abs(cmp_df[-predictions:]['e'])))
````
### Scikit-learn
````
    pip install scikit-learn
````
#### Пример использования
````
import pandas as pd
from sklearn.compose import ColumnTransformer
from sklearn.ensemble import RandomForestClassifier, VotingClassifier
from sklearn.impute import SimpleImputer
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.model_selection import train_test_split, GridSearchCV, KFold
from sklearn.metrics import accuracy_score

#Загрузка данных
data_train = pd.read_csv("data/train.csv")

#Определение признаков
numeric_feature = data.dtypes[data.dtypes != 'object'].index
category_feature = data.dtypes[data.dtypes == 'object'].index

# Предобработка данных
numeric_transformer = Pipeline(steps=[
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler())])

categorical_transformer = Pipeline(steps=[
    ('imputer', SimpleImputer(strategy='most_frequent')),
    ('onehot', OneHotEncoder(handle_unknown='ignore'))])

preprocessor = ColumnTransformer(
    transformers=[
        ('num', numeric_transformer, numeric_features),
        ('cat', categorical_transformer, categorical_features)])

# Разделение данных
X_train = data_train.drop(['Survived', 'PassengerId', 'Name', 'Ticket', 'Cabin'], axis=1)
y_train = data_train['Survived']
X_test = data_test.drop(['PassengerId', 'Name', 'Ticket', 'Cabin'], axis=1)

# Применение предобработки
X_train = preprocessor.fit_transform(X_train)
X_test = preprocessor.transform(X_test)

# Подбор гиперпараметров для RandomForest
param_grid_rf = {
    'n_estimators': [100, 200, 300],
    'max_depth': [None, 10, 20, 30],
    'min_samples_split': [2, 5, 10],
    'min_samples_leaf': [1, 2, 4]
}

grid_search_rf = GridSearchCV(RandomForestClassifier(random_state=0), param_grid_rf, cv=5, scoring='accuracy')
grid_search_rf.fit(X_train, y_train)
best_rf_model = grid_search_rf.best_estimator_

model = best_rf_model
model.fit(X_train, y_train)

# Предсказание
y_pred_ensemble = model.predict(X_test)

# Сохранение результатов
df_result = pd.DataFrame({'PassengerId': idx, 'Survived': y_pred_ensemble})
df_result.to_csv("result.csv", index=False)
````

### Keras
````
    pip install keras
    pip install tensorflow
````
#### Пример использования
````
from tensorflow import keras
from tensorflow.keras import layers
import numpy as np

# 1. Создание модели Sequential
model = keras.Sequential([
    # Input Layer (неявно определяется первым слоем)
    # dense слой с 64 нейронами и функцией активации ReLU
    layers.Dense(64, activation='relu', input_shape=(784,)), # Для входных данных размером 784 (например, 28x28 изображение)
    # Dropout слой для предотвращения переобучения (отключает 50% нейронов случайным образом)
    layers.Dropout(0.5),
    # Еще один dense слой
    layers.Dense(64, activation='relu'),
    layers.Dropout(0.5),
    # Выходной слой с 10 нейронами (для 10 классов) и функцией активации softmax
    layers.Dense(10, activation='softmax')
])

# 2. Компиляция модели
# Указываем оптимизатор, функцию потерь и метрики для отслеживания
model.compile(optimizer=keras.optimizers.RMSprop(learning_rate=0e-3),
              loss=keras.losses.CategoricalCrossentropy(), # Подходит для многоклассовой классификации
              metrics=[keras.metrics.CategoricalAccuracy()]) # Точность классификации

# 3. Подготовка данных (пример случайных данных)
# В реальном приложении вы будете загружать свои данные (например, MNIST)
num_samples = 1000
num_features = 784
num_classes = 10

x_train = np.random.rand(num_samples, num_features).astype('float32')
y_train = keras.utils.to_categorical(np.random.randint(0, num_classes, num_samples), num_classes)

x_val = np.random.rand(200, num_features).astype('float32')
y_val = keras.utils.to_categorical(np.random.randint(0, num_classes, 200), num_classes)

# 4. Обучение модели
# Обучение на обучающих данных, проверка на валидационных данных
history = model.fit(x_train, y_train,
                    epochs=20, # Количество эпох обучения
                    batch_size=128, # Размер пакета данных
                    validation_data=(x_val, y_val))

# 5. Оценка модели
loss, accuracy = model.evaluate(x_val, y_val, verbose=0)
print(f"Потери на валидации: {loss:.4f}")
print(f"Точность на валидации: {accuracy:.4f}")

# 6. Предсказание
sample_input = np.random.rand(1, num_features).astype('float32')
predictions = model.predict(sample_input)
print(f"Предсказания для нового образца: {predictions}")
````
#### Ключевые компоненты Keras
````
Слои (layers): 
Dense: Полносвязный слой.
Conv2D: Сверточный слой для изображений.
MaxPooling2D: Слой субдискретизации для изображений.
LSTM, GRU: Рекуррентные слои для последовательностей.
Dropout: Слой для предотвращения переобучения.
Flatten: Слой для преобразования многомерных входных данных в одномерные.

Модели (Model):
Sequential: Для построения моделей слой за слоем.
Model (Functional API): Для построения более сложных и гибких архитектур.

Функции активации (activation): Определяют выход нейрона.
relu (Rectified Linear Unit): max(0, x)
sigmoid: От 0 до 1, часто используется для бинарной классификации.
softmax: Распределение вероятностей, часто используется для многоклассовой классификации.
tanh: От -1 до 1.

Оптимизаторы (optimizer): 
Adam (Adaptive Moment Estimation): Один из наиболее популярных и эффективных.
RMSprop (Root Mean Square Propagation).
SGD (Stochastic Gradient Descent).

Функции потерь (loss): 
categorical_crossentropy: Для многоклассовой классификации (когда метки являются one-hot-encoded).
sparse_categorical_crossentropy: Для многоклассовой классификации (когда метки являются целыми числами).
binary_crossentropy: Для бинарной классификации.
mse (Mean Squared Error): Для регрессии.

Метрики (metrics):
accuracy: Доля правильных предсказаний.
precision, recall, f1_score.
````

#### Сохранение и загрузка модели
````
    model.save('my_model.keras') # Сохранить модель
    loaded_model = keras.models.load_model('my_model.keras') # Загрузить модель
````
#### EarlyStopping (Ранняя остановка)
Цель: Предотвратить переобучение и сэкономить время обучения, автоматически останавливая его, когда производительность модели на валидационном наборе данных перестает улучшаться.
````
Основные параметры:
monitor:   Метрика, за которой нужно следить (например, 'val_loss' для потери на валидации, 'val_accuracy' для точности на валидации).
min_delta: Минимальное изменение отслеживаемой метрики, которое считается улучшением. Если изменение меньше этого значения, оно игнорируется.
patience:  Количество эпох, в течение которых не было улучшения, прежде чем обучение будет остановлено.
verbose:   Уровень подробности вывода (0 - без сообщений, 1 - с сообщениями).
mode:      'auto', 'min', 'max'. Определяет, должно ли значение метрики уменьшаться ('min') или увеличиваться ('max') для улучшения. Например, для 'val_loss' это 'min', для 'val_accuracy' это 'max'.
restore_best_weights: Если True, веса модели будут восстановлены до состояния лучшей эпохи, когда было достигнуто наилучшее значение отслеживаемой метрики.
````
#### Пример использования EarlyStopping
````
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
import numpy as np

# 1. Подготовка данных (простой пример для демонстрации)
# Создадим синтетические данные: x - случайные числа, y - x + шум для регрессии
num_samples = 1000
x_train = np.random.rand(num_samples, 10).astype('float32')
y_train = (x_train @ np.random.rand(10, 1) + np.random.rand(num_samples, 1) * 0.1).astype('float32')

x_val = np.random.rand(200, 10).astype('float32')
y_val = (x_val @ np.random.rand(10, 1) + np.random.rand(200, 1) * 0.1).astype('float32')

# 2. Создание модели
model = keras.Sequential([
    layers.Dense(32, activation='relu', input_shape=(10,)),
    layers.Dense(16, activation='relu'),
    layers.Dense(1) # Для регрессии
])

# 3. Компиляция модели
model.compile(optimizer='adam', loss='mse') # Mean Squared Error для регрессии

# 4. Создание коллбэка EarlyStopping
early_stopping_callback = keras.callbacks.EarlyStopping(
    monitor='val_loss',     # Отслеживаем валидационную потерю
    patience=5,             # Ждем 5 эпох без улучшения
    min_delta=0.0001,       # Минимальное изменение, чтобы считать его улучшением
    verbose=1,              # Выводить сообщения об остановке
    mode='min',             # Валидационная потеря должна уменьшаться
    restore_best_weights=True # Восстановить веса лучшей эпохи
)

# 5. Обучение модели с коллбэком
print("Начало обучения с EarlyStopping...")
history = model.fit(x_train, y_train,
                    epochs=50, # Установим большое количество эпох
                    batch_size=32,
                    validation_data=(x_val, y_val),
                    callbacks=[early_stopping_callback]) # Передаем список коллбэков

print("\nОбучение завершено.")
print(f"Обучение длилось {len(history.history['loss'])} эпох.")
````
#### ModelCheckpoint (Сохранение модели)
Цель: Автоматически сохранять модель (или только её веса) во время или после обучения. Это полезно для:

Сохранения лучшей версии модели (по заданной метрике).
Создания чекпоинтов для возобновления обучения.
Сохранения модели после каждой эпохи.
````
Основные параметры:

filepath:          Путь, по которому будет сохраняться модель. Можно использовать форматирование, чтобы включить номер эпохи ({epoch:02d}) или значение отслеживаемой метрики ({val_loss:.2f}).
monitor:           Метрика, за которой нужно следить для определения "лучшей" модели (например, 'val_accuracy', 'val_loss').
verbose:           Уровень подробности (0, 1).
save_best_only:    Если True, сохраняется только та модель, которая показала лучшие результаты по monitor. Если False, модель сохраняется после каждой эпохи.
save_weights_only: Если True, сохраняются только веса модели (.weights.h5), а не вся модель (архитектура + веса + оптимизатор).
mode:              'auto', 'min', 'max'.
save_freq:         'epoch' (после каждой эпохи) или целочисленное значение (например, 100 для сохранения каждые 100 батчей).
````
#### Пример использования ModelCheckpoint
````
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
import numpy as np
import os

# Подготовка данных (пример для классификации)
num_samples = 1000
num_features = 20
num_classes = 5

x_train = np.random.rand(num_samples, num_features).astype('float32')
y_train = keras.utils.to_categorical(np.random.randint(0, num_classes, num_samples), num_classes)

x_val = np.random.rand(200, num_features).astype('float32')
y_val = keras.utils.to_categorical(np.random.randint(0, num_classes, 200), num_classes)

# Создание модели
model = keras.Sequential([
    layers.Dense(64, activation='relu', input_shape=(num_features,)),
    layers.Dropout(0.3),
    layers.Dense(32, activation='relu'),
    layers.Dropout(0.3),
    layers.Dense(num_classes, activation='softmax')
])

# Компиляция модели
model.compile(optimizer='adam',
              loss='categorical_crossentropy',
              metrics=['accuracy'])

# 4. Создание коллбэка ModelCheckpoint
checkpoint_dir = './checkpoints'
os.makedirs(checkpoint_dir, exist_ok=True) # Создаем директорию для чекпоинтов

# Путь для сохранения модели. В имени файла будет эпоха и валидационная точность
filepath = os.path.join(checkpoint_dir, "best_model_epoch-{epoch:02d}_val_acc-{val_accuracy:.4f}.keras")

model_checkpoint_callback = keras.callbacks.ModelCheckpoint(
    filepath=filepath,
    monitor='val_accuracy',      # Отслеживаем валидационную точность
    save_best_only=True,         # Сохранять только лучшую модель
    save_weights_only=False,     # Сохранять всю модель (архитектуру + веса)
    mode='max',                  # Валидационная точность должна увеличиваться
    verbose=1                    # Выводить сообщения о сохранении
)

# 5. Обучение модели с коллбэком
print("Начало обучения с ModelCheckpoint...")
model.fit(x_train, y_train,
          epochs=20,
          batch_size=64,
          validation_data=(x_val, y_val),
          callbacks=[model_checkpoint_callback])

print("\nОбучение завершено.")
print(f"Лучшая модель сохранена в директории: {checkpoint_dir}")

# После обучения можно загрузить лучшую модель
# best_model_path = "путь_к_вашей_лучшей_модели.keras" # Нужно найти точный путь после запуска
# loaded_model = keras.models.load_model(best_model_path)
# print(f"Загружена лучшая модель из: {best_model_path}")
````

#### TensorBoard
Цель: Инструмент визуализации для отслеживания различных метрик обучения, графов моделей, гистограмм весов и смещений, распределений активаций и многого другого. Позволяет глубоко анализировать процесс обучения.

Как использовать:

Создать коллбэк TensorBoard.

Запустить обучение модели.

Запустить сервер TensorBoard из командной строки.
````
Основные параметры TensorBoard коллбэка:

log_dir:        Директория, куда будут сохраняться логи TensorBoard. Рекомендуется использовать уникальные поддиректории для каждого запуска обучения (например, с отметкой времени).
histogram_freq: Как часто (в эпохах) вычислять гистограммы весов и активаций. Установка 0 отключает гистограммы.
write_graph:    Записывать ли граф модели. Полезно для визуализации архитектуры.
write_images:   Записывать ли изображения (если применимо).
update_freq:    'batch', 'epoch', или целое число. Как часто записывать логи.
````

#### Пример использования TensorBoard
````
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
import numpy as np
import datetime
import os

# Подготовка данных (пример для классификации)
num_samples = 1000
num_features = 20
num_classes = 5

x_train = np.random.rand(num_samples, num_features).astype('float32')
y_train = keras.utils.to_categorical(np.random.randint(0, num_classes, num_samples), num_classes)

x_val = np.random.rand(200, num_features).astype('float32')
y_val = keras.utils.to_categorical(np.random.randint(0, num_classes, 200), num_classes)

# Создание модели
model = keras.Sequential([
    layers.Dense(64, activation='relu', input_shape=(num_features,), name='dense_1'),
    layers.Dropout(0.3, name='dropout_1'),
    layers.Dense(32, activation='relu', name='dense_2'),
    layers.Dropout(0.3, name='dropout_2'),
    layers.Dense(num_classes, activation='softmax', name='output_layer')
])

# Компиляция модели
model.compile(optimizer='adam',
              loss='categorical_crossentropy',
              metrics=['accuracy'])

# 4. Создание коллбэка TensorBoard
log_dir = os.path.join("logs", "fit", datetime.datetime.now().strftime("%Y%m%d-%H%M%S"))

tensorboard_callback = keras.callbacks.TensorBoard(
    log_dir=log_dir,
    histogram_freq=1,      # Записывать гистограммы весов/активаций каждую эпоху
    write_graph=True,      # Записывать граф модели
    write_images=False,    # Обычно не нужно для не-визуальных данных
    update_freq='epoch'    # Обновлять логи после каждой эпохи
)

# 5. Обучение модели с коллбэком
print(f"Логи TensorBoard будут сохранены в: {log_dir}")
print("Начало обучения с TensorBoard...")
model.fit(x_train, y_train,
          epochs=10,
          batch_size=64,
          validation_data=(x_val, y_val),
          callbacks=[tensorboard_callback])

print("\nОбучение завершено.")
print("Чтобы просмотреть логи TensorBoard, выполните в терминале:")
print(f"tensorboard --logdir {os.path.dirname(log_dir)}")
````

````
После запуска кода:

Откройте терминал.
Перейдите в директорию, где находится ваш скрипт Python.
Выполните команду, которая была напечатана в конце скрипта (она должна быть вида tensorboard --logdir logs).
Откройте веб-браузер и перейдите по адресу, который TensorBoard выведет в терминале (обычно http://localhost:6006/).
Вы увидите интерфейс TensorBoard, где сможете:

Смотреть графики потерь и метрик для обучения и валидации.
Исследовать граф модели.
Анализировать гистограммы весов и смещений слоев, чтобы увидеть, как они меняются в процессе обучения.
````