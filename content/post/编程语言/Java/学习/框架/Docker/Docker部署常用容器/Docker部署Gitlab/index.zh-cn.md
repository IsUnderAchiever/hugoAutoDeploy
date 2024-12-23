---
title: Docker部署Gitlab
description: Docker部署Gitlab
date: 2024-12-23
slug: Docker部署Gitlab
image: dbc-docker-desktop-home.webp
categories:
    - Docker
---
# Docker部署Gitlab

> 参考[博客](https://blog.csdn.net/BThinker/article/details/124097795)

## 部署

```sh
# 拉取镜像
docker pull gitlab/gitlab-ce:latest

# 启动容器
docker run \
 -itd  \
 -p 9980:80 \
 -p 9922:22 \
 -v /home/docker/gitlab/etc:/etc/gitlab  \
 -v /home/docker/gitlab/log:/var/log/gitlab \
 -v /home/docker/gitlab/opt:/var/opt/gitlab \
 --restart always \
 --privileged=true \
 --name gitlab \
 gitlab/gitlab-ce
```

**`接下来的配置请在容器内进行修改，不要在挂载到宿主机的文件上进行修改。否则可能出现配置更新不到容器内，或者是不能即时更新到容器内，导致gitlab启动成功，但是无法访问`**

```sh
#进容器内部
docker exec -it gitlab /bin/bash
 
#修改gitlab.rb
vi /etc/gitlab/gitlab.rb
 
#加入如下
#gitlab访问地址，可以写域名。如果端口不写的话默认为80端口
external_url 'http://192.168.42.128'
#ssh主机ip
gitlab_rails['gitlab_ssh_host'] = '192.168.42.128'
#ssh连接端口
gitlab_rails['gitlab_shell_ssh_port'] = 9922
 
# 让配置生效
gitlab-ctl reconfigure
```

**注意不要重启**
/etc/gitlab/gitlab.rb文件的配置会映射到gitlab.yml这个文件，由于咱们在docker中运行，在gitlab上生成的http地址应该是http://192.168.42.128:9980,所以，要修改下面文件



```sh
vi /opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml
```

```yaml
gitlab:
  host: 192.168.42.128
  port: 9980 # 这里改为9980
  https: false
```

```sh
#重启gitlab 
gitlab-ctl restart
#退出容器 
exit
```

访问链接`http://192.168.42.128:9980/`即可，**注意**机器配置要大于4g，否则很容易启动不了，报502


## 修改root密码

```sh
# 进入容器内部
docker exec -it gitlab /bin/bash
 
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