---
title: 04_获取被增强方法的相关信息
description: 04_获取被增强方法的相关信息
date: 2023-01-16
slug: 04_获取被增强方法的相关信息
image: 202412212133331.png
categories:
    - Spring
---

## 04_获取被增强方法的相关信息
我们可以在除了环绕通知外的所有通知方法中增加一个`JoinPoint类型`的参数。这个参数封装了被增强方法的相关信息。`我们可以通过这个参数获取到除了异常对象和返回值之外的所有信息。`
```java
// 获取传入参数
joinPoint.getArgs();
// 获得签名，签名可以用来获得详细信息
joinPoint.getSignature();
// 获得增强的方法名，String
joinPoint.getSignature().getName();
// 获得全类名，String
joinPoint.getSignature().getDeclaringTypeName();
// 获得类的Class对象
joinPoint.getSignature().getDeclaringType();
```
> 虽然签名`Signature`有一些有用的方法，但其实还不够多，所以选择`MethodSignature`来进行接收
> `MethodSignature`是`Signature`的子接口
> `MethodSignature`的方法更丰富
```java
@Before("pointCut1()")
public void before(JoinPoint joinPoint) {
    System.out.println("before---");
    MethodSignature signature = (MethodSignature) joinPoint.getSignature();
    Method method = signature.getMethod();
    method.invoke();
    System.out.println(joinPoint);
}
```
## DeBug
> debug模式的`小计算器`可以直接测试你希望测试的方法并显示返回值
> 如上述的一些方法
> `joinPoint.getArgs();`、`joinPoint.getSignature();`
