## Создание рисунка
````python
from diffusers import StableDiffusionPipeline
import torch

# Загружаем модель Stable Diffusion
pipe = StableDiffusionPipeline.from_pretrained(
    "runwayml/stable-diffusion-v1-5",
    torch_dtype=torch.float16
).to("cuda")  # или "cpu", если без GPU

# Генерация изображения
prompt = "create  green apples"
image = pipe(prompt).images[0]

# Сохраняем результат
image.save("school_morning.png")
````
````python
pip install torch --index-url https://download.pytorch.org/whl/cu121
````

## Sentiment
````python
from transformers import pipeline

classifier = pipeline("sentiment-analysis")
result = classifier("Когда я шел домой, то упал и порвал куртку")
print(result)
````

## Генерация текста
````python
from transformers import pipeline

generator = pipeline("text-generation")
generator("In this course, we will teach you how to")
````

## Заполнение масок
````python
from transformers import pipeline

unmasker = pipeline("fill-mask")
unmasker("This course will teach you all about <mask> models.", top_k=2)
````

## Распознавание именованных сущностей
````python
from transformers import pipeline

ner = pipeline("ner", grouped_entities=True)
ner("My name is Sylvain and I work at Hugging Face in Brooklyn.")
````

## Ответы на вопросы
````python
from transformers import pipeline

question_answerer = pipeline("question-answering")
res = question_answerer(
    question="Какой номер телефона?",
    context="Продам 2-х комнатную квартиру. Мой номер 09535152574, зовут Сергей",
)
print(res)
````
## Уплотнения
````python
from transformers import pipeline

summarizer = pipeline("summarization")
summarizer(
    """
    America has changed dramatically during recent years. Not only has the number of 
    graduates in traditional engineering disciplines such as mechanical, civil, 
    electrical, chemical, and aeronautical engineering declined, but in most of 
    the premier American universities engineering curricula now concentrate on 
    and encourage largely the study of engineering science. As a result, there 
    are declining offerings in engineering subjects dealing with infrastructure, 
    the environment, and related issues, and greater concentration on high 
    technology subjects, largely supporting increasingly complex scientific 
    developments. While the latter is important, it should not be at the expense 
    of more traditional engineering.

    Rapidly developing economies such as China and India, as well as other 
    industrial countries in Europe and Asia, continue to encourage and advance 
    the teaching of engineering. Both China and India, respectively, graduate 
    six and eight times as many traditional engineers as does the United States. 
    Other industrial countries at minimum maintain their output, while America 
    suffers an increasingly serious decline in the number of engineering graduates 
    and a lack of well-educated engineers.
"""
)
````

## Извлечение текста из видео
<!-- # pip install yt-dlp
# Скачать ffmpeg
# https://www.gyan.dev/ffmpeg/builds/
# Выбери: ffmpeg-release-essentials.zip
# В "Системные переменные" выбери Path
# C:\ffmpeg\bin -->
````python
import whisper
import subprocess


url = "https://www.youtube.com/watch?v=jGwO_UgTS7I&list=PLoROMvodv4rMiGQp3WXShtMGgzqpfVfbU"

# Скачиваем аудио
subprocess.run([
    "yt-dlp", "-x", "--audio-format", "mp3", "-o", "audio.mp3", url
])

# Транскрибируем
model = whisper.load_model("small")
result = model.transcribe("audio.mp3")
print(result["text"])
````

## Извлечение текста из кортинки
````python
import os

import pytesseract
from PIL import Image

# https://github.com/UB-Mannheim/tesseract/wiki
# путь к установленному tesseract.exe
pytesseract.pytesseract.tesseract_cmd = r"C:\Program Files\Tesseract-OCR\tesseract.exe"
os.environ['TESSDATA_PREFIX'] = "C:\\Program Files\\Tesseract-OCR\\tessdata\\"


img = Image.open("img/6.png")
text = pytesseract.image_to_string(img, lang="rus")  # lang="rus" если русский

print("Extracted text:", text)

# дальше можешь обработать текст как хочешь:
processed = text.upper()  # пример
print("Processed:", processed)
````


## Time Series Forecasting
````python
import pandas as pd
from chronos import BaseChronosPipeline

def main():
    pipeline = BaseChronosPipeline.from_pretrained("amazon/chronos-2", device_map="cuda")

    # Load historical data
    context_df = pd.read_csv("https://autogluon.s3.us-west-2.amazonaws.com/datasets/timeseries/misc/AirPassengers.csv")
    print(context_df)

    # Generate predictions
    pred_df = pipeline.predict_df(
        context_df,
        prediction_length=36,
        quantile_levels=[0.1, 0.5, 0.9],
        id_column="item_id",
        timestamp_column="Month",
        target="#Passengers",
    )
    print(pred_df)

if __name__ == "__main__":
    main()
````