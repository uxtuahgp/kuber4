## Задание 1 - Настройка Service (ClusterIP и NodePort)

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
  

