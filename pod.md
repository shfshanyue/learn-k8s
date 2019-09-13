---
title: 应用

---

## pod

`pod` 是 `kubernetes` 中最小的编排单位，通常由一个容器组成 (有时候会有多个容器组成)。

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
```

``` shell
$ kubectl apply -f nginx.yaml
pod/nginx created

# -o wide 列出IP/Node等更多信息
$ kubectl get pods nginx -o wide -l 'app=nginx'
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          14m   10.244.1.9   shuifeng   <none>           <none>

# 每个 pod 都有一个IP地址，直接访问IP地址获取内容
$ curl 10.244.1.9
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

``` shell
$ kubectl exec -it nginx sh
```

``` shell
# 在 POD 中执行命令

$ netstat -tan
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN

$ wget -q -O - localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## deployment

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

``` shell
$ kubectl get pods -o wide -l 'app=nginx'
NAME                                READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
nginx                               1/1     Running   1          4h29m   10.244.1.9    shuifeng   <none>           <none>
nginx-deployment-54f57cf6bf-57g8l   1/1     Running   0          23m     10.244.1.10   shuifeng   <none>           <none>
nginx-deployment-54f57cf6bf-ltdf7   1/1     Running   0          23m     10.244.1.11   shuifeng   <none>           <none>
nginx-deployment-54f57cf6bf-n8ppt   1/1     Running   0          23m     10.244.1.12   shuifeng   <none>           <none>
```

## service

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

``` shell
$ kubectl get svc nginx-service  -o wide
NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE   SELECTOR
nginx-service   ClusterIP   10.108.9.49   <none>        80/TCP    11m   app=nginx

$ curl 10.108.9.49
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

+ ClusterIP
+ NodePort
+ LoadBalancer
+ ExternalName

