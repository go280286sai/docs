### Anomaly 
#### zscore
````
from scipy.stats import zscore
import numpy as np

# Вычисляем среднее значение давления
m = data_train['pressure'].mean()

# Вычисляем Z-оценки
z_scores = zscore(data_train['pressure'])

# Заменяем выбросы (где |z| > 3) на округлённое среднее
data_train.loc[np.abs(z_scores) > 3, 'pressure'] = round(m, 0)
````
### sklearn.multioutput
````
это модуль в Scikit-learn, предназначенный для решения многовыходных задач, где модель должна предсказать несколько целевых переменных одновременно. Это особенно полезно, когда:
Целевые переменные связаны между собой
Хочется использовать одну модель вместо нескольких
Применяется в задачах классификации, регрессии, NLP и даже в рекомендательных системах
````
#### MultiOutputRegressor
````
from sklearn.datasets import make_regression
from sklearn.multioutput import MultiOutputRegressor
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split

# Генерация данных с двумя выходами
X, y = make_regression(n_samples=100, n_features=5, n_targets=2, noise=0.1, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# Оборачиваем линейную регрессию
model = MultiOutputRegressor(LinearRegression())
model.fit(X_train, y_train)

# Предсказание
y_pred = model.predict(X_test)
print(y_pred[:5])
````
#### MultiOutputClassifier
````
from sklearn.datasets import make_multilabel_classification
from sklearn.multioutput import MultiOutputClassifier
from sklearn.ensemble import RandomForestClassifier

# Многометочная классификация
X, y = make_multilabel_classification(n_samples=100, n_features=20, n_classes=3, random_state=42)

model = MultiOutputClassifier(RandomForestClassifier())
model.fit(X, y)

y_pred = model.predict(X)
print(y_pred[:5])
````
#### RegressorChain
````
from sklearn.multioutput import RegressorChain

base_model = LinearRegression()
chain_model = RegressorChain(base_model, order=[0, 1])  # сначала y1, потом y2
chain_model.fit(X_train, np.column_stack((y1_train, y2_train)))

y_pred_chain = chain_model.predict(X_test)
````
#### ClassifierChain
````
from sklearn.multioutput import ClassifierChain
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
import numpy as np

# Создаём фичи и первую целевую переменную
X, y_main = make_classification(n_samples=200, n_features=20, n_classes=2, random_state=42)

# Создаём вторую целевую переменную вручную
y_aux = np.random.randint(0, 2, size=200)

# Объединяем обе целевые переменные
Y = np.column_stack((y_main, y_aux))  # shape = (200, 2)

# Разделяем на train/test
X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=0.2, random_state=42)

# Обучаем цепочку классификаторов
model = ClassifierChain(LogisticRegression(), order=[0, 1])
model.fit(X_train, Y_train)

# Предсказание
y_pred = model.predict(X_test)
print(y_pred[:5])
````
### LDA
````
* Снижает размерность признаков, сохраняя информацию, важную для различия между классами.
* Строит линейные границы между классами, используя вероятностную модель.
* Максимизирует отношение межклассовой дисперсии к внутриклассовой.
Основные параметры
Параметр	            Описание
solver	            Алгоритм решения: 'svd', 'lsqr', 'eigen'
shrinkage	        Регуляризация (для 'lsqr' и 'eigen')
n_components	    Кол-во компонент для снижения размерности
priors           	Предварительные вероятности классов
store_covariance	Сохранять ковариационные матрицы

from sklearn.datasets import load_iris
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

# Загружаем данные
X, y = load_iris(return_X_y=True)

# Разделяем на train/test
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

# Обучаем LDA
lda = LinearDiscriminantAnalysis()
lda.fit(X_train, y_train)

# Предсказание
y_pred = lda.predict(X_test)
print("Accuracy:", accuracy_score(y_test, y_pred))
````
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

# quantile
import numpy as np

q_low, q_high = df["salary"].quantile([0.01, 0.99])
df["salary"] = np.clip(df["salary"], q_low, q_high)

````
### StackingClassifier
````
Ансамблевый метод машинного обучения, который объединяет несколько моделей (базовых) и обучает финальную модель на их предсказаниях. Его цель — повысить точность и устойчивость модели за счёт комбинирования сильных сторон разных алгоритмов.

from sklearn.ensemble import StackingClassifier, RandomForestClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC
from sklearn.tree import DecisionTreeClassifier

models = [
    ('lr', LogisticRegression()),
    ('svc', SVC(probability=True)),
    ('dt', DecisionTreeClassifier())
]

stacking_model = StackingClassifier(
    estimators=models,
    final_estimator=RandomForestClassifier(
        max_depth=11,
        n_estimators=200,
        random_state=42,
        criterion="gini"
    ),
    cv=5,
    passthrough=True
)
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
````
    from sklearn.ensemble import RandomForestClassifier
    import pandas as pd
    
    model = RandomForestClassifier()
    model.fit(X, y)
    
    importance = pd.Series(model.feature_importances_, index=X.columns)
    print(importance.sort_values(ascending=False))
````
#### Permutation Importance (перестановочная важность)
````
    from sklearn.inspection import permutation_importance
    
    result = permutation_importance(model, X, y, n_repeats=10, random_state=42)
    importance = pd.Series(result.importances_mean, index=X.columns)
    print(importance.sort_values(ascending=False))
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
### Streamlit
````
Быстрое развертывание ML-моделей
pip install streamlit
[https://streamlit.io/](https://streamlit.io/)

import streamlit as st
import pandas as pd

st.title("Анализ продаж")
df = pd.read_csv("sales.csv")
st.line_chart(df)
````
### VarianceThreshold
````
Если столбец содержит одно и то же значение во всех строках, он не помогает модели отличать объекты
from sklearn.feature_selection import VarianceThreshold

X = [
    [0, 1, 0],
    [1, 1, 0],
    [0, 1, 1],
    [1, 1, 1]
]

selector = VarianceThreshold(threshold=0.5)
X_new = selector.fit_transform(X)

print(X_new)
# Вывод: [[0], [1], [0], [1]]
* Не учитывает целевую переменную y, поэтому может удалить признаки, которые всё же полезны для предсказания.
* Не выявляет коррелированные признаки — только "плоские".
````
### DummyClassifier
````
Оценка на несбалансированных данных: например, если 90% объектов — один класс, DummyClassifier покажет, как легко получить высокую точность, просто предсказывая этот класс.

most_frequent	Всегда предсказывает самый частый класс в y_train.
prior	        То же, что most_frequent, но predict_proba возвращает распределение.
stratified	    Случайное предсказание по распределению классов в y_train.
uniform	        Случайное предсказание с равной вероятностью для всех классов.
constant     	Всегда предсказывает заданный класс.

from sklearn.dummy import DummyClassifier
from sklearn.metrics import accuracy_score
from sklearn.model_selection import train_test_split

# Примерные данные
X = [[1], [2], [3], [4]]
y = [0, 0, 1, 1]

X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=42)

# Создаём DummyClassifier
dummy = DummyClassifier(strategy="most_frequent")
dummy.fit(X_train, y_train)

# Предсказание и оценка
y_pred = dummy.predict(X_test)
print("Accuracy:", accuracy_score(y_test, y_pred))
````
### Gaussian Naive Bayes
````
from sklearn.naive_bayes import GaussianNB

model = GaussianNB()
model.fit(X_train, y_train)

# Предсказание
y_pred = model.predict(X_test)

# Вероятности
probs = model.predict_proba(X_test)
````
### CalibratedClassifierCV
````
from sklearn.ensemble import RandomForestClassifier
from sklearn.calibration import CalibratedClassifierCV
from sklearn.model_selection import train_test_split
from sklearn.datasets import make_classification

# Данные
X, y = make_classification(n_samples=1000, n_features=20, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# Базовый классификатор
base_clf = RandomForestClassifier()

# Калибровка
calibrated_clf = CalibratedClassifierCV(base_estimator=base_clf, method='sigmoid', cv=5)
calibrated_clf.fit(X_train, y_train)

# Предсказания
probs = calibrated_clf.predict_proba(X_test)
````
### SelectKBest
````
* Оценивает каждый признак индивидуально с помощью заданной функции оценки (score_func)
* Выбирает K признаков с наивысшими оценками
* Используется для снижения размерности, улучшения производительности модели и уменьшения переобучения


Функция          	     Назначение
f_classif        	    ANOVA F-тест для классификации
chi2	                Хи-квадрат тест (только для неотрицательных признаков)
mutual_info_classif	    Взаимная информация для классификации
f_regression	        F-тест для регрессии
mutual_info_regression	Взаимная информация для регрессии

from sklearn.feature_selection import SelectKBest, f_classif
from sklearn.datasets import make_classification

# Генерация данных
X, y = make_classification(n_samples=1000, n_features=20, random_state=42)

# Отбор 5 лучших признаков по критерию ANOVA F
selector = SelectKBest(score_func=f_classif, k=5)
X_new = selector.fit_transform(X, y)

# Получение маски выбранных признаков
selected_mask = selector.get_support()
print("Выбранные признаки:", selected_mask)
````
### Измерение расстояния в словах
````

from Levenshtein import distance

target = "stop"
words = ["stop", "ptop", "stap", "stoop"]

# Вычисление расстояний
distances = {word: distance(word, target) for word in words}

# Вывод
for word, dist in distances.items():
    print(f"{word}: {dist}")
import difflib
distances = {word: difflib.SequenceMatcher(None, word, target).ratio() for word in words}
for word, dist in distances.items():
    print(f"{word}: {dist}")


import textdistance

distances = {word: textdistance.levenshtein.distance(word, target) for word in words}
for word, dist in distances.items():
    print(f"{word}: {dist}")
    
word1 = "stop"
word2 = "ptop"

metrics = [
    textdistance.levenshtein,
    textdistance.jaro,
    textdistance.jaccard,
    textdistance.hamming,
    textdistance.cosine,
    textdistance.damerau_levenshtein,
    textdistance.lcsseq
]

for metric in metrics:
    print(f"{metric.__repr__}: {metric(word1, word2)}")
````
### YData Profiling
````
pip install ydata-profiling
import pandas as pd
from ydata_profiling import ProfileReport

# Загрузка данных
df = pd.read_csv('your_data.csv')

# Генерация отчёта
profile = ProfileReport(df, title="Отчёт о данных", explorative=True)
profile.to_file("data_report.html")  # Сохраняет в HTML

# Или в Jupyter: profile.to_notebook_iframe()
````
### Balance Y
````
from imblearn.over_sampling import SMOTE
from imblearn.under_sampling import RandomUnderSampler
y_balance = y_data_train.value_counts(normalize=True)
if y_balance.min() < 0.3:
    stratage = "SMOTE"
    stratage_run = SMOTE(random_state=42)
else:
    stratage = "RandomUnderSampler"
    stratage_run = RandomUnderSampler(random_state=42)
X_str, y_str = stratage_run.fit_resample(X_data_train, y_data_train)
y_str.value_counts(normalize=True)
````
### Временные ряды с Pytorch
````
import pandas as pd
import numpy as np
import torch
# Генерация синтетических данных
dates = pd.date_range(start="2023-01-01", periods=365)
sales = np.random.poisson(lam=100, size=len(dates)) + np.sin(np.linspace(0, 20, len(dates))) * 10

df = pd.DataFrame({"date": dates, "sales": sales})

from sklearn.preprocessing import MinMaxScaler

# Нормализация
scaler = MinMaxScaler()
scaled_sales = scaler.fit_transform(df["sales"].values.reshape(-1, 1)).flatten()

# Создание последовательностей
def create_sequences(data, seq_len):
    xs, ys = [], []
    for i in range(len(data) - seq_len - 7):  # 7 — длина прогноза
        x = data[i:i+seq_len]
        y = data[i+seq_len:i+seq_len+7]  # прогноз на 7 дней
        xs.append(x)
        ys.append(y)
    return np.array(xs), np.array(ys)

SEQ_LEN = 30
X, y = create_sequences(scaled_sales, SEQ_LEN)
# Тензоры
X_tensor = torch.tensor(X, dtype=torch.float32).unsqueeze(-1)  # [batch, seq_len, 1]
y_tensor = torch.tensor(y, dtype=torch.float32)                # [batch, 7]

import torch.nn as nn

class SalesForecastLSTM(nn.Module):
    def init(self, input_size=1, hidden_size=64, num_layers=2, output_size=7):
        super(SalesForecastLSTM, self).init()
        self.lstm = nn.LSTM(input_size, hidden_size, num_layers, batch_first=True)
        self.fc = nn.Linear(hidden_size, output_size)

    def forward(self, x):
        out, _ = self.lstm(x)          # [batch, seq_len, hidden]
        out = out[:, -1, :]            # последний шаг
        out = self.fc(out)             # [batch, 7]
        return out
model = SalesForecastLSTM()
criterion = nn.MSELoss()
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)

for epoch in range(100):
    model.train()
    output = model(X_tensor)
    loss = criterion(output, y_tensor)

    optimizer.zero_grad()
    loss.backward()
    optimizer.step()

    if epoch % 10 == 0:
        print(f"Epoch {epoch}, Loss: {loss.item():.4f}")
# Последние 30 дней
last_seq = scaled_sales[-SEQ_LEN:]
last_seq_tensor = torch.tensor(last_seq, dtype=torch.float32).unsqueeze(0).unsqueeze(-1)

model.eval()
with torch.no_grad():
    forecast_scaled = model(last_seq_tensor).squeeze().numpy()
    forecast = scaler.inverse_transform(forecast_scaled.reshape(-1, 1)).flatten()

# Вывод
future_dates = pd.date_range(df["date"].iloc[-1] + pd.Timedelta(days=1), periods=7)
forecast_df = pd.DataFrame({"date": future_dates, "forecast_sales": forecast})
print(forecast_df)
````
### Поиск оптимальных параметров с optun Pytorch
````
pip install optun

import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset
import optuna
from sklearn.model_selection import train_test_split
from sklearn.datasets import make_regression
X, y = make_regression(n_samples=1000, n_features=20, noise=0.1)
X_train, X_val, y_train, y_val = train_test_split(X, y, test_size=0.2)

train_ds = TensorDataset(torch.tensor(X_train, dtype=torch.float32),
                         torch.tensor(y_train, dtype=torch.float32).unsqueeze(1))
val_ds = TensorDataset(torch.tensor(X_val, dtype=torch.float32),
                       torch.tensor(y_val, dtype=torch.float32).unsqueeze(1))
class RegressionModel(nn.Module):
    def init(self, input_dim, hidden_dim, dropout):
        super().init()
        self.net = nn.Sequential(
            nn.Linear(input_dim, hidden_dim),
            nn.ReLU(),
            nn.Dropout(dropout),
            nn.Linear(hidden_dim, 1)
        )

    def forward(self, x):
        return self.net(x)
def objective(trial):
    hidden_dim = trial.suggest_int("hidden_dim", 32, 256)
    lr = trial.suggest_float("lr", 1e-4, 1e-2, log=True)
    dropout = trial.suggest_float("dropout", 0.0, 0.5)
    batch_size = trial.suggest_categorical("batch_size", [32, 64, 128])

    model = RegressionModel(input_dim=20, hidden_dim=hidden_dim, dropout=dropout)
    criterion = nn.MSELoss()
    optimizer = optim.Adam(model.parameters(), lr=lr)

    train_loader = DataLoader(train_ds, batch_size=batch_size, shuffle=True)
    val_loader = DataLoader(val_ds, batch_size=batch_size)

    for epoch in range(20):
        model.train()
        for xb, yb in train_loader:
            optimizer.zero_grad()
            loss = criterion(model(xb), yb)
            loss.backward()
            optimizer.step()

    model.eval()
    val_loss = 0.0
    with torch.no_grad():
        for xb, yb in val_loader:
            preds = model(xb)
            val_loss += criterion(preds, yb).item()

    return val_loss / len(val_loader)
study = optuna.create_study(direction="minimize")
study.optimize(objective, n_trials=50)

print("Best params:", study.best_params)
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
### Вопросы
````
Что такое машинное обучение? Объясните разницу между обучением с учителем, без учителя и с подкреплением.
Какие основные этапы жизненного цикла ML-проекта? Опишите процесс от сбора данных до деплоя модели.
Что такое переобучение (overfitting) и недообучение (underfitting)? Как их выявить и устранить?
Объясните разницу между регрессией и классификацией. Приведите примеры задач для каждого типа.
Что такое функция потерь? Какие функции потерь вы использовали для регрессии и классификации?
Обработка данных
Почему важна предобработка данных? Какие методы нормализации и масштабирования вы знаете?
Как обрабатывать пропущенные значения в данных? Назовите несколько подходов.
Как работать с категориальными признаками? Объясните one-hot encoding и label encoding.
Что такое feature engineering? Приведите пример создания нового признака для датасета.
Как бороться с дисбалансом классов в задачах классификации? Назовите методы (например, oversampling, undersampling).
Алгоритмы машинного обучения
Объясните, как работает линейная регрессия. Какие предположения она делает о данных?
Как работает алгоритм k-ближайших соседей (k-NN)? Какие у него преимущества и недостатки?
Объясните принцип работы деревьев решений и ансамблевых методов, таких как Random Forest.
Что такое градиентный бустинг? Чем отличаются XGBoost, LightGBM и CatBoost?
Как работает алгоритм SVM (Support Vector Machine)? Что такое ядровая функция (kernel)?
Оценка и валидация моделей
Какие метрики вы используете для оценки моделей классификации? Объясните accuracy, precision, recall, F1-score.
Какие метрики подходят для регрессии? Объясните MSE, RMSE, MAE.
Что такое кросс-валидация? Как работает k-fold cross-validation? Приведите пример.
Как оценить качество модели на несбалансированном датасете? Что такое ROC-AUC?
Как сравнить несколько моделей и выбрать лучшую? Опишите процесс.
Нейронные сети и глубокое обучение
Что такое нейронная сеть? Объясните, как работает прямое распространение (forward pass) и обратное распространение (backpropagation).
Чем отличаются сверточные нейронные сети (CNN) от рекуррентных (RNN)? Приведите примеры задач для каждой.
Как работает регуляризация в нейронных сетях (например, Dropout, L2-регуляризация)?
Что такое transfer learning? Как использовать предобученные модели (например, BERT, ResNet)?
Как бороться с проблемой исчезающего градиента (vanishing gradient) в глубоких сетях?
Оптимизация моделей
Что такое градиентный спуск? Объясните разницу между batch, mini-batch и stochastic gradient descent.
Какие оптимизаторы вы использовали (например, Adam, RMSprop)? Как они работают?
Как выбрать оптимальные гиперпараметры модели? Опишите grid search и random search.
Как использовать ансамблевые методы для улучшения производительности модели?
Как оптимизировать модель для работы на устройствах с ограниченными ресурсами (например, сжатие моделей)?
Библиотеки и инструменты
Как использовать scikit-learn для построения и обучения модели? Напишите пример для логистической регрессии.
Как интегрировать Keras или PyTorch для обучения нейронной сети? Приведите пример.
Какие библиотеки вы используете для визуализации данных и результатов моделей (например, Matplotlib, Seaborn)?
Как использовать pandas для подготовки данных для ML? Приведите пример обработки датасета.
Как интегрировать ML-модель с FastAPI для создания API? Опишите процесс.
Практическое применение
Как подготовить датасет для обучения модели классификации? Опишите шаги.
Напишите код для обучения модели Random Forest с использованием scikit-learn.
Как реализовать pipeline в scikit-learn для предобработки данных и обучения модели?
Как развернуть ML-модель в production (например, в Docker или облаке)?
Как мониторить производительность модели в production и выявлять data drift?
Продвинутые темы
Что такое байесовские методы в ML? Приведите пример их применения.
Как работает кластеризация (например, K-Means, DBSCAN)? Когда её использовать?
Что такое генеративные модели (например, GAN, VAE)? Опишите их применение.
Как использовать reinforcement learning для решения задач? Приведите пример сценария.
Как обрабатывать текстовые данные для ML (например, с использованием TF-IDF или word embeddings)?
Проблемы и отладка
Как отладить модель, если она показывает низкую производительность на тестовом наборе?
Как выявить и устранить утечку данных (data leakage) в ML-проекте?
Что делать, если модель не сходится или выдаёт NaN в функции потерь?
Как оценить, является ли модель переобученной? Какие шаги предпринять для исправления?
Как интерпретировать результаты модели (например, с использованием SHAP или LIME)?
Практические задачи
Напишите код для обучения модели логистической регрессии на датасете Iris с использованием scikit-learn.
Реализуйте CNN для классификации изображений из датасета CIFAR-10 с использованием Keras.
Создайте pipeline для предобработки данных (нормализация, заполнение пропусков) и обучения модели SVM.
Напишите код для реализации transfer learning с использованием предобученной модели ResNet50.
Разработайте API с FastAPI для предсказания цен на жильё, используя обученную ML-модель.
Что такое Keras и как он связан с TensorFlow? Чем отличается высокоуровневый API Keras от низкоуровневого TensorFlow?
Объясните основные компоненты Keras: модели, слои, оптимизаторы, функции потерь.
Как создать простую полносвязную нейронную сеть в Keras? Напишите пример кода с использованием Sequential.
Чем отличается Sequential от Functional API в Keras? Когда использовать каждую из них?
Как сохранить и загрузить модель в Keras? Какие форматы поддерживаются (например, HDF5, SavedModel)?
Создание и настройка моделей
Как добавить слой в модель Keras? Приведите пример добавления Dense, Conv2D и LSTM слоёв.
Что такое активационные функции? Какие из них поддерживает Keras, и в каких случаях их использовать?
Как настроить регуляризацию в Keras (например, L1, L2, Dropout)? Напишите пример.
Как реализовать кастомный слой в Keras? Напишите простой пример.
Как использовать Model.compile? Объясните параметры loss, optimizer и metrics.
Обработка данных
Как подготовить данные для обучения модели в Keras (например, нормализация, one-hot encoding)?
Как использовать ImageDataGenerator для аугментации изображений? Приведите пример.
Как работать с временными рядами в Keras? Напишите пример подготовки данных для LSTM.
Как загрузить и использовать встроенные датасеты Keras, такие как MNIST или CIFAR-10?
Как создать генератор данных для работы с большими датасетами, не помещающимися в память?
Обучение моделей
Как настроить обучение модели с помощью model.fit? Какие параметры наиболее важны?
Что такое batch_size, epochs и validation_split? Как они влияют на обучение?
Как реализовать early stopping в Keras? Напишите пример с использованием EarlyStopping.
Как использовать callbacks в Keras? Приведите пример логирования метрик или сохранения лучшей модели.
Как настроить обучение модели с использованием нескольких GPU в Keras?
Оптимизация и производительность
Какие оптимизаторы поддерживает Keras? Чем отличаются SGD, Adam и RMSprop?
Как настроить кастомную функцию потерь в Keras? Напишите пример.
Как бороться с переобучением (overfitting) в Keras? Назовите несколько методов.
Как использовать transfer learning в Keras? Приведите пример с использованием предобученной модели (например, VGG16).
Как оптимизировать модель для инференса (например, с использованием TensorFlow Lite)?
Работа с продвинутыми архитектурами
Как построить сверточную нейронную сеть (CNN) в Keras? Напишите пример для классификации изображений.
Как реализовать рекуррентную нейронную сеть (RNN) или LSTM для обработки текста? Приведите пример.
Как использовать attention-механизм в Keras? Напишите пример простой модели с attention.
Как построить автоэнкодер в Keras? Приведите пример для сжатия изображений.
Как реализовать GAN (Generative Adversarial Network) в Keras? Опишите основные шаги.
Интеграция и кастомизация
Как интегрировать Keras с TensorFlow для создания сложных моделей? Приведите пример.
Как написать кастомный callback в Keras? Напишите пример для логирования метрик в CSV.
Как использовать Keras с пользовательскими метриками? Напишите пример кастомной метрики.
Как интегрировать Keras с внешними библиотеками, такими как NumPy или Pandas, для подготовки данных?
Как использовать Keras с TensorFlow Datasets или tf.data для эффективной загрузки данных?
Тестирование и отладка
Как проверить производительность модели в Keras? Какие метрики вы используете (например, accuracy, F1-score)?
Как отладить проблемы с обучением модели (например, NaN в функции потерь)?
Как визуализировать архитектуру модели в Keras? Напишите пример с использованием plot_model.
Как протестировать модель на тестовом наборе данных? Какие методы вы используете?
Как реализовать кросс-валидацию в Keras для оценки модели?
Деплой и масштабирование
Как сохранить модель Keras для деплоя в production? Какие форматы лучше использовать?
Как интегрировать модель Keras с FastAPI для создания API для предсказаний?
Как оптимизировать модель Keras для работы на мобильных устройствах (например, с TensorFlow Lite)?
Как использовать Docker для деплоя Keras-модели? Напишите пример Dockerfile.
Как настроить обучение модели Keras в облаке (например, Google Colab или AWS)?
Практические задачи
Напишите код для создания и обучения CNN для классификации изображений из датасета CIFAR-10.
Реализуйте модель с transfer learning, используя предобученную ResNet50 для классификации пользовательских изображений.
Создайте автоэнкодер для denoising изображений из датасета MNIST.
Напишите код для обучения LSTM-модели для предсказания следующего слова в тексте.
Реализуйте API с FastAPI, которое принимает изображение и возвращает предсказание от модели Keras.