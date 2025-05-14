---
title: Kubernetes_2
description: Kubernetes_2
date: 2025-04-30
slug: Kubernetes_2
image: 202412242125601.png
categories:
  - Kubernetes_2
---

# Kubernetes_2

## Service、Endpoint、Pod 之间的关系

> Service 负责东西流量、Ingress 负责南北流量

![202505122306614.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202505122306614.png)

> Service 根据选择器(比如 app=nginx-deploy)选中对应节点的 pod，如何找到对应 pod 的 IP 呢
> Endpoint 会对容器内 Pod 的 ip 进行管理
> 当执行 kubectl 命令操作时候，请求到了 master 的 api-server 上面，api-server 根据 ip 找到对应的 service，service 找到 endpoint，endpoint 根据 iptables 转发找到目标服务器的 kube-proxy，最终找到对应的 pod 容器

![202505122306617.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202505122306617.png)
新建一个`nginx-svc.yaml`

```yaml
apiVersion: v1
kind: Service # 资源类型为 Service
metadata:
  name: nginx-svc # Service 名字
  labels:
    app: nginx # Service 自己本身的标签
spec:
  selector: # 匹配哪些 pod 会被该 service 代理
    app: nginx-deploy # 所有匹配到这些标签的 pod 都可以通过该 service 进行访问
  ports: # 端口映射
    - port: 80 # service 自己的端口，在使用内网 ip 访问时使用
      targetPort: 80 # 目标 pod 的端口
      nodePort: 32000 # 固定绑定到所有 node 的 32000 端口上
      name: web # 为端口起个名字
  type:
    NodePort # 随机启动一个端口（30000~32767），映射到 ports 中的端口，该端口是直接绑定在 node 上的，且集群中的每一个 node 都会绑定这个端口
    # 也可以用于将服务暴露给外部访问，但是这种方式实际生产环境不推荐，效率较低，而且 Service 是四层负载
```

```YAML
apiVersion: apps/v1 # deployment api 版本
kind: Deployment # 资源类型为 deployment
metadata: # 元信息
  labels: # 标签
    app: nginx-deploy # 具体的 key: value 配置形式
  name: nginx-deploy # deployment 的名字
  namespace: default # 所在的命名空间
spec:
  replicas: 1 # 期望副本数
  revisionHistoryLimit: 10 # 进行滚动更新后，保留的历史版本数
  selector: # 选择器，用于找到匹配的 RS
    matchLabels: # 按照标签匹配
      app: nginx-deploy # 匹配的标签key/value
  strategy: # 更新策略
    rollingUpdate: # 滚动更新配置
      maxSurge: 25% # 进行滚动更新时，更新的个数最多可以超过期望副本数的个数/比例
      maxUnavailable: 25% # 进行滚动更新时，最大不可用比例更新比例，表示在所有副本数中，最多可以有多少个不更新成功
    type: RollingUpdate # 更新类型，采用滚动更新
  template: # pod 模板
    metadata: # pod 的元信息
      labels: # pod 的标签
        app: nginx-deploy
    spec: # pod 期望信息
      containers: # pod 的容器
      - image: nginx:1.7.9 # 镜像
        imagePullPolicy: IfNotPresent # 拉取策略
        name: nginx # 容器名称
        resources:
          limits:
            cpu: 200m
            memory: 128Mi
          requests:
            cpu: 100m
            memory: 128Mi
      restartPolicy: Always # 重启策略
      terminationGracePeriodSeconds: 30 # 删除操作最多宽限多长时间
```

```shell
root@k8s-master:/opt/k8s/service# kubectl create -f nginx-svc.yaml
service/nginx-svc created
root@k8s-master:/opt/k8s/service# kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        4d11h
nginx-svc    NodePort    10.97.20.225   <none>        80:32000/TCP   8s
# 创建一个deploy（包括pod）,这里使用的是nginx:1.7.9的镜像，如果没有请更换版本
root@k8s-master:/opt/k8s/deploy# kubectl create -f nginx-deploy.yaml
deployment.apps/nginx-deploy created
root@k8s-master:/opt/k8s/deploy# kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   1/1     1            1           108s
root@k8s-master:/opt/k8s/deploy# kubectl get po
NAME                           READY   STATUS    RESTARTS        AGE
dns-test                       1/1     Running   1 (2m45s ago)   12m
nginx-deploy-56696fbb5-lkgb2   1/1     Running   0               4m39s
# 新建一个busybox pod
root@k8s-master:~# kubectl run -it --image busybox:1.28.4 dns-test /bin/sh
# 如果本来就有这个pod的话
root@k8s-master:~# kubectl exec -it dns-test -- sh
/ # wget http://nginx-svc
Connecting to nginx-svc (10.97.20.225:80)
index.html           100% |**********************************************************************************************************************************************************************************|   615   0:00:00 ETA
# 查看一下内容
/ # cat index.html
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
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
/ #
# 查看一下10.97.20.225是指的哪个ip，这里显然是svc的ip
root@k8s-master:~# kubectl get po -o wide
NAME                           READY   STATUS    RESTARTS        AGE     IP               NODE         NOMINATED NODE   READINESS GATES
dns-test                       1/1     Running   1 (4m28s ago)   13m     10.244.235.197   k8s-master   <none>           <none>
nginx-deploy-56696fbb5-lkgb2   1/1     Running   0               6m22s   10.244.235.199   k8s-master   <none>           <none>
root@k8s-master:~# kubectl get svc -o wide
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE     SELECTOR
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        4d11h   <none>
nginx-svc    NodePort    10.97.20.225   <none>        80:32000/TCP   31m     app=nginx-deploy
```

> 这种方式实际上不够标准，service 应该用于内部服务的发现和负载均衡

| 资源        | 核心职责                 | 适用场景                             |
| ----------- | ------------------------ | ------------------------------------ |
| **Service** | 内部服务发现与负载均衡   | 所有服务间的通信（无论是否对外暴露） |
| **Ingress** | 对外暴露 HTTP/HTTPS 服务 | 需要基于域名/路径路由的 Web 服务     |

**1. Service 的核心作用**

Service 是 Kubernetes 中用于**内部服务发现与负载均衡** 的核心资源，主要职责是：

- **为 Pod 提供稳定的 IP 和 DNS 名称** ：即使后端 Pod 重启或扩缩容，Service 的 IP 和端口保持不变。
- **内部通信** ：集群内其他服务或 Pod 通过 Service 的 IP/端口访问目标应用。
- **负载均衡** ：将请求分发到后端多个 Pod 实例。

  **Service 的类型** ：

- **ClusterIP** （默认）：仅在集群内部可见，适用于内部服务间调用。
- **NodePort** ：在每个节点的 IP 上开放一个静态端口（范围：30000-32767），外部可通过 `节点IP:NodePort` 访问服务。
- **LoadBalancer** ：通过云服务商的负载均衡器（如 AWS ELB、阿里云 SLB）对外暴露服务，分配独立的公网 IP。

---

**2. Ingress 的核心作用**

Ingress 是 Kubernetes 中**对外暴露 HTTP/HTTPS 服务的 API 资源** ，主要职责是：

- **基于路径或域名的路由** ：将外部请求根据 URL 路径或 Host 头分发到不同的 Service。
- **统一入口** ：通过一个公网 IP 和端口（如 80/443）暴露多个服务，减少对外暴露的端口数量。
- **SSL/TLS 终止** ：支持 HTTPS 证书配置，统一处理加密流量。

  **Ingress 的典型场景** ：

- 将 `example.com/app` 路由到 `app-service`。
- 将 `api.example.com` 路由到 `api-service`。
- 提供 HTTPS 加密（需配置证书）。

---

**3. 两者的协作关系**

- **Service 是基础** ：Ingress 本身不直接连接到 Pod，而是通过 Service 作为后端。
- **Ingress 是扩展** ：在 Service 的基础上，提供更灵活的 HTTP 路由和外部访问能力。

---

**4. 对外暴露服务的典型架构**

1. **内部服务** ：使用 `ClusterIP` 类型的 Service，仅在集群内访问。
2. **HTTP 类服务** ：通过 Ingress 暴露，利用域名/路径路由和 HTTPS 支持。
3. **非 HTTP 类服务** （如数据库、TCP/UDP 服务）：
   - 使用 `NodePort` 或 `LoadBalancer` 类型的 Service。
   - 或通过 `Ingress` 的扩展实现（如 Nginx Ingress 的 TCP/UDP 配置）。

## 基于 Service 访问外部服务

此时在编写 svc 配置文件的时候，不编写 selector 属性，那么就不会自动创建 endpoint，我们需要自己手动创建 endpoint

```shell
root@k8s-master:/opt/k8s/service# ll
total 12
drwxr-xr-x 2 root root 4096 Apr 10 00:26 ./
drwxr-xr-x 5 root root 4096 Apr 10 00:50 ../
-rw-r--r-- 1 root root  919 Apr 10 00:18 nginx-svc.yaml
# 复制一份配置文件，用于编辑
root@k8s-master:/opt/k8s/service# cp -p nginx-svc.yaml nginx-svc-external.yaml
# 编辑一下文件内容
```

> `nginx-svc-external.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc-external # Service 名字
  labels:
    app: nginx
spec:
  ports:
    - port: 80
      targetPort: 80
      name: web
  type: ClusterIP # 使用 ClusterIP 类型
```

```shell
# 创建对应的服务
root@k8s-master:/opt/k8s/service# kubectl create -f nginx-svc-external.yaml
service/nginx-svc-external created
# 查看svc信息
root@k8s-master:/opt/k8s/service# kubectl get svc
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes           ClusterIP   10.96.0.1      <none>        443/TCP        20d
nginx-svc            NodePort    10.97.20.225   <none>        80:32000/TCP   16d
nginx-svc-external   ClusterIP   10.97.168.9    <none>        80/TCP         33s
# 查看endpoint信息，可以看到这里并没有自动创建endpoint，我们需要手动创建
root@k8s-master:/opt/k8s/service# kubectl get ep
NAME         ENDPOINTS              AGE
kubernetes   192.168.142.128:6443   20d
nginx-svc    10.244.235.212:80      16d
```

> `nginx-ep-external.yaml`
>
> 这里的 ip 我选择的是百度官网(103.235.46.115)，这样我们通过 service 请求，将会请求到百度官网

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  labels:
    app: nginx-svc-external # 与 service 一致
  name: nginx-svc-external # 与 service 一致
  namespace: default # 与 service 一致
subsets:
  - addresses:
      - ip: 103.235.46.115 # 目标 ip 地址
    ports: # 与 service 一致
      - name: web # 与 service 一致
        port: 80
        protocol: TCP
```

```shell
# 创建endpoint
root@k8s-master:/opt/k8s/service# kubectl create -f nginx-ep-external.yaml
endpoints/nginx-svc-external created
# 查看endpoint列表
root@k8s-master:/opt/k8s/service# kubectl get ep
NAME                 ENDPOINTS              AGE
kubernetes           192.168.142.128:6443   21d
nginx-svc            10.244.235.212:80      16d
nginx-svc-external   103.235.46.115:80      4s
# 查看详细信息
root@k8s-master:/opt/k8s/service# kubectl describe ep nginx-svc-external
Name:         nginx-svc-external
Namespace:    default
Labels:       app=nginx-svc-external
Annotations:  <none>
Subsets:
  Addresses:          103.235.46.115
  NotReadyAddresses:  <none>
  Ports:
    Name  Port  Protocol
    ----  ----  --------
    web   80    TCP

Events:  <none>
# 进入容器内部
root@k8s-master:/opt/k8s/service# kubectl exec -it dns-test -- sh
/ # wget http://nginx-svc-external
Connecting to nginx-svc-external (10.97.168.9:80)
wget: server returned error: HTTP/1.1 403 Forbidden
# 上面失败了，百度服务器可能配置了基于虚拟主机的访问控制，我们在请求里加一个Host头重试一下
/ # wget --header="Host: www.baidu.com" http://nginx-svc-external
Connecting to nginx-svc-external (10.97.168.9:80)
index.html           100% |**********************************************************************************************************************************************************************************|  2381   0:00:00 ETA
# 这次成功了，看一下效果
/ # cat index.html
<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Compatible content=IE=Edge><meta content=always name=referrer><link rel=stylesheet type=text/css href=http://s1.bdstatic.com/r/www/cache/bdorz/baidu.min.css><title>百度一下，你就知道</title></head> <body link=#0000cc> <div id=wrapper> <div id=head> <div class=head_wrapper> <div class=s_form> <div class=s_form_wrapper> <div id=lg> <img hidefocus=true src=//www.baidu.com/img/bd_logo1.png width=270 height=129> </div> <form id=form name=f action=//www.baidu.com/s class=fm> <input type=hidden name=bdorz_come value=1> <input type=hidden name=ie value=utf-8> <input type=hidden name=f value=8> <input type=hidden name=rsv_bp value=1> <input type=hidden name=rsv_idx value=1> <input type=hidden name=tn value=baidu><span class="bg s_ipt_wr"><input id=kw name=wd class=s_ipt value maxlength=255 autocomplete=off autofocus></span><span class="bg s_btn_wr"><input type=submit id=su value=百度一下 class="bg s_btn"></span> </form> </div> </div> <div id=u1> <a href=http://news.baidu.com name=tj_trnews class=mnav>新闻</a> <a href=http://www.hao123.com name=tj_trhao123 class=mnav>hao123</a> <a href=http://map.baidu.com name=tj_trmap class=mnav>地图</a> <a href=http://v.baidu.com name=tj_trvideo class=mnav>视频</a> <a href=http://tieba.baidu.com name=tj_trtieba class=mnav>贴吧</a> <noscript> <a href=http://www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1 name=tj_login class=lb>登录</a> </noscript> <script>document.write('<a href="http://www.baidu.com/bdorz/login.gif?login&tpl=mn&u='+ encodeURIComponent(window.location.href+ (window.location.search === "" ? "?" : "&")+ "bdorz_come=1")+ '" name="tj_login" class="lb">登录</a>');</script> <a href=//www.baidu.com/more/ name=tj_briicon class=bri style="display: block;">更多产品</a> </div> </div> </div> <div id=ftCon> <div id=ftConw> <p id=lh> <a href=http://home.baidu.com>关于百度</a> <a href=http://ir.baidu.com>About Baidu</a> </p> <p id=cp>&copy;2017&nbsp;Baidu&nbsp;<a href=http://www.baidu.com/duty/>使用百度前必读</a>&nbsp; <a href=http://jianyi.baidu.com/ class=cp-feedback>意见反馈</a>&nbsp;京ICP证030173号&nbsp; <img src=//www.baidu.com/img/gs.gif> </p> </div> </div> </div> </body> </html>
```

这里的请求流程实际上是从 busybox pod -> service -> endpoint -> 外部环境

## 代理外部域名及常用类型

```shell
root@k8s-master:/opt/k8s/service# cp nginx-svc-external.yaml nginx-svc-externalname.yaml
# 内容如下
root@k8s-master:/opt/k8s/service# vim nginx-svc-externalname.yaml
# 这里使用了别名，使用k代替kubectl，执行命令“alias k='kubectl'”
root@k8s-master:/opt/k8s/service# k create -f nginx-svc-externalname.yaml
service/baidu-external-domain created
root@k8s-master:/opt/k8s/service# k get svc
NAME                    TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
baidu-external-domain   ExternalName   <none>         www.baidu.com   <none>         12s
kubernetes              ClusterIP      10.96.0.1      <none>          443/TCP        23d
nginx-svc               NodePort       10.97.20.225   <none>          80:32000/TCP   18d
nginx-svc-external      ClusterIP      10.97.168.9    <none>          80/TCP         2d10h
root@k8s-master:/opt/k8s/service# k exec -it dns-test -- sh
/ # wget baidu-external-domain
Connecting to baidu-external-domain (153.3.238.127:80)
wget: server returned error: HTTP/1.1 403 Forbidden
# 需要加一下header，重试
/ # wget --header="Host: www.baidu.com" baidu-external-domain
Connecting to baidu-external-domain (103.235.46.115:80)
index.html           100% |**********************************************************************************************************************************************************************************|  2381   0:00:00 ETA
# 请自行查看index.html，这里就不贴出来了
```

> `nginx-svc-externalname.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: baidu-external-domain
  name: baidu-external-domain
spec:
  type: ExternalName
  externalName: www.baidu.com
```

> 这里看一下 service 的类型已经使用过哪几种了

```shell
root@k8s-master:/opt/k8s/service# ls
nginx-ep-external.yaml  nginx-svc-externalname.yaml  nginx-svc-external.yaml  nginx-svc.yaml
root@k8s-master:/opt/k8s/service# grep type nginx-svc.yaml
  type: NodePort
root@k8s-master:/opt/k8s/service# grep type nginx-svc-external.yaml
  type: ClusterIP
root@k8s-master:/opt/k8s/service# grep type nginx-svc-externalname.yaml
  type: ExternalName
```

1. ClusterIP

   只能在集群内部使用，不配置类型的话默认就是 ClusterIP

2. ExternalName

   返回定义的 CNAME 别名，可以配置为域名

3. NodePort

   会在所有安装了 kube-proxy 的节点都绑定一个端口，此端口可以代理至对应的 Pod，集群外部可以使用任意节点 ip + NodePort 的端口号访问到集群中对应 Pod 中的服务。

   当类型设置为 NodePort 后，可以在 ports 配置中增加 nodePort 配置指定端口，需要在下方的端口范围内，如果不指定会随机指定端口

4. LoadBalancer

   使用云服务商（阿里云、腾讯云等）提供的负载均衡器服务

## Ingress

> 左侧是传统 nginx 的负载均衡版本，右侧是 k8s 版本的负载均衡

![202505122307025.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202505122307025.png)

### 安装 ingress-nginx

> 官方[文档](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/)

这里有很多的控制器，我们使用 ingress-nginx
![202505122307136.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202505122307136.png)

以下是官方文档
![202505122307254.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202505122307254.png)
![202505122307604.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202505122307604.png)

#### 安装 helm

[下载链接](https://get.helm.sh/helm-v3.2.3-linux-amd64.tar.gz)

```shell
# 解压到当前目录
root@k8s-master:/opt/k8s/helm# ll
total 39456
drwxr-xr-x 2 root root     4096 Apr 29 00:42 ./
drwxr-xr-x 6 root root     4096 Apr 29 00:39 ../
-rwxr-xr-x 1 3434 3434 40378368 Jun  8  2020 helm*
-rw-r--r-- 1 3434 3434    11373 Jun  8  2020 LICENSE
-rw-r--r-- 1 3434 3434     3319 Jun  8  2020 README.md
# 将helm拷贝到/usr/local/bin/下
root@k8s-master:/opt/k8s/helm# cp helm /usr/local/bin/
# 查看版本
root@k8s-master:/opt/k8s/helm# helm version
version.BuildInfo{Version:"v3.2.3", GitCommit:"8f832046e258e2cb800894579b1b3b50c2d83492", GitTreeState:"clean", GoVersion:"go1.13.12"}
# 添加仓库
root@k8s-master:/opt/k8s/helm# helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
"ingress-nginx" has been added to your repositories
# 查看仓库列表
root@k8s-master:/opt/k8s/helm# helm repo list
NAME            URL
ingress-nginx   https://kubernetes.github.io/ingress-nginx
# 搜索 ingress-nginx
root@k8s-master:/opt/k8s/helm# helm search repo ingress-nginx
NAME                            CHART VERSION   APP VERSION     DESCRIPTION
ingress-nginx/ingress-nginx     4.12.1          1.12.1          Ingress controller for Kubernetes using NGINX a...
root@k8s-master:/opt/k8s/helm# helm pull ingress-nginx/ingress-nginx
# 查看下载软件包
root@k8s-master:/opt/k8s/helm# ls
helm  ingress-nginx-4.12.1.tgz  LICENSE  README.md
root@k8s-master:/opt/k8s/helm# tar -zxvf ingress-nginx-4.12.1.tgz
# 将仓库信息改为国内镜像
root@k8s-master:/opt/k8s/helm/ingress-nginx# vim values.yaml

```

```yaml
controller:
  name: controller
  enableAnnotationValidations: true
  image:
    chroot: false
    # 修改此处
    registry: registry.cn-hangzhou.aliyuncs.com
    # 修改此处
    image: google_containers/nginx-ingress-controller
    tag: "v1.2.1"
    # 注释掉这两个地方
    # digest: sha256:d2fbc4ec70d8aa2050dd91a91506e998765e86c96f32cffb56c503c9c34eed5b
    # digestChroot: sha256:90155c86548e0bb95b3abf1971cd687d8f5d43f340cfca0ad3484e2b8351096e
    pullPolicy: IfNotPresent
# ....
    patch:
      enabled: true
      image:
        # 修改此处
        # registry: registry.k8s.io
        registry: registry.cn-hangzhou.aliyuncs.com
        # 修改此处
        # image: ingress-nginx/kube-webhook-certgen
        image: google_containers/kube-webhook-certgen
        tag: v1.2.1
        # 注释这里
        # digest: sha256:e8825994b7a2c7497375a9b945f386506ca6a3eda80b89b74ef2db743f66a5ea
        pullPolicy: IfNotPresent
#...
      nodeSelector:
        kubernetes.io/os: linux
        # 修改这里
        ingress: "true"
#...
  # 改成ClusterFirstWithHostNet
  dnsPolicy: ClusterFirstWithHostNet
#...
  electionTTL: ""
  allowSnippetAnnotations: false
  # 改成true
  hostNetwork: true
#...
  service:
    enabled: true
    external:
      enabled: true
    annotations: {}
    labels: {}
    # 改成ClusterIP
    type: ClusterIP
#...
  admissionWebhooks:
    name: admission
    annotations: {}
    # 改为false
    enabled: false
#... false改为true
  hostNetwork: true
#... {} 改为false
  extraArgs:
    update-status: "false"
#...
  nodeSelector:
    kubernetes.io/os: linux
    # 添加
    ingress: "true"
```

```shell
# 标记一下node
root@k8s-master:/opt/k8s/helm/ingress-nginx# k label no k8s-master ingress=true
node/k8s-master labeled
# 查看node的label
root@k8s-master:/opt/k8s/helm/ingress-nginx# k get no --show-labels
NAME         STATUS   ROLES                  AGE   VERSION   LABELS
k8s-master   Ready    control-plane,master   24d   v1.23.6   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,ingress=true,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=,node.kubernetes.io/exclude-from-external-load-balancers=
root@k8s-master:/opt/k8s/helm/ingress-nginx# k create ns ingress-nginx
namespace/ingress-nginx created
helm install ingress-nginx -n ingress-nginx .
```
