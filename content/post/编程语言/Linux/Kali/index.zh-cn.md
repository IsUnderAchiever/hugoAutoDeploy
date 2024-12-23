---
title: Kali
description: Kali
date: 2024-10-11
slug: Kali
image: 202412232120751.png
categories:
    - Kali
---

# Metasploit

### kali安装Metasploit

> 参考[文章](https://cn.linux-console.net/?p=21188)
>
> **启动postgresql**

查看`postgresql`的状态，kali默认安装`postgresql`，无需再次安装

```sh
systemctl status postgresql
```

![image-20241011075553517](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412232113509.png)

如果`postgresql`没有启动，需要先启动

```sql
sudo systemctl enable --now postgresql
```

> **初始化postgresql**

PostgreSQL 数据库服务器运行后，继续初始化 Metasploit PostgreSQL 数据库

```sh
sudo msfdb init
```

> 安装`msfconsole`

**更新apt源**

```sh
vim /etc/apt/sources.list
```

**添加如下内容**

```conf
# 中科大
deb http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib

# 阿里云
deb http://mirrors.aliyun.com/kali kali-rolling main non-free contrib
deb-src http://mirrors.aliyun.com/kali kali-rolling main non-free contrib

# 清华大学
deb http://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main contrib non-free

# 浙大
deb http://mirrors.zju.edu.cn/kali kali-rolling main contrib non-free
deb-src http://mirrors.zju.edu.cn/kali kali-rolling main contrib non-free

# 官方源
deb http://http.kali.org/kali kali-rolling main non-free contrib
deb-src http://http.kali.org/kali kali-rolling main non-free contrib
```

```sh
# 更新源
apt-get update

# 下载
apt install metasploit-framework
```

> **启动msfconsole**

```sh
sudo msfconsole
```

