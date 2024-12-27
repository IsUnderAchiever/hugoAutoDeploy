---
title: Docker搭建私人Docker镜像仓库
description: Docker搭建私人Docker镜像仓库
date: 2024-12-27
slug: Docker搭建私人Docker镜像仓库
image: dbc-docker-desktop-home.webp
categories:
    - Docker
---

# Docker搭建私人Docker镜像仓库

```sh
docker pull registry

mkdir -p registry/data && cd registry

pwd
# /root/registry

docker run \
--restart=always \
-p 5000:5000 \
--name registry \
-v /root/registry/data:/var/lib/registry \
-d registry:latest

# 访问url，查看镜像仓库中的所有镜像
# http://ubuntu-notebook:5000/v2/_catalog
```

> docker私有仓库默认是基于https传输的，这里我们在`客户端`进行相关设置不使用https传输

```sh
vim /etc/docker/daemon.json

# 添加如下配置
"insecure-registries": ["192.168.50.11:5000"]
```

```json
{
  "registry-mirrors": [
    "https://dockerpull.org",
    "https://docker.1panel.dev",
    "https://docker.foreverlink.love",
    "https://docker.fxxk.dedyn.io",
    "https://docker.xn--6oq72ry9d5zx.cn",
    "https://docker.zhai.cm",
    "https://docker.5z5f.com",
    "https://a.ussh.net",
    "https://docker.cloudlayer.icu",
    "https://hub.littlediary.cn",
    "https://hub.crdz.gq",
    "https://docker.unsee.tech",
    "https://docker.kejilion.pro",
    "https://registry.dockermirror.com",
    "https://hub.rat.dev",
    "https://dhub.kubesre.xyz",
    "https://docker.nastool.de",
    "https://docker.udayun.com",
    "https://docker.rainbond.cc",
    "https://hub.geekery.cn",
    "https://docker.1panelproxy.com",
    "https://atomhub.openatom.cn",
    "https://docker.m.daocloud.io",
    "https://docker.1ms.run",
    "https://docker.linkedbus.com"
  ],
  "insecure-registries": [
    "192.168.50.11:5000"
  ],
  "exec-opts": [
    "native.cgroupdriver=systemd"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
```

> 重新加载docker配置

```sh
systemctl daemon-reload
systemctl restart docker
```

> 在推送镜像之前，还需要`重做镜像tag`

```sh
# 重做镜像tag
docker tag images:tag ip:5000/images:tag

# 比如
docker tag nginx:latest 192.168.50.11:5000/nginx:latest
# 或者
docker tag nginx 192.168.50.11:5000/nginx

# 推送镜像
docker push ip:5000/images:tag

# 比如
docker push 192.168.50.11:5000/nginx

# 访问url，查看镜像仓库中的所有镜像
# http://ubuntu-notebook:5000/v2/_catalog

# 或者在服务器上查看data目录下的内容
ll data/docker/registry/v2/repositories/
# nginx/
```

> 这里使用`registry-ui`配置可视化界面进行管理

新建`docker-componse.yml`文件，并上传到服务器`/root/registry`目录下

```yaml
version: '3.8'
services:
  registry:
    image: registry
    volumes:
      - /root/registry/data:/var/lib/registry
  registry-ui:
    image: joxit/docker-registry-ui:1.5-static
    ports:
      - 8010:80
    environment:
      - REGISTRY_TITLE=tong的私有镜像仓库
      - REGISTRY_URL=http://registry:5000
      - CATALOG_ELEMENTS_LIMIT=1000
    depends_on:
      - registry
```

> 执行以下命令

```sh
docker compose up -d
```

> 访问服务器url即可
>
> `http://ubuntu-notebook:8010/`

接下来需要更新客户端`daemon.json`指向`8010`端口

```json
{
  "registry-mirrors": [
    "https://dockerpull.org",
    "https://docker.1panel.dev",
    "https://docker.foreverlink.love",
    "https://docker.fxxk.dedyn.io",
    "https://docker.xn--6oq72ry9d5zx.cn",
    "https://docker.zhai.cm",
    "https://docker.5z5f.com",
    "https://a.ussh.net",
    "https://docker.cloudlayer.icu",
    "https://hub.littlediary.cn",
    "https://hub.crdz.gq",
    "https://docker.unsee.tech",
    "https://docker.kejilion.pro",
    "https://registry.dockermirror.com",
    "https://hub.rat.dev",
    "https://dhub.kubesre.xyz",
    "https://docker.nastool.de",
    "https://docker.udayun.com",
    "https://docker.rainbond.cc",
    "https://hub.geekery.cn",
    "https://docker.1panelproxy.com",
    "https://atomhub.openatom.cn",
    "https://docker.m.daocloud.io",
    "https://docker.1ms.run",
    "https://docker.linkedbus.com"
  ],
  "insecure-registries": [
    "192.168.50.11:8010"
  ],
  "exec-opts": [
    "native.cgroupdriver=systemd"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
```

> 重新加载配置

```sh
systemctl daemon-reload
systemctl restart docker

# 步骤和之前类似，仍然先要重构镜像，只是端口变为刚刚`docker-componse.yml`里配置的8010
# 重做镜像tag
docker tag images:tag ip:8010/images:tag

# 比如
docker tag nginx:latest 192.168.50.11:8010/nginx:latest
# 或者
docker tag nginx 192.168.50.11:8010/nginx

# 推送镜像
docker push ip:8010/images:tag

# 比如
docker push 192.168.50.11:8010/nginx
```

