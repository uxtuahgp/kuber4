## Домашнее задание по теме Сетевое взаимодействие в Kubernetes ##  
  
### Задание 1 - Настройка Service (ClusterIP и NodePort) ###  

1. Создан и применен [манифест развертывания двух контейнеров](deployment.yml).  
```
alex@uxtu-note:~/Study/kuber4/kuber4$ kubectl apply -f deployment.yml -n dep-ns
deployment.apps/dep-app configured
alex@uxtu-note:~/Study/kuber4/kuber4$ kubectl get pods -n dep-ns
NAME                        READY   STATUS    RESTARTS   AGE
dep-app-76cd889cd4-gtwtr    2/2     Running   0          110m
dep-app-76cd889cd4-m66jg    2/2     Running   0          107m
dep-app-76cd889cd4-mt6nc    2/2     Running   0          8s
dep2-app-8664778d65-2frz6   1/1     Running   0          32m
multitool-pod               1/1     Running   0          87m
alex@uxtu-note:~/Study/kuber4/kuber4$ kubectl get deployments -n dep-ns
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
dep-app    3/3     3            3           111m
dep2-app   1/1     1            1           33m
alex@uxtu-note:~/Study/kuber4/kuber4$ kubectl get replicasets -n dep-ns
NAME                  DESIRED   CURRENT   READY   AGE
dep-app-76cd889cd4    3         3         3       111m
dep2-app-8664778d65   1         1         1       33m
```  
  
2. Создан и применен [манифест сервиса](service.yml) для публикации портов подов с типом сервиса по-умолчанию (ClusterIP).  
```
alex@uxtu-note:~/Study/kuber4/kuber4$ kubectl apply -f service.yml -n dep-ns
service/dep-service configured
alex@uxtu-note:~/Study/kuber4/kuber4$ kubectl get svc -n dep-ns
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
dep-service    ClusterIP   10.152.183.221   <none>        9001/TCP,9002/TCP   108m
dep2-service   ClusterIP   10.152.183.36    <none>        10082/TCP           41m
```  
  
3. Проверена доступность сервиса изнутри кластера:  
```
alex@uxtu-note:~/Study/kuber4/kuber4$ kubectl run test-pod -n dep-ns --image=wbitt/network-multitool --rm -it -- bash 
If you don't see a command prompt, try pressing enter.
test-pod:/# curl dep-service:9001
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
test-pod:/# curl dep-service:9002
WBITT Network MultiTool (with NGINX) - dep-app-76cd889cd4-gtwtr - 10.1.69.204 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)

```  
  
4. Создал [манифест сервиса типа NodePort](service-nodeport.yml) для публикации сервисов подов на внешних интерфейсах узлов K8s  
5. Проверил доступность сервисов с локального компьютера:  
```
alex@uxtu-note:~/Study/kuber4/kuber4$ kubectl apply -f service-nodeport.yml -n dep-ns
service/dep-service-np created
alex@uxtu-note:~/Study/kuber4/kuber4$ kubectl get svc -o wide -n dep-ns
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                       AGE    SELECTOR
dep-service      ClusterIP   10.152.183.221   <none>        9001/TCP,9002/TCP             120m   app=dep-app
dep-service-np   NodePort    10.152.183.105   <none>        80:30101/TCP,8080:30102/TCP   22s    app=dep-app
dep2-service     ClusterIP   10.152.183.36    <none>        10082/TCP                     54m    app=dep2-app
alex@uxtu-note:~/Study/kuber4/kuber4$ curl localhost:30101
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
alex@uxtu-note:~/Study/kuber4/kuber4$ curl localhost:30102
WBITT Network MultiTool (with NGINX) - dep-app-76cd889cd4-gtwtr - 10.1.69.204 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
```  
  
### Задание 2: Настройка Ingress ###  
  
1. Создал манифесты для развертывания [frontend](task2/deployment-frontend.yml) и [backend](task2/deployment-backend.yml)  
```
alex@uxtu-note:~/Study/kuber4/kuber4/task2$ kubectl apply -f deployment-frontend.yml -n dep-ns
deployment.apps/dep-front created
alex@uxtu-note:~/Study/kuber4/kuber4/task2$ kubectl apply -f deployment-backend.yml -n dep-ns
deployment.apps/dep-back created
alex@uxtu-note:~/Study/kuber4/kuber4/task2$ kubectl get pods -o wide -n dep-ns
NAME                         READY   STATUS    RESTARTS   AGE    IP            NODE        NOMINATED NODE   READINESS GATES
dep-app-76cd889cd4-gtwtr     2/2     Running   0          159m   10.1.69.204   uxtu-note   <none>           <none>
dep-app-76cd889cd4-m66jg     2/2     Running   0          156m   10.1.69.208   uxtu-note   <none>           <none>
dep-app-76cd889cd4-mt6nc     2/2     Running   0          48m    10.1.69.223   uxtu-note   <none>           <none>
dep-back-5df4764697-g8z5l    1/1     Running   0          71s    10.1.69.237   uxtu-note   <none>           <none>
dep-back-5df4764697-njmlw    1/1     Running   0          71s    10.1.69.236   uxtu-note   <none>           <none>
dep-front-644c7d7796-cv7t6   1/1     Running   0          81s    10.1.69.233   uxtu-note   <none>           <none>
dep-front-644c7d7796-zl727   1/1     Running   0          81s    10.1.69.231   uxtu-note   <none>           <none>
dep2-app-8664778d65-2frz6    1/1     Running   0          81m    10.1.69.224   uxtu-note   <none>           <none>
multitool-pod                1/1     Running   0          135m   10.1.69.229   uxtu-note   <none>           <none>
```  
  
2. Создал манифесты для сервисов [frontend](task2/service-frontend.yml) и [backend](task2/service-backend.yml)  
```
alex@uxtu-note:~/Study/kuber4/kuber4/task2$ kubectl apply -f service-frontend.yml -n dep-ns
service/front-service created
alex@uxtu-note:~/Study/kuber4/kuber4/task2$ kubectl apply -f service-backend.yml -n dep-ns
service/back-service created
alex@uxtu-note:~/Study/kuber4/kuber4/task2$ kubectl get svc -n dep-ns -o wide
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                       AGE    SELECTOR
back-service     ClusterIP   10.152.183.54    <none>        9001/TCP                      23s    app=dep-back
dep-service      ClusterIP   10.152.183.221   <none>        9001/TCP,9002/TCP             149m   app=dep-app
dep-service-np   NodePort    10.152.183.105   <none>        80:30101/TCP,8080:30102/TCP   29m    app=dep-app
dep2-service     ClusterIP   10.152.183.36    <none>        10082/TCP                     83m    app=dep2-app
front-service    ClusterIP   10.152.183.250   <none>        9001/TCP                      31s    app=dep-front
```  
  
3. Включил ingress controller  для Microk8s  
```
alex@uxtu-note:~/Study/kuber4/kuber4/task2$  microk8s enable ingress
Infer repository core for addon ingress
Enabling Ingress
ingressclass.networking.k8s.io/public created
ingressclass.networking.k8s.io/nginx created
namespace/ingress created
serviceaccount/nginx-ingress-microk8s-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-microk8s-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-microk8s-role created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
configmap/nginx-load-balancer-microk8s-conf created
configmap/nginx-ingress-tcp-microk8s-conf created
configmap/nginx-ingress-udp-microk8s-conf created
daemonset.apps/nginx-ingress-microk8s-controller created
Ingress is enabled
```  
  
4. Создал и применил [ingress для маршрутизации запросов на frontend и backend](task2/ingress.yml)  
```  
lex@uxtu-note:~/Study/kuber4/kuber4/task2$ kubectl apply -f ingress.yml -n dep-ns
ingress.networking.k8s.io/back-front-ingress created
alex@uxtu-note:~/Study/kuber4/kuber4/task2$ kubectl describe ingress -n dep-ns
Name:             back-front-ingress
Labels:           <none>
Namespace:        dep-ns
Address:          127.0.0.1
Ingress Class:    public
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /      front-service:9001 (10.1.69.231:80,10.1.69.233:80)
              /api   back-service:9001 (10.1.69.236:8080,10.1.69.237:8080)
Annotations:  <none>
Events:
  Type    Reason  Age                 From                      Message
  ----    ------  ----                ----                      -------
  Normal  Sync    94s (x2 over 2m9s)  nginx-ingress-controller  Scheduled for sync
  alex@uxtu-note:~/Study/kuber4/kuber4/task2$ kubectl describe svc back-service -ndep-ns
  Name:                     back-service
  Namespace:                dep-ns
  Labels:                   <none>
  Annotations:              <none>
  Selector:                 app=dep-back
  Type:                     ClusterIP
  IP Family Policy:         SingleStack
  IP Families:              IPv4
  IP:                       10.152.183.54
  IPs:                      10.152.183.54
  Port:                     multitool-port  9001/TCP
  TargetPort:               8080/TCP
  Endpoints:                10.1.69.236:8080,10.1.69.237:8080
  Session Affinity:         None
  Internal Traffic Policy:  Cluster
  Events:                   <none>
  alex@uxtu-note:~/Study/kuber4/kuber4/task2$ kubectl describe svc front-service -ndep-ns
  Name:                     front-service
  Namespace:                dep-ns
  Labels:                   <none>
  Annotations:              <none>
  Selector:                 app=dep-front
  Type:                     ClusterIP
  IP Family Policy:         SingleStack
  IP Families:              IPv4
  IP:                       10.152.183.250
  IPs:                      10.152.183.250
  Port:                     nginx-port  9001/TCP
  TargetPort:               80/TCP
  Endpoints:                10.1.69.231:80,10.1.69.233:80
  Session Affinity:         None
  Internal Traffic Policy:  Cluster
  Events:                   <none>
  alex@uxtu-note:~/Study/kuber4/kuber4/task2$ curl localhost/
  <!DOCTYPE html>
  <html>
  <head>
  <title>Welcome to nginx!</title>
  <style>
  html { color-scheme: light dark; }
  body { width: 35em; margin: 0 auto;
  font-family: Tahoma, Verdana, Arial, sans-serif; }
  </style>
  </head>
  <body>
  <h1>Welcome to nginx!</h1>
  <p>If you see this page, nginx is successfully installed and working.
  Further configuration is required for the web server, reverse proxy, 
  API gateway, load balancer, content cache, or other features.</p>
  
  <p>For online documentation and support please refer to
  <a href="https://nginx.org/">nginx.org</a>.<br/>
  To engage with the community please visit
  <a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
  For enterprise grade support, professional services, additional 
  security features and capabilities please refer to
  <a href="https://f5.com/nginx">f5.com/nginx</a>.</p>
  
  <p><em>Thank you for using nginx.</em></p>
  </body>
  </html>
  alex@uxtu-note:~/Study/kuber4/kuber4/task2$ curl localhost/api
  <html>
  <head><title>404 Not Found</title></head>
  <body>
  <center><h1>404 Not Found</h1></center>
  <hr><center>nginx/1.28.0</center>
  </body>
  </html>
```  
  
  
