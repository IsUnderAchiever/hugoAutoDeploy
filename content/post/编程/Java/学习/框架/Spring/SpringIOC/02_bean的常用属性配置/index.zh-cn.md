---
title: 02_bean的常用属性配置
description: 02_bean的常用属性配置
date: 2023-01-14
slug: 02_bean的常用属性配置
image: 202412212133331.png
categories:
    - Spring
---

## bean的常用属性配置
### id
bean的唯一表示，同一个Spring容器中不允许重复
### class
全类名，用于反射创建对象
### scope
scope主要有两个值：singleton、prototype
如果设置为singleton，则一个容器中只会有一个bean对象，`多个方法共用一个bean对象`
如果设置为prototype，则一个容器中会有多个该bean对象。每次调用getBean方法都会创建一个新对象
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- scope="prototype"多例 ，scope="singleton"单例， 默认是单例的 -->
    <!-- 多例：每调用一次getBean方法会创建一个新的对象，debug可以看到对象地址，单例和多例的两次对象地址不一样 -->
    <bean class="com.example.dao.impl.StudentDaoImpl" id="studentDao" scope="prototype">
    </bean>
</beans>
```
