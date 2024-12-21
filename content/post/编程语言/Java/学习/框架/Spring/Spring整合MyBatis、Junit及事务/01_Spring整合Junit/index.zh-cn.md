---
title: 01_Spring整合Junit
description: 01_Spring整合Junit
date: 2023-01-20
slug: 01_Spring整合Junit
image: 202412212133331.png
categories:
    - Spring
---

## 1_Spring整合Junit
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>Spring_AOP_7_Junit</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.9</version>
        </dependency>
        <!--Junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <!--Spring整合Junit依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.3.18</version>
        </dependency>
    </dependencies>
</project>
```
> 编写测试类
>
> 在测试类上加上
> `@RunWith(SpringJUnit4ClassRunner.class)`注解，指定让测试运行于Spring环境
> `@ContextConfiguration`注解，指定Spring容器创建需要的配置文件或者配置类
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:配置文件.xml"})
@ContextConfiguration(classes = spring配置类.class)
```
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
public class MyTest {
    @Autowired
    private DoService doService;
    @Test
    public void test1(){
        System.out.println(doService.success("code"));
    }
}
```
