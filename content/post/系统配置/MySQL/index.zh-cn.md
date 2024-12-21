---
title: MySQL
description: MySQL配置
date: 2023-02-18
slug: MySQL
image: 202412211354776.png
categories:
    - 系统配置
---
## MySQL安装
![image-20230124182400827](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301241824907.png)
![image-20230124182419921](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301241824984.png)
![image-20230124182519527](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301241825584.png)
![image-20230124182714142](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301241827181.png)
![image-20230124182731556](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301241827607.png)
> 一直next
![image-20230124182753261](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301241827318.png)
![image-20230124182811676](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301241828735.png)
> 剩下的默认即可，一直next
## mysql环境变量配置
```
path添加
D:\MySQL\MySQL Server 8.0\bin
```
![image-20230124183117870](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301241831905.png)
![image-20230124183154560](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301241831631.png)
## MySQL时区配置
打开mysql的安装目录，比如我的是`D:\MySQL\MySQL Server 8.0`
打开my.ini，添加下面这句，重启服务
```ini
# [mysqld]
# 永久更改时区，解决连接mysql时timeout的情况
default-time_zone = '+8:00'
```
```bash
# 查看时区是否设置成功
show variables like "%time_zone%";
```
```
mysql> show variables like "%time_zone%";
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone |        |
| time_zone        | +08:00 |
+------------------+--------+
2 rows in set, 1 warning (0.01 sec)
```
若出现以上结果即可
