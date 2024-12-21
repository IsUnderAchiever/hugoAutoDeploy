---
title: 03_Spring_DI
description: 03_Spring_DI
date: 2023-01-14
slug: 03_Spring_DI
image: 202412212133331.png
categories:
    - Spring
---

## Set方法注入&&有参构造注入
> 需知：@Data注解会为类自动生成get、set方法、toString方法
> 使用property标签来为实体类赋值是需要生成set方法的，如果去掉了@Data注解，则需要生成set方法才行
> 若类中不存在set方法，applicationContext.xml会爆红
```java
import lombok.Data;
@Data
public class Student {
    private String name;
    private String gender;
    private Integer age;
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="com.example.entity.Student" id="studentEntity">
        <property name="name" value="学生1"/>
        <property name="gender" value="男"/>
        <property name="age" value="22"/>
    </bean>
</beans>
```
```java
public class Application_05 {
    public static void main(String[] args) {
        // 创建容器
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        // 获取对象 对应applicationContext里bean标签的id
        Student student= (Student) context.getBean("studentEntity");
        System.out.println(student);
    }
}
```
> `value`来设置`Student的属性`，这里bean标签里有`value`属性和`ref`属性需要注意，下面针对ref属性来进行举例
```java
import lombok.Data;
@Data
public class Student {
    private String name;
    private String gender;
    private Integer age;
    private Game game;
}
@Data
public class Game {
    private String name;
    private String country;
    private Double money;
}
// Application_05不变
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- game -->
    <bean class="com.example.entity.Game" id="gameEntity">
        <property name="name" value="王者荣耀"/>
        <property name="country" value="中国"/>
        <property name="money" value="88.8"/>
    </bean>
    <!-- student -->
    <bean class="com.example.entity.Student" id="studentEntity">
        <property name="name" value="学生1"/>
        <property name="gender" value="男"/>
        <property name="age" value="22"/>
        <!-- 这里ref指向上面的gameid -->
        <property name="game" ref="gameEntity"/>
    </bean>
</beans>
```
> ref="gameEntity"这里指向上面game的bean标签的id值
## 有参构造注入
> 和set注入一样，set注入需要存在set方法才能注入
> 这里同样需要存在有参构造方法才能进行注入
> 下图为了区别，就将原本的applicationContext改名了
```java
@Data
// 全参构造方法
@AllArgsConstructor
// 无参构造犯法
@NoArgsConstructor
public class Game {
    private String name;
    private String country;
    private Double money;
}
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Student {
    private String name;
    private String gender;
    private Integer age;
    private Game game;
}
```
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201919774.png)
![2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201920791.png)
**这里报错是正常的，因为你game类的属性还没写完，但是给的是全参构造方法，所以会报错，把属性写完整就不会报错了**
![3](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201920337.png)
这里直接换到测试方法里进行测试
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="com.example.entity.Game" id="gameEntity">
        <constructor-arg name="name" value="原神"/>
        <constructor-arg name="country" value="China"/>
        <constructor-arg name="money" value="136.9"/>
    </bean>
</beans>
```
```java
@Test
public void constructorTest(){
	ApplicationContext context=new ClassPathXmlApplicationContext("application_constructor.xml");
    Game studentEntity = (Game) context.getBean("gameEntity");
    System.out.println(studentEntity);
}
```
> 这里先测试Game实体类，显然是没问题，接下来测试Student类
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="com.example.entity.Game" id="gameEntity">
        <constructor-arg name="name" value="原神"/>
        <constructor-arg name="country" value="China"/>
        <constructor-arg name="money" value="136.9"/>
    </bean>
    <bean class="com.example.entity.Student" id="studentEntity">
        <constructor-arg name="name" value="学生2"/>
        <constructor-arg name="gender" value="女"/>
        <constructor-arg name="age" value="23"/>
        <constructor-arg name="game" ref="gameEntity"/>
    </bean>
</beans>
```
```java
package com.example.test;
import com.example.entity.Student;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MyTest {
    @Test
    public void setterTest(){
        ApplicationContext context=new ClassPathXmlApplicationContext("application_set.xml");
        Student studentEntity = (Student) context.getBean("studentEntity");
        System.out.println(studentEntity);
    }
    @Test
    public void constructorTest(){
        ApplicationContext context=new ClassPathXmlApplicationContext("application_constructor.xml");
        Student studentEntity = (Student) context.getBean("studentEntity");
        System.out.println(studentEntity);
    }
}
```
> ref和set注入的ref是一样的用法，用来关联game的bean
## 复杂属性怎么注入？
```java
package com.example.complexEntity;
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Phone {
    private double price;
    private String name;
    private String password;
    private String path;
}
import java.util.List;
import java.util.Map;
import java.util.Properties;
import java.util.Set;
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private int age;
    private String name;
    private Phone phone;
    private List<String> list;
    private List<Phone> phones;
    private Set<String> set;
    private Map<String,Phone> map;
    private int[] arr;
    private Properties properties;
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- plone -->
    <bean class="com.example.complexEntity.Phone" id="phoneEntity1">
        <property name="price" value="1999.9"/>
        <property name="name" value="骁龙"/>
        <property name="password" value="123456"/>
        <property name="path" value="红米"/>
    </bean>
    <bean class="com.example.complexEntity.Phone" id="phoneEntity2">
        <property name="price" value="2999.9"/>
        <property name="name" value="天玑"/>
        <property name="password" value="654321"/>
        <property name="path" value="vivo"/>
    </bean>
    <!-- user -->
    <bean class="com.example.complexEntity.User" id="userEntity">
        <property name="age" value="22"/>
        <property name="name" value="某用户"/>
        <property name="phone" ref="phoneEntity1"/>
        <property name="list">
            <list>
                <value>元素1</value>
                <value>元素2</value>
            </list>
        </property>
        <property name="phones">
            <list>
                <ref bean="phoneEntity1"/>
                <ref bean="phoneEntity2"/>
            </list>
        </property>
        <property name="set">
            <set>
                <value>aaa</value>
                <value>bbb</value>
                <value>ccc</value>
            </set>
        </property>
        <property name="map">
            <map>
                <entry key="aa" value-ref="phoneEntity1"/>
                <entry key="bb" value-ref="phoneEntity2"/>
            </map>
        </property>
        <property name="arr">
            <array>
                <value>1</value>
                <value>2</value>
                <value>3</value>
                <value>4</value>
                <value>5</value>
            </array>
        </property>
        <property name="properties">
            <props>
                <prop key="k1">v1</prop>
                <prop key="k2">v2</prop>
                <prop key="k3">v3</prop>
            </props>
        </property>
    </bean>
</beans>
```
```java
package com.example.test;
import com.example.complexEntity.User;
import com.example.entity.Student;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MyTest {
    @Test
    public void setterTest() {
        ApplicationContext context = new ClassPathXmlApplicationContext("application_set.xml");
        Student studentEntity = (Student) context.getBean("studentEntity");
        System.out.println(studentEntity);
    }
    @Test
    public void constructorTest() {
        ApplicationContext context = new ClassPathXmlApplicationContext("application_constructor.xml");
        Student studentEntity = (Student) context.getBean("studentEntity");
        System.out.println(studentEntity);
    }
    @Test
    public void complexTest() {
        ApplicationContext context = new ClassPathXmlApplicationContext("application_complex.xml");
        User studentEntity = (User) context.getBean("userEntity");
        System.out.println(studentEntity);
    }
}
```
