---
title: Git
description: Git配置
date: 2022-12-13
slug: Git
image: branching-illustration@2x.png
categories:
    - 系统配置
---

## git环境配置
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202015463.png)
![2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202015664.png)
安装一路默认即可【一直next】
![3](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202015837.png)
在桌面右键，多出了Git Bash Here
```bash
# 打开Git Bash Here，复制命令并粘贴，右键paste
# 配置用户名【填写自己的用户名】
git config --global user.name "your username"
# 配置邮箱【填写自己的邮箱】
git config --global user.email "your email"
```
## 同时配置gitee和github
[参考博客](https://cloud.tencent.com/developer/article/1774890)
```bash
git config --global --unset user.name "你的名字"
git config --global --unset user.email "你的邮箱"
```
### 生成新的 SSH keys
#### Github
```javascript
ssh-keygen -t rsa -f ~/.ssh/id_rsa.github -C "xxx@qq.com"
```
按三次回车
### Gitee
邮箱换一个，不与Github邮箱相同即可
```javascript
ssh-keygen -t rsa -f ~/.ssh/id_rsa.gitee -C "xxx@aliyun.com"
```
按三次回车
完成后会在~/.ssh / 目录下生成以下文件。
- id_rsa.github
- id_rsa.github.pub
- id_rsa.gitee
- id_rsa.gitee.pub
```javascript
ssh-agent bash
ssh-add ~/.ssh/id_rsa.github
ssh-add ~/.ssh/id_rsa.gitee
```
创建一个名为config的文件，不带后缀，以文本形式打开，写入如下内容
```javascript
#Default gitHub user Self
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa.github
# gitee
Host gitee.com
    Port 22
    HostName gitee.com
    User git
    IdentityFile ~/.ssh/id_rsa.gitee
```
添加 ssh到github和gitee中
```javascript
ssh -T git@gitee.com
ssh -T git@github.com
```
输入yes
### 补充
> 说一下我遇到的问题，我是全程开着加速器的，毕竟github的特殊性...
> 但是最后这一步测试github的连接始终是连不上，`Connection reset by 140.82.114.4 port 22`
```bash
$ ping  github.com
Pinging github.com [140.82.114.4] with 32 bytes of data:
Reply from 140.82.114.4: bytes=32 time=270ms TTL=41
Request timed out.
Reply from 140.82.114.4: bytes=32 time=273ms TTL=41
Reply from 140.82.114.4: bytes=32 time=273ms TTL=41
Ping statistics for 140.82.114.4:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 270ms, Maximum = 273ms, Average = 272ms
```
> 看来确实是有问题的
>
> 配置一下hosts文件就连上了
> hosts文件在C:\Windows\System32\drivers\etc
[点击测速](https://tool.chinaz.com/speedworld/)，搜索github.com
```
如我这里是20.205.243.166
则在hosts文件里添加
20.205.243.166 github.com
```
```bash
$ ssh -T git@github.com
The authenticity of host 'github.com (140.82.114.4)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
Hi IsUnderAchiever! You've successfully authenticated, but GitHub does not provide shell access.
```
> 但是`github`克隆项目还是报错，开不开‘特殊手段’都是这样
```bash
$ git clone https://github.com/nodejs/node.git
Cloning into 'node'...
fatal: unable to access 'https://github.com/nodejs/node.git/': Failed to connect to github.com port 443 after 21049 ms: Couldn't connect to server
```
## 只配置gitee或github
> 以gitee为例，github类似
配置ssh免密登录
```bash
ssh-keygen -t rsa -C "your email"
# 按三次回车后，生成密钥【在用户目录下】
cat ~/.ssh/id_rsa.pub
# 查看密钥
# 复制以上密钥
# 进入gitee，左侧找到SSH公钥，添加标题（自己随便写一个），公钥复制粘贴上一步查看的内容（注意不要携带空格）
# 测试是否成功 (选择yes)
ssh -T git@gitee.com
```
![4](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202015913.png)
![5](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202015239.png)
![6](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202015408.png)
