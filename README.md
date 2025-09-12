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

## 5.10. Домашнее задание - Работа с объектами

```bash
$ cat app-deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: conv-app-deployment
spec:
  replicas: 1 
  selector:
    matchLabels:
      components: frontend
  template:
    metadata:
      name: conv-app
      labels:
        components: frontend
    spec:
      containers:
        - name: conv-app
          image: antonlarichev/conv-app:1.2
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
          resources:
            limits:
              memory: "512Mi"
              cpu: "500m"

vg m22: ~/purplescool_kube/ps-kube-hw git(2-deployment)
$ kaf app-deployment.yaml 
deployment.apps/conv-app-deployment created

$ kaf node-port.yml 
service/conv-app-port created

$ kgall
NAME                                      READY   STATUS    RESTARTS   AGE
pod/conv-app-deployment-57474565c-gxvc4   1/1     Running   0          15s

NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/conv-app-port   NodePort    10.109.127.137   <none>        3000:31200/TCP   3s
service/kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP          45h

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/conv-app-deployment   1/1     1            1           15s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/conv-app-deployment-57474565c   1         1         1       15s


$ m service conv-app-port --url
http://192.168.49.2:31200
```

```html
$ curl http://192.168.49.2:31200
<!doctype html>
<html lang="en">
  <head>
    <script type="module">import { injectIntoGlobalHook } from "/@react-refresh";
injectIntoGlobalHook(window);
window.$RefreshReg$ = () => {};
window.$RefreshSig$ = () => (type) => type;</script>

    <script type="module" src="/@vite/client"></script>

    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite + React + TS</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

## 6.8. Домашнее задание - Работа с сетью

```
добавил в /etc/hosts
192.168.49.2 conv.test

vg m22: ~/purplescool_kube/ps-kube-hw git(2-deployment)

$ cat app-service.yaml 
apiVersion: v1
kind: Service
metadata:
  name: conv-app-clusterip
spec:
  type: ClusterIP
  ports:
    - port: 80
      protocol: TCP
  selector:
    components: frontend

$ cat ingress.yaml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myingress
  annotations:
    nginx.ingress.kubernetis.io/add-base-url: "true"
spec:
  ingressClassName: nginx
  rules:
  - host: conv.test
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: conv-app-clusterip
            port:
              number: 80

$ kaf app-deployment.yaml 
$ kaf app-service.yaml
$ kaf ingress.yaml

$ kgall
NAME                                      READY   STATUS        RESTARTS      AGE
pod/conv-app-deployment-d986d8548-hr4s6   1/1     Terminating   1 (42m ago)   77m
pod/conv-app-deployment-d986d8548-pkjdg   1/1     Running       1 (42m ago)   77m

NAME                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/conv-app-clusterip   ClusterIP   10.97.50.77      <none>        80/TCP           76m
service/conv-app-port        NodePort    10.107.248.181   <none>        3000:31200/TCP   21m
service/kubernetes           ClusterIP   10.96.0.1        <none>        443/TCP          13h

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/conv-app-deployment   1/1     1            1           77m

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/conv-app-deployment-d986d8548   1         1         1       77m

$ kgin
NAME        CLASS   HOSTS       ADDRESS   PORTS   AGE
myingress   nginx   conv.test             80      6s

$ curl conv.test
<!doctype html>
<html lang="en">
  <head>
    <script type="module">import { injectIntoGlobalHook } from "/@react-refresh";
injectIntoGlobalHook(window);
window.$RefreshReg$ = () => {};
window.$RefreshSig$ = () => (type) => type;</script>

    <script type="module" src="/@vite/client"></script>

    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite + React + TS</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```



