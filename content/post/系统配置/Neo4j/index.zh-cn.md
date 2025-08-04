---
title: Neo4j
description: Neo4j
date: 2025-06-02
slug: Neo4j
image: 202506081441416.png
categories:
    - Neo4j
---
# Neo4j

## 配置Neo4j

```sh
cd /home/tong/software/docker/neo4j
mkdir {logs,plugins,conf,data}

# 修改目录所有者为 neo4j 用户（UID 7474）
sudo chown -R 7474:7474 $PWD/data $PWD/plugins $PWD/logs

# cp一些配置出来
docker run --name neo4j -p 7474:7474 -p 7687:7687 -d neo4j:5.21.0
docker cp neo4j:/data /home/tong/software/docker/neo4j
docker cp neo4j:/plugins /home/tong/software/docker/neo4j
docker cp neo4j:/var/lib/neo4j/conf /home/tong/software/docker/neo4j
docker cp neo4j:/logs /home/tong/software/docker/neo4j
docker rm -f neo4j

# 前往github下载jar包并上传到服务器
# https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/download/5.21.0/apoc-5.21.0-extended.jar
# 这个jar包也得改
sudo chown -R 7474:7474 /home/tong/software/docker/neo4j/plugins/apoc-extended.jar

docker run -d \
    -p 7474:7474 -p 7687:7687 \
    -v $PWD/data:/data \
    -v $PWD/plugins:/plugins \
    -v $PWD/conf:/var/lib/neo4j/conf \
    -v $PWD/logs:/logs \
    --name neo4j-apoc \
    -e NEO4J_AUTH=neo4j/password \
    -e NEO4J_apoc_export_file_enabled=true \
    -e NEO4J_apoc_import_file_enabled=true \
    -e NEO4J_apoc_import_file_use__neo4j__config=true \
    -e NEO4J_PLUGINS='["apoc"]' \
    neo4j:5.21.0
```

启动日志如下

```bash
# Installing Plugin 'apoc' from /var/lib/neo4j/labs/apoc-*-core.jar to /plugins/apoc.jar
# Applying default values for plugin apoc to neo4j.conf
# SLF4J(W): Class path contains multiple SLF4J providers.
# SLF4J(W): Found provider [org.slf4j.jul.JULServiceProvider@4e07b95f]
# SLF4J(W): Found provider [org.neo4j.server.logging.slf4j.SLF4JLogBridge@28b46423]
# SLF4J(W): See https://www.slf4j.org/codes.html#multiple_bindings for an explanation.
# SLF4J(I): Actual provider is of type [org.slf4j.jul.JULServiceProvider@4e07b95f]
# Changed password for user 'neo4j'. IMPORTANT: this change will only take effect if performed before the database is started for the first time.
# 2025-06-02 07:06:22.473+0000 INFO  Logging config in use: File '/var/lib/neo4j/conf/user-logs.xml'
# 2025-06-02 07:06:22.540+0000 INFO  Starting...
# 2025-06-02 07:06:25.174+0000 INFO  This instance is ServerId{6ef3a538} (6ef3a538-da4e-4847-9a8c-ced5240172d2)
# 2025-06-02 07:06:28.773+0000 INFO  ======== Neo4j 5.21.0 ========
# SLF4J(W): Class path contains multiple SLF4J providers.
# SLF4J(W): Found provider [org.slf4j.jul.JULServiceProvider@1cde374]
# SLF4J(W): Found provider [org.neo4j.server.logging.slf4j.SLF4JLogBridge@6818fd48]
# SLF4J(W): See https://www.slf4j.org/codes.html#multiple_bindings for an explanation.
# SLF4J(I): Actual provider is of type [org.slf4j.jul.JULServiceProvider@1cde374]
# 2025-06-02 07:06:46.178+0000 INFO  Called db.clearQueryCaches(): Query cache already empty.
# 2025-06-02 07:06:46.210+0000 INFO  Anonymous Usage Data is being sent to Neo4j, see https://neo4j.com/docs/usage-data/
# 2025-06-02 07:07:06.371+0000 INFO  Bolt enabled on 0.0.0.0:7687.
# 2025-06-02 07:07:08.417+0000 INFO  HTTP enabled on 0.0.0.0:7474.
# 2025-06-02 07:07:08.419+0000 INFO  Remote interface available at http://localhost:7474/
# 2025-06-02 07:07:08.427+0000 INFO  id: 9548E13033C9911929EA0F5EB83CD0B567E495A0F9991AD4467060BB17E9FD1A
# 2025-06-02 07:07:08.428+0000 INFO  name: system
# 2025-06-02 07:07:08.429+0000 INFO  creationDate: 2025-06-02T06:35:32.111Z
# 2025-06-02 07:07:08.429+0000 INFO  Started.
```

访问`http://116.148.120.214:7474/browser/`
[链接](http://116.148.120.214:7474/browser/)

```bash
# 测试插件是否成功加载
return apoc.version()

RETURN apoc.convert.toJson({test: 'value'}) AS json
```

![202506081442358.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081442358.png)

## 安装Neo4j Desktop
[官网](https://neo4j.com/download/)

安装Neo4j Desktop后双击没反应，解决方案如下
1. 断网启动
2. 配置代理
这里选择**配置代理**
配置文件在`C:\Users\{{Administrator}}\.Neo4jDesktop\persist\userData.json`下
```json
{
  "appSettings": {
    "proxyHost": "代理ip",
    "proxyPort": "代理端口",
    "proxyType": "HTTP_PROXY"
  }
}
```
如果还是无法启动，日志文件在`C:\Users\{{Administrator}}\.Neo4jDesktop\log.log`
查看是否有报错
![202506081500340.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081500340.png)
## 安装GDB Console
[安装文档](https://help.aliyun.com/zh/gdb/developer-reference/use-the-open-source-gdb-console-to-log-on-to-gdb)
