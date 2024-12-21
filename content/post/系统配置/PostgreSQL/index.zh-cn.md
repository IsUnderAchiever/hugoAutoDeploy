---
title: PostgreSQL
description: PostgreSQL配置
date: 2024-10-20
slug: PostgreSQL配置
image: 202412211414934.png
categories:
    - 系统配置
---
# PostgreSQL

> [下载链接](https://www.postgresql.org/download/windows/)
>
> [参考博客](https://blog.csdn.net/weixin_54787369/article/details/141348101)
>
> [参考博客2](https://blog.csdn.net/hx7013/article/details/124126849)
>
> 不要跟着截图一路走下去，后续有报错，先看完解决方法，尽量避免报错！！！

![image-20241020145520220](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211411361.png)

![image-20241020145600987](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211411214.png)

![image-20241020145816874](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211411410.png)

![image-20241020145904513](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211411172.png)

> 这里报了一个警告

![image-20241020150149575](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211411168.png)

> [参考博客](https://blog.csdn.net/xiuxiuxiu666/article/details/107982981)
>
> 但我尝试之后并未解决

![image-20241020151553944](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211411192.png)

![image-20241020152123437](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211412635.png)

![image-20241020152319942](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211412870.png)

![image-20241020152400319](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211412221.png)

> 重新安装时，选择

![image-20241020152442599](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211412879.png)

**`依然报错!!!`**

> 参考如下[博客](https://blog.csdn.net/hx7013/article/details/124126849)后`依然无法`解决问题
>
> cd到`postgresql/bin`下，执行如下命令初始化数据库集群

```sh
initdb -D "D:\env\Database\PostgreSQL\data"
```

![image-20241020153759112](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211412997.png)

> 注册成windows服务

```sh
pg_ctl.exe register -N postgresql17 -D D:\env\Database\PostgreSQL\data
```

![image-20241020154157319](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211412644.png)

![image-20241020154216958](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211412210.png)

> 配置环境变量

新建如下配置

![image-20241020155244939](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211412976.png)

```
PGHOME
D:\env\Database\PostgreSQL\17
```

在path下新建

```
%PGHOME%\bin
```

> navicat连接postgresql

![image-20241020154822302](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211412136.png)

> 报错信息乱码
>
> [参考博客](https://blog.csdn.net/weixin_41377835/article/details/104943953)

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             0.0.0.0/0               trust
```

> **`再次卸载重装`**
>
> [参考博客](https://blog.csdn.net/clbdbc/article/details/138634314)
>
> 这次终于解决问题

```sh
initdb -D "D:\env\Database\PostgreSQL\data" -E UTF-8 --locale=Chinese
```

![image-20241020161130682](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211413204.png)

![image-20241020161209442](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211413582.png)

> 打开psql之后，一路默认，报错`postgres`角色不存在，所以接下来需要创建这个角色



>再次打开psql，username输入`Administrator`，其他默认

![image-20241020161340534](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211413512.png)

```sql
CREATE USER postgres SUPERUSER;
```

> 至此成功

![image-20241020161459281](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211413293.png)

![image-20241020161513628](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211413398.png)