---
title: Docker部署Jenkins
description: Docker部署Jenkins
date: 2024-12-23
slug: Docker部署Jenkins
image: dbc-docker-desktop-home.webp
categories:
    - Docker
---
# Docker部署Jenkins

## 部署

> 参考[博客](https://blog.csdn.net/Aaaaaaatwl/article/details/140314323)

```
docker pull jenkins/jenkins:lts-jdk8
```

```sh
# 创建容器
docker run -d \
--name jenkins \
-p 8080:8080 \
-p 50000:50000 \
-v /home/docker/jenkins_data:/var/jenkins_home \
-v $(which docker):/usr/bin/docker \
-v /var/run/docker.sock:/var/run/docker.sock \
-u 0 \
--restart=on-failure:3 \
-d jenkins/jenkins:lts-jdk8

# 查看日志
docker logs -f 8605cc78de0d
```

![image-20241222214704308](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412222147430.png)

> 这就是`管理员密码`

```
aaadd43cf529436ea4500df629d64226
```

> 修改文件`/home/docker/jenkins_data/config.xml`
>
> 将`excludeClientIPFromCrumb`标签里的`false`改为`true`

```xml
  <crumbIssuer class="hudson.security.csrf.DefaultCrumbIssuer">
    <excludeClientIPFromCrumb>true</excludeClientIPFromCrumb>
  </crumbIssuer>
```

## 插件下载缓慢

```sh
# 进入容器
docker exec -it jenkins /bin/bash


# 找到 default.json 文件
find / -name default.json

# 进入对应目录
cd /var/jenkins_home/updates

# 替换 default.json 中的内容
# 将 updates.jenkins-ci.org/download 替换为 mirrors.tuna.tsinghua.edu.cn/jenkins，
# 将 www.google.com 替换为 www.baidu.com

sed -i 's/www.google.com/www.baidu.com/g' default.json
sed -i 's/updates.jenkins-ci.org\/download/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json

# 退出
exit

# 重启容器
docker restart 8605cc78de0d
```

![image-20241222215448277](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412232126132.png)

> 我这边遇到了这个问题，查看日志发现启动成功了，但是无法访问

```sh
# 进入容器
docker exec -it jenkins /bin/bash

find / -name hudson.model.UpdateCenter.xml

# 进入对应目录
cd /var/jenkins_home/

sed -i 's|updates.jenkins.io/update-center.json|mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json|g' /var/jenkins_home/hudson.model.UpdateCenter.xml

# 重启容器
docker restart 8605cc78de0d
```

![image-20241222215855486](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412232126188.png)

将日志里的管理员密码粘贴进来，文章上面已经提过了

1. 点击安装推介的插件