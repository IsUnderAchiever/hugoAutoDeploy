---
title: 07_Spring低频知识点
description: 07_Spring低频知识点
date: 2023-01-14
slug: 07_Spring低频知识点
image: 202412212133331.png
categories:
    - Spring
---

## Spring低频知识点
> bean的配置
> 1. name属性
> 2. lazy-init
> 3. init-method
> 4. destory-method
> 5. factory-bean&factory-method
### bean的配置
#### name属性
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="com.example.entity.User" id="userEntity" name="user1">
        <property name="uid" value="1"/>
        <property name="username" value="admin"/>
        <property name="password" value="123456"/>
    </bean>
</beans>
```
```java
public class Application_08 {
    public static void main(String[] args) {
        ApplicationContext context=new ClassPathXmlApplicationContext("applicationContext.xml");
        User user= (User) context.getBean("userEntity");
        System.out.println(user);
    }
}
// User(uid=1, username=admin, password=123456)
public class Application_08 {
    public static void main(String[] args) {
        ApplicationContext context=new ClassPathXmlApplicationContext("applicationContext.xml");
        User user= (User) context.getBean("user1");
        System.out.println(user);
    }
}
// User(uid=1, username=admin, password=123456)
```
> ```java
> User user= (User) context.getBean("user1");
> ```
>
> 可以通过id or name来获取bean对象
> name可以赋多个名字，如下
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="com.example.entity.User" id="userEntity" name="user1,user2">
        <property name="uid" value="1"/>
        <property name="username" value="admin"/>
        <property name="password" value="123456"/>
    </bean>
</beans>
```
```java
public class Application_08 {
    public static void main(String[] args) {
        ApplicationContext context=new ClassPathXmlApplicationContext("applicationContext.xml");
        User user= (User) context.getBean("user2");
        System.out.println(user);
    }
}
```
#### lazy-init
>  `scope="singleton"`单例bean，默认在容器创建时创建
> 而`lazy-init`是在第一次调用getBean方法是创建对象
> 和`懒加载`类似，需要用到的时候再加载
> `lazy-init="true"`或者`lazy-init="false"`两种情况来进行对比
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="com.example.entity.User" lazy-init="true" id="userEntity" name="user1,user2">
        <property name="uid" value="1"/>
        <property name="username" value="admin"/>
        <property name="password" value="123456"/>
    </bean>
</beans>
```
#### init-method
> 设置出初始化方法，在对象创建之后，自动调用指定方法
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer uid;
    private String username;
    private String password;
    private void initUser(){
        System.out.println("学生对象初始化中---");
    }
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="com.example.entity.User" id="userEntity" init-method="initUser">
        <property name="uid" value="1"/>
        <property name="username" value="admin"/>
        <property name="password" value="123456"/>
    </bean>
</beans>
```
```java
// 学生对象初始化中---
// User(uid=1, username=admin, password=123456)
```
> 注意：该初始化方法必须是`空参`方法
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201921224.png)
#### destory-method
> 和`init-method`一样，这两个方法都需要是空参方法
> 怎么样销毁`user对象`？注意，配置文件中默认是`单例bean`，所以在`容器`销毁的时候，会销毁对象
```java
public class Application_08 {
    public static void main(String[] args) throws InterruptedException {
        ClassPathXmlApplicationContext context=new ClassPathXmlApplicationContext("applicationContext.xml");
        User user= (User) context.getBean("userEntity");
        System.out.println(user);
        context.close();
    }
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="com.example.entity.User" id="userEntity" init-method="initUser" destroy-method="destoryUser">
        <property name="uid" value="1"/>
        <property name="username" value="admin"/>
        <property name="password" value="123456"/>
    </bean>
</beans>
```
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer uid;
    private String username;
    private String password;
    private void initUser(){
        System.out.println("学生对象初始化中---");
    }
    private void destoryUser(){
        System.out.println("学生对象销毁前调用，用于释放资源---");
    }
}
```
#### factory-bean&factory-method
```java
@Data
public class Car {
    /**
     * 发动机
     */
    private Motor motor;
    /**
     * 方向盘
     */
    private SteeringWheel steeringWheel;
    /**
     * 轮胎
     */
    private Tyre tyre;
    /**
     * 测试发动机
     */
    public void testMotor(){
        System.out.println("测试发动机---");
    }
    /**
     * 测试方向盘
     */
    public void testSteeringWheel(){
        System.out.println("测试方向盘---");
    }
    /**
     * 测试轮胎
     */
    public void testTyre(){
        System.out.println("测试轮胎---");
    }
}
public class Motor {
}
public class SteeringWheel {
}
public class Tyre {
}
```
```java
/**
 * @author 深海
 * @version 1.0.0
 * @date 2023/01/14
 * @doc 汽车实例工厂
 * @since 1.0.0
 */
public class CarFactory {
    public Car getCar(){
        Car car = new Car();
        // 设置属性
        car.setMotor(new Motor());
        car.setSteeringWheel(new SteeringWheel());
        car.setTyre(new Tyre());
        // 调用相关方法进行调试
        car.testMotor();
        car.testSteeringWheel();
        car.testTyre();
        return car;
    }
}
/**
 * @author 深海
 * @version 1.0.0
 * @date 2023/01/14
 * @doc 汽车静态工厂
 * @since 1.0.0
 */
public class CarStaticFactory {
    public static Car getCar(){
        Car car = new Car();
        // 设置属性
        car.setMotor(new Motor());
        car.setSteeringWheel(new SteeringWheel());
        car.setTyre(new Tyre());
        // 调用相关方法进行调试
        car.testMotor();
        car.testSteeringWheel();
        car.testTyre();
        return car;
    }
}
```
```java
public class MyTest {
    @Test
    public void test1(){
        Car car = new Car();
        // 设置属性
        car.setMotor(new Motor());
        car.setSteeringWheel(new SteeringWheel());
        car.setTyre(new Tyre());
        // 调用相关方法进行调试
        car.testMotor();
        car.testSteeringWheel();
        car.testTyre();
        System.out.println(car);
    }
    /**
     * 使用实例工厂来创建Car对象
     */
    @Test
    public void test2(){
        CarFactory factory=new CarFactory();
        Car car=factory.getCar();
        System.out.println(car);
    }
    /**
     * 使用静态工厂来创建Car对象
     */
    @Test
    public void test3(){
        Car car=CarStaticFactory.getCar();
        System.out.println(car);
    }
}
```
> 总体来说，`实例工厂`和`静态工厂`就是提取代码、增加代码复用性的作用
> 上述方法还是手动调用方法，接下来使用Spring来自动调用
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 创建实例工厂 -->
    <bean class="com.example.factory.CarFactory" id="carFactory"/>
    <!-- 使用实例工厂创建Car，注入容器 -->
    <bean factory-bean="carFactory" factory-method="getCar" id="car1"/>
</beans>
```
```java
@Test
public void test4(){
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    Car car1 = (Car) context.getBean("car1");
    System.out.println(car1);
}
```
------
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
<!--    &lt;!&ndash; 创建实例工厂 &ndash;&gt;-->
<!--    <bean class="com.example.factory.CarFactory" id="carFactory"/>-->
<!--    &lt;!&ndash; 使用实例工厂创建Car，注入容器 &ndash;&gt;-->
<!--    <bean factory-bean="carFactory" factory-method="getCar" id="car1"/>-->
    <!-- 使用静态工厂创建Car，注入容器 -->
    <bean class="com.example.factory.CarStaticFactory" factory-method="getCar" id="car2"/>
</beans>
```
```java
@Test
public void test5(){
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    Car car2 = (Car) context.getBean("car2");
    System.out.println(car2);
}
```
> 细心地小伙伴可能发现了，为啥结果输出了两遍？
![2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201921146.png)
> 测试`实例工厂`的时候，注释掉`静态工厂`的配置
> 测试`静态工厂`的时候，注释掉`实例工厂`的配置
> 其实就是两个工厂的配置之间的影响，测试哪个，就留哪个的配置
