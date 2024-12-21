---
title: WSL
description: 开启WSL
date: 2024-06-02
slug: WSL
image: 202412211452323.png
categories:
    - 系统配置
---

# 开启WSL

> 参考[文章](https://blog.csdn.net/qq_60750453/article/details/128636298)

## 基本配置

> 打开设置，选中应用

![image-20240602120930045](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406021209094.png)

> 选择`程序和功能`

![image-20240602121020903](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406021210976.png)

> 选择`启用或关闭Windows功能`

![image-20240602121149020](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406021211070.png)

>勾选`Hyper-v`和`适用于Linux的Windows子系统`，然后重启电脑

![image-20240602121327604](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406021213639.png)

## 安装Linux子系统

> 注意使用`管理员权限`运行`cmd`或者`powershell`

```sh
# 下载或者更新
wsl --update
# 重新启动
wsl --shutdown
```

![image-20240602121840962](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406021218041.png)

> 前往`微软商店`下载Linux子系统

![image-20240602122004003](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406021220128.png)

> 搜索Linux，安装心仪的系统，这里以ubuntu为例

![image-20240602122124490](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406021221541.png)

> 启动系统后，会需要新建用户

![image-20240602122257262](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406021222300.png)

> 输入自定义用户名以及密码，即可新建用户

![image-20240602122403159](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406021224191.png)

## 配置docker

配置docker过程查看[博客](https://blog.csdn.net/qq_60750453/article/details/128636298)

> 以下是配置后的结果
>
> 启动了两个容器，一个nginx，一个redis

![image-20240602122619380](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406021226421.png)

> 输入`http://localhost:80/`访问nginx界面

![image-20240602122744455](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406021229951.png)

> 客户端连接Redis成功

![image-20240602122855884](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406021228921.png)

## 配置ssh

> 输入密码后切换为`root`用户

```sh
sudo su
```

> 执行以下命令
>
> `卸载WSL上自带的openssh并重装`

```sh
apt-get update

apt-get remove openssh-server 

apt-get install openssh-server
```

> 修改ssh的配置

```sh
vim /etc/ssh/sshd_config
```

> 先按住`shift : `
>
> 输入`set nu`，回车即可显示行数
>
> 找到`PermitRootLogin`，改为`yes`

![image-20240602211021605](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406022110343.png)

> 将如下位置的注释取消掉
>
> 然后保存并退出

![image-20240602211209601](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406022112256.png)

```sh
service ssh status

service ssh start
```

![image-20240602211333816](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406022115683.png)

> ip直接写`127.0.0.1`即可

![image-20240602211512950](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406022115467.png)

> 配置用户名、密码

![image-20240602211528562](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo02/202406022115033.png)
