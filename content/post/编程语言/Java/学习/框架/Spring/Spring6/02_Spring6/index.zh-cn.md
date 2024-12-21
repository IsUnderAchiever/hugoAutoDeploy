---
title: 02_Spring6
description: 02_Spring6
date: 2023-03-11
slug: 02_Spring6
image: 202412212133331.png
categories:
    - Spring
---

# Spring6学习02
## 引入外部文件
> 1. 引入数据库相关依赖
> 2. 创建外部属性文件，如properties，定义数据库连接信息(用户名、密码...)
> 3. 创建spring配置文件，引入context命名空间来引入属性文件，使用表达式完成注入
![image-20230311215722456](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303112157765.png)
```java
public class ApplicationTest {
    @Test
    public void test1() {
        try (DruidDataSource dataSource = new DruidDataSource()) {
            dataSource.setDriverClassName("oracle.jdbc.driver.OracleDriver");
            dataSource.setUrl("jdbc:oracle:thin:@localhost:1521:orcl");
            dataSource.setUsername("root");
            dataSource.setPassword("123456");
            System.out.println(dataSource);
        } catch (Exception e) {
            throw new RuntimeException("出现异常", e);
        }
    }
}
```
> 使用spring改造以上的代码
```properties
jdbc.driver=oracle.jdbc.driver.OracleDriver
jdbc.url=jdbc:oracle:thin:@localhost:1521:orcl
jdbc.username=root
jdbc.password=123456
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 引入属性 -->
    <context:property-placeholder location="jdbc.properties"/>
    <!-- 完成数据库信息注入 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```
```java
public class ApplicationTest {
    @Test
    public void test2(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        DruidDataSource dataSource = context.getBean(DruidDataSource.class);
        System.out.println(dataSource);
    }
}
```
![image-20230311220327244](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303112203370.png)
## bean的作用域
在Spring中可以通过配置bean标签的scope属性来指定bean的作用域范围，各取值含义参加下表：
| 取值              | 含义                                    | 创建对象的时机  |
| ----------------- | --------------------------------------- | --------------- |
| singleton（默认） | 在IOC容器中，这个bean的对象始终为单实例 | IOC容器初始化时 |
| prototype         | 这个bean在IOC容器中有多个实例           | 获取bean时      |
如果是在WebApplicationContext环境下还会有另外几个作用域（但不常用）：
| 取值    | 含义                 |
| ------- | -------------------- |
| request | 在一个请求范围内有效 |
| session | 在一个会话范围内有效 |
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
    <bean id="user1" class="com.example.domain.User" scope="singleton">
        <property name="id" value="1"/>
        <property name="name" value="哈哈"/>
    </bean>
    <bean id="user2" class="com.example.domain.User" scope="prototype">
        <property name="id" value="1"/>
        <property name="name" value="哈哈"/>
    </bean>
</beans>
```
```java
@Setter
public class User {
    private Integer id;
    private String name;
}
```
```java
@Test
public void test3() {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    User user1_1 = context.getBean("user1", User.class);
    User user1_2 = context.getBean("user1", User.class);
    System.out.println(user1_1);
    System.out.println(user1_2);
}
```
```java
// 输出
com.example.domain.User@121314f7
com.example.domain.User@121314f7
```
> 这里可以看到 两个对象都是同一个地址，即同一个对象
```java
@Test
public void test4() {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    User user1_1 = context.getBean("user2", User.class);
    User user1_2 = context.getBean("user2", User.class);
    System.out.println(user1_1);
    System.out.println(user1_2);
}
```
```java
// 输出
com.example.domain.User@130c12b7
com.example.domain.User@5e600dd5
```
> 这里显示对象的地址不同，即创建了多个对象
## bean的生命周期
> 1. bean对象创建（调用无参构造器）
> 2. 给bean对象设置属性
>
> 3. bean的后置处理器（初始化之前）
>
> 4. bean对象初始化（需在配置bean时指定初始化方法）
>
> 5. bean的后置处理器（初始化之后）
>
> 6. bean对象就绪可以使用
>
> 7. bean对象销毁（需在配置bean时指定销毁方法）
>
> 8. IOC容器关闭
