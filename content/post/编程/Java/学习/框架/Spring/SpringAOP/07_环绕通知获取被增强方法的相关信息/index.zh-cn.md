---
title: 07_环绕通知获取被增强方法的相关信息
description: 07_环绕通知获取被增强方法的相关信息
date: 2023-01-18
slug: 07_环绕通知获取被增强方法的相关信息
image: 202412212133331.png
categories:
    - Spring
---

## 07_环绕通知获取被增强方法的相关信息
> 直接在环绕通知方法中增加一个`ProceedingJoinPoint`类型的参数。这个参数封装了被增强方法的相关信息。该参数的proceed()方法被调用相当于被增强方法被执行，调用后的返回值就相当于被增强方法的返回值。例如:
```java
@Around("servicePointCut()")
public Object around(ProceedingJoinPoint point) {
    // 方法调用时传入的参数
    Object[] args = point.getArgs();
    // 被代理对象
    Object target = point.getTarget();
    // 获取被增强方法签名封装的对象
    MethodSignature signature = (MethodSignature) point.getSignature();
    Object proceed = null;
    try {
        proceed = point.proceed();
    } catch (Throwable e) {
        throw new RuntimeException(e);
    }
    return proceed;
}
```
## 环绕通知注意事项
> 需要给环绕通知的方法返回值定义为Object对象，如果不希望更改目标方法的返回值，直接返回目标方法的原本返回值即可。一句话概括，将目标方法的返回值，作为通知方法的返回值进行返回，详情看上述代码
>
> 如果希望篡改目标方法的返回值，更改通知方法的返回值即可
