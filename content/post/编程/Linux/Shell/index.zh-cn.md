---
title: Shell
description: Shell
date: 2024-02-15
slug: Shell
image: 202412212157770.png
categories:
    - Shell
---

# tar包
```bash
tar -zxvf XXX.tar.gz
```
# nacos
```bash
cd /usr/local/nacos/bin
# 启动命令
sudo sh startup.sh -m standalone
```
# zookeeper
> 启动失败，可参考博客
>
> [博客1](https://blog.csdn.net/m0_51796313/article/details/127726232)、[博客2](https://blog.csdn.net/weixin_44285445/article/details/109849311)、[博客3](https://blog.csdn.net/YYbLQQ/article/details/121803004)
```bash
cd /usr/local/zookeeper/bin
# 启动zookeeper
./zkServer.sh start
# 停止zookeeper
./zkServer.sh stop
# 查看状态
./zkServer.sh status
# 连接zookeeper
./zkCli.sh -server 192.168.56.10:2181
# 查看
ls /
ls /dubbo
```
