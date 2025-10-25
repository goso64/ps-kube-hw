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

```
vg m22: ~/purplescool_kubehelm/ps-kube-hw git(5-helm-final)
$ cat templates/conv-converter-deployment.yaml
apiVersion: apps/v1
kind: Deployment
{{- with .Values.converter }}
metadata:
  name: {{ .name }}-deployment
spec:
  replicas: {{ .replicas }}
  selector:
    matchLabels:
      components: {{ .components }}
  template:
    metadata:
      name: {{ .name }}
      labels:
        components: {{ .components }}
    spec:
      containers:
        - name: {{ .name }}
          image: "{{ .image.name }}:{{ .image.tag }}"
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: {{ .port }}
          resources:
            limits: {{ .limits | toYaml | nindent 14 }}
          env:
            {{- $rabbit_name := $.Values.rabbit.name }}
            {{- $rabbit_secret := cat $rabbit_name "-secret" | nospace }}
            - name: AMQP_EXCHANGE
              value: {{ $.Values.rabbit.amqp_exchange }}
            - name: AMQP_HOSTNAME
              value: {{ $rabbit_name }}-clusterip
            - name: AMQP_QUEUE
              value: "converter"
            - name: AMQP_USER
              valueFrom:
                secretKeyRef:
                  name: {{ $rabbit_secret }}
                  key: SEED_USERNAME
            - name: AMQP_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ $rabbit_secret }}
                  key: SEED_USER_PASSWORD
          volumeMounts:
            - name: {{ .name }}-data
              mountPath: /opt/app/uploads
      volumes:
        - name: {{ .name }}-data
          persistentVolumeClaim:
            claimName: {{ .name }}-pvc
{{- end -}}

$ helm template . --debug --show-only templates/conv-converter-deployment.yaml
install.go:225: 2025-10-20 17:44:29.18302265 +0300 MSK m=+0.039689233 [debug] Original chart version: ""
install.go:242: 2025-10-20 17:44:29.183349982 +0300 MSK m=+0.040016455 [debug] CHART PATH: /home/vg/purplescool_kubehelm/ps-kube-hw

---
# Source: converter/templates/conv-converter-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: converter-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      components: converter
  template:
    metadata:
      name: converter
      labels:
        components: converter
    spec:
      containers:
        - name: converter
          image: "antonlarichev/conv-service:1.0"
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8000
          resources:
            limits: 
              cpu: 200m
              memory: 256Mi
          env:
            - name: AMQP_EXCHANGE
              value: convert
            - name: AMQP_HOSTNAME
              value: rabbitmq-clusterip
            - name: AMQP_QUEUE
              value: "converter"
            - name: AMQP_USER
              valueFrom:
                secretKeyRef:
                  name: rabbitmq-secret
                  key: SEED_USERNAME
            - name: AMQP_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: rabbitmq-secret
                  key: SEED_USER_PASSWORD
          volumeMounts:
            - name: converter-data
              mountPath: /opt/app/uploads
      volumes:
        - name: converter-data
          persistentVolumeClaim:
            claimName: converter-pvc
```

## 14.10. Домашнее задание - Использование Charts

- Добавляем Notes.txt
- Пишем тест на проверку работы APP приложения
- Шифруем секреты


