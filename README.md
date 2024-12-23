# Домашнее задание к занятию «Конфигурация приложений»

### Цель задания

В тестовой среде Kubernetes необходимо создать конфигурацию и продемонстрировать работу приложения.

------

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8s).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым GitHub-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/configuration/secret/) Secret.
2. [Описание](https://kubernetes.io/docs/concepts/configuration/configmap/) ConfigMap.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment приложения и решить возникшую проблему с помощью ConfigMap. Добавить веб-страницу

1. Создать Deployment приложения, состоящего из контейнеров nginx и multitool.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-multitool
  labels:
    app: nginx-multi
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-multi
  template:
    metadata:
      labels:
        app: nginx-multi
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-html
          mountPath: /usr/share/nginx/html
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
        env:
          - name: HTTP_PORT
            valueFrom:
              configMapKeyRef:
                name: myconfigmap
                key: value
      volumes:
      - name: nginx-html
        configMap:
          name: myconfigmap
          items:
          - key: "index.html"
            path: "index.html"
```

Добавил configmap:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap
data:
  value: "9080"
  index.html: |
    <html>
    <body>
      <h1>"Домашнее задание к занятию «Конфигурация приложений»"</h1>
    </body>
    </html>
```

Добавил service:

```
apiVersion: v1
kind: Service
metadata:
  name: svc-ngmu
spec:
  selector:
    app: nginx-multi
  ports:
    - name: nginx
      protocol: TCP
      port: 80
      targetPort: 80
    - name: multitool
      protocol: TCP
      port: 8080
      targetPort: 9080
  type: ClusterIP
```

```
user@k8s:/opt/hw_k8s_8$ microk8s kubectl apply -f deployment.yml
deployment.apps/nginx-multitool created
user@k8s:/opt/hw_k8s_8$ microk8s kubectl apply -f configmap.yml
configmap/myconfigmap created
user@k8s:/opt/hw_k8s_8$ microk8s kubectl apply -f svc.yml
service/svc-ngmu created
```

2. Решить возникшую проблему с помощью ConfigMap.
3. Продемонстрировать, что pod стартовал и оба конейнера работают.

```
user@k8s:/opt/hw_k8s_8$ microk8s kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
nginx-multitool-64f7fc55d5-82c4v   2/2     Running   0          86s
user@k8s:/opt/hw_k8s_8$ microk8s kubectl get service
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP           33d
svc-ngmu     ClusterIP   10.152.183.178   <none>        80/TCP,8080/TCP   87s
user@k8s:/opt/hw_k8s_8$ microk8s kubectl get configmap
NAME               DATA   AGE
kube-root-ca.crt   1      33d
myconfigmap        2      111s
```

4. Сделать простую веб-страницу и подключить её к Nginx с помощью ConfigMap. Подключить Service и показать вывод curl или в браузере.

```
user@k8s:/opt/hw_k8s_8$ microk8s kubectl exec -ti nginx-multitool-64f7fc55d5-82c4v -c multitool -- bash
nginx-multitool-64f7fc55d5-82c4v:/# curl 127.0.0.1:9080
WBITT Network MultiTool (with NGINX) - nginx-multitool-64f7fc55d5-82c4v - 10.1.77.44 - HTTP: 9080 , HTTPS: 443 . (Formerly praqma/network-multitool)
nginx-multitool-64f7fc55d5-82c4v:/# curl 127.0.0.1
<html>
<body>
  <h1>"Домашнее задание к занятию «Конфигурация приложений»"</h1>
</body>
</html>
nginx-multitool-64f7fc55d5-82c4v:/# 
```

5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

[deployment.yml](https://github.com/stepynin-georgy/hw_k8s_8/blob/main/deployment.yml)

[configmap.yml](https://github.com/stepynin-georgy/hw_k8s_8/blob/main/configmap.yml)

[svc.yml](https://github.com/stepynin-georgy/hw_k8s_8/blob/main/svc.yml)

------

### Задание 2. Создать приложение с вашей веб-страницей, доступной по HTTPS 

1. Создать Deployment приложения, состоящего из Nginx.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-https-ssl
  labels:
    app: nginx-hs
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-hs
  template:
    metadata:
      labels:
        app: nginx-hs
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-html
          mountPath: /usr/share/nginx/html
      volumes:
      - name: nginx-html
        configMap:
          name: myconfigmap
          items:
          - key: "index.html"
            path: "index.html"
```

Добавил configmap:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap
data:
  index.html: |
    <html>
    <body>
      <h1>"Домашнее задание к занятию «Конфигурация приложений» - Задание 2 - Мурчин Артем"</h1>
    </body>
    </html>
```

Добавил service:

```
apiVersion: v1
kind: Service
metadata:
  name: svc-ng
spec:
  selector:
    app: nginx-hs
  ports:
    - name: nginx
      protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

Добавил ingress:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
    - hosts:
      - stg-test.ru
      secretName: web-tls
  rules:
  - host: stg-test.ru
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-ng
            port:
              number: 80
```

```
user@k8s:/opt/hw_k8s_8$ for i in `ls *-2.yml`; do microk8s kubectl apply -f $i; done
configmap/myconfigmap created
deployment.apps/nginx-https-ssl created
ingress.networking.k8s.io/my-ingress created
service/svc-ng created
user@k8s:/opt/hw_k8s_8$
```

2. Создать собственную веб-страницу и подключить её как ConfigMap к приложению.
3. Выпустить самоподписной сертификат SSL. Создать Secret для использования сертификата.

```
user@k8s:/opt/hw_k8s_8$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=stg-test.ru/0=stg-test.ru"
.+.+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+...............+..+.......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..+...+........+...+.......+...+.........+............+...+...+............+..+...+.+.....+.......+...+.....+.+.....+...................+.........+..+.+.........+........+......+................+..............+.......+...............+........+.......+...+..+...+...................+.....+.......+......+...+..............+.......+.....+...+......+.+........+...+....+...+........+...+......+.+....................+...+..........+......+.........+.....+.+.....+.+.....+...+......+.+........+....+...+...............+.....+............+.+..+............+...+...+.+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
....................+....+........+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*....+....+..............+....+...+..+.+.........+...............+.....+.+.....+.+......+..+.+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.......+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
req warning: Skipping unknown subject name attribute "0"
```

```
user@k8s:/opt/hw_k8s_8$ sudo microk8s kubectl create secret tls web-tls --cert=tls.crt --key=tls.key
secret/web-tls created
user@k8s:/opt/hw_k8s_8$  microk8s kubectl get secret
NAME      TYPE                DATA   AGE
web-tls   kubernetes.io/tls   2      20s
user@k8s:/opt/hw_k8s_8$ microk8s kubectl describe secret web-tls
Name:         web-tls
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/tls

Data
====
tls.key:  1704 bytes
tls.crt:  1119 bytes
```

4. Создать Ingress и необходимый Service, подключить к нему SSL в вид. Продемонстировать доступ к приложению по HTTPS.

```
user@k8s:/opt/hw_k8s_8$ curl https://stg-test.ru -k
<html>
<body>
  <h1>"Домашнее задание к занятию «Конфигурация приложений» - Задание 2"</h1>
</body>
</html>
```

4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

[deployment-2.yml](https://github.com/stepynin-georgy/hw_k8s_8/blob/main/deployment-2.yml)

[configmap-2.yml](https://github.com/stepynin-georgy/hw_k8s_8/blob/main/configmap-2.yml)

[svc-2.yml](https://github.com/stepynin-georgy/hw_k8s_8/blob/main/svc-2.yml)

[ingress-2.yml](https://github.com/stepynin-georgy/hw_k8s_8/blob/main/ingress-2.yml)

------

### Правила приёма работы

1. Домашняя работа оформляется в своём GitHub-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------
