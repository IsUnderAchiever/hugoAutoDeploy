---
title: 02_AOP概念
description: 02_AOP概念
date: 2023-01-16
slug: 02_AOP概念
image: 202412212133331.png
categories:
    - Spring
---

## SpringAOP核心概念
- Joinpoint(连接点)︰所谓连接点是指那些可以被增强到的点。在spring中,这些点指的是方法，因为spring只支持方法类型的连接点
- Pointcut(切入点)︰所谓切入点是指被增强的连接点(方法)
- Advice(通知/增强)︰所谓通知是指具体增强的代码
- Target(目标对象)︰被增强的对象就是目标对象
- Aspect(切面)︰是切入点和通知(引介)的结合
- Proxy (代理)︰一个类被AOP增强后，就产生一个结果代理类
### 切点表达式
```java
// 返回值，某个类下某个方法，..代表任意参数，不加代表无参方法
@Pointcut("execution(* com.example.service.*.sayHello(..))")
```
### 切点函数@annotation
我们也可以在要增强的方法上加上注解。然后使用@annotation来表示对加了什么注解的方法进行增强。
写法:@annotation(注解的全类名)
例如:
定义注解如下
```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface InvokeLog {
}
@Pointcut("@annotation(com.example.myannotation.InvokeLog)")
public void pointCut1(){}
@Before("pointCut1()")
public void before(){
	System.out.println("拜拜咯您嘞");
}
@InvokeLog
public void sayBye(){
    System.out.println("bye");
}
phoneService.sayBye();
```
