---
title: Maven
description: Maven配置
date: 2023-01-19
slug: Maven配置
image: maven-logo-black-on-white.png
categories:
    - 系统配置
---

## maven下载
[maven官网](https://maven.apache.org/)
### 最新版
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202036769.png)
### [历史版](https://archive.apache.org/dist/maven/maven-3/)
> 还是刚刚`最新版`的页面，往下拉，找到`archives`，或者点击[链接](https://archive.apache.org/dist/maven/maven-3/)前往
![2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202036175.png)
![3](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202036471.png)
![4](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202036603.png)
![5](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202036654.png)
## maven配置
> 新建环境变量`MAVEN_HOME`
```bash
D:\apache-maven-3.8.6
```
![6](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202036513.png)
> 系统变量`path`新建`%MAVEN_HOME%\bin`
```bash
%MAVEN_HOME%\bin
```
![7](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202036785.png)
> cmd窗口查看版本
```bash
mvn -v
```
> 新建maven仓库`repository`【新建repository文件夹】
![9](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202036584.png)
> 配置`setting.xml`
![10](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202036280.png)
```xml
<localRepository>D:\apache-maven-3.8.6\repository</localRepository>
<!-- 阿里云仓库 -->
<mirror>
    <id>alimaven</id>
    <mirrorOf>central</mirrorOf>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
</mirror>
<!-- java版本 -->
<profile>
    <id>jdk-1.8</id>
    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
    </activation>
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
</profile>
```
![11](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202036869.png)
![12](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202036321.png)
![13](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202036857.png)
