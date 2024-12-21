---
title: 05_读取properties配置文件
description: 05_读取properties配置文件
date: 2023-01-14
slug: 05_读取properties配置文件
image: 202412212133331.png
categories:
    - Spring
---

## 读取properties配置文件
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201920297.png)
可以看到这四个属性已经注入进去了
![2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201920656.png)
> 以前是下面这种写法
> 现在直接读取properties配置文件后，注入属性
![3](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201920369.png)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>_06_Read_Properties</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.18</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.9</version>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>RELEASE</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 读取properties配置文件 -->
    <context:property-placeholder location="classpath:jdbc.properties"/>
    <bean class="com.alibaba.druid.pool.DruidDataSource" id="dataSource">
        <property name="driverClassName" value="${mysql.driverClassName}"/>
        <property name="url" value="${mysql.url}"/>
        <property name="username" value="${mysql.username}"/>
        <property name="password" value="${mysql.password}"/>
    </bean>
</beans>
```
```properties
mysql.driverClassName=com.mysql.cj.jdbc.Driver
mysql.url=jdbc:mysql://localhost/test?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai
mysql.username=root
mysql.password=123456
```
```java
package com.example.test;
import com.alibaba.druid.pool.DruidDataSource;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MyTest {
    @Test
    public void dataSourceTest(){
        ApplicationContext context = new ClassPathXmlApplicationContext("application_datasource.xml");
        DruidDataSource dataSource = (DruidDataSource) context.getBean("dataSource");
        System.out.println(dataSource);
    }
    @Test
    public void dataSourceTest1(){
        //DruidDataSource dataSource = new DruidDataSource();
        //dataSource.setDriverClassName();
        //dataSource.setUrl();
        //dataSource.setUsername();
        //dataSource.setPassword();
    }
}
```
> 本文第二张图片报了`DruidDataSource`警告，提示使用`try-with-resources`的写法
> 注意：用`try-with-resources`来代替`try-catch-finally`
> 在JavaGuide里有这里的解释，[原文连接](https://javaguide.cn/java/basis/java-basic-questions-03.html#%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-try-with-resources-%E4%BB%A3%E6%9B%BFtry-catch-finally)
> 按Ctrl+F，搜索 "如何使用 `try-with-resources` 代替`try-catch-finally`" 即可找到该位置
一般情况下会在`finally`语句中关闭资源，如下
```java
@Test
public void dataSourceTest1(){
    DruidDataSource dataSource = null;
    try {
        dataSource = new DruidDataSource();
        dataSource.setDriverClassName("");
        dataSource.setUrl("");
        dataSource.setUsername("");
        dataSource.setPassword("");
    }catch (Exception e){
        throw new RuntimeException("出现异常",e);
    }finally {
        assert dataSource != null;
        dataSource.close();
    }
}
```
使用`try-with-resources`来改造上面的代码
```java
@Test
public void dataSourceTest1(){
    try (DruidDataSource dataSource = new DruidDataSource()) {
        dataSource.setDriverClassName("");
        dataSource.setUrl("");
        dataSource.setUsername("");
        dataSource.setPassword("");
    } catch (Exception e) {
        throw new RuntimeException("出现异常",e);
    }
}
```
> 感兴趣的可以去了解一下@Value注解，会有不一样的感觉
