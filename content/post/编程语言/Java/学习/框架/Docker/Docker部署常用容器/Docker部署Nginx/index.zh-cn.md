---
title: Docker部署Nginx
description: Docker部署Nginx
date: 2024-12-23
slug: Docker部署Nginx
image: dbc-docker-desktop-home.webp
categories:
    - Docker
---
# Docker部署Nginx

```sh
# 创建挂载目录
mkdir -p /home/tong/software/docker/nginx_angular15_docs/conf
mkdir -p /home/tong/software/docker/nginx_angular15_docs/log
mkdir -p /home/tong/software/docker/nginx_angular15_docs/html
```

> 复制配置文件
```sh
# 生成容器
docker run --name nginx -p 9001:80 -d nginx
# 将容器nginx.conf文件复制到宿主机
docker cp nginx:/etc/nginx/nginx.conf /home/nginx/conf/nginx.conf
# 将容器conf.d文件夹下内容复制到宿主机
docker cp nginx:/etc/nginx/conf.d /home/nginx/conf/conf.d
# 将容器中的html文件夹复制到宿主机
docker cp nginx:/usr/share/nginx/html /home/nginx/
# 删除容器
docker rm -f nginx
```

> 我这里本来就有文件在`/home/tong/software/docker/nginx`目录下，就不复制容器内的文件了
```sh
cp -rp /home/tong/software/docker/nginx/conf /home/tong/software/docker/nginx_angular15_docs
cp -rp /home/tong/software/docker/nginx/html /home/tong/software/docker/nginx_angular15_docs
```

> 运行容器
```sh
docker run \
-p 8015:80 \
--name angular15 \
-v /home/tong/software/docker/nginx_angular15_docs/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /home/tong/software/docker/nginx_angular15_docs/conf/conf.d:/etc/nginx/conf.d \
-v /home/tong/software/docker/nginx_angular15_docs/html:/usr/share/nginx/html \
-d nginx:latest
```

