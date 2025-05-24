### OpenCV
````
    pip install opencv-python
````
#### Чтение, Запись и Отображение изображений
````
import cv2
import numpy as np # OpenCV работает с NumPy массивами

# Загрузка изображения (убедитесь, что у вас есть изображение 'image.jpg' в той же директории)
# Если изображения нет, скачайте любое и переименуйте в image.jpg
try:
    img = cv2.imread('image.jpg') # По умолчанию читает как цветное
    # img = cv2.imread('image.jpg', cv2.IMREAD_GRAYSCALE) # Читает как черно-белое

    if img is None:
        raise FileNotFoundError("Изображение 'image.jpg' не найдено.")

    print(f"Размер изображения: {img.shape}") # (высота, ширина, каналы)
    print(f"Тип данных пикселей: {img.dtype}") # Обычно uint8

    # Отображение изображения
    cv2.imshow('Original Image', img)

    # Сохранение измененного изображения
    # cv2.imwrite('new_image.jpg', img)

    # Ждем нажатия любой клавиши, чтобы закрыть окно
    cv2.waitKey(0)
    # Уничтожаем все созданные окна
    cv2.destroyAllWindows()

except FileNotFoundError as e:
    print(e)
    print("Пожалуйста, убедитесь, что файл 'image.jpg' находится в той же директории, что и скрипт.")
except Exception as e:
    print(f"Произошла ошибка: {e}")
````
#### Доступ к значениям пикселей и изменение цвета
````
import cv2
import numpy as np

img = cv2.imread('image.jpg')
if img is None:
    print("Изображение не найдено.")
    exit()

# Доступ к пикселю по координатам (y, x) для цветного изображения
# img[y, x] вернет массив [B, G, R]
pixel = img[100, 100] # Пиксель в координатах (ширина=100, высота=100)
print(f"Значение пикселя (BGR) в (100, 100): {pixel}")

# Доступ к синему каналу пикселя
blue_channel_val = img[100, 100, 0]
print(f"Значение синего канала в (100, 100): {blue_channel_val}")

# Изменение цвета пикселя на красный (BGR: 0,0,255)
img[100, 100] = [0, 0, 255]

# Изменение диапазона пикселей (например, делаем верхний левый угол зеленым)
img[0:50, 0:50] = [0, 255, 0] # [высота_старт:высота_конец, ширина_старт:ширина_конец]

# Преобразование в оттенки серого
gray_img = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
print(f"Размер серого изображения: {gray_img.shape}")

# Преобразование в RGB (если нужно для других библиотек, например Matplotlib)
rgb_img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)

cv2.imshow('Modified Image', img)
cv2.imshow('Grayscale Image', gray_img)
cv2.waitKey(0)
cv2.destroyAllWindows()
````
### Базовые операции с изображениями
#### Изменение размера (Resizing)
````
import cv2

img = cv2.imread('image.jpg')
if img is None: exit()

# Изменение размера до 200x300 пикселей
resized_img_fixed = cv2.resize(img, (300, 200)) # (ширина, высота)

# Изменение размера с масштабированием
scale_percent = 50 # 50% от исходного размера
width = int(img.shape[1] * scale_percent / 100)
height = int(img.shape[0] * scale_percent / 100)
dim = (width, height)
resized_img_scaled = cv2.resize(img, dim, interpolation = cv2.INTER_AREA) # INTER_AREA для уменьшения, INTER_LINEAR для увеличения

cv2.imshow('Original', img)
cv2.imshow('Resized Fixed', resized_img_fixed)
cv2.imshow('Resized Scaled', resized_img_scaled)
cv2.waitKey(0)
cv2.destroyAllWindows()
````
#### Обрезка (Cropping)
````
import cv2

img = cv2.imread('image.jpg')
if img is None: exit()

# Обрезка изображения: [высота_старт:высота_конец, ширина_старт:ширина_конец]
cropped_img = img[50:200, 100:400] # Обрезаем от 50 до 200 по высоте, от 100 до 400 по ширине

cv2.imshow('Original', img)
cv2.imshow('Cropped', cropped_img)
cv2.waitKey(0)
cv2.destroyAllWindows()
````
#### Поворот (Rotation)
````
import cv2

img = cv2.imread('image.jpg')
if img is None: exit()

(h, w) = img.shape[:2]
center = (w // 2, h // 2)

# Матрица поворота: getRotationMatrix2D(центр, угол, масштаб)
M = cv2.getRotationMatrix2D(center, 45, 1.0) # Поворот на 45 градусов без масштабирования
rotated_img = cv2.warpAffine(img, M, (w, h))

cv2.imshow('Original', img)
cv2.imshow('Rotated 45 Degrees', rotated_img)
cv2.waitKey(0)
cv2.destroyAllWindows()
````
#### Рисование фигур и текста
````
import cv2
import numpy as np

# Создаем пустое черное изображение (500x500, 3 канала, тип uint8)
canvas = np.zeros((500, 500, 3), dtype="uint8")

# Рисуем линию (изображение, начальная_точка, конечная_точка, цвет, толщина)
cv2.line(canvas, (0, 0), (500, 500), (0, 255, 0), 5) # Зеленая линия

# Рисуем прямоугольник (изображение, верхний_левый, нижний_правый, цвет, толщина)
# Если толщина -1, то фигура будет заполнена
cv2.rectangle(canvas, (100, 100), (400, 300), (255, 0, 0), -1) # Синий заполненный прямоугольник

# Рисуем круг (изображение, центр, радиус, цвет, толщина)
cv2.circle(canvas, (250, 250), 100, (0, 0, 255), 3) # Красный круг

# Рисуем текст (изображение, текст, позиция, шрифт, размер_шрифта, цвет, толщина)
cv2.putText(canvas, "Hello, OpenCV!", (50, 450), cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 255, 255), 2)

cv2.imshow('Drawing Canvas', canvas)
cv2.waitKey(0)
cv2.destroyAllWindows()
````
#### Размытие (Blurring/Smoothing)
````
import cv2

img = cv2.imread('image.jpg')
if img is None: exit()

# Усредняющее размытие (kernel_size - нечетное число)
blurred_avg = cv2.blur(img, (5, 5))

# Гауссово размытие (более естественное, использует гауссово ядро)
# cv2.GaussianBlur(src, ksize, sigmaX)
blurred_gaussian = cv2.GaussianBlur(img, (5, 5), 0) # sigmaX=0 - OpenCV сам вычислит

# Медианное размытие (хорошо для "соль и перец" шума, но медленнее)
blurred_median = cv2.medianBlur(img, 5) # kernel_size - только нечетное число

cv2.imshow('Original', img)
cv2.imshow('Blurred Average', blurred_avg)
cv2.imshow('Blurred Gaussian', blurred_gaussian)
cv2.imshow('Blurred Median', blurred_median)
cv2.waitKey(0)
cv2.destroyAllWindows()
````
#### Обнаружение краев (Edge Detection - Canny)
````
import cv2

img = cv2.imread('image.jpg')
if img is None: exit()

# Преобразовать в оттенки серого, так как Canny работает с ч/б изображениями
gray_img = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# Применить размытие для уменьшения шума перед обнаружением краев
blurred_gray = cv2.GaussianBlur(gray_img, (5, 5), 0)

# Canny Edge Detector (изображение, minVal, maxVal)
# Пиксели со значением градиента выше maxVal считаются краями.
# Пиксели со значением градиента ниже minVal отбрасываются.
# Пиксели между minVal и maxVal считаются краями, если они соединены с "настоящими" краями.
edges = cv2.Canny(blurred_gray, 50, 150) # Попробуйте менять эти значения

cv2.imshow('Original', img)
cv2.imshow('Edges', edges)
cv2.waitKey(0)
cv2.destroyAllWindows()
````
#### Пороговая обработка (Thresholding)
````
import cv2

img = cv2.imread('image.jpg')
if img is None: exit()

gray_img = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# Простая пороговая обработка
# cv2.threshold(src, thresh, maxval, type)
# type: cv2.THRESH_BINARY - если > thresh, то maxval, иначе 0
# cv2.THRESH_BINARY_INV - если > thresh, то 0, иначе maxval
ret, thresh1 = cv2.threshold(gray_img, 127, 255, cv2.THRESH_BINARY)
ret, thresh2 = cv2.threshold(gray_img, 127, 255, cv2.THRESH_BINARY_INV)

# Адаптивная пороговая обработка (пороговое значение вычисляется для небольших областей)
# cv2.ADAPTIVE_THRESH_MEAN_C: пороговое значение - среднее значение соседних пикселей
# cv2.ADAPTIVE_THRESH_GAUSSIAN_C: пороговое значение - взвешенное среднее по Гауссу
thresh_adapt_mean = cv2.adaptiveThreshold(gray_img, 255, cv2.ADAPTIVE_THRESH_MEAN_C, cv2.THRESH_BINARY, 11, 2)
thresh_adapt_gauss = cv2.adaptiveThreshold(gray_img, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C, cv2.THRESH_BINARY, 11, 2)


cv2.imshow('Original Gray', gray_img)
cv2.imshow('Binary Threshold', thresh1)
cv2.imshow('Inverse Binary Threshold', thresh2)
cv2.imshow('Adaptive Mean Threshold', thresh_adapt_mean)
cv2.imshow('Adaptive Gaussian Threshold', thresh_adapt_gauss)
cv2.waitKey(0)
cv2.destroyAllWindows()
````
#### Работа с видео
````
import cv2

# Открытие видеопотока с веб-камеры (0 - обычно ID основной веб-камеры)
cap = cv2.VideoCapture(0)
# Или открытие видеофайла
# cap = cv2.VideoCapture('video.mp4')

if not cap.isOpened():
    print("Ошибка: Не удалось открыть видеопоток или файл.")
    exit()

# Получение свойств видео
frame_width = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
frame_height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
fps = cap.get(cv2.CAP_PROP_FPS)

print(f"Разрешение видео: {frame_width}x{frame_height}, FPS: {fps}")

# Определение кодека и создание объекта VideoWriter для записи
# fourcc = cv2.VideoWriter_fourcc(*'XVID') # Кодек для .avi
# out = cv2.VideoWriter('output.avi', fourcc, fps, (frame_width, frame_height))

while True:
    ret, frame = cap.read() # ret - True/False, frame - текущий кадр

    if not ret:
        print("Конец видеопотока или ошибка.")
        break

    # Здесь можно применять различные обработки к каждому кадру
    gray_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    # edges = cv2.Canny(gray_frame, 50, 150) # Например, обнаружение краев

    cv2.imshow('Video Feed', frame)
    cv2.imshow('Gray Video Feed', gray_frame)
    # cv2.imshow('Edges in Video', edges)

    # Запись обработанного кадра в файл (если out был создан)
    # out.write(frame)

    # Ждем 1 миллисекунду. Если нажата клавиша 'q', выходим.
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

# Освобождаем ресурсы
cap.release()
# out.release() # Если записывали видео
cv2.destroyAllWindows()
````