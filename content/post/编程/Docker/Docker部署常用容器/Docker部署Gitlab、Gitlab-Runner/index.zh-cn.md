---
title: Docker部署Gitlab、Gitlab-Runner
description: Docker部署Gitlab、Gitlab-Runner
date: 2024-12-28
slug: Docker部署Gitlab、Gitlab-Runner
image: dbc-docker-desktop-home.webp
categories:
    - Docker
---
# Docker部署Gitlab、Gitlab-Runner

> 参考[视频](https://www.bilibili.com/video/BV18z421e7pL/?spm_id_from=333.337.search-card.all.click)

## 部署Gitlab容器

```sh
# 删除之前创建的容器
docker rm -f gitlab
# 删除数据卷
docker volume rm gitlab-etc
docker volume rm gitlab-log
docker volume rm gitlab-opt

docker volume create gitlab-etc
docker volume create gitlab-log
docker volume create gitlab-opt

docker pull gitlab/gitlab-ce:17.7.0-ce.0

# 启动容器
# 注意这里指定了hostname，这点非常重要，会影响到后序配置runner，改成主机的ip即可
# 这里gitlab版本号需要和后续的gitlab-runner匹配
docker run \
 -itd  \
 -p 9980:80 \
 -p 9922:22 \
 -p 9943:443 \
 -v gitlab-etc:/etc/gitlab  \
 -v gitlab-log:/var/log/gitlab \
 -v gitlab-opt:/var/opt/gitlab \
 --restart always \
 --privileged=true \
 --name gitlab \
 --hostname 192.168.42.128 \
gitlab/gitlab-ce:latest

docker logs -f gitlab

# 查看gitlab初始root密码
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
# Password: DMLAQsQVh5YIi5+y4/b3nkAAXiZHscm2RGoACUc41R4=

#进容器内部
docker exec -it gitlab /bin/bash
 
#修改gitlab.rb
vi /etc/gitlab/gitlab.rb
 
#加入如下
# ==========================================
#gitlab访问地址，可以写域名。如果端口不写的话默认为80端口
external_url 'http://192.168.42.128:9980'
#ssh主机ip
gitlab_rails['gitlab_ssh_host'] = '192.168.42.128'
#ssh连接端口
gitlab_rails['gitlab_shell_ssh_port'] = 9922
# 显式设置 Nginx 监听端口
nginx['listen_port'] = 80
nginx['listen_https'] = false
# ==========================================

# 让配置生效
gitlab-ctl reconfigure

# /var/opt/gitlab/nginx/conf/gitlab-http.conf
# 这是容器内nginx的配置文件，如果有需要可以进行修改，我们这里默认即可

vi /opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml
# ==========================================
# 改为如下配置
gitlab:
  host: 192.168.42.128
  port: 9980 # 这里改为9980
  https: false
# ==========================================

#重启gitlab 
gitlab-ctl restart

# 可以直接在这里修改root密码，或者在ui页面进行修改

# 此时依然是在容器内部
# 进入控制台
gitlab-rails console -e production
 
# 查询id为1的用户，id为1的用户是超级管理员
user = User.where(id:1).first
# 修改密码为tong123456，另外密码设置有强度检测，不要设置`123456`，会失败的
user.password='tong123456'
# 保存
user.save!
# 退出
exit
```

> 登陆后，先重置一下密码

![image-20241227214209162](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281320950.png)

![image-20241227214225177](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281320530.png)

![image-20241227214244718](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281320146.png)

![image-20241227214309591](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281320526.png)

## 准备Gitlab-Runner

```sh
# 配置jdk21
cd /usr/local/
wget https://download.oracle.com/java/21/latest/jdk-21_linux-x64_bin.tar.gz

tar -zxvf jdk-21_linux-x64_bin.tar.gz

cat >> /etc/profile <<-'EOF'
export JAVA_HOME=/usr/local/jdk-21.0.5
export PATH=$PATH:$JAVA_HOME/bin
EOF

source /etc/profile
java -version

# 配置maven
cd /usr/local/
wget https://dlcdn.apache.org/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.tar.gz

tar -zxvf apache-maven-3.9.9-bin.tar.gz

cat >> /etc/profile <<-'EOF'
export MVN_HOME=/usr/local/apache-maven-3.9.9
export PATH=$PATH:$MVN_HOME/bin
EOF

source /etc/profile
mvn --version

# 配置gitlab-runner
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner//script.rpm.sh | sudo bash
# 提示:The repository is setup! You can now install packages.

# 下载一直超时，最后还是使用代理下载的
export https_proxy=http://192.168.0.16:7899
export http_proxy=http://192.168.0.16:7899
# 下载完之后一定要取消代理
unset http_proxy
unset https_proxy

# 安装gitlab-runner，版本与gitlab镜像保持一致(版本相差不大即可)
yum install -y gitlab-runner-17.7.0-1

# 虽然gitlab runner安装好了，但是现在还不能直接用
# 默认gitlab runner使用的是一个低权限的用户`gitlab-runner`，这会对之后的cicd有很大的影响
# 查看用户，这个用户权限太低了
cut -d: -f1 /etc/passwd

# 执行以下命令，删除gitlab-runner原始信息
gitlab-runner uninstall

# 安装并设置--user(设置为root),工作目录是/home/gitlab-runner
gitlab-runner install --working-directory /home/docker/gitlab/gitlab-runner --user root

# 重启gitlab-runner
systemctl daemon-reload
systemctl start gitlab-runner
systemctl enable gitlab-runner

# 查看当前runner用户
ps aux|grep gitlab-runner
```

![image-20241227224134959](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281320796.png)

![image-20241227224211312](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281320765.png)

> 写上`shared`，直接点击`create`

![image-20241228102543566](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281320057.png)

> 复制命令进行注册
>
> 我这里注册失败过很多次，分享一下排查过程，首先说我的原因，是因为挂了`http`代理

```sh
# 首先检查gitlab和gitlab-runner版本是否匹配
# 我刚开始curl http://192.168.42.128:9980是没有响应的
# 然后修改容器内部的配置文件/etc/gitlab/gitlab.rb，以下是原来的配置文件
external_url 'http://192.168.42.128'
gitlab_rails['gitlab_ssh_host'] = '192.168.42.128'
gitlab_rails['gitlab_shell_ssh_port'] = 9922
# 修改为如下配置
external_url 'http://192.168.42.128:9980'
gitlab_rails['gitlab_ssh_host'] = '192.168.42.128'
gitlab_rails['gitlab_shell_ssh_port'] = 9922
nginx['listen_port'] = 80
nginx['listen_https'] = false

# 执行命令，生效配置
gitlab-ctl reconfigure

# /var/opt/gitlab/nginx/conf/gitlab-http.conf文件时gitlab内部nginx的配置文件
# 我这里默认即可，监听80端口

# /opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml配置文件配置如下
gitlab:
  host: 192.168.42.128
  port: 9980
  https: false

# 查看nginx的配置文件/var/log/gitlab/nginx/error.log
# 我这里文件为空，没有内容

# 这样修改配置之后执行以下命令
curl -v http://192.168.42.128:9980

# 此时有响应了，响应如下
* Rebuilt URL to: http://192.168.42.128:9980/
* Uses proxy env variable http_proxy == 'http://192.168.0.16:7899'
*   Trying 192.168.0.16...
* TCP_NODELAY set
* Connected to 192.168.0.16 (192.168.0.16) port 7899 (#0)
> GET http://192.168.42.128:9980/ HTTP/1.1
> Host: 192.168.42.128:9980
> User-Agent: curl/7.61.1
> Accept: */*
> Proxy-Connection: Keep-Alive
> 
< HTTP/1.1 502 Bad Gateway
< Connection: keep-alive
< Keep-Alive: timeout=4
< Proxy-Connection: keep-alive
< Content-Length: 0
< 
* Connection #0 to host 192.168.0.16 left intact

# 原因是因为挂了http代理，前面yum下载超时才挂的代理
# 取消代理
unset http_proxy
unset https_proxy

# 重新注册之后成功，结果如下
[root@192 ~]# gitlab-runner register  --url http://192.168.42.128:9980  --token glrt-t1_tjt1C9_KNQJRRiRc37-k
Runtime platform                                    arch=amd64 os=linux pid=209111 revision=3153ccc6 version=17.7.0
Running in system-mode.                            
                                                   
Enter the GitLab instance URL (for example, https://gitlab.com/):
[http://192.168.42.128:9980]: 
Verifying runner... is valid                        runner=t1_tjt1C9
Enter a name for the runner. This is stored only in the local config.toml file:
[192.168.42.128]: 
Enter an executor: ssh, docker, docker-windows, docker-autoscaler, shell, parallels, virtualbox, docker+machine, kubernetes, instance, custom:
shell
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
 
Configuration (with the authentication token) was saved in "/etc/gitlab-runner/config.toml" 
```

![image-20241228103712513](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281320103.png)

此时runner已经进入`Online`状态

> gitlab配置`ssh`参考[博客](https://blog.csdn.net/weixin_45710060/article/details/121752880)
>
> 这里我就直接省略了
>
> 这里在`gitlab`新建一个项目

![image-20241228104030641](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281321015.png)

![image-20241228104048753](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281321867.png)

![image-20241228104127834](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281321485.png)

![image-20241228104300647](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281321622.png)

> 这里先把代码`clone`下来
>
> 先随便写个demo即可，重点关注项目下的两个文件
>
> 新建两个文件`.gitlab-ci.yml`和`Dockerfile`

![image-20241228115239957](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281321320.png)

**Dockerfile**

```dockerfile
FROM openjdk:21
ADD ./target/boot-demo-0.0.1-SNAPSHOT.jar /usr/local
CMD ["java","-jar","/usr/local/boot-demo-0.0.1-SNAPSHOT.jar"]
EXPOSE 8080
```

**.gitlab-ci.yml**

```yaml
stages:
  - build
  - build-image
  - push-image

build:
  stage: build
  rules:
    - if: '$CI_COMMIT_TAG'
      allow_failure: false
    - when: never
  tags:
    - shared
  script:
    - mvn clean
    - mvn package
  artifacts:
    paths:
      - target/*.jar

build-image:
  stage: build-image
  rules:
    - if: '$CI_COMMIT_TAG'
      allow_failure: false
    - when: never
  tags:
    - shared
  script:
    - docker build -t tong/boot-demo:$CI_COMMIT_TAG .

push-image:
  stage: push-image
  rules:
    - if: '$CI_COMMIT_TAG'
      allow_failure: false
    - when: never
  tags:
    - shared
  script:
    - docker push 192.168.0.16:8010/tong/boot-demo:$CI_COMMIT_TAG
```

> 为防止后续部署报错，建议先手动执行下，看看是否能运行起来

```sh
mvn clean package

# 进入target目录
java -jar boot-demo-0.0.1-SNAPSHOT.jar

# 启动没问题就行
```

> 创建token，在push代码时需要用到

![image-20241228114809975](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281321228.png)

> 勾选需要的权限，点击创建

![image-20241228114922692](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281321277.png)

> push代码时，填入对应的token

![image-20241228114937324](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281321438.png)

> 可以看到现在代码已经push上来了

![image-20241228115504787](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281321995.png)

![image-20241228115529433](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281321522.png)

![image-20241228115546297](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281322187.png)

> 注意分支要选择自己提交代码的分支

![image-20241228115637008](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281322181.png)

> 这里build失败是因为`gitlab yml`里的命令有问题，先看看`build`的镜像是否能正常运行

![image-20241228124144668](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281322268.png)

稍作修改

![image-20241228124227087](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281322319.png)

前面`build`是成功了

查看下本地镜像

```sh
[root@192 jar]# docker image ls tong/boot-demo
REPOSITORY       TAG              IMAGE ID       CREATED          SIZE
tong/boot-demo   0.0.1-SNAPSHOT   58b8922d0b3d   13 minutes ago   524MB
[root@192 jar]# docker run -d --name boot-demo -p 8012:8080 tong/boot-demo:0.0.1-SNAPSHOT
6af3dad70224e9d613da84ad5d7e1317733107749a5f8546683c3f353928a4ff
[root@192 jar]# docker logs boot-demo

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::               (v2.6.13)

2024-12-28 04:44:56.844  INFO 1 --- [           main] org.tong.TongApplication                 : Starting TongApplication using Java 21 on 6af3dad70224 with PID 1 (/usr/local/boot-demo-0.0.1-SNAPSHOT.jar started by root in /)
2024-12-28 04:44:56.846  INFO 1 --- [           main] org.tong.TongApplication                 : No active profile set, falling back to 1 default profile: "default"
2024-12-28 04:44:57.332  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2024-12-28 04:44:57.339  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2024-12-28 04:44:57.339  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.68]
2024-12-28 04:44:57.382  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2024-12-28 04:44:57.383  INFO 1 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 505 ms
2024-12-28 04:44:57.575  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2024-12-28 04:44:57.580  INFO 1 --- [           main] org.tong.TongApplication                 : Started TongApplication in 0.969 seconds (JVM running for 1.201)
```

> 可以看到这里已经成功启动了，访问下试试

![image-20241228124606450](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281322496.png)

> 接口正常，接下来重新修改下`dockerfile`完整过一遍

```sh
# 先把原来的容器删掉
docker rm -f boot-demo
```

> 我这里是搭建了私人的docker仓库，如果需要push到`dockerhub`，需要先登录
>
> 命令改成`docker push tong/boot-demo:$CI_COMMIT_TAG`即可

```sh
# 登录dockerhub
docker login
```

```dockerfile
stages:
  - build
  - build-image
  - tag-image
  - push-image
  - run-image

build:
  stage: build
  rules:
    - if: '$CI_COMMIT_TAG'
      allow_failure: false
    - when: never
  tags:
    - shared
  script:
    - mvn clean
    - mvn package
  artifacts:
    paths:
      - target/*.jar

build-image:
  stage: build-image
  rules:
    - if: '$CI_COMMIT_TAG'
      allow_failure: false
    - when: never
  tags:
    - shared
  script:
    - docker build -t tong/boot-demo:$CI_COMMIT_TAG .

tag-image:
  stage: tag-image
  rules:
    - if: '$CI_COMMIT_TAG'
      allow_failure: false
    - when: never
  tags:
    - shared
  script:
    - docker tag tong/boot-demo:$CI_COMMIT_TAG 192.168.0.16:8010/tong/boot-demo:$CI_COMMIT_TAG

push-image:
  stage: push-image
  rules:
    - if: '$CI_COMMIT_TAG'
      allow_failure: false
    - when: never
  tags:
    - shared
  script:
    - docker push 192.168.0.16:8010/tong/boot-demo:$CI_COMMIT_TAG

run-image:
  stage: push-image
  rules:
    - if: '$CI_COMMIT_TAG'
      allow_failure: false
    - when: never
  tags:
    - shared
  script:
    - docker run -d --name boot-demo -p 8012:8080 192.168.0.16:8010/tong/boot-demo:$CI_COMMIT_TAG
```

![image-20241228124846469](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412281322016.png)

可以看到这里已经成功了，访问接口也没问题

> pom.xml
>
> 设置`<finalName>boot-demo</finalName>`表示打包之后的名字

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>boot-demo</artifactId>
    <version>0.0.2-SNAPSHOT</version>
    <name>boot-demo</name>
    <packaging>jar</packaging>
    <description>boot-demo</description>
    <properties>
        <java.version>17</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.6.13</spring-boot.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <finalName>boot-demo</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.10.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot.version}</version>
                <configuration>
                    <mainClass>org.tong.TongApplication</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

> dockerfile

```dockerfile
FROM openjdk:21
ADD ./target/boot-demo.jar /usr/local
CMD ["java","-jar","/usr/local/boot-demo.jar"]
EXPOSE 8080
```

>`.gitlab-ci.yml`

```yml
stages:
  - build
  - build-image
  - tag-image
  - push-image
  - run-image

build:
  stage: build
  rules:
    - if: '$CI_COMMIT_TAG'
      allow_failure: false
    - when: never
  tags:
    - shared
  script:
    - mvn clean
    - mvn package
  artifacts:
    paths:
      - target/*.jar

build-image:
  stage: build-image
  rules:
    - if: '$CI_COMMIT_TAG'
      allow_failure: false
    - when: never
  tags:
    - shared
  script:
    - docker rmi tong/boot-demo:${CI_COMMIT_TAG} || true
    - docker build -t tong/boot-demo:$CI_COMMIT_TAG .

tag-image:
  stage: tag-image
  rules:
    - if: '$CI_COMMIT_TAG'
      allow_failure: false
    - when: never
  tags:
    - shared
  script:
    - docker tag tong/boot-demo:$CI_COMMIT_TAG 192.168.0.16:8010/tong/boot-demo:$CI_COMMIT_TAG

push-image:
  stage: push-image
  rules:
    - if: '$CI_COMMIT_TAG'
      allow_failure: false
    - when: never
  tags:
    - shared
  script:
    - docker push 192.168.0.16:8010/tong/boot-demo:$CI_COMMIT_TAG

run-image:
  stage: push-image
  rules:
    - if: '$CI_COMMIT_TAG'
      allow_failure: false
    - when: never
  tags:
    - shared
  script:
    - docker rm -f boot-demo || true
    - docker run -d --name boot-demo -p 8012:8080 192.168.0.16:8010/tong/boot-demo:$CI_COMMIT_TAG
```

