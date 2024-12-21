---
title: 10_多切面顺序问题
description: 10_多切面顺序问题
date: 2023-01-20
slug: 10_多切面顺序问题
image: 202412212133331.png
categories:
    - Spring
---
---
permalink: "/编程语言/Java/学习/框架/Spring/SpringAOP/10_多切面顺序问题.html"
title: 10_多切面顺序问题
date: 2023-1-20 8:00:00
categories: "SpringAOP"
tags: "SpringAOP"
---
## 10_多切面顺序问题
>在实际项目中我们可能会存在配置了多个切面的情况。这种情况下我们很可能需要控制切面的顺序。我们在默认情况下Spring有它自己的排序规则。(按照类名排序)
>默认排序规则往往不符合我们的要求，我们需要进行特殊控制。
>如果是注解方式配置的AOP可以在切面类上加`@Order`注解来控制顺序。如果是XML方式配置的AOP,可以通过配置顺序来控制。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">
    <context:component-scan base-package="com.example"/>
    <aop:aspectj-autoproxy/>
</beans>
```
```properties
@Service
public class DoService {
    public void doService(){
        System.out.println("doService");
    }
}
```
```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface Log {
}
```
```java
@Component
@Aspect
@Order(2)
public class MyAspect {
    @Pointcut("execution(* com.example.service..*.doService(..))")
    public void doService(){}
    @Around("doService()")
    public Object doAround(ProceedingJoinPoint pjp){
        Object result = null;
        try {
            System.out.println("MyAspect增强");
            result=pjp.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        return result;
    }
}
```
```java
@Component
@Aspect
@Order(1)
public class MyAspect2 {
    @Pointcut("execution(* com.example.service..*.doService(..))")
    public void doService(){}
    @Around("doService()")
    public Object doAround(ProceedingJoinPoint pjp){
        Object result = null;
        try {
            System.out.println("MyAspect2增强");
            result=pjp.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        return result;
    }
}
```
```java
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        DoService bean = context.getBean(DoService.class);
        bean.doService();
    }
}
```
```java
// 执行结果
MyAspect2增强
MyAspect增强
doService
```
## 总结
> order越小，优先级越高
