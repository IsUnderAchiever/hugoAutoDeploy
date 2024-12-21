---
title: 03_通知分类
description: 03_通知分类
date: 2023-01-16
slug: 03_通知分类
image: 202412212133331.png
categories:
    - Spring
---

## 02_Before前置通知
- @Before:前置通知,在方法执行前执行
- @AfterReturning:返回后通知，在目标方法执行后执行，如果出现异常不会执行
- @After:后置通知，在目标方法返回结果之后执行，无论是否出现异常都会执行
- @AfterThrowing:异常通知，在目标方法抛出异常后执行
- @Around:环绕通知，围绕着方法执行
```java
public Object test(){
    // @Before前置通知
    beafore();
    try {
        // 目标方法调用
        Object ret=目标方法();
        // @AfterReturning返回后通知
        afterReturn();
    }catch (Throwable throwable){
        throwable.printStackTrace();
        // @AfterThrowing 异常通知
        afterThrowing();
    }finally {
        // @After 后置通知
        after();
    }
    return ret;
}
```
## 04_AfterReturning返回后异常
> 若目标方法出现`异常`，不执行
```java
System.out.println(1/0);
```
## 05_After后置通知
> 无论目标方法出现`异常`与否，都会执行
## 06_AfterThrowing异常通知
> 如果目标方法出现`异常`，则执行；反之不执行
## 07_Around环绕通知
> 围绕着目标方法执行
```java
@Around("pointCut1()")
public void around(ProceedingJoinPoint point) {
    System.out.println("目标方法前---");
    try {
        // 目标方法执行
        point.proceed();
        System.out.println("目标方法后---");
    } catch (Throwable e) {
        e.printStackTrace();
        System.out.println("出现异常后---");
    }finally {
        System.out.println("finally中进行增强---");
    }
}
```
## 总结
> Around的功能更加强大，IDEA中@Around报黄，提示`Join point return value will be lost `
> 是因为可以返回一个Object对象
