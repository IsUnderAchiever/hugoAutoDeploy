---
title: 04_实现声明式事务
description: 04_实现声明式事务
date: 2023-01-20
slug: 04_实现声明式事务
image: 202412212133331.png
categories:
    - Spring
---

# 4_实现声明式事务
![202301201818047.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201818047.png)
> 这里会发生异常，用来模拟支付过程中的突发情况导致支付失败，如网络原因等
>
> 测试后发现，前者的余额已经增加，但后者的余额由于发生异常导致没有减少
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>Spring_MyBatis_1</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
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
        <!--Spring JDBC-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.3.9</version>
        </dependency>
        <!--ioc-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.9</version>
        </dependency>
        <!--aop-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.6</version>
        </dependency>
        <!--mybatis整合spring-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.6</version>
        </dependency>
        <!--druid数据源-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.9</version>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.9</version>
        </dependency>
        <!--mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.30</version>
        </dependency>
    </dependencies>
</project>
```
## 注解实现
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd">
    <context:component-scan base-package="com.example"/>
    <context:property-placeholder location="classpath:jdbc.properties"/>
    <!--配置Druid连接池-->
    <bean class="com.alibaba.druid.pool.DruidDataSource" id="dataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
    <!--配置Mapper扫描器，扫描到的mapper对象会被注入Spring容器中-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer" id="mapperScannerConfigurer">
        <!--mapper.xml的路径-->
        <property name="basePackage" value="com.example.mapper"/>
    </bean>
    <bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactory">
        <property name="dataSource" ref="dataSource"/>
        <!--设置MyBatis配置文件的路径-->
        <property name="configLocation" value="mybatis-config.xml"/>
    </bean>
    <!--把事务管理器注入Spring容器，需要配置一个连接池-->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--开启事务注解驱动，配置使用的事务管理器-->
    <tx:annotation-driven transaction-manager="txManager"/>
</beans>
```
```java
@Service
public class CountServiceImpl implements CountService {
    @Autowired
    private CountMapper countMapper;
    @Transactional
    @Override
    public void transfer(Integer outUserId, Integer inUserId, Integer money) {
        // 增加
        countMapper.updateMoney(inUserId, money);
        //System.out.println(1/0);
        // 减少
        countMapper.updateMoney(outUserId, -money);
    }
}
```
> `@Transactional`可以加到方法上也可以加到类上，若在类上，则类中所有方法都被事务管理
## XML实现
```xml
<!--把事务管理器注入Spring容器，需要配置一个连接池-->
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
```
```xml
<!--定义事务管理的通知类-->
<tx:advice transaction-manager="txManager" id="txAdvice">
    <tx:attributes>
        <!--针对transfer进行事务控制-->
        <tx:method name="transfer"/>
    </tx:attributes>
</tx:advice>
<aop:config>
    <aop:pointcut id="pointCut" expression="execution(* com.example.service..*.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="pointCut"/>
</aop:config>
```
