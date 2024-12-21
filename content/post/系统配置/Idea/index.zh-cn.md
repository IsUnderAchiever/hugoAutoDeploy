---
title: Idea
description: Idea配置
date: 2023-01-19
slug: Idea配置
image: overview-heading-screenshot.webp
categories:
    - 系统配置
---
# Idea配置

## 快捷键
```
代码重排(reformat code)->(ctrl+k)
run context configuration->(F5) # 可有可无
delete line->(ctrl+d)
代码提示(basic)->alt+?
方法提取(Extract method)->(ctrl+alt+M)
全局查找(Find in files)->(ctrl+H)
向下复制(Duplicate Line or Selection)->(alt+D)
```
## 自动注释模板
![13](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202034968.png)
```java
/**
 * @Auther: ${USER}
 * @Date: ${YEAR}/${MONTH}/${DAY}/${TIME}
 * @Description: 
 */
```
## 修改编码
![4](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202035922.png)
## 自动导包
![5](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202035660.png)
## 忽视代码大小写
![6](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202035162.png)
## 关闭自动更新
![7](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202035515.png)
## tab页多行显示设置
![8](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202035522.png)
## Java注释优化代码注释前空格格式
![9](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202035055.png)
## 开启代码自动编译
![10](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202035166.png)
## 去除idea自带的.iml文件，以及.idea文件夹
![11](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202035328.png)
## 配置maven
![12](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202035538.png)
## idea启动时展开项目列表
![3](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202035773.png)
## pom文件依赖报黄
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202035003.png)
## 配置YML模板
> 1.这种是直接添加new的文件
![14](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202035663.png)
> 2.这种才类似添加用户的模板
![image-20230120205948293](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202059382.png)
![image-20230120210127291](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202101373.png)
```yml
server:
  # 应用服务 WEB 访问端口
  port: 8080
spring:
  # 应用名称 TODO
  application:
    name: demo
  datasource:
    # 数据库驱动：
    driver-class-name: com.mysql.cj.jdbc.Driver
    # 数据源名称
    name: defaultDataSource
    # 数据库连接地址
    url: jdbc:mysql://localhost:3306/blue?serverTimezone=UTC
    # 数据库用户名&密码：
    username: root
    password: 123456
```
```yml
spring:
  application:
    name: demo
  cloud:
    nacos:
      # 服务注册
      discovery:
        server-addr: localhost:8848
      # 服务配置
      config:
        server-addr: localhost:8848
        # 命名空间
#        namespace: # b6625ff9-0949-464b-bad3-fe4cf7858223
#        ext-config:
#          - data-id: # oss.yml
#            group: # DEFAULT_GROUP
#            refresh: true
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>demo</artifactId>
        <groupId>com.example.demo</groupId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example.common</groupId>
    <artifactId>demo-common</artifactId>
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <druid.version>1.1.10</druid.version>
        <servlet-api.version>2.3</servlet-api.version>
        <validation-api.version>2.0.1.Final</validation-api.version>
        <httpcore.version>4.4.15</httpcore.version>
        <commons-lang.version>2.6</commons-lang.version>
        <lombok.version>1.18.24</lombok.version>
        <mybatis-plus.version>3.5.1</mybatis-plus.version>
        <mysql.version>8.0.30</mysql.version>
        <spring-cloud-alibaba.version>2.2.9.RELEASE</spring-cloud-alibaba.version>
    </properties>
    <dependencies>
        <!--druid连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>${druid.version}</version>
        </dependency>
        <!--servlet-api-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>${servlet-api.version}</version>
            <scope>provided</scope>
        </dependency>
        <!--参数校验-->
        <dependency>
            <groupId>javax.validation</groupId>
            <artifactId>validation-api</artifactId>
            <version>${validation-api.version}</version>
        </dependency>
        <!--httpcore-->
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpcore</artifactId>
            <version>${httpcore.version}</version>
        </dependency>
        <!--commons-lang-->
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>${commons-lang.version}</version>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
        </dependency>
        <!--mybatis-plus-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis-plus.version}</version>
        </dependency>
        <!--mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
        <!--服务注册/发现-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--配置中心-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```
```properties
# 应用名称 TODO
spring.application.name=demo
# 应用服务 WEB 访问端口
server.port=8080
# 数据库驱动：
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 数据库连接地址
spring.datasource.url=jdbc:mysql://localhost:3306/blue?serverTimezone=UTC
# 数据库用户名&密码：
spring.datasource.username=root
spring.datasource.password=123456
# 数据源名称
# spring.datasource.name=com.alibaba.druid.pool.DruidDataSource
spring.datasource.name=defaultDataSource
# 配置mybatis-plus 打印sql日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
# xml文件路径
mybatis-plus.mapper-locations=classpath:/mapper/**/*.xml
# 配置mybatis-plus 包路径 TODO
mybatis-plus.type-aliases-package=com.example.demo.entity
# mybatis-plus下划线转驼峰配置，默认为true
mybatis-plus.configuration.map-underscore-to-camel-case=true
# 配置全局默认主键类型，实体类不用加@TableId(value ="id",type = IdType.AUTO)
mybatis-plus.global-config.db-config.id-type=auto
# 1代表已经删除，0代表没有删除
mybatis-plus.global-config.db-config.logic-delete-value=1
mybatis-plus.global-config.db-config.logic-not-delete-value=0
```
## idea本地缓存异常
![image-20230120204329350](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202043446.png)
![image-20230120204454572](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202044621.png)
## mybatis-plus xml报黄
![image-20230120204713724](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202047784.png)
![image-20230120204742889](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202047981.png)
## 类上注释报黄
![image-20230120204913655](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202049704.png)
![image-20230120205050381](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202050458.png)
![image-20230120205119196](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202051282.png)
> 找到java下的javadoc
>
> description,createDate  ==  @description  @createDate  
>
> description:,createDate:  ==  @description:  @createDate:  
## mapper层注入爆红，红色波浪线
![image-20230120205540071](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202055161.png)
## Lombok requires enabled annotation processin
<img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202114492.png" alt="image-20230120211444376" style="zoom:80%;" />
![image-20230120211525558](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202115626.png)
## lombok注解失效
> lombok注解失效，运行时提示找不到某个参数或方法
解决方法有以下三种可能（我个人是第三种）：
1. 更新lombok版本
2. mvn clean后重新导入
3. 加入 `-Djps.track.ap.dependencies=false`
![image-20230311100422978](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303111004136.png)
## pom文件变成灰色
> 原因，可能是之前创建项目后删除重建，所以idea排除了这个模块
>
> Setting->Build Tools->Maven->Ignored Files `去掉勾选项即可`

![image-20230311100659882](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303111006961.png)
## 打开项目后没有显示项目结构
![image-20230325120519766](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303251205970.png)
> 选择import

![image-20230325120540273](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303251205358.png)
![image-20230325120619785](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303251206868.png)
![image-20230325120640914](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303251206023.png)


## idea配置项目根目录

![image-20230402220239326](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304022202424.png)

## idea好用的插件
### Alibaba Java Coding Guidelines
阿里代码规范检查工具
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202027067.png)
### CodeGlance2
代码缩略图
![2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202027408.png)
### Easy Javadoc
自动生成类和方法的注释
![3](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202027132.png)
### Gitee
将项目分享至gitee
将gitee项目克隆值idea
![4-1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202027083.png)
![4-2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202027557.png)
### LeetCode Editor
LeetCode刷题
![5](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202027757.png)
### Maven Helper
解决maven jar包冲突问题
![6](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202027673.png)
### MyBatisX
快速生成实体类、mapper、mapper.xml、service、serviceimpl代码
![7](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202027619.png)
### RestfulTool
帮助基于restful服务开发的插件
查看请求路径，在项目运行之后还能发送请求查看返回结果
![8](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202027517.png)
### SequenceDiagram
生成时序图
![9-1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202027187.png)
![9-2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202027656.png)
### Show comment
显示注解，在方法上写好注解后，以后用到该方法的地方会看到写好的注解
![10](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202027409.png)
### statistic
统计代码量
![11](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202028590.png)
### Tabnine
生成代码提示
![12](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202028606.png)
### Translation
翻译插件
![13-1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202028247.png)
![13-2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202028373.png)
### Rainbow Brackets
彩虹括号，以不同的颜色显示每队括号，方便括号配对
![14](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202028408.png)

## 快捷键
### 同时编辑多处
> 按住`alt+shift`，点击多处即可
### 方法提取
> 方法提取(Extract method)`alt+shift+m`
