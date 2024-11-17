**Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера**

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.

2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.

3. Создать отдельный Pod с приложением multitool и убедиться с помощью curl, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.

4. Продемонстрировать доступ с помощью curl по доменному имени сервиса.

5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.




**Решение 1**

Создадим файл деплоймента

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: dep-multitool
  name: dep-multitool
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: multitool
  template:
    metadata:
      labels:
        app: multitool
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool:latest
        ports:
        - containerPort: 8080
        env:
        - name: HTTP_PORT
          value: "8080"
```

![Image alt](https://github.com/mezhibo/kubernetes4/blob/c2e77f203614557ece29ee3d2c99e977527fde08/IMG/1.jpg)

Далее создадим Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080

```
apiVersion: v1
kind: Service
metadata:
  name: svc-multitool
spec:
  selector:
    app: multitool
  ports:
    - name: for-nginx
      port: 9001
      targetPort: 80
    - name: for-multitool
      port: 9002
      targetPort: 8080
```

![Image alt](https://github.com/mezhibo/kubernetes4/blob/c2e77f203614557ece29ee3d2c99e977527fde08/IMG/2.jpg)



Теперь создаем отдельный Pod с приложением multitool и проверяем с помощью curl, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры 


```
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: pod-multitool
  name: pod-multitool
  namespace: default
spec:
  containers:
  - name: multitool
    image: wbitt/network-multitool:latest
    ports:
      - containerPort: 8090
        name: multitool-port
```


![Image alt](https://github.com/mezhibo/kubernetes4/blob/c2e77f203614557ece29ee3d2c99e977527fde08/IMG/3.jpg)


Теперь изнутрки контейнера через curl проверим подключение

![Image alt](https://github.com/mezhibo/kubernetes4/blob/c2e77f203614557ece29ee3d2c99e977527fde08/IMG/4.jpg)

![Image alt](https://github.com/mezhibo/kubernetes4/blob/c2e77f203614557ece29ee3d2c99e977527fde08/IMG/5.jpg)


![Image alt](https://github.com/mezhibo/kubernetes4/blob/c2e77f203614557ece29ee3d2c99e977527fde08/IMG/6.jpg)


![Image alt](https://github.com/mezhibo/kubernetes4/blob/c2e77f203614557ece29ee3d2c99e977527fde08/IMG/7.jpg)


Проверяем доступ с помощью curl по доменному имени сервиса

![Image alt](https://github.com/mezhibo/kubernetes4/blob/c2e77f203614557ece29ee3d2c99e977527fde08/IMG/8.jpg)



**Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера**

1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.

2. Продемонстрировать доступ с помощью браузера или curl с локального компьютера.

3. Предоставить манифест и Service в решении, а также скриншоты или вывод команды п.2.


**Решение 2**


Создадим сервис service-dep-nodeport.yaml


```
apiVersion: v1
kind: Service
metadata:
  name: svc-multitool-nodeport
spec:
  selector:
    app: multitool
  ports:
    - name: for-nginx
      nodePort: 30500
      targetPort: 80
      port: 80
    - name: for-multitool
      nodePort: 30501
      port: 8080
      targetPort: 8080
  type: NodePort
```

![Image alt](https://github.com/mezhibo/kubernetes4/blob/9fcdbac7e0550f31c19a0e442089715a1a68d49c/IMG/9.jpg)


Переходим в браузер и проверяем, видим что у нас приложение пробросилось

![Image alt](https://github.com/mezhibo/kubernetes4/blob/9fcdbac7e0550f31c19a0e442089715a1a68d49c/IMG/10.jpg)

![Image alt](https://github.com/mezhibo/kubernetes4/blob/9fcdbac7e0550f31c19a0e442089715a1a68d49c/IMG/11.jpg)
