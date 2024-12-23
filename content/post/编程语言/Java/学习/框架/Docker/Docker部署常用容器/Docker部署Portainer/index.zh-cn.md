---
title: Docker部署Portainer
description: Docker部署Portainer
date: 2024-12-23
slug: Docker部署Portainer
image: dbc-docker-desktop-home.webp
categories:
    - Docker
---
# Docker部署Portainer

> 参考[资料](https://blog.csdn.net/jks212454/article/details/130828096)
>
> 拉取镜像

```sh
docker pull portainer/portainer-ce:latest
```

> 创建数据卷

```sh
docker volume create portainer_data
```

```sh
docker run \
--name=portainer \
-p 9000:9000 \
-p 8000:8000 \
--restart=always \
-v /var/run/docker.sock:/var/run/docker.sock \
-v portainer_data:/data \
-d portainer/portainer-ce:latest
```

> 访问[url](http://docker-learning.com:9000/#!/2/docker/containers)即可
>
> `http://docker-learning.com:9000/#!/2/docker/containers`

