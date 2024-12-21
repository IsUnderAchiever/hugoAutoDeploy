---
title: Activiti报错
description: Activiti报错
date: 2023-07-10
slug: Activiti报错
image: 202412212108679.png
categories:
    - Activiti
---

# 遭遇问题
## 404
`http://localhost:8080/activiti-app`访问报404
在activiti官网上下的文件，[官网链接](https://www.activiti.org/get-started)，[网盘链接](https://www.123pan.com/s/tMU0Vv-REzUd.html)
看了一下tomcat，是因为`activiti-app.war`没有被解压
怀疑是war包有问题，又去github下载了（实际可能不是），[链接](https://www.123pan.com/s/tMU0Vv-DEzUd.html)
后来没办法，试过直接用zip解压，但是启动tomcat时候报错了，即便删掉重新启动还是会报错
最后清了一下tomcat缓存就成功了
**以下是清理Tomcat的步骤：**
```
1. 停止Tomcat服务器：在命令行中输入“shutdown.bat”（Windows）或“shutdown.sh”（Linux）。
2. 删除Tomcat工作目录：在Tomcat安装目录下找到“work”文件夹并删除。
3. 删除Tomcat日志文件：在Tomcat安装目录下找到“logs”文件夹并删除其中的日志文件。
4. 删除Tomcat临时文件：在Tomcat安装目录下找到“temp”文件夹并删除其中的临时文件。
5. 清理Tomcat缓存：在Tomcat安装目录下找到“catalina”文件夹并删除其中的缓存文件。
6. 清理Tomcat应用程序：在Tomcat安装目录下找到“webapps”文件夹并删除其中的应用程序。
7. 清理Tomcat配置文件：在Tomcat安装目录下找到“conf”文件夹并删除其中的配置文件。
8. 清理Tomcat插件：在Tomcat安装目录下找到“lib”文件夹并删除其中的插件。
9. 重新启动Tomcat服务器：在命令行中输入“startup.bat”（Windows）或“startup.sh”（Linux）。
注意：在清理Tomcat之前，请备份重要的文件和配置。
```
## 配置MySQL
![202306242243706](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152350068.png)
> 查看[博客](https://blog.csdn.net/zxw75192/article/details/94749796)
>
> 启动依然还有报错，需要将mysql连接包复制到lib文件夹下
>
> 不确定对mysql连接包的版本有没有要求，我刚开始用的是`8.0.30`版本，依然存在报错，改为`8.0.17`后即可，记得删除原本5.xx版本的jar包
![202306242243379](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152349359.png)
![202306242243046](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152349743.png)
> 再次启动，报错如下
```
### Error querying database. Cause: java.sql.SQLSyntaxErrorException: Table 'activit.ACT_GE_PROPERTY' doesn't exist
### The error may exist in org/activiti/db/mapping/entity/Property.xml
### The error may involve org.activiti.engine.impl.persistence.entity.PropertyEntityImpl.selectProperty-Inline
### The error occurred while setting parameters
### SQL: select * from ACT_GE_PROPERTY where NAME_ = ?
### Cause: java.sql.SQLSyntaxErrorException: Table 'activit.ACT_GE_PROPERTY' doesn't exist
```
> 修改配置文件
>
> 添加`nullCatalogMeansCurrent=true`即可
datasource.url=jdbc:mysql://127.0.0.1:3306/activiti6ui?`nullCatalogMeansCurrent=true`&characterEncoding=utf8&useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
![202306242243027](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152349643.png)
## junit报错
> 导入`activiti`依赖后，junit报错
```
java.lang.IllegalStateException: Failed to load ApplicationContext
```
> 更换junit依赖即可
```java
package com.example.demo;
import org.junit.Test;
import org.springframework.boot.test.context.SpringBootTest;
@SpringBootTest
public class ActivitiDemoApplicationTests {
    @Test
    public void contextLoads() {
        System.out.println("aaa");
    }
}
```
**注意**
> `activiti`和`mybatis`版本问题，可能会导致版本冲突
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>activiti-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>activiti-demo</name>
    <description>activiti-demo</description>
    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.3.12.RELEASE</spring-boot.version>
        <mybatis-plus.version>3.5.2</mybatis-plus.version>
        <activiti.version>7.0.0.GA</activiti.version>
    </properties>
    <dependencies>
        <!-- web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- mysql -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!-- test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- mybatis-plus -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis-plus.version}</version>
        </dependency>
        <!-- activiti -->
        <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-spring-boot-starter</artifactId>
            <version>${activiti.version}</version>
            <exclusions>
                <exclusion>
                    <artifactId>mybatis</artifactId>
                    <groupId>org.mybatis</groupId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot.version}</version>
                <configuration>
                    <mainClass>com.example.demo.ActivitiDemoApplication</mainClass>
                    <skip>true</skip>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```
## 获取ProcessEngine对象报错
![202306242243865](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152348161.png)
> jdk从8更换为11即可
![image-20230622131628898](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242243610.png)
![image-20230622131642302](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242243919.png)
![image-20230622131646360](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242243195.png)
## 流程部署后查询不到
> 那是因为没有部署成功
![image-20230623095814972](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242243154.png)
> 这两个`xml`文件，图标都不一样，是文件的命名有问题
![image-20230623095858036](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242243943.png)
> 将文件名改为`xxx.bpmn20`即可
