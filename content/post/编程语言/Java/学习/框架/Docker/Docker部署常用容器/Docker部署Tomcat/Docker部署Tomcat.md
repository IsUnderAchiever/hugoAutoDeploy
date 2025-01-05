---
title: Docker部署Tomcat
description: Docker部署Tomcat
date: 2025-01-01
slug: Docker部署Tomcat
image: dbc-docker-desktop-home.webp
categories:
    - Docker
---
# Docker部署Tomcat

> 文末处有下载链接

## 部署流程
```sh
docker run -d -it -p 8080:8080 --name=tomcat tomcat:8.5.87
```

> 访问8080端口发现404，详情可以查看该[博客](https://cloud.tencent.com/developer/article/1704573)
> 进行修改操作，首先进入容器内部,在`/usr/local/tomcat/`目录下有`webapps`目录和`webapps.dist`目录
> 直接删除`webapps`目录，将`webapps.dist`改名为`webapps`，然后重启docker容器即可

```sh
# centos检查防火墙
# 查看防火墙状态
service iptables status
# 停止防火墙
service iptables stop

# firewalld 防火墙
# 查看firewall服务状态(出现Active:active(running)是启动状态，Active:inactive(dead)是停止状态)
systemctl status firewalld

# 关闭firewall.service服务
systemctl stop firewalld

# ubuntu防火墙操
# 查看防火墙状态
sudo ufw status
# 默认允许外部访问本机
sudo ufw default allow
# 默认拒绝外部访问主机
sudo ufw default deny
# 关闭防火墙
sudo ufw disable

# 进入容器内部
docker exec -it tomcat /bin/bash

# 查看下当前目录
pwd
# /usr/local/tomcat

rm -rf webapps
mv webapps.dist webapps

# 重启容器
docker restart tomcat
# 再次访问8080，发现可以访问了

# 永久解决这个问题
# 如果使用tomcat继续部署容器的话，还是需要重复以上操作，我们不希望每次都要这样
# 所以自己`commit`容器副本之后使之成为一个新的镜像

docker commit -m=“提交的描述信息” -a=“作者” 容器ID 要创建的目标镜像名:[标签名]

# 比如
docker commit -m="修复404" -a="tong" tomcat my_tomcat:8.5.87

# 删除原来的容器，并使用这个镜像部署新的tomcat容器
docker rm -f tomcat

docker run -d -it -p 8080:8080 --name=tomcat my_tomcat:8.5.87
```

## 部署Activiti流程设计器

### 步骤

> 接下来将部署Activiti流程设计器,下载[链接](https://www.activiti.org/get-started)
> 解压之后在`activiti-6.0.0\wars`下有三个`war`包，我们需要部署到`tomcat`容器上

```sh
# 这里我已经将三个war包上传到服务器的`/home/tong/software/docker/images/activiti/war`里了
# 将war包复制到tomcat容器内
docker cp /home/tong/software/docker/images/activiti/war/activiti-admin.war tomcat:/usr/local/tomcat/webapps/
docker cp /home/tong/software/docker/images/activiti/war/activiti-app.war tomcat:/usr/local/tomcat/webapps/
docker cp /home/tong/software/docker/images/activiti/war/activiti-rest.war tomcat:/usr/local/tomcat/webapps/

# 重启一下容器
docker restart tomcat

# 查看日志
docker logs -f tomcat

# 报了一堆错
# Caused by: javax.persistence.PersistenceException: [PersistenceUnit: persistenceUnit] Unable to build EntityManagerFactory
# Caused by: org.hibernate.cfg.beanvalidation.IntegrationException: Error activating Bean Validation integration
# Caused by: java.lang.NoClassDefFoundError: Could not initialize class org.hibernate.validator.internal.engine.ConfigurationImpl
# 访问http://ubuntu-notebook:8080/activiti-app也404

# 进入容器查看java版本，发现是17，我记得好像不支持jdk17，换一个jdk8的tomcat试试吧
docker exec -it tomcat /bin/bash
java -version

docker rm -f tomcat
# tomcat:7.0.94并不会报404，所以不必重复上述操作
# 进入容器内部发现是jdk8
docker run -d -it -p 8080:8080 --name=tomcat tomcat:7.0.94

docker cp /home/tong/software/docker/images/activiti/war/activiti-admin.war tomcat:/usr/local/tomcat/webapps/
docker cp /home/tong/software/docker/images/activiti/war/activiti-app.war tomcat:/usr/local/tomcat/webapps/
docker cp /home/tong/software/docker/images/activiti/war/activiti-rest.war tomcat:/usr/local/tomcat/webapps/

# 重启一下容器
docker restart tomcat

# 查看日志
docker logs -f tomcat

# 虽然也出现了一些报错，不过页面能进去了
# 可能是默认配置的持久化，后面我们改成mysql即可
### Cause: org.h2.jdbc.JdbcSQLException: Table "ACT_RU_JOB" not found; SQL statement:
```

`用户名/密码`是:`admin/test`

![image-20250101120727141](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202501011601471.png)

### 持久化

> 修改`activiti-app.war\WEB-INF\classes\META-INF\activiti-app\activiti-app.properties`文件
>
> 记得提前新建`activiti6ui`数据库，表不用管，启动时会自动初始化表

```properties
# datasource.driver=org.h2.Driver
# datasource.url=jdbc:h2:mem:activiti;DB_CLOSE_DELAY=-1

datasource.driver=com.mysql.cj.jdbc.Driver
datasource.url=jdbc:mysql://192.168.0.16:3306/activiti6ui?characterEncoding=UTF-8&nullCatalogMeansCurrent=true&characterEncoding=utf8&useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true

datasource.username=root
datasource.password=123456

# hibernate.dialect=org.hibernate.dialect.H2Dialect
hibernate.dialect=org.hibernate.dialect.MySQLDialect
#hibernate.dialect=org.hibernate.dialect.Oracle10gDialect
#hibernate.dialect=org.hibernate.dialect.SQLServerDialect
#hibernate.dialect=org.hibernate.dialect.DB2Dialect
#hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
```

> 直接使用`winrar`打开war包，将文件拖出来修改，改完之后直接拖回去覆盖即可，无需解压
>
> 注意mysql版本，如果是mysql8，需要将maven里的mysql包放入war包中
>
> 在`activiti-app.war\WEB-INF\lib`目录下，删除原来的`mysql-connector-java-5.1.30.jar`
>
> 上传现在的`mysql-connector-java-8.0.17.jar`
>
> 将修改好的war包上传到服务器上进行部署，依然是上述流程，这里不再演示

![image-20250101123807292](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202501011601630.png)

可以看到已经自动初始化了这些表，持久化已经完成了

### 汉化

> 参考[博客1](https://www.jianshu.com/p/f3947908d18a)、[博客2](https://blog.csdn.net/qq_19734597/article/details/87784216)、[博客3](https://blog.csdn.net/qq_26458903/article/details/90409026)
>
> 这里不再演示，直接放链接

原版链接

1. [activiti-admin](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202501011253297.war)
2. [activiti-app](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202501011253298.war)
3. [activiti-rest](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202501011253299.war)

汉化版链接:

> 这款汉化版不够彻底，需要参考下这个[博客](https://blog.csdn.net/qq_26458903/article/details/90409026)的配置

[activiti-app](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202501011253300.war)

### DockerCompose部署

> 在连接mysql时候直接写`ip`还是硬伤，如果有一种方法能直接指定就好了
>
> 在Docker里如果多个容器在同一个网络`(network)`下，可以直接使用`容器名`进行通信，无需使用ip
>
> 创建一个`docker-compose.yml`文件，内容如下
>
> `activiti_tomcat`镜像是我打包的的一个`activiti汉化`镜像，下载链接在文末
>
> `activiti_tomcat`会链接在同一个网络下的`activiti_mysql`容器作为持久化数据库
>
> 这里mysql为了避免端口冲突，我改成了`3307`端口
> 但是`activiti_tomcat`配置文件里连接还是填写`3306`端口，否则会连不上

```yml
services:
  activiti_mysql:
    image: mysql:latest
    container_name: activiti_mysql # 指定容器名称
    environment:
      # MYSQL_USER: root 注意不要加这一行，删掉
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_PASSWORD: 123456
    volumes:
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql # 挂载本地的init.sql到容器内的/docker-entrypoint-initdb.d/
    networks:
      - activiti_network
    ports:
      - "3307:3306"
    restart: always

  activiti_tomcat:
    image: activiti_tomcat:latest
    container_name: activiti_tomcat # 指定容器名称
    ports:
      - "8012:8080"
    depends_on:
      - activiti_mysql
    networks:
      - activiti_network
    restart: always

networks:
  activiti_network:
```

**`注意`**如果加了`MYSQL_USER: root`这一行，在mysql容器初始化的时候就会报错`ERROR 1396 (HY000) at line 1: Operation CREATE USER failed for 'root'@'%'`导致没有初始化数据库，从而导致activiti容器连不上数据库，这一行要删掉

> `init.sql`

```sql
-- init.sql 内容如下：
CREATE DATABASE IF NOT EXISTS activiti6ui;
USE activiti6ui;

-- 这里可以添加更多初始化SQL语句，如创建表、插入初始数据等。
```

这里演示一下创建过程

>主要注意下端口是否被占用，`activiti_tomcat`容器绑定的是宿主机的`8012`端口
>
>`activiti_mysql`绑定的是宿主机的`3307`端口

![image-20250101155458911](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202501011602976.png)

```sh
# 在当前目录下有两个文件`docker-compose.yml`和`init.sql`，内容在上面已经贴出来了
root@tong-HP-Laptop-15s-du1xxx:/../docker-compose# ll
总计 16
drwxr-xr-x 2 root root 4096  1月  1 15:00 ./
drwxr-xr-x 4 root root 4096  1月  1 14:55 ../
-rw-r--r-- 1 root root  713  1月  1 15:48 docker-compose.yml
-rw-r--r-- 1 root root  180  1月  1 14:55 init.sql
# 开始创建
root@tong-HP-Laptop-15s-du1xxx:/../docker-compose# docker compose up -d
[+] Running 3/3
 ✔ Network docker-compose_activiti_network  Created                                                                                                                                                                                     0.2s 
 ✔ Container activiti_mysql                 Started                                                                                                                                                                                     0.6s 
 ✔ Container activiti_tomcat                Started                                                                                                                                                                                     1.1s 
# 查看容器是否创建
root@tong-HP-Laptop-15s-du1xxx:/../docker-compose# docker ps --filter name=activiti_tomcat --filter name=activiti_mysql
CONTAINER ID   IMAGE                    COMMAND                   CREATED              STATUS              PORTS                                                    NAMES
087392a2c4ac   activiti_tomcat:latest   "catalina.sh run"         About a minute ago   Up About a minute   0.0.0.0:8012->8080/tcp, [::]:8012->8080/tcp              activiti_tomcat
3574a781ae9f   mysql:latest             "docker-entrypoint.s…"   About a minute ago   Up About a minute   33060/tcp, 0.0.0.0:3307->3306/tcp, [::]:3307->3306/tcp   activiti_mysql
```

![image-20250101160015096](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202501011602152.png)

![image-20250101160036444](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202501011602432.png)

> 以上是汉化之后的的效果
>
> 以下是下载链接

activiti_tomcat镜像[链接](https://pan.baidu.com/s/1A9lVyppWHkxwb0HFvnigcw?pwd=qigs)

activiti_tomcat是自制的activiti流程设计器docker镜像，默认会连接同一个网络下activiti_mysql容器进行持久化(注意提前创建activiti6ui数据库)

这里镜像有点大我是用命令压缩过一次了，这里传不上去github，我就直接传百度网盘了

```sh
# 解压xxx.tar.gz压缩包
tar -zxvf xxx.tar.gz
```
