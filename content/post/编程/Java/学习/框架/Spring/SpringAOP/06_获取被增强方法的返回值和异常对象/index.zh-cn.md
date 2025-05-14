---
title: 06_获取被增强方法的返回值和异常对象
description: 06_获取被增强方法的返回值和异常对象
date: 2023-01-18
slug: 06_获取被增强方法的返回值和异常对象
image: 202412212133331.png
categories:
    - Spring
---

## 06_获取被增强方法的返回值和异常对象
> 获取方法返回值，由于前置通知是在目标方法执行之前，所以前置通知是无法获取到返回值的
> 应该在@AfterReturning里获取，配置returning属性
> 获取异常对象是在@AfterThrowing里获取，配置throwing属性
> 示例如下：
```java
@AfterReturning(value = "servicePointCut()",returning = "ret")
public void afterReturning(JoinPoint point,Object ret){
    System.out.println("方法的返回对象是:"+ret);
}
@AfterThrowing(value = "servicePointCut()",throwing = "throwable")
public void afterReturning(JoinPoint point,Throwable throwable){
    System.out.println("方法的异常对象是:"+throwable);
}
```
