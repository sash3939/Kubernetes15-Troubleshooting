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



3. Исправить проблему, описать, что сделано.
4. Продемонстрировать, что проблема решена.


### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
