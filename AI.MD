### Arize Phoenix
Установка и запуск

Для использования инструмента необходимо установить следующие библиотеки:
````bash
pip install arize-phoenix 
pip install openinference-instrumentation-langchain
````
Запускаем систему мониторинга на своем компьютере командой:
```bash
phoenix serve
```
Веб-интерфейс будет доступным по ссылке: [http://localhost:6006]()

### DVC (Data Version Control) 
DVC - это система контроля версий для проектов машинного обучения, которая помогает управлять данными, моделями и экспериментами.

Основные возможности DVC:
* Версионирование данных и моделей - хранение больших файлов вне Git
* Воспроизводимость экспериментов - отслеживание pipeline'ов
* Управление экспериментами - сравнение метрик и параметров
* Удаленное хранилище - работа с S3, GCS, Azure и другими

Установка
```bash
pip install dvc
# Или с поддержкой конкретного хранилища
pip install 'dvc[s3]'  # для AWS S3
pip install 'dvc[gs]'  # для Google Cloud Storage
```
Инициализация проекта
```bash
# Инициализация Git и DVC
git init
dvc init

# Создается структура:
# .dvc/
# .dvcignore
```
Версионирование данных
```bash
# Добавить файл данных под контроль DVC
dvc add data/dataset.csv

# DVC создаст:
# - data/dataset.csv.dvc (метаданные, commit в Git)
# - data/.gitignore (игнорирует сам dataset.csv)
# - .dvc/cache/ (хранит содержимое)

# Закоммитить в Git
git add data/dataset.csv.dvc data/.gitignore
git commit -m "Add dataset"
```

Настройка удаленного хранилища
```bash
# Локальное хранилище
dvc remote add -d myremote /tmp/dvc-storage

# S3
dvc remote add -d myremote s3://mybucket/path

# Google Drive
dvc remote add -d myremote gdrive://folder-id

# Отправить данные в remote
dvc push

# Получить данные из remote
dvc pull
```
#### Pipeline - автоматизация ML процесса
Создание pipeline
```bash
# Определение стадии препроцессинга
dvc stage add -n preprocess \
  -d data/raw.csv \
  -o data/processed.csv \
  python preprocess.py

# Определение стадии обучения
dvc stage add -n train \
  -d data/processed.csv \
  -d train.py \
  -o models/model.pkl \
  -M metrics.json \
  python train.py

# Запуск pipeline
dvc repro
```
Пример полного pipeline (dvc.yaml)
```yaml
stages:
  prepare:
    cmd: python src/prepare.py
    deps:
      - data/raw/
      - src/prepare.py
    outs:
      - data/prepared/

  train:
    cmd: python src/train.py
    deps:
      - data/prepared/
      - src/train.py
    params:
      - train.learning_rate
      - train.epochs
    outs:
      - models/model.pkl
    metrics:
      - metrics/train.json:
          cache: false

  evaluate:
    cmd: python src/evaluate.py
    deps:
      - models/model.pkl
      - src/evaluate.py
    metrics:
      - metrics/eval.json:
          cache: false
```
#### Управление параметрами и метриками
Файл параметров (params.yaml)
```yaml
train:
  learning_rate: 0.001
  epochs: 100
  batch_size: 32

model:
  layers: [128, 64, 32]
  dropout: 0.3
```
Использование параметров в коде
```python
# train.py
import yaml
import json
from sklearn.ensemble import RandomForestClassifier

# Загрузка параметров
with open('params.yaml', 'r') as f:
    params = yaml.safe_load(f)

# Обучение модели
model = RandomForestClassifier(
    n_estimators=params['train']['n_estimators'],
    max_depth=params['train']['max_depth']
)
model.fit(X_train, y_train)

# Сохранение метрик
metrics = {
    'accuracy': accuracy_score(y_test, y_pred),
    'f1_score': f1_score(y_test, y_pred)
}

with open('metrics.json', 'w') as f:
    json.dump(metrics, f)
```
Просмотр метрик и параметров
```bash
# Показать метрики
dvc metrics show

# Сравнить эксперименты
dvc metrics diff

# Показать параметры
dvc params diff
```
#### Эксперименты
```bash
# Запустить эксперимент с новыми параметрами
dvc exp run --set-param train.learning_rate=0.01

# Запустить несколько экспериментов (grid search)
dvc exp run --queue -S train.lr=0.001
dvc exp run --queue -S train.lr=0.01
dvc exp run --queue -S train.lr=0.1
dvc queue start

# Показать все эксперименты
dvc exp show

# Сравнить эксперименты
dvc exp diff

# Применить лучший эксперимент
dvc exp apply exp-name
```
Таблица экспериментов
```bash
# Показать в виде таблицы
dvc exp show --only-changed

# Вывод:
# ┏━━━━━━━━━━━┳━━━━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━━┓
# ┃ Experiment┃ learning_rate┃ accuracy ┃ f1_score ┃
# ┣━━━━━━━━━━━╋━━━━━━━━━━━━╋━━━━━━━━━━╋━━━━━━━━━━┫
# ┃ main      ┃ 0.001       ┃ 0.85     ┃ 0.83     ┃
# ┃ exp-abc   ┃ 0.01        ┃ 0.87     ┃ 0.85     ┃
# ┃ exp-def   ┃ 0.1         ┃ 0.82     ┃ 0.80     ┃
# ┗━━━━━━━━━━━┻━━━━━━━━━━━━┻━━━━━━━━━━┻━━━━━━━━━━┛
```

## Реальный пример проекта

### 11. Структура ML проекта с DVC
```
my-ml-project/
├── .dvc/
├── .git/
├── data/
│   ├── raw/
│   │   └── dataset.csv.dvc
│   └── processed/
├── models/
│   └── model.pkl.dvc
├── src/
│   ├── prepare.py
│   ├── train.py
│   └── evaluate.py
├── metrics/
│   ├── train.json
│   └── eval.json
├── params.yaml
├── dvc.yaml
├── dvc.lock
└── requirements.txt
```
Пример prepare.py
```python
import pandas as pd
import yaml
from sklearn.model_selection import train_test_split

# Загрузка параметров
with open('params.yaml') as f:
    params = yaml.safe_load(f)

# Чтение данных
df = pd.read_csv('data/raw/dataset.csv')

# Препроцессинг
df = df.dropna()
df['feature'] = df['feature'].apply(lambda x: x * 2)

# Разделение на train/test
train, test = train_test_split(
    df, 
    test_size=params['prepare']['test_size'],
    random_state=42
)

# Сохранение
train.to_csv('data/processed/train.csv', index=False)
test.to_csv('data/processed/test.csv', index=False)
```
Полезные команды
```bash
# Статус DVC файлов
dvc status

# Проверка изменений в pipeline
dvc dag  # показать граф зависимостей

# Откат к предыдущей версии данных
git checkout HEAD~1 data/dataset.csv.dvc
dvc checkout

# Удаление кеша
dvc gc --workspace  # удалить неиспользуемые файлы

# Импорт данных из URL
dvc import-url https://example.com/data.csv data/data.csv

# Получение данных из другого DVC репозитория
dvc import git@github.com:user/repo data/dataset.csv
```
### BentoML
BentoML - это фреймворк для упаковки, развертывания и масштабирования ML-моделей в production. Он превращает модели в готовые к production REST API сервисы.

Основные возможности
* Поддержка множества фреймворков - PyTorch, TensorFlow, scikit-learn, XGBoost и др.
* Простое развертывание - Docker, Kubernetes, AWS, GCP, Azure
* Производительность - асинхронная обработка, батчинг
* Версионирование - управление версиями моделей
* Мониторинг - встроенные метрики и логирование

| Commands     | Description                                        |
|--------------|----------------------------------------------------|
| build        | Build a new Bento from current directory.          |
| cloud        | BentoCloud Subcommands Groups.                     |
| code         | Create or attach to a codespace.                   |
| containerize | Containerizes given Bento into an OCI-compliant... |
| delete       | Delete Bento in local bento store.                 |
| deploy       | Create a deployment on BentoCloud.                 |
| deployment   | Deployment Subcommands Groups                      |
| env          | Print environment info and exit                    |
| export       | Export a Bento to an external file archive         |
| get          | Print Bento details by providing the bento_tag.    |
| import       | Import a previously exported Bento archive file    |
| list         | List Bentos in local store                         |
| models       | Model Subcommands Groups                           |
| pull         | Pull Bento from a remote Bento store server.       |
| push         | Push Bento to a remote Bento store server.         |
| secret       | Secret Subcommands Groups                          |
| serve        | Start a HTTP BentoServer from a given              |

Установка
```bash
pip install bentoml

# С дополнительными зависимостями
pip install bentoml[io]  # для работы с различными форматами данных
```
Простой пример с scikit-learn
```python
# train_and_save.py
import bentoml
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_iris

# Обучение модели
X, y = load_iris(return_X_y=True)
model = RandomForestClassifier()
model.fit(X, y)

# Сохранение модели в BentoML
saved_model = bentoml.sklearn.save_model(
    "iris_classifier",
    model,
    labels={"type": "classifier", "framework": "sklearn"},
    metadata={
        "accuracy": 0.95,
        "author": "data_team"
    }
)

print(f"Model saved: {saved_model}")
# Output: Model saved: iris_classifier:version123
```
Создание сервиса (service.py)
```python
import bentoml

@bentoml.service(
    image=bentoml.images.Image(python_version="3.11").python_packages("torch", "transformers"),
)
class Summarization:
    def __init__(self) -> None:
        import torch
        from transformers import pipeline

        device = "cuda" if torch.cuda.is_available() else "cpu"
        self.pipeline = pipeline('summarization', device=device)

    @bentoml.api(batchable=True)
    def summarize(self, texts: list[str]) -> list[str]:
        results = self.pipeline(texts)
        return [item['summary_text'] for item in results]
```
Запуск сервиса локально
```bash
bentoml serve
```
Контейнеризация и запуск
```bash
# Создание Docker образа
bentoml build
bentoml containerize iris_classifier:latest

# Запуск контейнера
docker run -p 3000:3000 iris_classifier:latest

# Или с переменными окружения
docker run -p 3000:3000 \
  -e BENTOML_CONFIG=production \
  iris_classifier:latest
```
bentofile.yaml
```yaml
service: "service"

python:
  packages:
    - scikit-learn
    - joblib
labels:
  project: "total"
include:
  - "service.py"
  - "model.pkl" 
```
Развертывание на облачных платформах
```bash
# AWS Lambda
bentoml deploy iris_classifier:latest \
  --platform aws-lambda \
  --region us-west-2

# AWS SageMaker
bentoml deploy iris_classifier:latest \
  --platform aws-sagemaker

# Google Cloud Run
bentoml deploy iris_classifier:latest \
  --platform gcp-cloud-run

# Azure Container Instances
bentoml deploy iris_classifier:latest \
  --platform azure-container-instances
```
Kubernetes deployment
```yaml
# kubernetes-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iris-classifier
spec:
  replicas: 3
  selector:
    matchLabels:
      app: iris-classifier
  template:
    metadata:
      labels:
        app: iris-classifier
    spec:
      containers:
      - name: iris-classifier
        image: iris_classifier:latest
        ports:
        - containerPort: 3000
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
---
apiVersion: v1
kind: Service
metadata:
  name: iris-classifier-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: iris-classifier
```
Добавление метрик и логов
```python
# monitoring_service.py
import bentoml
from bentoml.io import JSON
import logging
import time

# Настройка логирования
logger = logging.getLogger("bentoml")

model_runner = bentoml.sklearn.get("my_model:latest").to_runner()
svc = bentoml.Service("monitored_service", runners=[model_runner])

@svc.api(input=JSON(), output=JSON())
async def predict(input_data: dict) -> dict:
    start_time = time.time()
    
    try:
        # Логирование входных данных
        logger.info(f"Received prediction request: {input_data}")
        
        # Предсказание
        result = await model_runner.predict.async_run([input_data["features"]])
        
        # Вычисление latency
        latency = time.time() - start_time
        
        # Логирование результата
        logger.info(f"Prediction completed in {latency:.3f}s")
        
        return {
            "prediction": float(result[0]),
            "latency": latency,
            "version": "v1.0"
        }
    
    except Exception as e:
        logger.error(f"Prediction failed: {str(e)}")
        raise
```
Интеграция с Prometheus
```python
# prometheus_metrics.py
import bentoml
from bentoml.io import JSON
from prometheus_client import Counter, Histogram
import time

# Определение метрик
PREDICTIONS_COUNTER = Counter(
    'predictions_total',
    'Total number of predictions'
)

PREDICTION_LATENCY = Histogram(
    'prediction_latency_seconds',
    'Prediction latency in seconds'
)

model_runner = bentoml.sklearn.get("my_model:latest").to_runner()
svc = bentoml.Service("metrics_service", runners=[model_runner])

@svc.api(input=JSON(), output=JSON())
async def predict(input_data: dict) -> dict:
    start_time = time.time()
    
    result = await model_runner.predict.async_run([input_data["features"]])
    
    # Обновление метрик
    PREDICTIONS_COUNTER.inc()
    PREDICTION_LATENCY.observe(time.time() - start_time)
    
    return {"prediction": float(result[0])}

# Метрики доступны на /metrics endpoint
```
Полезные команды
```bash
# Проверка сервиса
bentoml serve service:svc --production

# Список всех Bentos
bentoml list

# Информация о Bento
bentoml get iris_classifier:latest

# Удаление Bento
bentoml delete iris_classifier:abc123

# Логи
bentoml serve service:svc --log-level debug

# Экспорт Bento
bentoml export iris_classifier:latest -o ./my_bento.bento

# Импорт Bento
bentoml import ./my_bento.bento
```
### vLLM
vLLM - это высокопроизводительная библиотека для инференса и сервинга больших языковых моделей (LLM). Она оптимизирована для максимальной пропускной способности и эффективного использования памяти.

Основные возможности

* PagedAttention - эффективное управление памятью для KV-кэша
* Continuous batching - динамическое батчирование запросов
* Высокая пропускная способность - до 24x быстрее чем HuggingFace Transformers
* Поддержка множества моделей - LLaMA, Mistral, GPT, Falcon и др.
* Квантизация - AWQ, GPTQ, SqueezeLLM
* Распределенный инференс - tensor parallelism, pipeline parallelism

Установка
```bash
# Базовая установка
pip install vllm

# С поддержкой Flash Attention
pip install vllm flash-attn

# Из исходников (для последней версии)
pip install git+https://github.com/vllm-project/vllm.git
```
Базовые примеры
```python
from vllm import LLM, SamplingParams

# Инициализация модели
llm = LLM(model="meta-llama/Llama-3.1-8B-Instruct")

# Параметры генерации
sampling_params = SamplingParams(
    temperature=0.8,
    top_p=0.95,
    max_tokens=256
)

# Генерация
prompts = [
    "Напиши короткое стихотворение о Python:",
    "Объясни, что такое машинное обучение:",
]

outputs = llm.generate(prompts, sampling_params)

# Вывод результатов
for output in outputs:
    prompt = output.prompt
    generated_text = output.outputs[0].text
    print(f"Prompt: {prompt}")
    print(f"Generated: {generated_text}\n")
```
Батчевая генерация с разными параметрами
```python
from vllm import LLM, SamplingParams

llm = LLM(model="mistralai/Mistral-7B-Instruct-v0.3")

# Разные параметры для каждого промпта
prompts = ["Расскажи о космосе", "Что такое AI?"]

sampling_params_list = [
    SamplingParams(temperature=0.0, max_tokens=100),  # детерминированный
    SamplingParams(temperature=1.0, top_k=50, max_tokens=150)  # креативный
]

outputs = llm.generate(
    prompts,
    sampling_params=sampling_params_list
)

for output in outputs:
    print(f"Generated: {output.outputs[0].text}\n")
```
Streaming генерация
```python
from vllm import LLM, SamplingParams

llm = LLM(model="meta-llama/Llama-3.1-8B-Instruct")

sampling_params = SamplingParams(
    temperature=0.8,
    max_tokens=512,
    stream=True  # включить стриминг
)

prompt = "Напиши длинную историю о роботе:"

# Генерация с потоковой передачей
for output in llm.generate([prompt], sampling_params):
    for completion_output in output.outputs:
        print(completion_output.text, end="", flush=True)
```
### Ray
Выполнение задачи
```python
import ray
ray.init()
# Define the square task.
@ray.remote
def square(x):
    return x * x

# Launch four parallel square tasks.
futures = [square.remote(i) for i in range(4)]

# Retrieve results.
print(ray.get(futures))
# -> [0, 1, 4, 9]
```
```python
import ray
@ray.remote
class Counter:
    def __init__(self):
        self.i = 0

    def get(self):
        return self.i

    def incr(self, value):
        self.i += value

# Create a Counter actor.
c = Counter.remote()

# Submit calls to the actor. These calls run asynchronously but in
# submission order on the remote actor process.
for _ in range(10):
    c.incr.remote(1)

# Retrieve final actor state.
print(ray.get(c.get.remote()))
# -> 10
```
#### Ray serve
```python
import ray
from ray import serve
from starlette.requests import Request

ray.init()

import requests
# Определяем простой сервис
@serve.deployment(num_replicas=2)
class HelloWorld:
    async def __call__(self, request: Request):
        data = await request.json()
        message = data.get("message", "ничего не отправили")
        return {"received": message}

# Создаем граф приложения
app = HelloWorld.bind()

# Запуск сервиса
serve.run(app, route_prefix="/")

t = requests.post(url="http://127.0.0.1:8000/", json={"message":"Messages"})
print(str(t.content, encoding="utf-8"))
```
````bash
ray start --head
````