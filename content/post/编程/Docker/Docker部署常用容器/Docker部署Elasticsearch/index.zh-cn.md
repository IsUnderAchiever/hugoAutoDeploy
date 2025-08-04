---
title: Docker部署Elasticsearch
description: Docker部署Elasticsearch
date: 2025-06-09
slug: Docker部署Elasticsearch
image: dbc-docker-desktop-home.webp
categories:
    - Docker
---
# Docker部署Elasticsearch

## 安装Elasticsearch

```sh
mkdir /home/tong/software/docker/elasticsearch/{data,config,plugins}

chmod -R 777 /home/tong/software/docker/elasticsearch/{data,config,plugins}

docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -p 9300:9300 \
  -v /home/tong/software/docker/elasticsearch/data:/usr/share/elasticsearch/data \
  -v /home/tong/software/docker/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
  -v /home/tong/software/docker/elasticsearch/config:/usr/share/elasticsearch/config \
  -e "discovery.type=single-node" \
  -e "ES_JAVA_OPTS=-Xms1g -Xmx1g" \
  --memory="4g" \
  --memory-swap="-1" \
  elasticsearch:8.18.2
```

> 修改`elasticsearch.yaml`

```yaml
xpack.security.http.ssl:
  enabled: false
xpack.security.enabled: false
```

访问`ip:9200`即可

![image-20250609224350686](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506092303101.png)

## 安装Kibana

```sh
# 启动容器用来复制配置文件到宿主机
docker run --name kibana \
  -p 5601:5601 \
  -d kibana:8.18.2

# 复制配置文件
docker cp kibana:/usr/share/kibana/config /home/tong/software/docker/kibana/

# 启动容器
# ELASTICSEARCH_HOSTS配置为自己es的url
docker run --name kibana \
  -p 5601:5601 \
  -e ELASTICSEARCH_HOSTS=http://116.148.120.214:9200 \
  -v /home/tong/software/docker/kibana/config:/usr/share/kibana/config \
  -d kibana:8.18.2
```

访问`ip:5601`即可



