---
title: STS
description: Spring Tools Suite安装及配置
date: 2023-03-04
slug: STS
image: 202412211431425.png
categories:
    - 系统配置
---
# 安装
## Spring Tools Suite 3
> spring-tool-suite-3.9.18 遇到的问题如下

![image-20230304203127888](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042031029.png)

> [解决方法](https://blog.csdn.net/boss_way/article/details/89762651)
![image-20230304204451982](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042044014.png)

![image-20230304204356225](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042043262.png)

> 装的jdk1.8，却显示17，这不是离谱吗？
>
> java_home配置也没有问题，确实是1.8的路径
`原因`出在oracle这里，将oracle一道java环境变量之下即可

![image-20230304204632564](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042046603.png)

![image-20230304204720626](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042047656.png)

>到这里还是依然报错，java版本不在11以上
>
>网上说把ini文件的11改为1.8即可，但我的还是报错，还是直接配置3.9.11版本或者sts4吧

1. [官网](https://github.com/spring-attic/toolsuite-distribution/wiki/Spring-Tool-Suite-3)
2. [spring-tool-suite-3.9.18`（不推介）`](https://www.123pan.com/s/tMU0Vv-TwiUd.html)
3. [spring-tool-suite-3.9.11`（推介）`](https://www.123pan.com/s/tMU0Vv-jMiUd.html)
## Spring Tools Suite 4
[spring-tool-suite-4-4.3.1](https://www.123pan.com/s/tMU0Vv-vwiUd.html)
# 配置
### 取消空格或分号自动补全

![image-20230304215525059](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042155213.png)

![image-20230304215748584](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042157756.png)

```
.abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXY
```
### 快捷键配置
> 这个是按照我的习惯配置的，如果不需要配置，请直接跳过
>复制整行，如idea的ctrl+d，我习惯配置alt+d

![image-20230304220121647](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042201724.png)

> 这个是代码提示快捷键

![image-20230304220228843](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042202915.png)

> format 格式化代码快捷键
>
> 方法提取(Extract method)
>
> 全局查找(Find Text in File)
### 开启注解提示

![image-20230304220839131](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042208202.png)

### 配置maven

![image-20230304221833484](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042218553.png)

![image-20230304221911956](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042219006.png)

![image-20230304221924265](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042219331.png)

![image-20230304221011377](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042210436.png)

### 配置本地java

![image-20230304221216278](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042212336.png)

![image-20230304221149953](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042211998.png)

### 编码配置

![image-20230304221448268](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042214349.png)

![image-20230304221731074](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042217141.png)

![image-20230304222222892](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042222959.png)

![image-20230304222347323](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042223387.png)

![image-20230304223150268](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042231342.png)

![image-20230304223218691](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042232754.png)

### 配置Lombok插件
[lombok jar包](https://www.123pan.com/s/tMU0Vv-dwiUd.html)
> 1. 将lombok放在sts.exe文件处
> 2. 双击jar包

![image-20230304223506132](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042235194.png)

![image-20230304223727831](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042237892.png)

![image-20230304223743408](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042237464.png)

### 配置字体

![image-20230304224008789](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042240852.png)

![image-20230304224019268](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042240321.png)

### 配置代码模板

![image-20230304224639136](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042246198.png)

![image-20230304224622520](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042246563.png)

### 自动导包

![image-20230304225933835](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042259918.png)

### 配置tomcat

window》show view》other》servers

![image-20230304232300462](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042323516.png)

![image-20230304232319905](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042323961.png)

## 普通的java项目如何使用jar包

![image-20230304230517623](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042305719.png)

![image-20230304230535585](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042305639.png)

![image-20230304230551076](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303042305134.png)
