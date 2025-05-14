---
title: 01_AOP入门
description: 01_AOP入门
date: 2023-01-15
slug: 01_AOP入门
image: 202412212133331.png
categories:
    - Spring
---

## 01_AOP入门
> AOP为Aspect Oriented Programming的缩写，意为:面向切面编程。他是一种可以在不修改原来的核心代码的情况下给程序动态统—进行`增强`的一种技术。
> SpringAOP:批量对Spring容器中bean的方法做增强，并且这种增强不会与原来方法中的代码耦合。
> `需求`：要求让service包下所有类的所有方法在调用前都输出:方法被调用了。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>_01_Spring_AOP</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <!-- lombok依赖 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
        </dependency>
        <!-- IOC相关依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.18</version>
        </dependency>
        <!-- AOP相关依赖 -->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.7</version>
        </dependency>
    </dependencies>
</project>
```
```java
@Service
public class PhoneService {
    public void toInit(){
        System.out.println("PhoneService中toInit的核心代码");
    }
}
@Service
public class UserService {
    public void toInit(){
        System.out.println("UserService中toInit的核心代码");
    }
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">
    <!-- 开启组件扫描 -->
    <context:component-scan base-package="com.example"/>
    <!-- 开启aop注解支持 -->
    <aop:aspectj-autoproxy/>
</beans>
```
> 创建切面类
> 创建一个类，在类上加上`@Component`和`@Aspect`使用`@Pointcut`注解来指定要被增强的方法
> 使用`@Before`注解来给我们的增强代码所在的方法进行标识，并且指定了增强代码是在被增强方法执行之前执行的。
```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;
@Component
@Aspect
public class MyAspect {
    // 对哪些方法进行增强
    @Pointcut("execution(* com.example.service.*.toInit(..))")
    public void pt(){}
    // 怎么增强
    @Before("pt()")
    public void methodBefore(){
        System.out.println("方法被调用了");
    }
}
```
```java
public class Application_01 {
    public static void main(String[] args) {
        // 创建容器
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        // 获取对象
        PhoneService phoneService = context.getBean(PhoneService.class);
        UserService userService = context.getBean(UserService.class);
        // 方法调用
        phoneService.toInit();
        userService.toInit();
    }
}
```
