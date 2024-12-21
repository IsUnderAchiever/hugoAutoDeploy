---
title: 03_Spring6
description: 03_Spring6
date: 2023-03-13
slug: 03_Spring6
image: 202412212133331.png
categories:
    - Spring
---

Spring6学习03
====================================================================
注解开发
---------------------------------------------------------------
> 1.  引入依赖
> 2.  开启组件扫描
```xml
<dependencies>
    <!--spring ioc-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>6.0.2</version>
    </dependency>
    <!--junit5测试-->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.3.1</version>
    </dependency>
    <!--lombok-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.24</version>
    </dependency>
    <!--log4j2的依赖-->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.19.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-slf4j2-impl</artifactId>
        <version>2.19.0</version>
    </dependency>
</dependencies>
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 开启组件扫描 -->
    <context:component-scan base-package="com.example"/>
</beans>
```
> 以上的基本组件扫描是最常用的
>
> `同时还有以下两种组件扫描的写法`
**指定要排除的组件**
```xml
<context:component-scan base-package="com.atguigu.spring6">
    <!-- context:exclude-filter标签：指定排除规则 -->
    <!-- 
 		type：设置排除或包含的依据
		type="annotation"，根据注解排除，expression中设置要排除的注解的全类名
		type="assignable"，根据类型排除，expression中设置要排除的类型的全类名
	-->
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
        <!--<context:exclude-filter type="assignable" expression="com.atguigu.spring6.controller.UserController"/>-->
</context:component-scan>
```
**仅扫描指定组件**
```xml
<context:component-scan base-package="com.atguigu" use-default-filters="false">
    <!-- context:include-filter标签：指定在原有扫描规则的基础上追加的规则 -->
    <!-- use-default-filters属性：取值false表示关闭默认扫描规则 -->
    <!-- 此时必须设置use-default-filters="false"，因为默认规则即扫描指定包下所有类 -->
    <!-- 
 		type：设置排除或包含的依据
		type="annotation"，根据注解排除，expression中设置要排除的注解的全类名
		type="assignable"，根据类型排除，expression中设置要排除的类型的全类名
	-->
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	<!--<context:include-filter type="assignable" expression="com.atguigu.spring6.controller.UserController"/>-->
</context:component-scan>
```
> 定义bean
>
> 此时将以前的`<bean id="" class=""></bean>`这种写法更换为一下写法
```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.stereotype.Component;
@Data
// 不写则默认是首字母小写，即user
@Component("userEntity")
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer uid;
    private String name;
    private String gender;
    private Integer age;
}
```
```java
public class ApplicationTest {
    @Test
    public void test1(){
        ApplicationContext context=new ClassPathXmlApplicationContext("applicationContext.xml");
        // 此处应该写component中写的名称，即userEntity
        User userEntity = context.getBean("userEntity", User.class);
        System.out.println(userEntity);
    }
}
```
### Autowired
> 1.  属性注入
单独使用@Autowired注解，**默认根据类型装配**。【默认是byType】
```java
@Controller
public class UserController {
    @Autowired
    private UserService userService;
    public void userController1(){
        System.out.println("userController1方法已执行...");
        userService.userService1();
    }
}
```
```java
public interface UserService {
    public void userService1();
}
```
```java
@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserDao userDao;
    @Override
    public void userService1() {
        System.out.println("userService1方法执行了...");
        userDao.userDao1();
    }
}
```
```java
public interface UserDao {
    void userDao1();
}
```
```java
@Repository
public class UserDaoImpl implements UserDao {
    @Override
    public void userDao1() {
        System.out.println("userDao1方法执行了...");
    }
}
```
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer id;
    private String name;
}
```
> 2.  set方法注入
```java
@Controller
public class UserController {
    private UserService userService;
    // set方法注入
    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
    public void userController1(){
        System.out.println("userController1方法已执行...");
        userService.userService1();
    }
}
```
> 3.  构造方法注入
```java
@Controller
public class UserController {
    private UserService userService;
    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }
    @Autowired
    public void userController1(){
        System.out.println("userController1方法已执行...");
        userService.userService1();
    }
}
```
> 4.  形参注入
```java
@Controller
public class UserController {
    private UserService userService;
    public UserController(@Autowired UserService userService) {
        this.userService = userService;
    }
    @Autowired
    public void userController1(){
        System.out.println("userController1方法已执行...");
        userService.userService1();
    }
}
```
> 5.  只有一个有参构造函数，注解可以省略
```java
@Controller
public class UserController {
    private UserService userService;
    public UserController(UserService userService) {
        this.userService = userService;
    }
    @Autowired
    public void userController1(){
        System.out.println("userController1方法已执行...");
        userService.userService1();
    }
}
```
> `必须只有一个有参的构造函数`，若多一个无参构造函数，则报错  
> `@Controller不能省略`若省略@Controller，也报错
> 6.  Autowired+Qualifier联合注解  
>     接口有多个实现类，用Qualifier来指定bean名称
```java
@Controller
public class UserController {
    @Autowired
    // 指定bean名称
    @Qualifier("userServiceImpl")
    private UserService userService;
    public UserController(UserService userService) {
        this.userService = userService;
    }
    @Autowired
    public void userController1(){
        System.out.println("userController1方法已执行...");
        userService.userService1();
    }
}
```
### Resource
@Resource注解也可以完成属性注入。那它和@Autowired注解有什么区别？
*   @Resource注解是JDK扩展包中的，也就是说属于JDK的一部分。所以该注解是标准注解，更加具有通用性。(JSR-250标准中制定的注解类型。JSR是Java规范提案。)
*   @Autowired注解是Spring框架自己的。
*   **@Resource注解默认根据名称装配byName，未指定name时，使用属性名作为name。通过name找不到的话会自动启动通过类型byType装配。**
*   **@Autowired注解默认根据类型装配byType，如果想根据名称装配，需要配合@Qualifier注解一起用。**
*   `@Resource注解用在属性上、setter方法上。`
*   `@Autowired注解用在属性上、setter方法上、构造方法上、构造方法参数上。`
@Resource注解属于JDK扩展包，所以不在JDK当中，需要额外引入以下依赖：【如果是JDK8的话不需要额外引入依赖。`高于JDK11或低于JDK8需要引入以下依赖。`】
```xml
<dependency>
    <groupId>jakarta.annotation</groupId>
    <artifactId>jakarta.annotation-api</artifactId>
    <version>2.1.1</version>
</dependency>
```
全注解开发
--------------------------------------------------------------------------
全注解开发就是不再使用spring配置文件了，写一个配置类来代替配置文件。
```java
package com.atguigu.spring6.config;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
@Configuration
//@ComponentScan({"com.atguigu.spring6.controller", "com.atguigu.spring6.service","com.atguigu.spring6.dao"})
@ComponentScan("com.atguigu.spring6")
public class Spring6Config {
}
```
测试类
```java
@Test
public void testAllAnnotation(){
    ApplicationContext context = new AnnotationConfigApplicationContext(Spring6Config.class);
    UserController userController = context.getBean("userController", UserController.class);
    userController.out();
    logger.info("执行成功");
}
```
