---
title: Nexus搭建Docker私服
description: Nexus搭建Docker私服
date: 2025-06-03
slug: Nexus搭建Docker私服
image: 202506081432952.png
categories:
    - 系统配置
---
# Nexus搭建Docker私服

# 基础配置

```sh
docker run -itd \
        --name=nexus \
        -p 8081:8081 \
        -p 8082:8082 \
        -p 8083:8083 \
        -p 5000:5000 \
        --privileged=true \
        --restart=always \
        --ulimit nofile=655350 \
        --ulimit memlock=-1 \
        --memory=16G \
        --memory-swap=-1 \
        --cpuset-cpus='1-7' \
        -e INSTALL4J_ADD_VM_PARAMS="-Xms4g -Xmx4g -XX:MaxDirectMemorySize=8g -Djava.util.prefs.userRoot=/nexus-data/javaprefs" \
        -v nexus-localtime:/etc/localtime \
        -v nexus-data:/nexus-data \
        sonatype/nexus3:latest
```
> 配置镜像源

```sh
sudo vim /etc/docker/daemon.json

# 添加如下配置
{
  "registry-mirrors": [
    "https://a.ussh.net",
    "https://hub.littlediary.cn",
    "https://hub.crdz.gq",
    "https://docker.unsee.tech",
    "https://docker.kejilion.pro",
    "https://hub.rat.dev",
    "https://docker.m.daocloud.io",
    "https://docker.1ms.run"
  ],
  "exec-opts": [
    "native.cgroupdriver=systemd"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "debug": true,
  "storage-driver": "overlay2"
}

# 重启 Docker 服务
sudo systemctl restart docker
```


> 配置docker日志级别为debug`("debug": true)`，查看下载镜像时使用的是哪个源
> 这里我提供的都是暂时可用的源
> 官方源地址`https://registry-1.docker.io`

```sh
sudo journalctl -u docker.service -n 100 | grep "pull"
```

![Pasted image 20250601094848.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428086.png)
# Nexus新建仓库
## Proxy仓库
接下来在nexus新建proxy仓库，刚刚我已经测试过了`daemon.json`里的源暂时都是可用的
随便挑一个源配置为nexus的proxy仓库即可

>url：`https://a.ussh.net`
>新建`docker(proxy)`仓库，name随意即可，我这里取名`docker-proxy-ussh`
>以下是需要注意的几个地方，其他位置默认即可

![Pasted image 20250601101120.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428087.png)

## Group仓库
> 新建`docker(group)`仓库，我这里取名`docker-public`
> 以下时需要注意的几个地方
> http端口选择8082，允许匿名pull
> 最重要的是将刚刚创建的proxy仓库添加为成员

![Pasted image 20250601101548.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428088.png)

## Hosted仓库
> hosted仓库用于`docker push`将本地镜像push到nexus仓库
> 而以上的proxy主要用于`docker pull`

配置如下
![Pasted image 20250601101919.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428089.png)

## Realms

> 点击将`Docker Bearer Token Realm`配置到右侧的`Active`,然后点击`Save`

![Pasted image 20250601102150.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428090.png)

# pull
## 配置Daemon

```sh
# 配置daemon.json为如下内容
vim /etc/docker/daemon.json

sudo systemctl restart docker
```

```json
{
  "registry-mirrors": [
    "http://116.148.120.214:8081"
  ],
  "insecure-registries": [
    "116.148.120.214:8082",
    "116.148.120.214:8083"
  ],
  "exec-opts": [
    "native.cgroupdriver=systemd"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "debug": true,
  "storage-driver": "overlay2"
}
```

> **等待nexus容器启动完成后**，执行如下命令登录

```sh
docker login -u admin --password-stdin tongwenhao123 116.148.120.214:8082
```

> 报错如下
> 这应该是较新版本nexus的问题，如果未出现此问题，忽略即可

```sh
docker login -u admin -p tongwenhao123 116.148.120.214:8082

# Error response from daemon: Get http://116.148.120.214:8082/v2/: Get null://116.148.120.214:8082/v2/token?account=admin&client_id=docker&offline_token=true&service=null%3A%2F%2F116.148.120.214%3A8082%2Fv2%2Ftoken: unsupported protocol scheme "null"
```

在github上看到相同的issue，且还未关闭
[Docker repository - Unsupported protocol scheme "null" #565](https://github.com/sonatype/nexus-public/issues/565)

此issue里提供了两种解决方案
1. 取消匿名pull选项
2. 修改`org.sonatype.nexus.bootstrap.jetty.NexusRequestCustomizer`类如下
```java
  @Override
  public void customize(final Connector connector, final HttpConfiguration channelConfig, final Request request) {
    HttpURI uri = request.getHttpURI();
    String path = uri.getPath();

    request.setScheme("http"); // <=== ADD THIS

    [...]
  }
```

这里我使用第一种方案，取消匿名pull后登陆成功
![Pasted image 20250601121259.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428091.png)
拉取一个镜像试试
```sh
docker pull 116.148.120.214:8082/mysql
```
![Pasted image 20250601122141.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428092.png)

## nginx反向代理携带认证信息

这里是因为neuxs的bug，取消匿名pull之后无法login，所以需要配置代理
如果你的版本可以使用匿名pull，则直接开启匿名pull即可，无需再配置代理

> 但是如果直接`docker pull mysql`则会报错如下

```sh
# 查看pull日志信息
sudo journalctl -u docker.service -n 100 | grep "pull"

# 发现存在如下报错
# no basic auth credentials

# 6月 01 12:41:42 tongwh dockerd[2878504]: time="2025-06-01T12:41:42.083347618+08:00" level=debug msg="Trying to pull mysql from http://116.148.120.214:8082/ v2"
# 6月 01 12:41:42 tongwh dockerd[2878504]: time="2025-06-01T12:41:42.211650285+08:00" level=info msg="Attempting next endpoint for pull after error: Head http://116.148.120.214:8082/v2/library/mysql/manifests/latest: no basic auth credentials"
```

参考[Docker私有镜像拉取错误no basic auth credentials](https://www.kubernetes.org.cn/7994.html#:~:text=%E4%B8%8D%E5%B9%B8%E7%9A%84%E6%98%AF%EF%BC%8CDocker%E5%81%8F%E7%88%B1%E4%BD%BF%E7%94%A8auths%E4%B8%AD%E7%9A%84https%3A%2F%2F%E7%9A%84%E5%87%AD%E8%AF%81%EF%BC%8C%E5%B9%B6%E5%B0%9D%E8%AF%95%E4%BD%BF%E7%94%A8%E7%A9%BA%E5%87%AD%E8%AF%81%E6%8B%89%E5%8F%96%E9%95%9C%E5%83%8F%E3%80%82%20%E5%9B%A0%E6%AD%A4%EF%BC%8C%E4%BC%9A%E6%9C%89no%20basic%20auth%20credentials%E9%94%99%E8%AF%AF%E3%80%82,%E9%80%9A%E8%BF%87%E4%B8%8A%E6%96%87%EF%BC%8C%E6%88%91%E4%BB%AC%E7%A1%AE%E5%AE%9A%E4%BA%86%E9%97%AE%E9%A2%98%E6%98%AF%E4%B8%80%E4%B8%AA%E7%A9%BA%E5%87%AD%E8%AF%81%E8%A2%AB%E6%B7%BB%E5%8A%A0%E5%88%B0%20Docker%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6config.json%20%E4%B8%AD%EF%BC%8C%E6%88%91%E4%BB%AC%E5%B0%B1%E5%BE%88%E5%AE%B9%E6%98%93%20%E8%A7%A3%E5%86%B3%E8%AF%A5%E9%97%AE%E9%A2%98%E3%80%82%20%E6%88%91%E4%BB%AC%E9%9C%80%E8%A6%81%E5%81%9A%E7%9A%84%E5%B0%B1%E6%98%AF%E6%B7%BB%E5%8A%A0%E4%B8%80%E6%9D%A1if%E8%AF%AD%E5%8F%A5%E4%BB%A5%E8%B7%B3%E8%BF%87%E7%A9%BA%E5%87%AD%E6%8D%AE%EF%BC%9A%20%E5%BE%88%E5%A4%9A%E4%BD%BF%E7%94%A8Docker%E7%9A%84%E7%BB%84%E7%BB%87%EF%BC%8C%E5%8F%AF%E8%83%BD%E4%BB%8D%E5%9C%A8%E4%BD%BF%E7%94%A8%E4%BC%A0%E7%BB%9F%E7%9A%84auths%E6%96%B9%E6%B3%95%E3%80%82)
知道了为啥会出现这个报错，但是**Docker Credentials Store** 无法直接解决我们现在的问题，考虑使用nginx反向代理解决认证传递

```shell
mkdir -p /home/tong/software/docker/ngxin/nexus/{conf,logs,html}

# 生成容器
docker run --name nginx -p 9001:80 -d nginx
# 将容器nginx.conf文件复制到宿主机
docker cp nginx:/etc/nginx/nginx.conf /home/tong/software/docker/ngxin/nexus/nginx.conf
# 将容器conf.d文件夹下内容复制到宿主机
docker cp nginx:/etc/nginx/conf.d /home/tong/software/docker/ngxin/nexus/conf.d
# 将容器中的html文件夹复制到宿主机
docker cp nginx:/usr/share/nginx/html /home/tong/software/docker/ngxin/nexus/
# 删除容器
docker rm -f nginx

docker run \
-p 8010:80 \
--name nexus_docker \
-v /home/tong/software/docker/ngxin/nexus/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /home/tong/software/docker/ngxin/nexus/conf/conf.d:/etc/nginx/conf.d \
-v /home/tong/software/docker/ngxin/nexus/html:/usr/share/nginx/html \
-d nginx:latest

# 将your_password改为自己的neuxs密码
echo -n 'admin:your_password' | base64
```

> `/home/tong/software/docker/ngxin/nexus/conf/conf.d/default.conf`文件内容如下

主要配置如下内容
`Authorization` 更新成自己的base64密码
`proxy_pass`这里如果连接域名失败，就改成ip试试


```conf
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    location /v2/ {
        proxy_pass http://116.148.120.214:8082/v2/;  # 指向 Nexus 的 Docker 仓库
        proxy_set_header Host $host;  # 保留原始 Host 头
        proxy_set_header Authorization "Basic YWRtaW46eW91cl9wYXNzd29yZA==";  # 替换为你的 Base64 编码认证信息
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # 可选：代理其他 Docker 相关路径（如 token 服务）
    location /service/token {
        proxy_pass http://116.148.120.214:8082/service/token;
        proxy_set_header Host $host;
        proxy_set_header Authorization "Basic YWRtaW46eW91cl9wYXNzd29yZA==";
    }
}
```

![Pasted image 20250601131936.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428093.png)

# push
> 执行下面的命令进行push

```shell
docker tag mysql:8.0.20 116.148.120.214:8011/mysql:8.0.20
docker push 116.148.120.214:8011/mysql:8.0.20
```

但是失败，查看日志

```sh
sudo journalctl -u docker.service -n 100 | grep "push"

# 6月 01 20:54:46 tongwh dockerd[278791]: time="2025-06-01T20:54:46.027457332+08:00" level=debug msg="Calling POST /v1.41/images/116.148.120.214:8083/mysql/push?tag=8.0.20"
# 6月 01 20:54:46 tongwh dockerd[278791]: time="2025-06-01T20:54:46.027889502+08:00" level=debug msg="Trying to push 116.148.120.214:8083/mysql to https://116.148.120.214:8083 v2"
# 6月 01 20:54:46 tongwh dockerd[278791]: time="2025-06-01T20:54:46.030593299+08:00" level=info msg="Attempting next endpoint for push after error: Get https://116.148.120.214:8083/v2/: http: server gave HTTP response to HTTPS client"
# 6月 01 20:54:46 tongwh dockerd[278791]: time="2025-06-01T20:54:46.030641890+08:00" level=debug msg="Trying to push 116.148.120.214:8083/mysql to http://116.148.120.214:8083 v2"
# 6月 01 20:54:47 tongwh dockerd[278791]: time="2025-06-01T20:54:47.621102657+08:00" level=error msg="Not continuing with push after error: context canceled"
# 6月 01 20:54:51 tongwh dockerd[278791]: time="2025-06-01T20:54:51.404756735+08:00" level=debug msg="Calling POST /v1.41/images/116.148.120.214:8083/mysql/push?tag=8.0.20"
# 6月 01 20:54:51 tongwh dockerd[278791]: time="2025-06-01T20:54:51.405206602+08:00" level=debug msg="Trying to push 116.148.120.214:8083/mysql to https://116.148.120.214:8083 v2"
# 6月 01 20:54:51 tongwh dockerd[278791]: time="2025-06-01T20:54:51.407736012+08:00" level=info msg="Attempting next endpoint for push after error: Get https://116.148.120.214:8083/v2/: http: server gave HTTP response to HTTPS client"
# 6月 01 20:54:51 tongwh dockerd[278791]: time="2025-06-01T20:54:51.407788029+08:00" level=debug msg="Trying to push 116.148.120.214:8083/mysql to http://116.148.120.214:8083 v2"
```

原因大概是出在nginx上，由于上面nginx只配置了到8082的代理，而我们的hosted却在8083，这显然有问题
有一个思路是新增一个nginx容器,然后再nginx针对路由到8083，比如
```sh
docker run \
-p 8011:80 \
--name nexus_docker_push \
-v /home/tong/software/docker/ngxin/nexus/docker/push/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /home/tong/software/docker/ngxin/nexus/docker/push/conf/conf.d:/etc/nginx/conf.d \
-v /home/tong/software/docker/ngxin/nexus/docker/push/html:/usr/share/nginx/html \
-d nginx:latest

docker tag mysql:8.0.20 116.148.120.214:8011/mysql:8.0.20
docker push 116.148.120.214:8011/mysql:8.0.20
```
接下来顺着这个思路尝试做一下

> nginx配置如下
> `client_max_body_size`一定要加，否则nginx会报错`413 Request Entity Too Large`

配置`/home/tong/software/docker/ngxin/nexus/docker/push/conf/conf.d/default.conf`
```conf
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    # 允许最大请求体大小（示例设置为无限制）
    client_max_body_size 0;

    location /v2/ {
        proxy_pass http://116.148.120.214:8083/v2/;  # 指向 Nexus 的 Docker 仓库
        proxy_set_header Host $host;  # 保留原始 Host 头
        proxy_set_header Authorization "Basic YWRtaW46dG9uZ3dlbmhhbzEyMw==";  # 替换为你的 Base64 编码认证信息
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
![Pasted image 20250601220219.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428094.png)
![Pasted image 20250601220514.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428095.png)

