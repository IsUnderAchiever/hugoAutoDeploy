---
title: Docker部署Nextcloud
description: Docker部署Nextcloud
date: 2025-12-15
slug: Docker部署Nextcloud
image: dbc-docker-desktop-home.webp
categories:
    - Docker
---
# Docker部署Nextcloud

> Docker compose yaml

```bash
services:
  db:
    image: postgres:15
    restart: always
    environment:
      POSTGRES_DB: nextcloud
      POSTGRES_USER: nextcloud
      POSTGRES_PASSWORD: tongwenhao123
    volumes:
      - /data/volumes/nextcloud/postgres_data:/var/lib/postgresql/data
    networks:
      - nextcloud-net

  app:
    image: nextcloud:latest
    restart: always
    ports:
      - "51117:80"
    volumes:
      - /data/volumes/nextcloud/nextcloud_data:/var/www/html
    environment:
      POSTGRES_HOST: db
      POSTGRES_DB: nextcloud
      POSTGRES_USER: nextcloud
      POSTGRES_PASSWORD: tongwenhao123
    depends_on:
      - db
    networks:
      - nextcloud-net

networks:
  nextcloud-net:
    driver: bridge
```

访问[127.0.0.1:51117](http://127.0.0.1:51117)

![image-20251215230142299](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo06/image-20251215230142299.png)
