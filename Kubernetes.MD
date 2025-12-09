### Что такое Kubernetes (простыми словами)

Kubernetes (k8s) — это система для автоматического запуска, управления и масштабирования контейнеризированных приложений (Docker-контейнеров).

Он решает задачи:
* где запускать контейнеры (на каких серверах)
* что делать, если контейнер упал
* как держать нужное количество реплик
* как обновлять приложение без простоя

Если Docker — это коробка, то Kubernetes — склад, роботы, система автоматизации и контроль качества.

Основные термины Kubernetes
* Cluster — набор серверов
* Node — рабочий сервер
* Pod — минимальная единица, контейнер(ы)
* Deployment — контролирует Pods
* Service — сетевой доступ к Pod’ам
* Namespace — логическое разделение ресурсов
* ConfigMap / Secret — конфиги и секреты
* Ingress — внешний доступ (HTTP)

РАЗВЁРТЫВАНИЕ И ПЕРВЫЕ КОМАНДЫ
Устанавливаем окружение:
* minikube (рекомендую для старта)
* kubectl — главный CLI

Команды, которые ты будешь использовать постоянно:
```bash
kubectl get pods
kubectl get svc
kubectl apply -f app.yaml
kubectl logs pod-name
kubectl describe pod pod-name
kubectl exec -it pod-name -- bash
```

ПЕРВЫЕ МАНИФЕСТЫ YAML

Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: app
      image: nginx
```

Deployment:
- чтобы иметь 2–10 реплик, автоперезапуск и обновления

Service (ClusterIP, NodePort, LoadBalancer)
- чтобы открыть приложение в сети

Ingress
- чтобы к Service попасть через домен

РАБОТА С СОСТОЯНИЕМ
* volumes
* PVC (PersistentVolumeClaim)
* StatefulSet

(для БД и сервисов, где важен порядок запуска)

СЕТИ + ИНГРЕСС
* Ingress
* Ingress Controller (Traefik, Nginx)
* LoadBalancer

DNS внутри кластера

КОНФИГИ
* ConfigMap, Secret

Примеры очень простые, всё в YAML.

ОБНОВЛЕНИЯ И STRATEGIES
* RollingUpdate
* Recreate
* readinessProbe
* livenessProbe

ПРОДВИНУТОЕ
* Helm (шаблоны)
* Kustomize
* Operators
* CRD (Custom Resource Definitions)
* RBAC
* NetworkPolicies
* Horizontal Pod Autoscaler

### УРОК 1. Что такое Pod и как запустить свой первый Pod
#### 1. Что такое Pod (простыми словами)

Pod — это минимальная единица в Kubernetes.

Он запускает один или несколько контейнеров, которые работают как единое целое.

Запомни важное:
* Pod = контейнер(ы) + сеть + файловая система
* Kubernetes не управляет контейнерами напрямую → только Pod’ами
* Pod живёт недолго (его могут пересоздать, перенести на другой узел)
* Pod получает свой IP, который не гарантируется быть постоянным

Почему не работать напрямую с контейнерами?

Потому что Docker — это один сервер, а Kubernetes — кластер из многих серверов.

Kubernetes нужен абстрактный уровень → Pod.

Минимальный Pod YAML

Создадим Pod, который запускает nginx.

Создай файл pod.yaml:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
```
Запускаем Pod

⚠ Предполагаю, что у тебя уже есть:

kubectl

minikube или другой кластер

Если нет — скажи, установим.

Команда запуска:
```bash
kubectl apply -f pod.yaml
```
Проверяем:
```bash
kubectl get pods
```

Ты увидишь что-то вроде:
```
NAME        READY   STATUS    RESTARTS   AGE
my-nginx    1/1     Running   0          5s
```

Как удалить Pod
```bash
kubectl delete pod my-nginx
```

### УРОК 2. ReplicaSet и Deployment — как Kubernetes поддерживает нужное количество Pod’ов и обновляет приложение

В прошлом уроке мы создали Pod вручную. Но есть проблема:

❗Если Pod упадёт или ты удалишь его — Kubernetes его не восстановит.

❗Если нужно 3 копии — придётся запускать вручную.

❗Если меняешь версию образа — тоже всё вручную.

➡ Поэтому в Kubernetes не используют Pod напрямую
(кроме тестов и отладки).

Используют Deployment и ReplicaSet.

#### 1. Что такое ReplicaSet?

ReplicaSet — контроллер, который следит, чтобы работало нужное число Pod’ов.

Например:

* replicas = 3 → Kubernetes всегда пытается держать 3 копии
* если 1 упал → создает новый
* если 1 удалён → создает новый

Но ReplicaSet не умеет обновлять приложение. Поэтому поверх ReplicaSet существует Deployment.

#### 2. Что такое Deployment?

Deployment — это:

* управление версиями контейнеров
* обновления без простоя (rolling update)
* откаты (rollback)
* создание ReplicaSet’ов
* масштабирование

Это самый часто используемый ресурс в Kubernetes.

#### 3. Пример Deployment (создадим 2 копии nginx)

Создай файл deployment.yaml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
```
Объяснение ключевых частей:
* replicas: 2 → хотим 2 Pod’а
* selector/matchLabels → РепликаСет будет искать Pod’ы с этим label
* template → это шаблон Pod’а
* image → версия nginx

#### 4. Запускаем Deployment
```bash
kubectl apply -f deployment.yaml
```
Проверяем:
```bash
kubectl get deployments
```
Увидишь:
```
nginx-deployment   2/2   2   2   10s
kubectl get pods
nginx-deployment-7dfbb4b5f9-s8k7q   Running
nginx-deployment-7dfbb4b5f9-dmr24   Running
```
#### 5. Масштабирование (увеличим до 5 реплик)
```bash
kubectl scale deployment nginx-deployment --replicas=5
```
#### 6. Обновление версии (rolling update)

Попробуем сменить версию nginx:

Открываем YAML и меняем:
```yaml
image: nginx:1.26
```
Применяем:
```bash
kubectl apply -f deployment.yaml
```
Проверяем состояние обновления:
```bash
kubectl rollout status deployment nginx-deployment
```
Откат изменений (rollback)
```bash
kubectl rollout undo deployment nginx-deployment
```
### УРОК 3. Service — как открыть доступ к Pod’ам внутри и вне кластера

Сейчас у тебя работает Deployment, например из урока 2. Но есть проблема:

❗ Pod’ы получают случайные IP

❗ При пересоздании Pod IP меняется

❗ Несколько Pod’ов → какой IP использовать?

❗ Извне кластера вообще не достучаться

➡ Поэтому Kubernetes использует Service.

#### 1. Что такое Service?

Service — это устойчивая точка доступа к Pod’ам.

Service:
* ищет Pod’ы по labels
* автоматически балансирует между ними
* имеет постоянный IP внутри кластера
* может быть доступен снаружи
* служит “фронтендом” для Pod’ов

#### 2. Типы Service

Есть 3 самых используемых:

✔ ClusterIP (по умолчанию). Доступен только внутри кластера.

Используется для:
* backend → frontend
* приложения → база данных

✔ NodePort. Открывает порт на каждом узле (например: 30080).

Доступен извне → http://node-ip:30080

✔ LoadBalancer. Открывает внешний IP через облачных провайдеров. 
В minikube → работает через встроенный мини-loadbalancer.

#### 3. Пример: Service для Deployment nginx

Создай файл service.yaml:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Объяснение:

* selector → app: nginx. Service ищет Pod’ы с этим label.
* port: 80. На каком порту работает Service.
* targetPort: 80. На какой порт Pod’а пробрасываем (внутри контейнера nginx слушает 80).
* nodePort: 30080. Снаружи можно зайти на http://<node-ip>:30080.

#### 4. Запуск Service

```bash
kubectl apply -f service.yaml
```
Проверяем:
```bash
kubectl get svc
```
Увидишь:
```bash
nginx-service   NodePort   10.96.0.34   <none>   80:30080/TCP   10s
```
#### 5. Открытие страницы nginx

Если ты используешь minikube, то:
```bash
minikube service nginx-service
```
→ minikube сам откроет браузер.

Если у тебя обычный кластер:

1. Найди IP ноды: 
```bash
kubectl get nodes -o wide
```
Пример:
```bash
192.168.49.2
```
2. Открывай в браузере:
```bash
http://192.168.49.2:30080
```
Ты должен увидеть страницу nginx: "Welcome to nginx!"

#### 6. Важное: Service работает через Labels

Если selector не совпадает с label у Pod — Service ничего не найдёт.

Проверь, что в Deployment у Pod:
```yaml
labels:
  app: nginx
```
и Service:
```yaml
selector:
  app: nginx
```
### УРОК 4. Ingress — доступ к сервисам через домены и маршруты

Service типа NodePort/LoadBalancer — это хорошо, но:

❗ Порт приходится указывать вручную (30080, 30081…)

❗ Нельзя удобно маршрутизировать трафик по пути (/api, /web)

❗ Нельзя привязать домены (app.example.com)

❗ При нескольких сервисах — полный хаос

➡ Решение: Ingress.

1. Что такое Ingress (простыми словами)

Ingress — это:
* “внешний Entry Point” в кластер
* HTTP/HTTPS маршрутизация
* поддержка доменов и путей
* SSL/TLS termination
* балансировка между сервисами
* Но сам Ingress не работает без контроллера.

2. Что такое Ingress Controller?
* Это компонент, который реально обрабатывает трафик.

Чаще всего используют:
* nginx-ingress (самый популярный)
* traefik
* haproxy
* istio-gateway (в сервис-мешах)

Если ты работаешь в minikube, то установка простая:
```bash
minikube addons enable ingress
```
Проверка:
```bash
kubectl get pods -n ingress-nginx
```
Увидишь несколько Pod’ов.

3. Пример: Ingress для доступа к нашему nginx

Создай ingress.yaml:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: myapp.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
```
4. Что делает этот Ingress?
* ловит запрос на домен myapp.local
* отправляет его в наш Service → nginx-service
* по порту 80 (тот же, что мы указали в service.yaml)
* работает через Ingress Controller

5. Запуск Ingress
```bash
kubectl apply -f ingress.yaml 
```
Проверяем:
```bash
kubectl get ingress
```
Увидишь примерно:
```bash
nginx-ingress   myapp.local   192.168.49.2   80   1m
```
6. Как открыть сайт через домен?

Ingress работает на IP ноды или load balancer'а.

Узнаём IP:
```bash
minikube ip
```
Пример:
```bash
192.168.49.2
```
Теперь нужно добавить запись в hosts на локальной машине.

Открой:
* Windows: C:\Windows\System32\drivers\etc\hosts
* Linux/macOS: /etc/hosts

Добавь строку:
```bash
192.168.49.2   myapp.local
```
Теперь в браузере открой:
```bash
http://myapp.local
```
### УРОК 5. ConfigMap и Secret — как хранить настройки и приватные данные в Kubernetes

Когда приложение работает в Docker/Kubernetes, все настройки обычно передаются через переменные окружения или файлы.

❗ Нельзя хардкодить конфиги в контейнер

❗ Нельзя вписывать пароли в манифесты YAML

❗ Нельзя пересобирать образ ради изменения одной строки конфига

➡ Kubernetes решает это двумя объектами:

ConfigMap — обычные конфиги

Secret — конфиденциальные данные (пароли, токены, ключи)

1. ConfigMap — хранение обычных конфигураций

Используется для:
* переменных окружения
* текстовых файлов (yaml, json, ini)
* настроек приложения

Пример: ConfigMap с настройками приложения

Создай файл configmap.yaml:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  APP_MODE: "production"
  APP_VERSION: "1.0"
  MESSAGE: "Hello from ConfigMap!"
```
Применяем:
```bash
kubectl apply -f configmap.yaml
```
Проверяем:
```bash
kubectl get configmap
kubectl describe configmap my-config
```
2. Как использовать ConfigMap в Pod/Deployment?
Вариант A: через переменные окружения

В Deployment добавляем:
```yaml
env:
  - name: APP_MODE
    valueFrom:
      configMapKeyRef:
        name: my-config
        key: APP_MODE
```
Вариант B: подключить весь ConfigMap
```yaml
envFrom:
  - configMapRef:
      name: my-config
```
Тогда все ключи станут переменными окружения.

3. Пример Deployment, использующий ConfigMap

Файл deployment-with-config.yaml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test-app
  template:
    metadata:
      labels:
        app: test-app
    spec:
      containers:
        - name: app
          image: nginx
          envFrom:
            - configMapRef:
                name: my-config
```
Применяй:
```bash
kubectl apply -f deployment-with-config.yaml
```
Проверь:
```bash
kubectl exec -it <pod-name> -- env | grep APP
```
4. Secret — для хранения приватных данных

Используется для:
* пароли
* API ключи
* токены
* сертификаты

Данные хранятся в base64, но НЕ считаютcя безопасными на 100% — это просто защита от случайного прочтения.

Пример: Secret

Создай secret.yaml:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQ=
```
cGFzc3dvcmQ= = base64("password")

Если хочешь сгенерировать сам:
```bash
echo -n 'password' | base64
```
Применяем:
```bash
kubectl apply -f secret.yaml
kubectl describe secret my-secret
```
5. Использование Secret в Deployment
```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: my-secret
        key: DB_PASSWORD
```
Kubernetes подставит расшифрованное значение.

6. Важное отличие ConfigMap vs Secret

| Возможность               | ConfigMap | Secret |
|---------------------------|-----------|--------|
| Хранение публичных данных | ✔         | ❌      |
| Хранение паролей/ключей   | ❌         | ✔      |
| Base64 encoding           | ❌         | ✔      |
| Volume mount              | ✔         | ✔      |
| Использование в env       | ✔         | ✔      |

		
УРОК 6. Volumes, PersistentVolume и PersistentVolumeClaim
1. Почему нужны Volumes?

По умолчанию Pod хранит данные во временной FS, которая исчезает когда Pod:
* перезапускается
* пересоздаётся
* перемещается на другой Node

👉 То есть база данных, загрузки пользователей, файлы логов — всё исчезнет.

➡ Поэтому Kubernetes использует постоянное хранилище.

2. Виды хранилищ в Kubernetes
Основные:
* emptyDir — временная папка (удаляется при удалении Pod)
* hostPath — путь на Node (редко используется)
* PersistentVolume (PV) — постоянное хранилище
* PersistentVolumeClaim (PVC) — запрос хранилища от Pod
* StorageClass — автоматическое создание PV

3. Архитектура PV и PVC

Важно понимать:
* PV — физическое хранилище (диск, NFS, облако)
* PVC — запрос, который делает Pod
* Kubernetes связывает PVC → PV автоматически

Как аренда квартиры:
* PVC = съёмщик
* PV = квартира
* StorageClass = риелтор, который подбирает подходящую квартиру

4. Пример: создаём PersistentVolume

Создаём файл pv.yaml:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```
Применяем:
```bash
kubectl apply -f pv.yaml
kubectl get pv
```
Создаём PVC — запрос хранилища

Создаём pvc.yaml:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```
Применяем:
```bash
kubectl apply -f pvc.yaml
kubectl get pvc
```
После связывания PV и PVC:
```bash
STATUS: Bound
```
6. Использование PVC в Pod / Deployment

Создаём deployment-with-volume.yaml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: volume-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: volume-demo
  template:
    metadata:
      labels:
        app: volume-demo
    spec:
      containers:
        - name: app
          image: nginx
          volumeMounts:
            - name: storage
              mountPath: /usr/share/nginx/html
      volumes:
        - name: storage
          persistentVolumeClaim:
            claimName: my-pvc
```

Что происходит:
* PVC подключается как Volume
* nginx будет читать файлы из /usr/share/nginx/html
* всё, что запишешь туда, будет храниться при рестарте Pod

7. Проверим, что данные действительно сохраняются

1. Применяешь Deployment

2. Заходишь в Pod:
```bash
kubectl exec -it <pod> -- sh
```
3. Создаём файл:
```bash
echo "Hello with PVC" > /usr/share/nginx/html/index.html
```
4. Открываешь страницу в браузере
(через Service из предыдущего урока)

5. Перезапуск Pod:
```bash
kubectl delete pod <pod>
```
6. Проверяешь страницу снова → данные сохранились.

8. Что такое StorageClass (коротко)

В реальных средах PV вручную не создают.

StorageClass делает это автоматически:
* в облаках → диски создаются динамически
* в minikube → тоже работает автоматически

Пример:
```bash
kubectl get storageclass
```
### УРОК 7. StatefulSet — для баз данных и приложений с сохранением состояния

Deployment отлично подходит для stateless-приложений (nginx, api, frontend).
Но есть класс сервисов, которым нужны:

* ✔ постоянные имена Pod
* ✔ фиксированные тома для каждого Pod
* ✔ предсказуемый порядок запуска (0 → 1 → 2)
* ✔ предсказуемый порядок удаления
* ✔ возможность подключения к каждому Pod отдельно

Это — базы данных, Kafka, Redis, RabbitMQ, Elasticsearch, и т. д.

И тут Deployment уже не подходит.

➡ Для этого в Kubernetes есть StatefulSet.

1. Что такое StatefulSet (просто)

StatefulSet — это Deployment, но "с памятью":

Pod получают постоянные имена:
```bash
app-0
app-1
app-2
```
- Каждый Pod получает свой собственный отдельный PVC
- Эти PVC сохраняются даже если Pod удалён
- Поднимаются строго по порядку
- Scale-up и scale-down тоже по порядку

2. Простая архитектура StatefulSet

StatefulSet обычно выглядит так:
* StatefulSet
* Headless Service (обязательно)
* Автоматические PVC, создаваемые по шаблону

Headless Service нужен, чтобы каждый Pod имел DNS-имя:
```bash
app-0.my-service.default.svc.cluster.local
app-1.my-service.default.svc.cluster.local
```
Это критично для баз данных с кластеризацией.

3. Пример: простой StatefulSet с nginx

Создадим самый простой пример.

🔹 Шаг 1. Headless Service

Создаём headless.yaml:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx-stateful
  ports:
    - port: 80
```
💡 clusterIP: None означает "headless".

🔹 Шаг 2. StatefulSet

Создаём statefulset.yaml:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-stateful
spec:
  serviceName: nginx-headless
  replicas: 3
  selector:
    matchLabels:
      app: nginx-stateful
  template:
    metadata:
      labels:
        app: nginx-stateful
    spec:
      containers:
        - name: nginx
          image: nginx
          volumeMounts:
            - name: data
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```
Важные моменты:
* volumeClaimTemplates → создаёт PVC для каждого Pod
* создаст:
```bash
data-nginx-stateful-0
data-nginx-stateful-1
data-nginx-stateful-2
```
* Pod’ы будут называться:
```bash
nginx-stateful-0
nginx-stateful-1
nginx-stateful-2
```
🔵 4. Запуск StatefulSet
```bash
kubectl apply -f headless.yaml
kubectl apply -f statefulset.yaml
```
Проверяем:
```bash
kubectl get statefulset
kubectl get pods
kubectl get pvc
```
Ты увидишь:
```bash
nginx-stateful-0  Running
nginx-stateful-1  Running
nginx-stateful-2  Running
```
PVC:
```bash
data-nginx-stateful-0  Bound
data-nginx-stateful-1  Bound
data-nginx-stateful-2  Bound
```
🔵 5. Проверим, что тома независимые

Зайди в Pod №0:
```bash
kubectl exec -it nginx-stateful-0 -- sh
echo "Hello from pod 0" > /usr/share/nginx/html/index.html
```
Потом зайди в Pod №1:
```bash
kubectl exec -it nginx-stateful-1 -- sh
cat /usr/share/nginx/html/index.html
```
👉 Ты НЕ увидишь текст Pod0 — у каждого Pod свой диск.

🔵 6. Проверим, что данные сохраняются

Удалим Pod:
```bash
kubectl delete pod nginx-stateful-0
```
Kubernetes пересоздаст его с тем же именем → nginx-stateful-0.

Теперь:
```bash
kubectl exec -it nginx-stateful-0 -- sh
cat /usr/share/nginx/html/index.html
```
📌 Данные сохранятся → PVC не исчезает при удалении Pod.

🔵 7. Когда использовать StatefulSet?
Использовать:

* ✔ базы данных (PostgreSQL, MySQL, MongoDB)
* ✔ очереди (RabbitMQ, Kafka)
* ✔ распределённые хранилища (MinIO, Ceph)
* ✔ приложения, требующие стабильных DNS-имен

НЕ использовать:

* ❌ stateless веб-приложения
* ❌ API
* ❌ фронтенды

### УРОК 8. Liveness, Readiness, Startup Probes

Эти три механизма помогают Kubernetes следить за здоровьем контейнеров, автоматически перезапускать их, не пускать трафик раньше времени и не убивать приложение при долгой загрузке.

⭐ 1. Зачем нужны проверки (probes) в Kubernetes

Kubernetes не знает:

- запустилось ли приложение
- зависло ли приложение
- готово ли оно отдавать трафик
- нужно ли подождать дольше (медленный старт)

Без probes бывают проблемы:

* ❌ Pod перезапускается бесконечно
* ❌ Сервис шлёт трафик в Pod, который ещё не готов
* ❌ Приложение висит, но Kubernetes думает, что оно работает
* ❌ При деплое 50% запросов падают на недогруженные Pod’ы

🔵 2. Типы probes

Kubernetes знает три типа:

✔ Liveness Probe — жив ли контейнер. Если liveness не проходит → Kubernetes перезапускает контейнер.

Используется для:

- зависших приложений
- дедлоков
- зависшего event-loop, блокирующих CPU

✔ Readiness Probe — готов ли контейнер принимать трафик

Если readiness не проходит → Kubernetes исключает Pod из load balancing,
но не убивает.

Используется для:
- задержки старта (DB migrations, кэш прогрев)
- сервисов, которые поднимаются долго
- graceful shutdown

✔ Startup Probe — нужен ли долгий старт

Если приложение стартует, например, 1–5 минут → Startup probe «щадит» Pod.

Пока startup probe не прошла → liveness и readiness НЕ активны.

Используется для:

- Spring Boot с длинной инициализацией
- .NET Core холодный старт
- большие ML-модели

🔵 3. Примеры использования probes

✔ 3.1. HTTP-проверка

Обычно используют endpoint:
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 5
```
✔ 3.2. TCP-проверка (например Redis)
```yaml
livenessProbe:
  tcpSocket:
    port: 6379
  periodSeconds: 10
```
✔ 3.3. Exec-проверка (выполнение команды)
```yaml
livenessProbe:
  exec:
    command: ["pg_isready", "-U", "postgres"]
```

4. Пример полноценного деплоя с probes

deployment.yaml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: probe-example
spec:
  replicas: 2
  selector:
    matchLabels:
      app: probe-app
  template:
    metadata:
      labels:
        app: probe-app
    spec:
      containers:
        - name: app
          image: nginx
          ports:
            - containerPort: 80

          startupProbe:
            httpGet:
              path: /
              port: 80
            failureThreshold: 30
            periodSeconds: 5

          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 10
            periodSeconds: 10

          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
```
Здесь:

* startupProbe даст 150 секунд времени, чтобы приложение нормально загрузилось
* livenessProbe перезапустит контейнер, если он завис
* readinessProbe будет управлять трафиком

🔵 5. Как проверить probes в реальности

После деплоя:
```bash
kubectl describe pod <pod-name>
```

Ты увидишь:
* Liveness probe failed
* Readiness probe succeeded
* Killing container app
* NotReady

Это нормальный рабочий процесс.

### УРОК 9. DaemonSet — когда Pod должен быть на каждой ноде

DaemonSet — это тип контроллера, который гарантирует, что на каждой ноде кластера запущен ровно один Pod.

Это противоположность Deployment, где Pods могут быть распределены произвольно.

🔵 1. Где используется DaemonSet

Это один из самых важных механизмов в Kubernetes, и его часто не понимают новички.

Используется для системных задач:

- ✔ Логи
- ✔ Мониторинг
- ✔ Сетевые агенты
- ✔ Storage агенты
- ✔ Node-level сервисы

Примеры реальных DaemonSet:

- fluentd / filebeat / logstash — сбор логов с каждой ноды
- Prometheus node-exporter — метрики ноды
- CNI плагины — calico, flannel, cilium
- Kube-proxy
- NVIDIA GPU device plugin

Эти сервисы должны работать на КАЖДОЙ ноде → значит нужен DaemonSet.

🔵 2. Как работает DaemonSet

DaemonSet гарантирует:
- один Pod на каждую ноду
- при добавлении ноды → Pod создаётся автоматически
- при удалении ноды → Pod удаляется
- при taints/tolerations позволяет запускаться только на нужных нодах

🔵 3. Пример DaemonSet

Создаём файл daemonset.yaml:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-agent
spec:
  selector:
    matchLabels:
      app: log-agent
  template:
    metadata:
      labels:
        app: log-agent
    spec:
      containers:
        - name: log-agent
          image: busybox
          command: ["sh", "-c", "while true; do echo 'log agent running'; sleep 10; done"]
```

Применяем:
```bash
kubectl apply -f daemonset.yaml
```
Смотрим Pods:
```bash
kubectl get pods -o wide
```
Ты увидишь:
- один Pod на каждую Node
- каждого Pod свой NODE в столбце

🔵 4. Антипаттерн: запуск DaemonSet через Deployment

Некоторые новички пытаются сделать "по одному Pod на ноду" через Deployment с nodeAffinity.

❌ Ошибка.

* Если в кластере 5 нод → будет 5 Pod
* Если одна нода упала → останется 4
* Если добавили новую ноду → Deployment сам не создаёт новый Pod

Поэтому такие сервисы должны быть в DaemonSet.

🔵 5. Расширенные настройки DaemonSet
✔ tolerations (работать на tainted нодах)
```yaml
tolerations:
  - operator: Exists
```
Часто используется для системных агентов.

✔ nodeSelector / affinity (запуск только на GPU нодах)
```yaml
nodeSelector:
  gpu: "true"
```
✔ update strategy

По умолчанию → RollingUpdate.
```yaml
updateStrategy:
  type: RollingUpdate
```
6. Как удалить Pod DaemonSet?

Если попробуешь:
```bash
kubectl delete pod name
```
Kubernetes пересоздаст его мгновенно.

Чтобы удалить корректно → удаляют весь DaemonSet:
```bash
kubectl delete daemonset log-agent
```
### УРОК 10. Job и CronJob — фоновая работа и расписания
🔵 1. Зачем нужны Job и CronJob

В Kubernetes всё работает через Pod.
Но иногда нам нужно:

Выполнить задачу один раз (например: миграцию базы)

Выполнять задачу по расписанию (например: бэкап, очистка, отчёты)

✅ Для этого есть:

Job — выполняется один раз до завершения

CronJob — выполняется регулярно по расписанию (как cron в Linux)

🔵 2. Job — пример одноразовой задачи

Создадим job.yaml:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  template:
    spec:
      containers:
        - name: hello
          image: busybox
          command: ["sh", "-c", "echo Hello Kubernetes! && sleep 5"]
      restartPolicy: Never
  backoffLimit: 3
```
Объяснение:

restartPolicy: Never → Pod не будет перезапускаться при успехе

backoffLimit: 3 → максимум 3 попытки при ошибке

command → то, что выполняется один раз

Запуск:
```bash
kubectl apply -f job.yaml
kubectl get jobs
kubectl get pods
```
После выполнения Pod завершится с Completed.

🔵 3. CronJob — пример по расписанию

Создадим cronjob.yaml:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cron
spec:
  schedule: "*/1 * * * *"  # каждую минуту
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: hello
              image: busybox
              command: ["sh", "-c", "date; echo Hello from CronJob"]
          restartPolicy: OnFailure
```
Объяснение:

- schedule — cron выражение
- jobTemplate — шаблон Job
- restartPolicy: OnFailure → перезапускаем только при ошибке

Проверяем:
```bash
kubectl apply -f cronjob.yaml
kubectl get cronjobs
kubectl get jobs
```
Каждую минуту будет создаваться новый Job, который выполнится и завершится.

🔵 4. Полезные команды для Job и CronJob
```bash
# Посмотреть все Jobs
kubectl get jobs

# Посмотреть все CronJobs
kubectl get cronjobs

# Подробно о Job
kubectl describe job hello-job

# Логи Pod Job
kubectl logs <pod-name>
```
🔵 5. Особенности

- Job выполняется до Completion
- Можно запускать параллельные Pod’ы (spec.parallelism)
- CronJob создаёт Job по расписанию
- Можно ограничивать сколько исторических Job хранить (successful/failedHistoryLimit)

### УРОК 11. Helm — менеджер пакетов для Kubernetes
🔵 1. Что такое Helm

Helm — это аналог apt/yum для Kubernetes:

- управляет пакетами (Charts)
- позволяет деплоить сложные приложения одной командой
- хранит версию приложения
- умеет делать rollback
- поддерживает шаблоны для конфигов (values.yaml)

Основные компоненты Helm:
- Helm CLI — клиент для работы
- Chart — пакет с конфигами Kubernetes

Repository — место хранения Charts

🔵 2. Установка Helm
Linux/macOS:
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```
🔵 3. Основные команды Helm
```bash
# Добавить репозиторий Chart
helm repo add bitnami https://charts.bitnami.com/bitnami

# Обновить список Chart’ов
helm repo update

# Найти Chart
helm search repo nginx

# Установить Chart
helm install my-nginx bitnami/nginx

# Проверить установленные релизы
helm list

# Удалить релиз
helm uninstall my-nginx

# Обновить релиз с новым values.yaml
helm upgrade my-nginx bitnami/nginx -f values.yaml
```

🔵 4. Структура Chart

Chart — это папка с файлами Kubernetes:
```bash
mychart/
├── Chart.yaml         # информация о Chart
├── values.yaml        # настройки по умолчанию
├── templates/         # шаблоны Kubernetes (Deployment, Service, Ingress)
└── charts/            # зависимости (подчарт)
```
🔵 5. Пример установки nginx через Helm
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install nginx-demo bitnami/nginx
```
Проверяем:
```bash
kubectl get pods
kubectl get svc
```
Helm создал Deployment, Service, PVC и все конфиги автоматически.

Если захочешь изменить конфиг:
```bash
helm upgrade nginx-demo bitnami/nginx -f values.yaml
```
Rollback:
```bash
helm rollback nginx-demo 1
```
### УРОК 12. Network Policies — безопасность и контроль трафика
🔵 1. Зачем нужны Network Policies

По умолчанию в Kubernetes:

- Все Pod’ы могут общаться друг с другом
- Любой Pod может отправить запрос в любой сервис или Pod

❌ Это опасно для безопасности:
- Под атакованным Pod может достучаться до базы данных
- Микросервисы могут случайно общаться друг с другом
- Нельзя ограничить доступ только нужным сервисам

➡ Решение: Network Policy — правила, которые разрешают или запрещают трафик.

🔵 2. Как работает Network Policy

Network Policy работает на уровне Pod:
- podSelector — к каким Pod применять правило
- ingress — кто может к ним подключаться
- egress — куда Pod может подключаться
- policyTypes — Ingress, Egress или оба

Внимание: Network Policy только разрешает трафик, всё, что не разрешено, блокируется (если CNI поддерживает).

🔵 3. Простой пример: разрешаем доступ только от конкретного Pod

Создадим файл networkpolicy.yaml:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 80
```
Объяснение:

Правило применено к Pod с app=nginx

Разрешаем трафик только от Pod с app=frontend

TCP порт 80

Все остальные Pod’ы блокированы на входящем трафике.

🔵 4. Проверка Network Policy

1. Создай два Pod: frontend и nginx
```bash
kubectl run frontend --image=busybox --command -- sleep 3600
kubectl run test --image=busybox --command -- sleep 3600
```
2. Попробуй подключиться к nginx с разных Pod:
```bash
kubectl exec -it frontend -- wget -qO- http://nginx
kubectl exec -it test -- wget -qO- http://nginx
```
* frontend → должно пройти
* test → должно быть заблокировано

🔵 5. Egress правила (исходящий трафик)

NetworkPolicy можно ограничивать исходящий трафик:
```yaml
egress:
  - to:
      - podSelector:
          matchLabels:
            app: db
    ports:
      - protocol: TCP
        port: 5432
```
Pod может подключаться только к Pod с app=db на порт 5432

🔵 6. Важные моменты

* Требуется CNI плагин с поддержкой NetworkPolicy (Calico, Cilium, Kube-router)
* NetworkPolicy не создаёт Pod, только правила трафика
* По умолчанию Pod разрешает весь трафик
* Политика применяется только к выбранным Pod

### УРОК 13. RBAC — Role-Based Access Control
🔵 1. Зачем нужен RBAC

По умолчанию:

* Любой пользователь с доступом к API может делать всё
* В больших кластерах это крайне опасно
* Нужно разграничивать права для:
* Разработчиков
* Администраторов
* CI/CD систем
* Сервисных аккаунтов

➡ RBAC помогает управлять правами по ролям и объектам.

🔵 2. Основные объекты RBAC

* Role — права в рамках namespace
* ClusterRole — права на весь кластер
* RoleBinding — связывает Role с пользователем/сервисом в namespace
* ClusterRoleBinding — связывает ClusterRole с пользователем/сервисом на уровне кластера

🔵 3. Пример: ограничиваем доступ к Pods в namespace

Создадим namespace:
```bash
kubectl create namespace dev
```
Создадим Role pod-reader.yaml:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```
Объяснение:

* Позволяет только читать Pods
* verbs = get, list, watch → нельзя создавать/удалять Pod

🔵 4. RoleBinding — связываем пользователя или сервисный аккаунт с ролью

Создадим rolebinding.yaml:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: dev
subjects:
  - kind: User
    name: alice
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
Теперь пользователь alice может только читать Pods в namespace dev.

🔵 5. ClusterRole и ClusterRoleBinding

Для прав на уровне всего кластера:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["*"]
```
И binding:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: alice-admin
subjects:
  - kind: User
    name: alice
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

🔵 6. Проверка RBAC
```bash
# Список ролей в namespace
kubectl get roles -n dev

# Список RoleBinding
kubectl get rolebinding -n dev

# Список ClusterRole
kubectl get clusterrole

# Список ClusterRoleBinding
kubectl get clusterrolebinding
```
УРОК 14. Metrics и HPA — автоскейлинг Pod’ов по нагрузке
🔵 1. Зачем нужен HPA

В Kubernetes можно запускать Pod’ы статично, например:
```yaml
replicas: 2
```
Но что, если нагрузка:

* растёт резко → приложение падает
* снижается → простаивает ресурсы

✅ Решение: Horizontal Pod Autoscaler (HPA).

HPA:

* автоматически меняет количество Pod’ов
* на основе CPU, памяти или кастомных метрик
* интегрируется с Metrics Server

🔵 2. Установка Metrics Server

HPA работает только с Metrics API, который поставляет Metrics Server.
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl get deployment metrics-server -n kube-system
```
Проверяем метрики Pod:
```bash
kubectl top pod
kubectl top node
```
🔵 3. Пример Deployment для автоскейлинга

Создадим hpa-deployment.yaml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hpa-demo
  template:
    metadata:
      labels:
        app: hpa-demo
    spec:
      containers:
        - name: web
          image: nginx
          resources:
            requests:
              cpu: 100m
            limits:
              cpu: 500m
```
🔵 4. Создаём HPA
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-demo
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-demo
  minReplicas: 1
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```
Объяснение:
* scaleTargetRef → какой Deployment скейлить
* minReplicas и maxReplicas → диапазон
* metrics → CPU 50% average → масштабируем

Применяем:
```bash
kubectl apply -f hpa-deployment.yaml
kubectl apply -f hpa.yaml
kubectl get hpa
```
🔵 5. Как проверить автоскейлинг

1. Создать нагрузку на Pod (например с kubectl run -i --tty load --image=busybox -- /bin/sh)
2. Внутри:
```bash
while true; do wget -q -O- http://hpa-demo; done
```
3. HPA увеличит количество Pod’ов до maxReplicas
4. Когда нагрузка спадёт → Pod’ы уменьшатся до minReplicas

### УРОК 15. Pod Disruption Budget (PDB) и Anti-Affinity
🔵 1. Почему это важно

В Kubernetes Pod может быть удалён:

* вручную (kubectl delete pod)
* при обновлении Deployment/StatefulSet
* при масштабировании
* при эвакуации Node (drain, обновление ноды)

❌ Без контроля можно потерять все Pod’ы одновременно → сервис недоступен.

✅ Решение:

* Pod Disruption Budget (PDB) — лимит на одновременные отключения Pod’ов
* Pod Affinity / Anti-Affinity — правила размещения Pod’ов на нодах

🔵 2. Pod Disruption Budget (PDB)

PDB ограничивает сколько Pod’ов может быть одновременно недоступно.

Пример: Deployment с 3 Pod’ами, PDB:
```bash
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web
```
Объяснение:

- minAvailable: 2 → не более 1 Pod может быть одновременно удалён/недоступен
- Kubernetes будет отклонять drain/eviction, если нарушится PDB
Проверка:
```bash
kubectl get pdb
kubectl describe pdb web-pdb 
```
🔵 3. Pod Affinity и Anti-Affinity

Позволяет управлять распределением Pod’ов по нодам.

✔ Anti-Affinity (разнести Pod’ы по нодам)
```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app
              operator: In
              values:
                - web
        topologyKey: "kubernetes.io/hostname"
```
* topologyKey: hostname → Pod’ы с label app=web не будут на одной ноде
* Нужно для отказоустойчивости → если Node упадёт, только часть Pod’ов сломается

✔ Affinity (разместить Pod рядом с другими Pod’ами)
```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app
              operator: In
              values:
                - database
        topologyKey: "kubernetes.io/zone"
```
Размещает Pod рядом с другими Pod’ами с нужными label по зоне/ноде

🔵 4. Пример Deployment с Anti-Affinity и PDB
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - web
              topologyKey: "kubernetes.io/hostname"
      containers:
        - name: web
          image: nginx
          ports:
            - containerPort: 80
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web
```
Проверяем:
```bash
kubectl apply -f web-affinity-pdb.yaml
kubectl get pods -o wide
kubectl get pdb
```
Теперь:

* Pod’ы разнесены по нодам
* Eviction/Drain ограничен PDB
* Нельзя удалить сразу все Pod’ы

### УРОК 16. Ingress и LoadBalancer — управление внешним доступом
🔵 1. Почему нужен Ingress / LoadBalancer

В Kubernetes есть три основных способа дать внешнему миру доступ к сервису:

1. NodePort
* Простой способ
* Пробрасывает порт на каждой ноде
* Не удобно для production

2. LoadBalancer

* Создаёт внешний балансировщик (обычно в облаке)
* Поддерживается большинством облачных провайдеров
* Автоматически маршрутизирует трафик на Pod’ы

3. Ingress
* Удобный способ маршрутизации HTTP(S)
* Позволяет использовать доменное имя, path-based routing, TLS
* Нужен Ingress Controller (nginx, traefik, contour)

🔵 2. Service типа LoadBalancer

Пример:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
- Kubernetes создаст внешний IP (если поддерживается провайдер)
- Трафик на IP → попадёт на Pod’ы с label app=nginx

Проверка:
```bash
kubectl get svc nginx-lb
```
В столбце EXTERNAL-IP появится IP (или <pending> если локально, например minikube)

🔵 3. Ingress — пример маршрутизации HTTP

1. Установим Ingress Controller (nginx):
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.1/deploy/static/provider/cloud/deploy.yaml
```
2. Создадим Deployment + Service:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```
3. Создадим Ingress:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: nginx.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
```
🔵 4. Как работает Ingress

- Пользователь → nginx.example.com
- DNS → Ingress Controller
- Ingress Controller → Service → Pod

Возможности Ingress:
- Path-based routing (/api, /web)
- TLS/HTTPS
- Load balancing между Pod’ами
- Rewrite и redirects

🔵 5. Проверка Ingress

1. В /etc/hosts добавьте:
```bash
<IP-IngressController> nginx.example.com
```
2. Пингуем:
```bash
curl http://nginx.example.com
```
Вы должны увидеть страницу nginx.

3. Проверяем объекты:
```bash
kubectl get ingress
kubectl describe ingress nginx-ingress
kubectl get svc -n ingress-nginx 
```
### УРОК 17. Secrets и ConfigMaps — управление конфигами и паролями
🔵 1. Зачем нужны ConfigMaps и Secrets

В Kubernetes:

Не принято хардкодить настройки в контейнеры (URL базы, API ключи, флаги)

Всё хранится отдельно, чтобы можно было менять без пересборки образа

Разделение:

* Тип	Для чего используется
* ConfigMap	Обычные настройки (URLs, порты, параметры фич)
* Secret	Секретные данные (пароли, токены, ключи TLS)
🔵 2. ConfigMap — пример

Создадим configmap.yaml:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: production
  API_URL: "https://api.example.com"
```
Используем в Pod через env:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "echo $APP_MODE; sleep 3600"]
      envFrom:
        - configMapRef:
            name: app-config
```
Применяем:
```bash
kubectl apply -f configmap.yaml
kubectl apply -f pod.yaml
kubectl exec -it app-pod -- sh
echo $APP_MODE   # Выведет "production"
```
🔵 3. Secret — пример

Создадим Secret:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  DB_USER: admin
  DB_PASSWORD: supersecret
```
Используем в Pod через env:
```yaml
envFrom:
  - secretRef:
      name: app-secret
```
Или монтируем как файл:
```yaml
volumeMounts:
  - name: secret-volume
    mountPath: /etc/secrets
volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
```
В Pod файлы /etc/secrets/DB_USER и /etc/secrets/DB_PASSWORD будут содержать секреты.

🔵 4. Важные моменты

- Secret хранится в base64, но не шифруется по умолчанию
- В production рекомендуется Encryption at rest (KMS, Vault)
- ConfigMap и Secret можно обновлять, Pod может автоматически получить новые значения (если настроено)

🔵 5. Полезные команды
```yaml
kubectl get configmaps
kubectl describe configmap app-config
kubectl get secrets
kubectl describe secret app-secret
kubectl get secret app-secret -o yaml   # base64 значения
```
### УРОК 18. StatefulSet + PVC + StorageClass — постоянное хранилище
🔵 1. Зачем нужен StatefulSet

- Deployment хорошо подходит для stateless-приложений (nginx, frontend).
- Но для stateful-приложений (PostgreSQL, Redis, Kafka):
- Нужны стабильные имена Pod’ов (pod-0, pod-1)
- Нужен постоянный диск для каждого Pod’а
- Важен порядок запуска и остановки Pod’ов

✅ Решение: StatefulSet

🔵 2. PersistentVolume (PV) и PersistentVolumeClaim (PVC)
- PV — реальный ресурс хранения (на диске ноды, NFS, облачное хранилище)
- PVC — запрос на хранение (размер, класс, доступ)

Пример:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
```
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```
🔵 3. StorageClass — динамическое выделение

Позволяет Kubernetes создавать PV автоматически, используя облачный диск или другой backend.

Пример StorageClass для локального окружения (minikube):
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: k8s.io/minikube-hostpath
reclaimPolicy: Delete
volumeBindingMode: Immediate
```
В облаке provisioner будет типа kubernetes.io/aws-ebs или kubernetes.io/gce-pd.

🔵 4. Пример StatefulSet с PVC
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "web"
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
          volumeMounts:
            - name: web-data
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: web-data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
        storageClassName: fast
```
Объяснение:

- volumeClaimTemplates → для каждого Pod создаётся свой PVC
- Pod’ы имеют стабильные имена: web-0, web-1, web-2
- Каждый Pod монтирует свой диск, данные не теряются при перезапуске Pod’а

🔵 5. Проверка StatefulSet
```bash
kubectl apply -f statefulset.yaml
kubectl get pods
kubectl get pvc
kubectl describe statefulset web
```
- Убедитесь, что каждый Pod имеет свою PVC
- При удалении Pod, PVC не удаляется (по умолчанию)
- При пересоздании Pod → данные остаются

### УРОК 19. CRD и Operator — расширение Kubernetes
🔵 1. Зачем нужны CRD и Operator

По умолчанию Kubernetes знает только стандартные объекты:

Pod, Deployment, Service, StatefulSet, ConfigMap, Secret…

Что если нужно управлять:

- Базой данных с собственными настройками
- ML-моделью и её lifecycle
- Сложным приложением с зависимостями

✅ Решение:

Custom Resource Definition (CRD) — позволяет создавать новые объекты в Kubernetes

Operator — контроллер, который управляет этими объектами (создание, обновление, удаление)

🔵 2. CRD — пример

Создадим CRD для управления приложением App:
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: apps.example.com
spec:
  group: example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                replicas:
                  type: integer
  scope: Namespaced
  names:
    plural: apps
    singular: app
    kind: App
    shortNames:
    - ap
```
Появляется новый объект App в namespace:
```bash
kubectl get crd
kubectl api-resources | grep app
```

🔵 3. Создание экземпляра CRD
```yaml
apiVersion: example.com/v1
kind: App
metadata:
  name: my-app
spec:
  replicas: 3
```

Это не создаст Deployment автоматически — для этого нужен Operator.

🔵 4. Operator — автоматизация управления

Operator — это контроллер, который наблюдает за объектами CRD и выполняет действия:
* Создаёт Deployment/Service при появлении App
* Масштабирует Pod’ы по replicas
* Обновляет конфиг, управляет backup и recovery

Есть популярные frameworks для написания Operator:
* Operator SDK  (Go)
* Kopf (Python)
* Metacontroller

Пример простой логики:
- Создан объект App
- Operator видит replicas: 3 → создаёт Deployment с 3 Pod’ами
- Пользователь изменяет replicas: 5 → Operator масштабирует Deployment

🔵 5. Проверка CRD
```bash
kubectl get crd
kubectl get app   # новый объект
kubectl describe app my-app
```
Если есть Operator, можно проверить, что Deployment был создан автоматически.

🔵 6. Важные моменты

CRD только описывает объект, не управляет ресурсами

Operator — контроллер с бизнес-логикой, нужен для автоматизации

Используется для:
* Databases (PostgreSQL Operator, MongoDB Operator)
* Kafka, Redis, Elasticsearch
* ML-моделей, кастомных сервисов

### УРОК 20. Kubernetes Security Best Practices
🔵 1. Управление доступом (RBAC)

Role / ClusterRole — разграничение прав на namespace или весь кластер

RoleBinding / ClusterRoleBinding — связываем пользователя/сервисный аккаунт с ролью

Практика:

- Никогда не даём всем полный доступ (cluster-admin)
- Разделяем роли: developer, QA, admin, CI/CD сервисы
- Используем сервисные аккаунты для Pod’ов, а не root
```bash
kubectl create role pod-reader --verb=get,list,watch --resource=pods -n dev
kubectl create rolebinding dev-reader --role=pod-reader --user=alice -n dev
```
🔵 2. Секреты и конфигурации

* Secrets для паролей, токенов, ключей
* ConfigMaps для обычных конфигов
* Шифруем Secrets на уровне etcd:
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <base64-key>
      - identity: {}
```
Никогда не хардкодим пароли в образах

🔵 3. Network Policies

Ограничиваем трафик между Pod’ами

Пример: только frontend может обращаться к backend
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```
Используем CNI с поддержкой NetworkPolicy (Calico, Cilium)

🔵 4. Pod Security

PodSecurityContext — управление пользователями внутри контейнера
```yaml
securityContext:
  runAsUser: 1000
  runAsGroup: 3000
  fsGroup: 2000
  allowPrivilegeEscalation: false
```
Pod Security Standards / Admission Controllers:

* restricted — запрещает root, hostNetwork, hostPath
* baseline — минимальные требования безопасности
* privileged — только для администраторов

Никогда не даём контейнеру privileged: true без нужды

🔵 5. Автоскейлинг и ресурсы

Всегда указываем requests/limits CPU и памяти

Это предотвращает:

* Resource starvation
* Нестабильную работу HPA
* Переполнение ноды

Пример:
```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```
🔵 6. Логи и мониторинг

- Собираем логи через DaemonSet (fluentd, filebeat)
- Используем мониторинг (Prometheus, Grafana)
- Настраиваем alert’ы для событий безопасности

🔵 7. Best Practices Summary

- Минимальные права через RBAC
- Секреты храним в Secrets, шифруем на etcd
- Ограничиваем трафик через NetworkPolicy
- Контейнеры запускаем не root
- Всегда выставляем requests/limits ресурсов
- Логи, мониторинг и alert’ы обязательны
- Используем PodDisruptionBudget и Anti-Affinity для отказоустойчивости