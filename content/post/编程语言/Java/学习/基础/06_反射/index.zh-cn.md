---
title: 06_反射
description: 06_反射
date: 2023-01-05
slug: 06_反射
image: 202412212036798.png
categories:
    - Java
---

# java 反射
[详细信息推介查看](https://blog.csdn.net/qq_51515673/article/details/124830558)
```java
//获取包名、类名
clazz.getPackage().getName()//包名
clazz.getSimpleName()//类名
clazz.getName()//完整类名
 
//获取成员变量定义信息
getFields()//获取所有公开的成员变量,包括继承变量
getDeclaredFields()//获取本类定义的成员变量,包括私有,但不包括继承的变量
getField(变量名)
getDeclaredField(变量名)
 
//获取构造方法定义信息
getConstructor(参数类型列表)//获取公开的构造方法
getConstructors()//获取所有的公开的构造方法
getDeclaredConstructors()//获取所有的构造方法,包括私有
getDeclaredConstructor(int.class,String.class)
 
//获取方法定义信息
getMethods()//获取所有可见的方法,包括继承的方法
getMethod(方法名,参数类型列表)
getDeclaredMethods()//获取本类定义的的方法,包括私有,不包括继承的方法
getDeclaredMethod(方法名,int.class,String.class)
 
//反射新建实例
clazz.newInstance();//执行无参构造创建对象
clazz.newInstance(222,"韦小宝");//执行有参构造创建对象
clazz.getConstructor(int.class,String.class)//获取构造方法
 
//反射调用成员变量
clazz.getDeclaredField(变量名);//获取变量
clazz.setAccessible(true);//使私有成员允许访问
f.set(实例,值);//为指定实例的变量赋值,静态变量,第一参数给null
f.get(实例);//访问指定实例变量的值,静态变量,第一参数给null
 
//反射调用成员方法
Method m = Clazz.getDeclaredMethod(方法名,参数类型列表);
m.setAccessible(true);//使私有方法允许被调用
m.invoke(实例,参数数据);//让指定实例来执行该方法
```
## 获取对象字节码的3种方式
```java
// 1.
public static void getClazz1() {
    User user = new User();
    Class<? extends User> clazz = user.getClass();
    System.out.println(clazz);
}
// 2.
public static void getClazz2() {
    Class<User> clazz = User.class;
    System.out.println(clazz);
}
// 3.
public static void getClazz3() {
    try {
        Class<?> clazz = Class.forName("com.example.domain.User");
        System.out.println(clazz);
    } catch (ClassNotFoundException e) {
        throw new RuntimeException(e);
    }
}
```
```JAVA
public class Demo1 {
    public static void main(String[] args) {
        System.out.println(getClazz()); // class com.example.domain.User
        System.out.println(getClazz().getName()); // com.example.domain.User
        System.out.println(getClazz().getSimpleName()); // User
        System.out.println(getClazz().getPackage()); // package com.example.domain
        System.out.println(getClazz().getPackage().getName()); // com.example.domain
    }
    public static Class<?> getClazz(){
        try {
            return Class.forName("com.example.domain.User");
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```
```JAVA
package com.example.mytest;
import com.example.domain.User;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.util.List;
public class MyTest {
    public static void main(String[] args) {
        createObjectByReflect();
    }
    /**
     * 获取 Class 对象
     * 方法1
     */
    public static void getClazz1() {
        User user = new User();
        Class<? extends User> clazz = user.getClass();
        System.out.println(clazz);
    }
    /**
     * 获取 Class 对象
     * 方法2
     */
    public static void getClazz2() {
        Class<User> clazz = User.class;
        System.out.println(clazz);
    }
    /**
     * 获取 Class 对象
     * 方法3
     */
    public static void getClazz3() {
        try {
            Class<?> clazz = Class.forName("com.example.domain.User");
            System.out.println(clazz);
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }
    /**
     * 获取类的所有方法信息
     */
    public static void getAllMethods() {
        Class<?> clazz = getClazz();
        Method[] methods = clazz.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println(method);
        }
    }
    /**
     * 获取类的所有成员属性信息
     */
    public static void getAllFields() {
        Field[] declaredFields = getClazz().getDeclaredFields();
        for (Field f : declaredFields) {
            System.out.println(f);
        }
    }
    /**
     * 获取类的构造方法信息
     */
    public static void getAllConstructors() {
        try {
            Constructor<?>[] declaredConstructors = getClazz().getDeclaredConstructors();
            for (Constructor<?> constructor : declaredConstructors) {
                System.out.println(constructor);
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    // 利用反射动态创建对象实例
    /**
     * 通过反射---创建对象
     */
    public static void createObjectByReflect() {
        Class<?> clazz = getClazz();
        try {
            //User user = (User) clazz.newInstance();
            Constructor<?> declaredConstructor = clazz.getDeclaredConstructor(Integer.class,String.class,Integer.class,String.class, List.class);
            User user = (User) declaredConstructor.newInstance(1, "张三", 12, "男", null);
            System.out.println(user);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    /**
     * 提取方法：---获取class对象
     *
     * @return {@link Class}<{@link ?}>
     */
    private static Class<?> getClazz() {
        try {
            return Class.forName("com.example.domain.User");
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }
}
```
```JAVA
// User
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer uid;
    private String name;
    private Integer age;
    private String gender;
    private List<User> children;
}
```
