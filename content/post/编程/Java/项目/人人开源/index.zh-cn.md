---
title: 人人开源配置
description: 人人开源配置
date: 2022-12-13
slug: 人人开源配置
image: pawel-czerwinski-8uZPynIu-rQ-unsplash_hu3425483315149503896.jpg
categories:
    - Java
---

进入[人人开源](https://gitee.com/renrenio)，用git的方式下载三个项目
1. [renren-fast](https://gitee.com/renrenio/renren-fast)
2. [renren-fast-vue](https://gitee.com/renrenio/renren-fast-vue)
3. [renren-generator](https://gitee.com/renrenio/renren-generator)

![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201950475.png)

![2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201950091.png)

![3](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201950038.png)

![4](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201950741.png)

## renren-fast

![6](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201950552.png)

![5](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201950941.png)

**新建renren-fast数据库，并执行mysql.sql内的sql语句**

![7](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201950460.png)

![8](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201950776.png)

![9](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951709.png)

#### 这里前端项目报错了，解决方法如下：
```
cnpm install -g node-gyp
npm install --global --production windows-build-tools
```
如果一直卡在Successfully installed Python 2.7不动。
我电脑的解决方法是
```
npm install --global --production windows-build-tools@4.0.0
```
如果还解决不了，请看这篇[博客](https://blog.csdn.net/weixin_48391379/article/details/117187522)
#### 若后端启动报错
报错信息：jar:file:/D:/apache-maven-3.5.3/repository/javax/servlet/servlet-api...
表示jar包冲突，我的解决方法是**先去repository删了这些包**，然后重新在项目中下载

![12](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951646.png)

运行renren-fast和renren-fast-vue两个项目，前者是后端项目，后者是前端项目

![10](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951110.png)

![11](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951109.png)

账号和密码都是admin
## renren-generator
先去配置数据库的信息

![13](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951490.png)

以demo数据库为例：

![14](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951164.png)

新建一个springboot项目，来操作demo数据库的这几个表
springcloud涉及到多个数据库，则需要创建多个项目来对应每个数据库【项目与数据库一一对应】

![15](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951577.png)

![16](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951752.png)

![17](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951373.png)

**【微服务还需要导入openfeign依赖】**
打开generator.properties进行配置

![18](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951706.png)

![19](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951724.png)

![20](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951803.png)

运行项目

![21](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951877.png)

![22](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951882.png)

![23](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951019.png)

直接复制生成的main文件夹，粘贴到项目的main处

![24](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951458.png)

有不少报错，别急，这是因为缺少了一些工具包和相应的依赖
考虑到各种原因，删除刚刚生成的内容
我们将工具包和其他依赖【如mybatis-plus、lombok等通用的依赖】集成成一个springboot项目
将其他springboot的公共依赖、公共包进行整合

![25](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201951173.png)

![26](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201952113.png)

复制粘贴renren-fast/renren-generator内的类即可
在renren-common中导入依赖
```xml
        <!--mybatis-plus-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.2</version>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
        </dependency>
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpcore</artifactId>
            <version>4.4.16</version>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.6</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.30</version>
        </dependency>
```
在demo示例项目中导入renren-common的依赖
```xml
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>renren-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
```
在运行项目前，还需要在demo配置mysql的连接
```properties
# 应用名称
spring.application.name=demo
# 应用服务 WEB 访问端口
server.port=8080
# 数据库驱动：
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 数据源名称
spring.datasource.name=defaultDataSource
# 数据库连接地址
spring.datasource.url=jdbc:mysql://localhost:3306/demo?serverTimezone=UTC
# 数据库用户名&密码：
spring.datasource.username=root
spring.datasource.password=123456
# 配置mybatis-plus 打印sql日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
# xml文件路径
mybatis-plus.mapper-locations=classpath:/mapper/**/*.xml
# 配置最新全局配置文件
#mybatis-plus.config-location=classpath:mybatis-config.xml
# 配置mybatis-plus 包路径
mybatis-plus.type-aliases-package=com.example.demo.domain
# mybatis-plus下划线转驼峰配置，默认为true
mybatis-plus.configuration.map-underscore-to-camel-case=true
# 配置全局默认主键类型，实体类不用加@TableId(value ="id",type = IdType.AUTO)
mybatis-plus.global-config.db-config.id-type=auto
# 逻辑删除 （1为删除，0为未删除）
mybatis-plus.global-config.db-config.logic-delete-value=1
mybatis-plus.global-config.db-config.logic-not-delete-value=0
# 如果java实体类没加注解@TableLogic，则可以配置这个，推介这里配置
mybatis-plus.global-config.db-config.logic-delete-field=isDeleted
```
运行demo项目

![27](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201952508.png)
链接：https://pan.baidu.com/s/1OP2oPB1onrzMMai7jOmneg?pwd=j4nw 
提取码：j4nw 
--来自百度网盘超级会员V3的分享
