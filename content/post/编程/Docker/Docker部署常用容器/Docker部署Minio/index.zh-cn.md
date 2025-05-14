---
title: Docker部署Minio
description: Docker部署Minio
date: 2025-01-04
slug: Docker部署Minio
image: dbc-docker-desktop-home.webp
categories:
    - Docker
---
# Docker部署Minio


```sh
docker run \
-p 5000:9000 \
-p 5001:5001 \
--name minio \
-v /home/tong/software/docker/volume/minio/data:/data \
-e "MINIO_ROOT_USER=minioadmin" \
-e "MINIO_ROOT_PASSWORD=minioadmin" \
-d ubuntu-notebook:8010/minio/minio server /data --console-address ":5001"
```