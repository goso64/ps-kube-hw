# ps-kube-hw
for purpleschool kube homeworks

## 10.8. Домашнее задание - Знакомство с Helm

Пустота, заготовка для заполнения на последующих уроках.

## 11.9. Домашнее задание - Шаблоны

Делаем шаблон для conv-app:
- deployment
- configMap
- clusterip
- ingress

```
$ helm install --dry-run converter .
NAME: converter
LAST DEPLOYED: Mon Oct 13 03:41:17 2025
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: converter/templates/vitedomain-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vitedomain-config
data:
  VITE_DOMAIN: "https://conv.test"
---
# Source: converter/templates/conv-app-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: "conv-app-clusterip"
spec:
  type: ClusterIP
  ports:
    - port: 80
      protocol: TCP
  selector:
    components: frontend
---
# Source: converter/templates/conv-app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: "conv-app-deployment"
spec:
  replicas: 1
  selector:
    matchLabels:
      components: frontend
  template:
    metadata:
      labels:
        components: frontend
    spec:
      containers:
        - name: conv-app
          image: "antonlarichev/conv-app:1.2"
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
          resources:
            limits: 
              cpu: 250m
              memory: 512Mi
          env:
            - name: VITE_DOMAIN
              valueFrom:
                configMapKeyRef:
                  name: vitedomain-config
                  key: VITE_DOMAIN
---
# Source: converter/templates/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: converter
  annotations:
    nginx.ingress.kubernetes.io/add-base-url: "true"
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
            name: "conv-app-clusterip"
            port:
              number: 80

```
## 12.13. Домашнее задание - Продвинутые шаблоны

Добавил шаблоны для api converter mq
