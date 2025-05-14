---
title: 04_SPEL表达式
description: 04_SPEL表达式
date: 2023-01-14
slug: 04_SPEL表达式
image: 202412212133331.png
categories:
    - Spring
---

## SPEL表达式
> 先写原写法，再看spel表达式和原写法之间的区别
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- game -->
    <bean class="com.example.entity.Game" id="gameEntity">
        <property name="name" value="王者荣耀"/>
        <property name="country" value="中国"/>
        <property name="money" value="88.8"/>
    </bean>
    <!-- student -->
    <bean class="com.example.entity.Student" id="studentEntity">
        <property name="name" value="学生1"/>
        <property name="gender" value="男"/>
        <property name="age" value="22"/>
        <!-- 这里ref指向上面的gameid -->
        <property name="game" ref="gameEntity"/>
    </bean>
</beans>
```
**SPEL表达式**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- game -->
    <bean class="com.example.entity.Game" id="gameEntity">
        <property name="name" value="王者荣耀"/>
        <property name="country" value="中国"/>
        <property name="money" value="#{88+0.8}"/>
    </bean>
    <!-- student -->
    <bean class="com.example.entity.Student" id="studentEntity">
        <property name="name" value="学生1"/>
        <property name="gender" value="男"/>
        <property name="age" value="22"/>
        <!-- 这里ref指向上面的gameid -->
        <property name="game" value="#{gameEntity}"/>
    </bean>
</beans>
```
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201920189.png)
> 注意：SPEL需要写到`value`属性中，而不能写在`ref`属性中
