---
title: SpringSecurity
description: SpringSecurity
date: 2023-03-26
slug: SpringSecurity
image: 202412212133331.png
categories:
    - Spring
---

SpringSecurity密码加密存储
=====
```java
BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
String encode = passwordEncoder.encode(user.getPassword());
// 配置加密后的密码
user.setPassword(encode);
public static void main(String[] args) {
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        String encode = passwordEncoder.encode("123");
        String encode1 = passwordEncoder.encode("123");
        boolean matches = passwordEncoder.matches("123", encode); //true
        boolean matches1 = passwordEncoder.matches("123", encode1); //true
        boolean matches2 = passwordEncoder.matches(encode,encode1); //false
        System.out.println(matches);
        System.out.println(matches1);
        System.out.println(matches2);
}
```
