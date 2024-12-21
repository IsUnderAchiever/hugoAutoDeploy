---
title: 01_SpringIOC入门
description: 01_SpringIOC入门
date: 2023-01-14
slug: 01_SpringIOC入门
image: 202412212133331.png
categories:
    - Spring
---

## SpringIOC快速入门
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201919655.png)
```tree
_04_SpringIOC_test                           
├─ src                                       
│  ├─ main                                   
│  │  ├─ java                                
│  │  │  └─ com                              
│  │  │     └─ example                       
│  │  │        ├─ dao                        
│  │  │        │  ├─ impl                    
│  │  │        │  │  └─ StudentDaoImpl.java  
│  │  │        │  └─ StudentDao.java         
│  │  │        ├─ entity                     
│  │  │        │  └─ Student.java            
│  │  │        └─ Application_04.java        
│  │  └─ resources                           
│  │     └─ applicationContext.xml           
│  └─ test                                   
│     └─ java                                                        
├─ pom.xml                                   
└─ _04_SpringIOC_test.iml                    
```
```xml
<!-- pom.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.example</groupId>
    <artifactId>_04_SpringIOC_test</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <!--lombok依赖-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
        </dependency>
        <!--Spring依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.14.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```
```xml
<!-- applicationContext.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="com.example.dao.impl.StudentDaoImpl" id="studentDao">
    </bean>
</beans>
```
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Student {
    private String name;
    private Integer id;
    private Integer age;
}
public interface StudentDao {
    Student getStudentById(Integer id);
}
public class StudentDaoImpl implements StudentDao {
    @Override
    public Student getStudentById(Integer id) {
        return new Student("张三",1,20);
    }
}
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class Application_04 {
    public static void main(String[] args) {
        // 创建容器
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        // 获取对象
        StudentDao studentDao= (StudentDao) context.getBean("studentDao");
        Student studentById = studentDao.getStudentById(5);
        System.out.println(studentById);
    }
}
// 运行Application_04
// Student(name=张三, id=1, age=20)
```
>1. 读取applicationContext文件
>2. 根据bean标签的class以反射的方式来创建对象，以bean标签的id为key存入map集合当中
>3. 通过getBean()方法传入key来获取
