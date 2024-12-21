---
title: Dubbo
description: Dubbo
date: 2024-06-01
slug: Dubbo
image: 202412212113029.png
categories:
    - Dubbo
---

# Dubbo
> 本次使用`JDK17、Dubbo3`
1. JDK8与Dubbo3.1.x以前的版本匹配，在使用Zookeeper注册作为注册中心时，消费者会出现节点已经存在的异常
2. JDK17与Dubbo3.1.x之前的版本搭配使用会出现如下问题
   1. JDK9之后的深反射问题，需要通过JVM参数配置解决
      ```
      -Dio.netty.tryReflectionSetAccessible-true
      --add-opens
      java.base/jdk.internal.misc=ALL - UNNAMED
      --add-opens
      java.base/java.nio=ALL-UNNAMED
      --add-opens
      java.base/java.lang=ALL-UNNAMED
      ```
   2. Dubbo3.2.0.beat4以前的版本使用的是Spring5.2.x不能支持JDK17会产生如下异常
      `Unsupported class file major version 61` [major 61对应17 ]
      版本需要升级到Dubbo3.2.0.beat5以上版本
