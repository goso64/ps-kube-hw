# ps-kube-hw
for purpleschool kube homeworks

## 3.11. Домашнее задание - Знакомство с Kubernetes

Необходимо описать в YML следующие объекты:

Пользователь:
Админ или нет
Имя
Возраст
Должность
Массив навыков
Место жительства
Массив домашних животных

Навык:
Название
Уровень владение от 1 до 5
Многострочное описание
Место жительство:
  Страна
  Город
  Улица
  Номер дома
  Номер квартиры

Домашнее животные
Название
Порода
Умеет ли мяукать да / нет

## 4.8. Домашнее задание - Первый pod

```
vg m22: ~/purplescool_kube/ps-kube-hw git(dz4.8)
$ cat pod.yml 
apiVersion: v1
kind: Pod
metadata:
  name: conv-app
  labels:
    components: frontend
spec:
  containers:
    - name: conv-app
      image: antonlarichev/conv-app:1.2
      ports:
        - containerPort: 80
      resources:
        limits:
          memory: "1000Mi"
          cpu: "500m"

$ cat node-port.yml 
apiVersion: v1
kind: Service
metadata:
  name: conv-app-port
spec:
  type: NodePort
  ports:
    - port: 3001
      targetPort: 80
      nodePort: 31201
  selector:
    components: frontend

$ k apply -f pod.yml
pod/conv-app created

$ k apply -f node-port.yml
service/conv-app-port created

$ k get pods
NAME        READY   STATUS    RESTARTS   AGE
conv-app    1/1     Running   0          5s

$ m service conv-app-port --url
http://192.168.49.2:31201

$ curl 192.168.49.2:31201
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Short App</title>
    <script type="module" crossorigin src="/assets/index-725ddef7.js"></script>
    <link rel="stylesheet" href="/assets/index-0c6c0b70.css">
  </head>
  <body>
    <div id="root"></div>
    
  </body>
</html>
```