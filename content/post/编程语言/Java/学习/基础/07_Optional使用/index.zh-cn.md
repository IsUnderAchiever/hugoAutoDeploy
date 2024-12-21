---
title: 07_Optional使用
description: 07_Optional使用
date: 2023-03-05
slug: 07_Optional使用
image: 202412212036798.png
categories:
    - Java
---

## Optional使用
### 安全的消费值
```java
	/**
	 * 得到用户对象
	 * 
	 * @return
	 */
	public static User getUser() {
		User user = new User();
		user.setUid(1);
		user.setUsername("admin");
		user.setPassword("123456");
		return user;
	}
	
	public static void main(String[] args) {
		User user = getUser();
		Optional<User> userOptional = Optional.ofNullable(user);
      // 如果user对象不为空，则执行ifPresent的lambda表达式；反之，不执行
		userOptional.ifPresent(item -> System.out.println(item.getUsername()));
	}
```
### 安全的获取值
> 1. Optional对象的get方法`(不推介)`
> 2. orElseGet()方法和orElseThrow()方法`(推介)`
```java
	/**
	 * 得到Optional<User>对象
	 * 
	 * @return
	 */
	public static Optional<User> getUser() {
		User user = new User();
		user.setUid(1);
		user.setUsername("admin");
		user.setPassword("123456");
		return Optional.ofNullable(user);
	}
	public static void main(String[] args) {
		Optional<User> userEntity = getUser();
      // 使用Optional对象的get方法来获取User值
		System.out.println(userEntity.get());
	}
```
```java
// 不推介使用Optional对象的get方法，如果user对象为空，会抛出以下异常
Exception in thread "main" java.util.NoSuchElementException: No value present
	at java.util.Optional.get(Optional.java:135)
	at com.example.test.Test1.main(Test1.java:21)
```
`推介使用orElseGet()方法和orElseThrow()方法`
```java
	/**
	 * 得到Optional<User>对象
	 * 
	 * @return
	 */
	public static Optional<User> getUser() {
		User user = new User();
		user.setUid(1);
		user.setUsername("admin");
		user.setPassword("123456");
		return Optional.ofNullable(user);
	}
	public static void main(String[] args) {
		Optional<User> userEntity = getUser();
      // 如果user对象为null，则会返回orElseGet内定义的默认值
		User user = userEntity.orElseGet(() -> new User(2, "哈哈", "123"));
		System.out.println(user);
	}
```
```java
	/**
	 * 得到Optional<User>对象
	 * 
	 * @return
	 */
	public static Optional<User> getUser() {
		User user = new User();
		user.setUid(1);
		user.setUsername("admin");
		user.setPassword("123456");
		return Optional.ofNullable(null);
	}
	public static void main(String[] args) {
		Optional<User> userEntity = getUser();
      // 如果该值为空，则会抛出异常，可通过Spring的全局异常处理优雅的返回值
		User user = userEntity.orElseThrow(() -> new RuntimeException("该值不可为空"));
		System.out.println(user);
	}
```
