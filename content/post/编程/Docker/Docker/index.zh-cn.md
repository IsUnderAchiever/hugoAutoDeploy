---
title: Docker
description: Docker
date: 2024-04-21
slug: Docker
image: dbc-docker-desktop-home.webp
categories:
    - Docker
---

# Docker概念
`镜像`:相当于容器的模板
`容器`:可以理解成轻量级的虚拟机
`镜像仓库`:存放镜像的仓库，官方的镜像仓库 https://hub.docker.com/
以下内容可能会使用到`某些文件`[下载链接](https://wwl.lanzouj.com/iKTW01shae1c)
> 可以使用镜像来不断创建容器，而镜像仓库则是存放镜像的地方
# 命令
> [官网文档](https://docs.docker.com/reference/cli/docker/image/ls/)
```sh
# 查看帮助文档
docker --help
# 查看某一个命令的用法，如 pull
docker pull --help
# 下列命令中，[]括起来的可写可不写
# 下载镜像 标签如版本等，如果不指定标签，则默认是latest，如果不指定仓库，则默认是官方仓库
docker pull [镜像仓库/]镜像名[:标签]
# 查看镜像
docker images [选项]
# 衍生
# 使用帮助手册查看参数 docker images --help
# 查看镜像id
docker images -q
# 查看所有镜像
docker images -a
# 查看所有镜像id
# docker images -aq/docker images -a -q
# 列出镜像名为mysql的镜像
# 例子 docker images --filter=reference='busy*:*libc'
# REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
# busybox             uclibc              e02e811dd08f        5 weeks ago         1.09 MB
# busybox             glibc               21c16b6787c6        5 weeks ago         4.19 MB
docker images --filter=reference='mysql'
# 列出容器信息
# 默认显示正在运行的信息
docker ps [选项]
# 显示全部容器
docke ps -a
# 列出所有退出状态的容器
    docker ps -f status=exited
# 列出所有退出状态的容器id
docker ps -qf status=exited
```
## 容器
### 容器继续运行的条件
> `正常情况下`docker容器运行必须有一个前台进程，如果没有前台进程执行，容器认为空闲，就会自动退出
```sh
# 创建并运行容器
docker run [OPTIONS] IMAGES [COMMAND] [ARG...]
# options 选项，重点关注-d,-p,-v,-e,--restart
# images 镜像信息，推介使用 镜像名:tag 的写法
# command 创建容器后要执行的命令
# ars command命令的参数
# 如
docker run nginx:latest
# 容器运行后执行 ls命令
docker run nginx:latest ls
docker run nginx:latest ls -a
docker run -d nginx:latest ls -a
```
### 容器的运行方式
1. 后台运行 `docker run nginx:latest`
2. 默认运行 `docker run -d nginx:latest`
3. 交互式运行 `docker run -it nginx:latest`
> -i 以交互模式运行容器，通常与-t一起使用
>
> -t 启动容器后，为容器分配一个命令行，通常与-i一起使用
>
> 主要在学习阶段和调试的时候会使用到，并且一般容器的执行命令会使用bash,这样才能在进入容器后去执行命令
>
> `docker run -it nginx:latest bash`如果这样执行，则会覆盖nginx镜像默认的执行命令，此时并不会运行nginx，而是进入容器内部执行bash脚本
>
> 退出交互式`exit`
### 删除容器
```sh
docker rm [选项] [容器ID或容器名...]
# 例如 删除id为"d34295b85f69"的容器
docker rm d34295b85f69
# 删除id为"d34295b85f69"和"c0c9bf25c6a7"的容器
# 容器id使用空格隔开
docker rm d34295b85f69 c0c9bf25c6a7
# 运行中的容器无法直接删除，解决办法
# 1.停止运行容器，然后删除
# 2.强制删除
docker rm -f c0c9bf25c6a7
# $()命令替换
ls $(pwd)
# pwd 会显示当前目录
# ls $(pwd) 会打印当前目录下的内容
# 删除所有容器
docker rm $(docker ps -aq)
# 强制删除所有容器
docker rm -f $(docker ps -aq)
# 删除所有非运行状态的容器
docker rm $(docker ps -f status=exited -q)
```
### 进入容器执行命令
```sh
docker exec [选项] 容器ID或容器名 命令[参数...]
# 如 docker exec 容器id ls
docker exec 295731ae1fdf pwd
# 但是一次只能携带一个命令，是否可以使用交互式的方式
docke exec -it 295731ae1fdf bash
# 使用curl的方式来查看nginx容器是否运行
curl http://127.0.0.1
```
### 查看容器日志
```sh
docker logs [选项] 容器ID或容器名
docker logs 295731ae1fdf
# 持续输出容器中的日志
# 执行完并不会返回，而是会等待是否有新的日志
docker logs -f 295731ae1fdf
# 查看最近的20条日志
docker logs -n 20 295731ae1fdf
```
### 容器文件拷贝
> 使用`docker cp`命令来实现容器与宿主机之间`文件和目录`的相互拷贝
```sh
# 把容器的文件拷贝到宿主机中
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH
# docker cp 295731ae1fdf:/usr/share/nginx/html/index.html /home/index.html
# 把宿主机的文件拷贝到容器中
docker cp [OPTIONS] SRC_PATH CONTAINER:DEST_PATH
# docker cp /home/cp_test 295731ae1fdf:/home/cp_test
```
### 停止容器
```sh
docker stop [容器ID或容器名...]
# docker stop 295731ae1fdf
```
### 运行容器
```sh
docker start [容器ID或容器名...]
# docker start 295731ae1fdf
```
### run命令详解
```sh
-p
# 将宿主机的端口映射到docker容器端口，通过宿主机的端口访问容器端口
# 发布一个端口
docker run -p 宿主机端口:容器端口 镜像名
#发布多个端口
docker run -p 宿主机端口1:容器端口1 -p 宿主机端口2:容器端口2 镜像名
# 怎么知道容器需要发布哪些端口呢？
# 1.DockerHub的介绍
# 2.构建容器的DockerFile(EXPOSE 80)
-v
# 数据卷
```
## 数据卷
为什么要使用数据卷？
1. 需要把容器中的数据持久化保存
2. 需要在宿主机和容器之间完成数据共享
> 将宿主机目录或文件挂载到容器中，实现宿主机和容器之间的数据共享和持久化
### -v
1. -p 对外端口发布，即端口映射
2. -v 数据卷
3. -e 设置环境变量
4. --name 容器命名
5. --restart 容器退出后的重启策略
```sh
docker run -v 数据卷别名:容器目录[:读写权限] 镜像名
docker run -d -p 80:80 -v nginx_html:/usr/share/nginx/html nginx
# 查看数据卷信息
docker inspect fcbc47e85323
# 其中有一段信息如下
"Mounts": [
            {
                "Type": "volume",
                "Name": "nginx_html",
                "Source": "/var/lib/docker/volumes/nginx_html/_data",
                "Destination": "/usr/share/nginx/html",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
# Type为volume，表示数据卷，Source即为我们数据卷的路径
# 可以cd到/var/lib/docker/volumes/nginx_html/_data里ls看一下，对应的是nginx中/usr/share/nginx/html的内容
# 列出数据卷
docker volume ls
# 查看数据卷详情
# 这里nginx_html是别名
docker volume inspect nginx_html
docker volume inspect 06f4db3290e6759861b085989a859257121aac46ad30e35fe26a1904f8ff7aea
# 创建数据卷
# volume_test为数据卷的名字
docker volume create volume_test
# 删除数据卷
docker volume rm volume_test
# 如果需要删除的数据卷正在被使用，是没有办法直接删除的
# 强制删除容器然后删除数据卷
```
```sh
# 手动安装jdk 也可以使用docker安装jdk
# docker安装jdk在后文
# 在/usr/local 目录下安装jdk
cd /usr/local
wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.tar.gz
tar -zxvf jdk-17_linux-x64_bin.tar.gz 
# 将jdk-17改名为java
mv jdk-17.0.10 jdk17
# 进入profile文件，按i进入编辑模式
vim /etc/profile
# 在文件最下方添加
export JAVA_HOME=/usr/local/jdk17
export PATH=$PATH:$JAVA_HOME/bin;
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar;
# 按下Esc退出编辑模式
# 下一步按住shift 再按俩次 z 键，保存配置文件信息
# 重新加载环境变量
source /etc/profile
# 
cd /
java -version
```
**练习**
将某html挂载到nginx目录下进行显示
> **第一个80**端口是宿主机的端口，**第二个80**端口是容器nginx的端口
>
> 将宿主机的`/home/html/index.html`挂载到容器的`/usr/share/nginx/html/index.html`
>
> 不过一般不会直接挂载html，一般挂载整个目录，包括html、css、js等
```sh
docker run -d -p 80:80 -v /home/html/index.html:/usr/share/nginx/html/index.html nginx
```
访问`http://虚拟机ip:80/`
> 如果我们将宿主机的`/home/html/index.html`改变内容，再次访问nginx，是否会发生变化
### -e
容器中某些变量不能直接写死，需要让使用者在创建容器的时候指定，这种情况镜像中一般是定义环境变量来使用。例如mysql容器的root密码。遇到这种镜像创建的容器我就可以使用-e来设置环境变量的值。
用法如下

```sh
docker run -e 变量名=变量值 镜像名
```
**练习**
1. 后台运行mysql容器
2. 容器中的mysql可以被外部连接
3. mysql容器的数据需要持久化存储，不能因为容器被删除而丢失
4. mysql的root用户密码设置为root
```sh
docker run -d -p 3306:3306 -v /home/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root mysql:8.0.20
```
### --name
容器命名
**练习**
发布一个nginx页面，为容器命名为nginx-test
```sh
docker run -d -p 81:80 --name nginx-test -v /home/html:/usr/share/nginx/html nginx
```
### --restart
容器退出后的自动重启策略
```sh
docker run --restart 重启策略 镜像名
# no 不自动重启
# always 总自动重启
# on-failure[:max-retries] 仅在非正常退出时，自动重启
# unless-stopped 容器会在退出后自动重启，除非手动停止
```
### 其他镜像和容器命令
```sh
# 查看镜像详细内容
docker images inspect [OPTIONS] IMAGE [IMAGE...]
docker image inspect 7614ae9453d1
# 查看容器中的进程
docker top CONTAINER[ps CONTAINER]
docker top redis
# 查看容器详细内容
docker inspect 51764b94357b
```
### 运行jar包
```sh
# docker拉取java17
docker pull openjdk:17-jdk
# 运行jar包
java -jar xxx.jar
# 根据application.properties来穿参数
# 运行时参数，比如运行端口为8080，mysql的密码为root
java -jar xxx.jar --server.port=8080 --spring.datasource.username=root --spring.datasource.password=root
docker run -d \
-p 8080:8080 \
-v /usr/blog:/usr/blog \
--restart always \
--name jar_test \
openjdk:17-jdk java -jar /usr/blog/demo-0.0.1-SNAPSHOT.jar 
```
### 网络
```sh
docker inspect mysql
```
![image-20240323184204498](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240323184204498.png)
```sh
# 查看运行的容器
docker ps -f name=jar_test
# 进入容器内部
docker exec -it jar_test bash
# ping mysql试试
ping 172.17.0.3
# 创建网络
docker network create 网络名
# 列出网络信息
docker network ls
# 加入网络有两种方式
# 1.在创建容器时加入
docker run --network 网络名 镜像名
# 2.容器创建后加入
docker network connect [选项] 网络名 容器名或容器id
# 查看网络详情
docker network inspect 网络名或网络id
docker run -d \
-p 8080:8080 \
-v /usr/blog:/usr/blog \
--network blog_test jar_test
--restart always \
--name jar_test \
openjdk:17-jdk java -jar /usr/blog/demo-0.0.1-SNAPSHOT.jar \
"--spring.datasource.url=jdbc:mysql://mysql:3306/test?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&serverTimezone=UTC&useSSL=false&zeroDateTimeBehavior=CONVERT_TO_NULL&allowPublicKeyRetrieval=true"
# 在加入同一个网络后，这里spring.datasource.url并没有写ip地址了，而是写的容器名称mysql
```
## DockerFile
### 基本语法
1. 不区分大小写，但是最好习惯大写
2. 基本以FROM开头
3. `#`开头代表注释
**练习**
自定义一个镜像，运行容器后输出helloworld(如:echo helloworld)
```sh
FROM centos:7
CMD ["echo","helloworld"]
```
> 将文件编译成镜像
>
> 使用`-t`可以指定镜像的名字和标签、`-f`指定基于哪个dockerfile文件来进行构建镜像、`.`表示文件在当前路径下
```sh
docker build -t hello:1.0 -f HelloWorld .
# 测试是否成功
docker run hello:1.0
```
### FROM
> 用来定义基础镜像
>
> `作用时机:构建镜像的时候`
```sh
FROM 镜像名:标签名
FROM centos:7
```
### CMD
> 用来定义容器运行时的默认命令，可以在docker run的时候覆盖掉CMD中定义的命令
```sh
CMD ["命令1","参数1","参数2"]
# CMD ["echo","helloworld"]
# 这种形式可以解析环境变量
# CMD echo hello$HOME
# 这种也可以解析环境变量
# CMD ["sh","-c","hello $HOME"]
# 如果在一个dockerfile中有多个CMD，则只执行最后一个CMD
FROM centos:7
CMD ["echo","helloworld"]
CMD echo hello,java
CMD echo hello,vue
# 测试一下
docker build -t hello:2.0 -f HelloWorld .
```
### ENV
> 用来定义环境变量
>
> `ENV 变量名="变量值"` 如`ENV DIR="/root"`
>
> 作用时机为`构建镜像时`
```sh
ENV DIR="/root"
```
> 需求：要求对打印内容进行改造，需要实现可以打印任意内容，具体内容在运行容器的时候通过环境变量的方式去指定
```sh
FROM centos:7
ENV NAME="小花"
CMD echo hello,$NAME
```
> 编译测试
```sh
docker build -t test01:1.0 -f Test01 .
docker run test01:1.0
# 也可以在运行时指定环境变量
docker run -e NAME=小红 test01:1.0
```
### WORKDIR
> 用于设置当前工作的目录，如果该目录不存在会自动创建。
```sh
WORKDIR 目录
WORKDIR /root/app
```
**示例1**
> pwd打印出来是什么？
```sh
FROM centos:7
WORKDIR /a
WORKDIR b
WORKDIR c
CMD pwd
```
pwd打印`/a/b/c`
**示例2**
> WORKDIR指定目录的父目录不存在会怎么样？
>
> `WORKDIR /a/b/c`
>
> `如果指定目录的父目录不存在，会自动创建`
**示例3**
> WORKDIR指定的目录是否能使用环境变量
```sh
FROM centos:7
ENV PATH="/a"
WORKDIR $PATH
CMD pwd
```
WORKDIR指定的目录`可以`使用环境变量
### RUN
> 用来在构建过程中要执行的命令
>
> 而`CMD`则是在容器运行时执行命令
```sh
RUN echo abc
```
**练习**
> 定义一个CONTENT变量,默认值为hellodocker, 在镜像的/app目录下创建一个sg目录，在其中创建一个content.txt文件, 文件的内容为CQNTENT变量的值。容器启动时打印content.txt的内容
```sh
# 如果是sh脚本，则可以这么实现
export CONTENT="hellodocker"
mkdir -p /app/sg
echo "$CONTENT" > /app/sg/content.txt
cat /app/sg/content.txt
```
> 如何改成`DockerFile`的写法
```sh
FROM centos:7
ENV CONTENT="hellodocker"
WORKDIR /app/sg
RUN echo "$CONTENT" > /app/sg/content.txt
CMD cat /app/sg/content.txt
```
> 测试
```sh
docker build -t test01:1.0 -f Test01 .
docker run test01:1.0
# 或者
docker run -it test01:1.0 bash
ls
cat content.txt 
```
**思考**
```sh
[root@10 dockerfile_test]# docker run -e CONTENT=java  test01:1.0
hellodocker
```
> 为什么`docker run -e CONTENT=java  test01:1.0`的时候，输出的不是`java`
>
> 因为只有`CMD`是在docker运行时执行，而其他指令如`ENV、RUN等`是在构建镜像时执行
>
> 所以输出的还是`hellodocker`
### ADD
> 把构建上下文的文件或者网络文件添加到镜像
>
> 如果文件是压缩包则自动解压，但是需要根据情况，如果是本地压缩包大概率没问题
>
> 但如果是网络上的如`oss压缩包`可能又不会解压，需要根据实际情况来定
>
> 作用时机为构建镜像的时候
```sh
ADD 原路径 目标路径
ADD xxx.tar.gz .
# 随便找一个 html+js+css的网页
# 使用如下命令将html目录打成html.tar.gz压缩包
tar zcvf html.tar.gz html
```
**练习**
在构建目录下存放一个`gz压缩包`， 构建镜像的时候把这个包添加到镜像的/app录下解压，然后把其中的dist目录的内容存放到存放在nginx的htmI目录下，声明开放80端口
> dockerfile:Test01
```sh
FROM nginx:latest
WORKDIR /app
ADD html.tar.gz .
RUN tar -xzvf html.tar.gz
RUN cp -r html/* /usr/share/nginx/html
CMD ["nginx","-g","daemon off;"]
```
> 测试
```
docker build -t test01:1.0 -f Test01 .
```
> 报错如下
```
[+] Building 0.4s (6/7)                                                                                                                                                                                              docker:default
 => [internal] load build definition from Test01                                                                                                                                                                               0.0s
 => => transferring dockerfile: 217B                                                                                                                                                                                           0.0s
 => [internal] load metadata for docker.io/library/nginx:latest                                                                                                                                                                0.0s
 => [internal] load .dockerignore                                                                                                                                                                                              0.0s
 => => transferring context: 2B                                                                                                                                                                                                0.0s
 => [1/4] FROM docker.io/library/nginx:latest                                                                                                                                                                                  0.0s
 => [2/4] WORKDIR /app                                                                                                                                                                                                         0.0s
 => ERROR [3/4] RUN tar -xzvf html.tar.gz                                                                                                                                                                                      0.3s
------                                                                                                                                                                                                                              
 > [3/4] RUN tar -xzvf html.tar.gz:                                                                                                                                                                                                 
0.263 tar (child): html.tar.gz: Cannot open: No such file or directory
0.263 tar (child): Error is not recoverable: exiting now
0.263 tar: Child returned status 2
0.263 tar: Error is not recoverable: exiting now
------
Test01:3
--------------------
   1 |     FROM nginx:latest
   2 |     WORKDIR /app
   3 | >>> RUN tar -xzvf html.tar.gz
   4 |     RUN cp html/* /usr/share/nginx/html
   5 |     CMD ["nginx","-g","daemon off;"]
--------------------
ERROR: failed to solve: process "/bin/sh -c tar -xzvf html.tar.gz" did not complete successfully: exit code: 2
```
可以看到是在解压tar.gz包的时候报错了，好像是没有找到文件
**那怎么调试呢？现在镜像都没构建，更别说进入到容器内部查看文件是否存在了**
> 将Test01改成如下内容
>
> 注释掉报错部分，先创建镜像，然后进入容器内部查看报错原因
```sh
FROM nginx:latest
WORKDIR /app
ADD html.tar.gz .
#RUN tar -xzvf html.tar.gz
#RUN cp -r html/* /usr/share/nginx/html
CMD ["nginx","-g","daemon off;"]
```
> 构建成功
```sh
docker build -t test01:1.0 -f Test01 .
docker run -it test01:1.0 bash
# ls发现tar.gz已经被自动解压了
# 说明ADD一个压缩包，会自动解压
ls
```
> 最终将dockerFile改为如下
```sh
FROM nginx:latest
WORKDIR /app
ADD html.tar.gz .
RUN cp -r html/* /usr/share/nginx/html
CMD ["nginx","-g","daemon off;"]
```
> 构建后运行镜像
>
> 因为我虚拟机已经运行了一个nginx容器了，所以80端口已经被占用了，这里指定的端口是81
```sh
docker run -d -p 81:80 --name=dockerfile_test01 test01:1.0
```
> 访问链接，这里`docker1`是指虚拟机的ip
>
> 我这里是因为配了主机的`hosts`文件
>
> hosts文件路径：C:\Windows\System32\drivers\etc
**`hosts`文件**
```
192.168.56.10   docker1
```
http://docker1:81/
![image-20240324181016560](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240324181016560.png)
### EXPOSE
> 暴露需要发布的端口，让镜像使用者知道应该发布哪些端口
```sh
EXPOSE 端口1 端口2
EXPOSE 80 8080
```
> dockerfile
```sh
FROM nginx:latest
WORKDIR /app
ADD html.tar.gz .
RUN cp -r html/* /usr/share/nginx/html
EXPOSE 80
CMD ["nginx","-g","daemon off;"]
```
### COPY
> 从构建上下文中复制内容到镜像中
```sh
COPY 原路径 目标路径
COPY html.tar.gz .
```
COPY的作用和ADD好像非常类似，具体如何抉择？
1. `ADD`可以下载网络文件,并且可以自动解压
2. `COPY`就是单纯的拷贝
**练习**
在构建目录下存放一个html.tar.gz包，构建镜像的时候把这个包复制到镜像的/app目录下解压，然后把其中的dist目录的内容存放到存放在nginx的htmI目录下，声明开放80端口
```sh
FROM nginx:latest
WORKDIR /app
COPY html.tar.gz .
RUN tar -xzvf html.tar.gz
RUN cp -r html/* /usr/share/nginx/html
EXPOSE 80
CMD ["nginx","-g","daemon off;"]
```
### ENTRYPOINT
> 用来定义容器运行时的默认命令。docker run的时候无法覆盖ENTRYPOINT里的内容。
>
> 运行时机在运行容器的时候
```sh
ENTRYPOINT ["命令1","参数1","参数2"]
ENTRYPOINT ["echo","helloworld"]
```
**实际使用**
> 实际使用往往是存在部分命令不能更改，也存在部分命令可以被更改
>
> 比如`java -jar aaa.jar`
>
> 我们希望`java -jar`这部分命令是固定的，无法在运行时被修改
>
> 而`jar`包的名称可以在运行时修改
```sh
ENTRYPOINT ["java","-jar"]
CMD ["aaa.jar"]
```
**练习**
在上个案例要求的基础上，还要求容器的nginx的启动命令不能在运行容器时被覆盖
> 分析一下这个练习
```sh
docker run -d -p 82:80 test01:1.0 bash
```
执行上述命令后，容器处于退出状态`Exited `
为啥呢？因为`bash`将dockerfile最后一行的cmd命令覆盖了`CMD ["nginx","-g","daemon off;"]`
```sh
FROM nginx:latest
WORKDIR /app
COPY html.tar.gz .
RUN tar -xzvf html.tar.gz
RUN cp -r html/* /usr/share/nginx/html
EXPOSE 80
ENTRYPOINT ["nginx","-g","daemon off;"]
```
```sh
docker build -t entrypoint_test02:1.0 -f Test01 .
docker run -d -p 82:80 entrypoint_test02:1.0 bash
```
> 发现容器依然没有运行，查看容器日志
```sh
docker run -d -p 82:80 entrypoint_test02:1.0 bash
#fa075623cdbf3d01ead7b405c8d61b224e0f70e9fffcdb2a3c5bd91100ecd35d
docker logs fa075623cdbf3d
#nginx: invalid option: "bash"
```
> 因为此时在`nginx`容器内执行了`bash`命令，nginx无法识别
## 优化部署流程
> 我们希望后端的镜像当中就包含了后端的jar包，镜像启动的时候默认就是会启动该jar包，前端的镜像中就包含了前端的包
### 基础版
#### 配置nginx反向代理
`/etc/nginx/conf.d/default.conf`
```conf
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;
    #access_log  /var/log/nginx/host.access.log  main;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    location ^~/api/{
        proxy_pass http://192.168.56.10:8080/;
    }
    #error_page  404              /404.html;
    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}
    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}
    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```
> 主要是添加如下代码
```conf
...
	location ^~/api/{
        proxy_pass http://192.168.56.10:8080/;
    }
...
```
```sh
# 我这里的nginx的镜像名为nginx-test
docker inspect volume nginx-test
#        "Mounts": [
#            {
#                "Type": "bind",
#                "Source": "/home/html",
#                "Destination": "/usr/share/nginx/html",
#                "Mode": "",
#                "RW": true,
#                "Propagation": "rprivate"
#            }
#        ],
```
而nginx镜像配置的数据卷在宿主机的路径为`/home/html`，所以我们将前端文件放在宿主机的`/home/html`路径下
```sh
ll
#total 16
#drwxr-xr-x. 2 root root   57 Mar 24 13:13 assets
#-rw-r--r--. 1 root root 1169 Mar 24 15:07 default.conf
#-rw-r--r--. 1 root root  456 Mar 24 13:13 index.html
#-rw-r--r--. 1 root root  834 Mar 24 14:06 nginx.conf
#-rw-r--r--. 1 root root 1497 Mar 20 23:01 vite.svg
pwd
#/home/html
```
到时候前端输入`虚拟机ip:80/`就会显示`/home/html/index.html`文件
如`http://192.168.56.10:80/`显示前端页面
#### 运行后端
> 注意更换自己mysql的账号和密码
```sh
docker run -d \
-p 8080:8080 \
-v /usr/blog:/usr/blog \
--restart always \
--name jar_test \
openjdk:17-jdk java -jar /usr/blog/demo.jar  --spring.datasource.username=root --spring.datasource.password=123456
```
> 这里将jar包放在了`/usr/blog`下，也可以自己定义路径
#### MySQL
还需要配置mysql,新建一个`test`数据库
```mysql
create table user
(
    id        int auto_increment comment '用户编号'
        primary key,
    username  varchar(20)  null comment '账号',
    password  varchar(20)  null comment '密码',
    nick_name varchar(20)  null comment '昵称',
    avatar    varchar(255) null comment '头像'
)
    comment '用户表';
INSERT INTO test.user (id, username, password, nick_name, avatar) VALUES (1, 'admin', '123456', '默认昵称', null);
```
### 优化
> dockerfile
```sh
FROM openjdk:17-jdk
WORKDIR /app
ADD demo.jar .
EXPOSE 8080
ENTRYPOINT ["java","-jar"]
CMD ["demo.jar"]
```
![image-20240325062118495](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240325062118495.png)
```sh
cd /usr/dockerfile_test
docker build -t demo_jar:1 -f /usr/dockerfile_test/demo_jar .
# network 添加到同一个网络
# mysql也需要
docker run -d \
-p 8080:8080 \
--network blog_test \
--restart always \
--name jar_demo \
demo_jar:1
```
> 前端
>
> `demo_ngxin`是dockerfile文件
>
> blog-vue下则是前端文件
![image-20240325071054576](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240325071054576.png)
```sh
FROM nginx:latest
WORKDIR /app
COPY blog-vue .
RUN cp -r dist/* /usr/share/nginx/html
EXPOSE 80
ENTRYPOINT ["nginx","-g","daemon off;"]
```
```sh
docker build -t blog_vue:1 -f demo_nginx .
docker run -d \
-p 80:80 \
--restart always \
-d \
--name blog_vue \
blog_vue:1
```
> 访问发现后端接口404，是因为基于nginx镜像`FROM nginx:latest`，这里反向代理依然需要配置
>
> 解决方法：进入`blog_vue`容器内部重新配置反向代理
```sh
docker exec -it blog_vue bash
```
## 镜像推送到镜像仓库
### 注册`Docker Hub`账号
官网:https://hub.docker.com/
### 登录镜像仓库
```sh
docker login
# 然后输入账号密码即可登录
```
### 构建镜像
```sh
docker build -t username/镜像名:tag .
# username是Docker Hub的帐号
```
### 给镜像打上标签
> 以便于与`Docker Hub`上的仓库关联
```sh
# 第一个镜像名是本地的镜像名
# 第二个镜像名是Docker Hub仓库中的镜像名
docker tag username/镜像名:tag username/镜像名:tag
```
### 推送镜像
```sh
docker push username/镜像名:tag
```
## DockerCompose
### 入门
> DockerCompose是用来定义和运行一个或多个容器(通常都是多个)运行和应用的工具
>
> 当前版本的Docker不需要单独安装
```sh
docker compose version
# 查看版本
```
```yaml
services:
  test:
    image: nginx:latest
  test2:
    image: nginx:latest
```
```sh
# 运行以上定义的所有镜像
docker compose up
# -d 后台运行
docker compose up -d
# 会停止并删除yaml中运行的容器
docker compose down
```
### 元素
1. command 覆盖容器启动后的默认指令
2. environment 指定环境变量，相当于run的-e选项
3. image 用来指定镜像
4. networks 指定网络, 相当于run的--network
5. ports 用来指定要发布的端口 ，相当于run的-p
6. volumes 用来指定数据卷， 相当于-V
7. restart 用来指定重启策略,相当于--restart

docker-compose.yml:

```yaml
services:
  # mysql
  blog_mysql:
    image: mysql:8.0.20
    # 已经存在的数据卷 mysql_data
    volumes:
      - mysql_data:/var/lib/mysql
    ports:
      - 3306:3306
    environment:
      MYSQL_ROOT_PASSWORD: 123456
    restart: always
    networks:
      - blog_net
  # redis
  blog_redis:
    image: redis:latest
    volumes:
      - redis_data:/data
    ports:
      - 6379:6379
    restart: always
    command: ['redis-server','--appendonly','yes']
    networks:
      - blog_net
  # 后端服务
  sg_blog:
    image: sb_blog:01
    ports:
      - 8080:8080
    networks:
      - blog_net
    restart: always
  # 前端服务
  sg_blog_vue:
    image: sb_blog_vue:01
    ports:
      - 80:80
    restart: always
networks:
  blog_net:
volumes:
  mysql_data:
    # 表示数据卷已经存在，不需要自动创建
    external: true
  redis_data:
    external: true
```
> 设置好`docker-compose.yaml`后
```sh
docker compose up -d
docker compose down
```
