---
title: 部署远程开发环境
description: 部署远程开发环境
date: 2025-12-14
slug: 部署远程开发环境
image: dbc-docker-desktop-home.webp
categories:
    - Docker
---
# 部署远程开发环境

> 先大概讲一下远程开发环境分哪几个部分
> 1. 代码仓库托管能力（gitlab）
> 2. 流水线（gitlab-runner、jenkins）
> 3. 制品仓库（nexus）
> 4. 远程代码编辑器（vscode、jetbrains gateway）

整个流程是这样的，在服务端部署了以上服务之后，客户端可以通过网页访问vscode进行开发工作，无需安装任何客户端工具
通过gitlab实现代码托管能力，提交代码后触发gitlab-runner流水线，自动打包并将制品（jar、pypi、docker images等）推送到nexus仓库
此时可在jenkins上从nexus拉取对应制品部署到环境上从而实现远程开发+部署的目的。如果还需要服务端监控能力，可以部署prometheus+grafana实现

## Gitlab、Gitlab Runner

> `hostname` 配置为自己的ip+端口
> 创建之后稍等一会即可访问`http://10.0.0.7:8080`

```bash
docker run -d \
  --name gitlab \
  --restart unless-stopped \
  --hostname 10.0.0.7:8080 \
  -p 8080:80 \
  -p 8443:443 \
  -p 2222:22 \
  -v /data/volumes/gitlab/config:/etc/gitlab \
  -v /data/volumes/gitlab/logs:/var/log/gitlab \
  -v /data/volumes/gitlab:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

```bash
docker run -d \
  --name gitlab-runner \
  --restart always \
  -v /data/volumes/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

![image-20251215225343474](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo06/image-20251215225343474.png)

![image-20251215225551627](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo06/image-20251215225551627.png)

![image-20251215225617911](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo06/image-20251215225617911.png)

![image-20251215225632442](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo06/image-20251215225632442.png)




> 点击设置、CI/CD,选择Runner，点击`创建项目Runner`，标签随便写一个即可，这里我写的是`shared`

```bash
# 这是直接在宿主机安装Gitlab Runner的情况，但是我们使用的是Runner容器
# 注册Runner，注意替换成自己的token
gitlab-runner register  --url http://10.0.0.7:8080  --token glrt-pXG2PMYcZhORoOGq49THO286MQpwOjIKdDozCnU6Mg8.01.171upp0gg

# Runner容器使用下面的注册命令
docker exec -it gitlab-runner gitlab-runner register \
  --non-interactive \
  --url "http://10.0.0.7:8080" \
  --clone-url "http://10.0.0.7:8080" \
  --token "glrt-pXG2PMYcZhORoOGq49THO286MQpwOjIKdDozCnU6Mg8.01.171upp0gg" \
  --executor "docker" \
  --docker-image "alpine:latest"
```

此时重新查看CICD设置，可以看到流水线已经注册成功了

![image-20251215225652128](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo06/image-20251215225652128.png)

> 注意这里有个坑，如果流水线注册之后，看到代码运行流水线失败，此时你可能会删除掉流水线重新注册。
> 你应该重新提交代码运行新的流水线，而不是重新运行旧的流水线，否则仍然会运行旧的流水线导致报错。

在项目根目录下创建`.gitlab-ci.yml`文件

![image-20251215225717480](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo06/image-20251215225717480.png)

```yaml
test-job:
  script:
    - echo "Runner is working!"
    - uname -a
```

然后自动就触发流水线了（也可以配置打tag时触发，而不是提交代码触发）

> 项目详情页面 代码commit左侧有一个`圆形按钮`，点进去就能查看流水线详情
> 这里15是jobid，根据自己实际情况来

访问`http://10.0.0.7:8080/tongwh/bilibili-download/-/jobs/15`

![image-20251215225743041](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo06/image-20251215225743041.png)

## 外网访问

这里我是使用`ddns`+`动态公网ip`+`域名`实现的远程访问方案

动态公网ip可以找运营商申请
ddns项目我推介使用`ddns-go`，可使用docker部署

```bash
docker run -d --name ddns-go --restart=always --net=host -v /data/volumes/ddns-go:/root jeessy/ddns-go
```

