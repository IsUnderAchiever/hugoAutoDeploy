---
title: 02_Spring整合MyBatis
description: 02_Spring整合MyBatis
date: 2023-01-20
slug: 02_Spring整合MyBatis
image: 202412212133331.png
categories:
    - Spring
---

## 2_MyBatis复习
> 目录结构
```tree
Spring_MyBatis                        
├─ src                                
│  ├─ main                            
│  │  ├─ java                         
│  │  │  └─ com                       
│  │  │     └─ example                
│  │  │        ├─ domain              
│  │  │        │  └─ User.java        
│  │  │        ├─ mapper              
│  │  │        │  └─ UserMapper.java  
│  │  │        └─ Demo.java           
│  │  └─ resources                    
│  │     ├─ com                       
│  │     │  └─ example                
│  │     │     └─ mapper              
│  │     │        └─ UserMapper.xml   
│  │     ├─ applicationContext.xml    
│  │     ├─ jdbc.properties           
│  │     └─ mybatis-config.xml        
│  └─ test                            
│     └─ java                       
└─ pom.xml                            
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>Spring_MyBatis</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.9</version>
        </dependency>
        <!--mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.30</version>
        </dependency>
    </dependencies>
</project>
```
> mybatis-config.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC
        "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--设置文件所在路径-->
    <properties resource="jdbc.properties"/>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!--指定包-->
        <package name="com.example.mapper"/>
    </mappers>
</configuration>
```
> jdbc.properties
```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/test?serverTimezone=UTC
jdbc.username=root
jdbc.password=123456
```
> UserMapper.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">
    <resultMap id="BaseResultMap" type="com.example.domain.User">
            <id property="id" column="id" jdbcType="INTEGER"/>
            <result property="name" column="name" jdbcType="VARCHAR"/>
            <result property="gender" column="gender" jdbcType="VARCHAR"/>
            <result property="age" column="age" jdbcType="INTEGER"/>
            <result property="address" column="address" jdbcType="VARCHAR"/>
            <result property="email" column="email" jdbcType="VARCHAR"/>
            <result property="phone" column="phone" jdbcType="VARCHAR"/>
    </resultMap>
    <sql id="Base_Column_List">
        id,name,gender,
        age,address,email,
        phone
    </sql>
    <select id="selectByPrimaryKey" parameterType="java.lang.Long" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List" />
        from user
        where  id = #{id,jdbcType=INTEGER} 
    </select>
    <select id="selectAll" resultType="com.example.domain.User">
        select
        <include refid="Base_Column_List" />
        from user
    </select>
    <delete id="deleteByPrimaryKey" parameterType="java.lang.Long">
        delete from user
        where  id = #{id,jdbcType=INTEGER} 
    </delete>
    <insert id="insert" keyColumn="id" keyProperty="id" parameterType="com.example.domain.User" useGeneratedKeys="true">
        insert into user
        ( id,name,gender
        ,age,address,email
        ,phone)
        values (#{id,jdbcType=INTEGER},#{name,jdbcType=VARCHAR},#{gender,jdbcType=VARCHAR}
        ,#{age,jdbcType=INTEGER},#{address,jdbcType=VARCHAR},#{email,jdbcType=VARCHAR}
        ,#{phone,jdbcType=VARCHAR})
    </insert>
    <insert id="insertSelective" keyColumn="id" keyProperty="id" parameterType="com.example.domain.User" useGeneratedKeys="true">
        insert into user
        <trim prefix="(" suffix=")" suffixOverrides=",">
                <if test="id != null">id,</if>
                <if test="name != null">name,</if>
                <if test="gender != null">gender,</if>
                <if test="age != null">age,</if>
                <if test="address != null">address,</if>
                <if test="email != null">email,</if>
                <if test="phone != null">phone,</if>
        </trim>
        <trim prefix="values (" suffix=")" suffixOverrides=",">
                <if test="id != null">#{id,jdbcType=INTEGER},</if>
                <if test="name != null">#{name,jdbcType=VARCHAR},</if>
                <if test="gender != null">#{gender,jdbcType=VARCHAR},</if>
                <if test="age != null">#{age,jdbcType=INTEGER},</if>
                <if test="address != null">#{address,jdbcType=VARCHAR},</if>
                <if test="email != null">#{email,jdbcType=VARCHAR},</if>
                <if test="phone != null">#{phone,jdbcType=VARCHAR},</if>
        </trim>
    </insert>
    <update id="updateByPrimaryKeySelective" parameterType="com.example.domain.User">
        update user
        <set>
                <if test="name != null">
                    name = #{name,jdbcType=VARCHAR},
                </if>
                <if test="gender != null">
                    gender = #{gender,jdbcType=VARCHAR},
                </if>
                <if test="age != null">
                    age = #{age,jdbcType=INTEGER},
                </if>
                <if test="address != null">
                    address = #{address,jdbcType=VARCHAR},
                </if>
                <if test="email != null">
                    email = #{email,jdbcType=VARCHAR},
                </if>
                <if test="phone != null">
                    phone = #{phone,jdbcType=VARCHAR},
                </if>
        </set>
        where   id = #{id,jdbcType=INTEGER} 
    </update>
    <update id="updateByPrimaryKey" parameterType="com.example.domain.User">
        update user
        set 
            name =  #{name,jdbcType=VARCHAR},
            gender =  #{gender,jdbcType=VARCHAR},
            age =  #{age,jdbcType=INTEGER},
            address =  #{address,jdbcType=VARCHAR},
            email =  #{email,jdbcType=VARCHAR},
            phone =  #{phone,jdbcType=VARCHAR}
        where   id = #{id,jdbcType=INTEGER} 
    </update>
</mapper>
```
>UserMapper
```java
public interface UserMapper {
    List<User> selectAll();
    int deleteByPrimaryKey(Long id);
    int insert(User record);
    int insertSelective(User record);
    User selectByPrimaryKey(Long id);
    int updateByPrimaryKeySelective(User record);
    int updateByPrimaryKey(User record);
}
```
> User
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User implements Serializable {
    /**
     * 用户id
     */
    private Integer id;
    /**
     * 用户名字
     */
    private String name;
    /**
     * 用户性别
     */
    private String gender;
    /**
     * 用户年龄
     */
    private Integer age;
    /**
     * 用户地址
     */
    private String address;
    /**
     * 用户邮箱
     */
    private String email;
    /**
     * 用户手机号
     */
    private String phone;
    private static final long serialVersionUID = 1L;
    @Override
    public boolean equals(Object that) {
        if (this == that) {
            return true;
        }
        if (that == null) {
            return false;
        }
        if (getClass() != that.getClass()) {
            return false;
        }
        User other = (User) that;
        return (this.getId() == null ? other.getId() == null : this.getId().equals(other.getId()))
            && (this.getName() == null ? other.getName() == null : this.getName().equals(other.getName()))
            && (this.getGender() == null ? other.getGender() == null : this.getGender().equals(other.getGender()))
            && (this.getAge() == null ? other.getAge() == null : this.getAge().equals(other.getAge()))
            && (this.getAddress() == null ? other.getAddress() == null : this.getAddress().equals(other.getAddress()))
            && (this.getEmail() == null ? other.getEmail() == null : this.getEmail().equals(other.getEmail()))
            && (this.getPhone() == null ? other.getPhone() == null : this.getPhone().equals(other.getPhone()));
    }
    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + ((getId() == null) ? 0 : getId().hashCode());
        result = prime * result + ((getName() == null) ? 0 : getName().hashCode());
        result = prime * result + ((getGender() == null) ? 0 : getGender().hashCode());
        result = prime * result + ((getAge() == null) ? 0 : getAge().hashCode());
        result = prime * result + ((getAddress() == null) ? 0 : getAddress().hashCode());
        result = prime * result + ((getEmail() == null) ? 0 : getEmail().hashCode());
        result = prime * result + ((getPhone() == null) ? 0 : getPhone().hashCode());
        return result;
    }
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append(getClass().getSimpleName());
        sb.append(" [");
        sb.append("Hash = ").append(hashCode());
        sb.append(", id=").append(id);
        sb.append(", name=").append(name);
        sb.append(", gender=").append(gender);
        sb.append(", age=").append(age);
        sb.append(", address=").append(address);
        sb.append(", email=").append(email);
        sb.append(", phone=").append(phone);
        sb.append(", serialVersionUID=").append(serialVersionUID);
        sb.append("]");
        return sb.toString();
    }
}
```
> 测试MyBatis
```java
public class Demo {
    public static void main(String[] args) throws Exception {
        // 定义mybatis配置文件的路径
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        // 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 获取UserMapper实现类对象
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        // 调用方法测试
        List<User> users = userMapper.selectAll();
        System.out.println(users);
        // 释放资源
        sqlSession.close();
    }
}
```
## 14_Spring整合MyBatis
[参考文档](https://mybatis.org/spring/zh/index.html)
> MyBatis-String连接包版本对应
| MyBatis-Spring | MyBatis | Spring Framework | Spring Batch | Java     |
| :------------- | :------ | :--------------- | :----------- | :------- |
| **3.0**        | 3.5+    | 6.0+             | 5.0+         | Java 17+ |
| **2.1**        | 3.5+    | 5.x              | 4.x          | Java 8+  |
| **2.0**        | 3.5+    | 5.x              | 4.x          | Java 8+  |
| **1.3**        | 3.4+    | 3.2.2+           | 2.1+         | Java 6+  |
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>Spring_MyBatis</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
        </dependency>
        <!--Junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <!--Spring整合Junit依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.3.18</version>
        </dependency>
        <!--Spring JDBC-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.3.9</version>
        </dependency>
        <!--ioc-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.9</version>
        </dependency>
        <!--mybatis整合spring-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.6</version>
        </dependency>
        <!--druid数据源-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.9</version>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.9</version>
        </dependency>
        <!--mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.30</version>
        </dependency>
    </dependencies>
</project>
```
> 往容器中注入整合相关对象
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="com.example"/>
    <context:property-placeholder location="classpath:jdbc.properties"/>
    <!--配置Druid连接池-->
    <bean class="com.alibaba.druid.pool.DruidDataSource" id="dataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
    <!--配置Mapper扫描器，扫描到的mapper对象会被注入Spring容器中-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer" id="mapperScannerConfigurer">
        <!--mapper.xml的路径-->
        <property name="basePackage" value="com.example.mapper"/>
    </bean>
    <bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactory">
        <property name="dataSource" ref="dataSource"/>
        <!--设置MyBatis配置文件的路径-->
        <property name="configLocation" value="mybatis-config.xml"/>
    </bean>
</beans>
```
> mybatis-config.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC
        "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--别名配置-->
    <typeAliases>
        <package name="com.example.domain"/>
    </typeAliases>
</configuration>
```
> 测试
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")
public class MyTest {
    @Autowired
    private UserMapper userMapper;
    @Test
    public void test1(){
        System.out.println(userMapper.selectAll());
    }
}
```
## MyBatis配置别名的作用
> 例子
![database](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20230120142044740.png)
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")
public class MyTest {
    @Autowired
    private UserMapper userMapper;
    @Test
    public void test1(){
        User user = new User(4,"哈哈","男",12,"湖南","xxx@qq.com","131xxxx5494");
        userMapper.insert(user);
    }
}
```
![image-20230120145745385](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201457499.png)
`测试结果`：
![image-20230120145944254](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201459301.png)
> 在配置了别名后，可继续使用上面的`全类名`写法，也可以只写`类名`
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")
public class MyTest {
    @Autowired
    private UserMapper userMapper;
    @Test
    public void test1(){
        User user = new User(5,"呵呵","女",15,"湖南","xxx@qq.com","131xxxx5494");
        userMapper.insert(user);
    }
}
```
![image-20230120150113451](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201501580.png)
![image-20230120150143868](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201501920.png)
>它的作用是让Mapper.xml中的参数找到对应类,如下面parameterType="User"，如果没有配置别名，则要改为parameterType="com.example.domain.User">
