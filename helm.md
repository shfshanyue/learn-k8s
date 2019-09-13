# 使用 helm 管理 kubernetes 集群

`helm` 之于 `kubernetes` 就如 `yum` 之于 `centos`，`pip` 之于 `python`，`npm` 之于 `javascript`。

`helm` 分为客户端与服务端两部分，在服务端又叫 `Tiller`。

那 `helm` 的引入对于管理集群有哪些帮助呢？

1. 更方便地安装 postgres/elk/grafana/gitlab/prometheus 等工具
1. 更方便地管理自己的应用

这里参考文档： [安装 helm](https://helm.sh/docs/using_helm/#installing-helm)

## 安装客户端

安装 `Tiller` 也需要客户端支持，所以客户端安装在本地与master 节点上。

在 mac 上进行安装

``` shell
$ brew install kubernetes-helm
```

在 linux 上进行安装

``` shell
$ curl -LO https://git.io/get_helm.sh
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

> 使用脚本安装在国内会出现网络问题，需要在代理节点安装并移动到目标位置

根据源码包进行安装，比较推荐(毕竟使用以上两种方案可能有网络问题)

``` shell
# 在 mac 上，本次安装的
# -L: 追踪重定向链接
# -O: 保存到本地
# -S: 打印错误
$ curl -SLO https://get.helm.sh/helm-v2.14.3-darwin-amd64.tar.gz 

# 在 centos 上
$ curl -SLO https://get.helm.sh/helm-v2.14.3-linux-amd64.tar.gz

# 如果有网络问题，请在代理节点下载并 rsync 到目标节点，如果没有，跳过此步
$ rsync -avhzP proxy:/root/helm-v2.14.3-linux-amd64.tar.gz .

# 如果在 mac 上
$ tar -zxvf helm-v2.14.3-darwin-amd64.tar.gz 

# 如果在 centos 上
$ tar -zxvf helm-v2.14.3-linux-amd64.tar.gz

# 进入相应目录，移至 /bin 目录
$ mv linux-amd64/helm /usr/local/bin/helm
```

## 安装服务端: tiller

### 下载镜像

tiller 的镜像：`gcr.io/kubernetes-helm/tiller:v2.14.3` 也在 gcr.io 上，这意味着在国内网络需要先下载到代理节点，再移动到目标位置

### 安装 tiller

当安装好 `helm` 命令行工具后，使用 `helm init` 安装 tiller

``` shell
$ helm init
Creating /root/.helm
Creating /root/.helm/repository
Creating /root/.helm/repository/cache
Creating /root/.helm/repository/local
Creating /root/.helm/plugins
Creating /root/.helm/starters
Creating /root/.helm/cache/archive
Creating /root/.helm/repository/repositories.yaml
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com
Adding local repo with URL: http://127.0.0.1:8879/charts
$HELM_HOME has been configured at /root/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation

# 查看 tiller 是否出在运行状态
$ kubectl get pods --all-namespaces

# 查看 helm 与 tiller 版本
$ helm version
Client: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}
```

## RBAC

关于更多文档可以查看官方文档 [RBAC](https://helm.sh/docs/using_helm/#role-based-access-control) 或者以上章节

``` yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

## 参考链接

+ [helm 官方文档](https://helm.sh/docs/)

## 编辑

## 使用 helm 搭建 postgres

> 搭建 postgres 时确保服务器有充足的磁盘空间

``` shell
$ helm search postgres
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION
stable/postgresql                       6.3.6           11.5.0          Chart for PostgreSQL, an object-relational database manag...
stable/prometheus-postgres-exporter     0.7.2           0.5.1           A Helm chart for prometheus postgres-exporter
stable/stolon                           1.1.2           0.13.0          Stolon - PostgreSQL cloud native High Availability.
stable/gcloud-sqlproxy                  0.6.1           1.11            DEPRECATED Google Cloud SQL Proxy
$ helm install stable/postgresql
```

## 使用 helm 部署应用

helm 对于 k8s 有两大用处

1. 更快地搭建基础设施，如 es/postgres/redis/grafana/prometheus
1. 更快地部署应用

``` shell
# 创建一个 chart
$ helm create todo
Creating todo

$ cd todo

# 打印 chart 目录，主要文件有 Chart.yaml 与 values.yaml
# --dirsfirst 先打印文件夹名称
$ tree --dirsfirst
.
├── charts
├── templates
│   ├── tests
│   │   └── test-connection.yaml
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── ingress.yaml
│   └── service.yaml
├── Chart.yaml
└── values.yaml

3 directories, 8 files

# 查看主要的两个文件 Chart.yaml 与 values.yaml 内容

$ cat Chart.yaml
apiVersion: v1
appVersion: "1.0"
description: A Helm chart for Kubernetes
name: helm
version: 0.1.0

$ cat values.yaml
# Default values for helm.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: nginx
  tag: stable
  pullPolicy: IfNotPresent

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths: []

  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

nodeSelector: {}

tolerations: []

affinity: {}
```

