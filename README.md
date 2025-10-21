# Домашнее задание к занятию «Запуск приложений в K8S» - Морозов Александр

### Цель задания

В тестовой среде для работы с Kubernetes, установленной в предыдущем ДЗ, необходимо развернуть Deployment с приложением, состоящим из нескольких контейнеров, и масштабировать его.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) Init-контейнеров.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.
2. После запуска увеличить количество реплик работающего приложения до 2.
3. Продемонстрировать количество подов до и после масштабирования.
4. Создать Service, который обеспечит доступ до реплик приложений из п.1.
5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.

Ответ:
1. Готовим манифест Deployment deployment.yml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment
  labels:
    app: main
spec:
  replicas: 1
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 8080
      - name: network-multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8085
        env:
          - name: HTTP_PORT
            value: "8080"
```

2. Колличество подов до масштабирования
![alt text](https://github.com/Mars12121/kuber-homeworks_1.3/blob/main/img/1.png)

Обновляем манифест deployment.yml с увеличением реплик до 2
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment
  labels:
    app: main
spec:
  replicas: 2
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 8080
      - name: network-multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8085
        env:
          - name: HTTP_PORT
            value: "8080"
```

Получаем количество подов 2
![alt text](https://github.com/Mars12121/kuber-homeworks_1.3/blob/main/img/2.png)

3. Создать Service service.ylm
```
apiVersion: v1
kind: Service
metadata:
  name: svc
spec:
  ports:
  - name: svc
    port: 80
  selector:
    app: main
```
![alt text](https://github.com/Mars12121/kuber-homeworks_1.3/blob/main/img/3.png)

4. Создать отдельный Pod с приложением multitool multitool.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: multitool
  labels:
    app: main-mt
spec:
  containers:
  - name: network-multitool
    image: wbitt/network-multitool
    env:
      - name: HTTP_PORT
        value: "8080"
```

![alt text](https://github.com/Mars12121/kuber-homeworks_1.3/blob/main/img/4.png)

------

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
3. Создать и запустить Service. Убедиться, что Init запустился.
4. Продемонстрировать состояние пода до и после запуска сервиса.

Ответ:
1. Создаем Deployment приложени nginx deploy-nginx.yml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netology-deployment-ng
  labels:
    app: main-ng
spec:
  replicas: 1
  selector:
    matchLabels:
      app: main-ng
  template:
    metadata:
      labels:
        app: main-ng
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      initContainers:
      - name: busybox
        image: busybox:1.28
        command: ['sh', '-c', "until nslookup nginx-svc-ng.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for nginx-svc-ng; sleep 2; done"]
```

Под не стартует
![alt text](https://github.com/Mars12121/kuber-homeworks_1.3/blob/main/img/5.png)

2. Создаем сервис servise-nginx.yml
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc-ng
spec:
  ports:
  - name: nginx-svc
    port: 80
  selector:
    app: main-ng
```

3. Проверяем состояния пода после запуска сервиса
![alt text](https://github.com/Mars12121/kuber-homeworks_1.3/blob/main/img/6.png)

------

### Правила приема работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать файлы манифестов и ссылки на них в файле README.md.

------
