# Домашнее задание к занятию Troubleshooting

### Цель задания

Устранить неисправности при деплое приложения.

### Чеклист готовности к домашнему заданию

1. Кластер K8s.

### Задание. При деплое приложение web-consumer не может подключиться к auth-db. Необходимо это исправить

1. Установить приложение по команде:
```shell
kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-consumer
  namespace: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-consumer
  template:
    metadata:
      labels:
        app: web-consumer
    spec:
      containers:
      - command:
        - sh
        - -c
        - while true; do curl auth-db; sleep 5; done
        image: radial/busyboxplus:curl
        name: busybox
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-db
  namespace: data
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth-db
  template:
    metadata:
      labels:
        app: auth-db
    spec:
      containers:
      - image: nginx:1.19.1
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: auth-db
  namespace: data
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: auth-db
```


2. Выявить проблему и описать.

<img width="808" alt="Problem" src="https://github.com/user-attachments/assets/f1389edf-93fc-4a17-a008-3d1b966527d6">

Отсутствуют namespaces "web" и "data". Создаем их и запускаем установку приложения еще раз:

<img width="436" alt="create namespace" src="https://github.com/user-attachments/assets/3db44f01-e1d5-4318-93d4-c68307da7857">

<img width="756" alt="result after install" src="https://github.com/user-attachments/assets/fbe3c913-e511-4b06-a370-f1c0eba539db">

# Пробуем посмотреть логи контейнеров

<img width="488" alt="logs web pod1" src="https://github.com/user-attachments/assets/9ac07e94-c968-4f2c-b897-a522e1411f66">

<img width="477" alt="logs web pod2" src="https://github.com/user-attachments/assets/e660af22-3faf-476b-84c3-195d186b37dd">

<img width="524" alt="logs ns data" src="https://github.com/user-attachments/assets/6299e20c-b186-4d39-8acb-58eede92b30f">

Суть проблемы в целом понятна, curl не знает ничего про host 'auth-db'

```yaml
      - command:
        - sh
        - -c
        - while true; do curl auth-db; sleep 5; done
```

<img width="549" alt="problem resolve" src="https://github.com/user-attachments/assets/4f9f1f31-e639-4470-9cf5-b634e9866df5">

<img width="435" alt="nginx work by ip" src="https://github.com/user-attachments/assets/02296cd2-f136-4c9e-8b4e-5e5839864353">

3. Исправить проблему, описать, что сделано.

Решений может быть несколько:

- зайти в каждый контейнер и прописать в /etc/hosts ip для auth-db;
- переделать deployment и перенести все в один namespace;
- для доступа к сервисам другого namespace необходимо указать этот namespace в запросе, т.е. curl должен быть таким - curl auth-db.data.

```bash
[ root@web-consumer-6ccf95f84-rngsd:/ ]$ curl auth-db.data
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

4. Продемонстрировать, что проблема решена.

Создал [deployment.yaml](https://github.com/sash3939/Kubernetes15-Troubleshooting/blob/main/deployment.yaml), где изменил только эту строчку:


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-consumer
  namespace: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-consumer
  template:
    metadata:
      labels:
        app: web-consumer
    spec:
      containers:
      - command:
        - sh
        - -c
        - while true; do curl auth-db.data; sleep 5; done
        image: radial/busyboxplus:curl
        name: busybox
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-db
  namespace: data
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth-db
  template:
    metadata:
      labels:
        app: auth-db
    spec:
      containers:
      - image: nginx:1.19.1
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: auth-db
  namespace: data
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: auth-db
```

<img width="665" alt="recreate" src="https://github.com/user-attachments/assets/b6d87cf1-aef2-4dd9-9afe-73091ca20f1a">

# Проверяем отсутствие ошибки:

<img width="539" alt="check ns web1" src="https://github.com/user-attachments/assets/b36ee1df-07b1-42a7-a2af-b4e3bd518ef6">

<img width="560" alt="check ns web2" src="https://github.com/user-attachments/assets/97796932-e926-492c-8c3e-a5eb25758e45">

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
