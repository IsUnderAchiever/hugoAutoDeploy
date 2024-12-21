---
title: 03_声明式事务
description: 03_声明式事务
date: 2023-01-20
slug: 03_声明式事务
image: 202412212133331.png
categories:
    - Spring
---

## 3_事务
> 事务的概念
>
> 保证—组数据库的操作，要么同时成功，要么同时失败
>
> 如：A转账给B，A的钱减少，B的钱增加
### 事务的四大特性
- 隔离性
  多个事务之间要相互隔离，不能互相干扰
- 原子性
  指事务是一个不可分割的整体，类似一个不可分割的原子
- —致性
  保障事务前后这组数据的状态是一致的。要么都是成功的，要么都是失败的。
- 持久性
  指事务一旦被提交，这组操作修改的数据就真的的发生变化了。即使接下来数据库故障也不应该对其有影响。
## 声明式事务
> 如果我们自己去对事务进行控制的话我们就需要值原来核心代码的基础上加上事务控制相关的代码。而在我们的实际开发中这种事务控制的操作也是非常常见的。所以Spring提供了声明式事务的方式让我们去控制事务。
> 只要简单的加个注解(或者是xml配置)就可以实现事务控制，不需要事务控制的时候只需要去掉相应的注解即可。
### 转账案例-环境搭建
![image-20230120154010810](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201540933.png)
```tree
Spring_MyBatis_1                               
├─ src                                         
│  ├─ main                                     
│  │  ├─ java                                  
│  │  │  └─ com                                
│  │  │     └─ example                         
│  │  │        ├─ domain                       
│  │  │        │  └─ Count.java                
│  │  │        ├─ mapper                       
│  │  │        │  └─ CountMapper.java          
│  │  │        ├─ service                      
│  │  │        │  ├─ impl                      
│  │  │        │  │  └─ CountServiceImpl.java  
│  │  │        │  └─ CountService.java         
│  │  │        └─ Main.java                    
│  │  └─ resources                             
│  │     ├─ com                                
│  │     │  └─ example                         
│  │     │     └─ mapper                       
│  │     │        └─ CountMapper.xml           
│  │     ├─ applicationContext.xml             
│  │     ├─ jdbc.properties                    
│  │     └─ mybatis-config.xml                 
│  └─ test                                     
│     └─ java                                  
│        └─ com                                
│           └─ example                         
│              └─ test                         
│                 └─ MyTest.java                              
└─ pom.xml                                     
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>Spring_MyBatis_1</artifactId>
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
        <!--aop-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.6</version>
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
```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/test?serverTimezone=UTC
jdbc.username=root
jdbc.password=123456
```
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC
        "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!-- 打印sql日志 -->
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
    <!--别名配置-->
    <typeAliases>
        <package name="com.example.domain"/>
    </typeAliases>
</configuration>
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.CountMapper">
    <resultMap id="BaseResultMap" type="com.example.domain.Count">
        <id property="id" column="id" jdbcType="INTEGER"/>
        <result property="name" column="name" jdbcType="VARCHAR"/>
        <result property="money" column="money" jdbcType="INTEGER"/>
    </resultMap>
    <sql id="Base_Column_List">
        id
        ,name,money
    </sql>
    <select id="selectByPrimaryKey" parameterType="java.lang.Long" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List"/>
        from count
        where id = #{id,jdbcType=INTEGER}
    </select>
    <delete id="deleteByPrimaryKey" parameterType="java.lang.Long">
        delete
        from count
        where id = #{id,jdbcType=INTEGER}
    </delete>
    <insert id="insert" keyColumn="id" keyProperty="id" parameterType="com.example.domain.Count"
            useGeneratedKeys="true">
        insert into count
            (id, name, money)
        values (#{id,jdbcType=INTEGER}, #{name,jdbcType=VARCHAR}, #{money,jdbcType=INTEGER})
    </insert>
    <insert id="insertSelective" keyColumn="id" keyProperty="id" parameterType="com.example.domain.Count"
            useGeneratedKeys="true">
        insert into count
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="id != null">id,</if>
            <if test="name != null">name,</if>
            <if test="money != null">money,</if>
        </trim>
        <trim prefix="values (" suffix=")" suffixOverrides=",">
            <if test="id != null">#{id,jdbcType=INTEGER},</if>
            <if test="name != null">#{name,jdbcType=VARCHAR},</if>
            <if test="money != null">#{money,jdbcType=INTEGER},</if>
        </trim>
    </insert>
    <update id="updateByPrimaryKeySelective" parameterType="com.example.domain.Count">
        update count
        <set>
            <if test="name != null">
                name = #{name,jdbcType=VARCHAR},
            </if>
            <if test="money != null">
                money = #{money,jdbcType=INTEGER},
            </if>
        </set>
        where id = #{id,jdbcType=INTEGER}
    </update>
    <update id="updateByPrimaryKey" parameterType="com.example.domain.Count">
        update count
        set name  = #{name,jdbcType=VARCHAR},
            money = #{money,jdbcType=INTEGER}
        where id = #{id,jdbcType=INTEGER}
    </update>
    <update id="updateMoney">
        update count
        set money = money + #{updateMoney,jdbcType=INTEGER}
        where id = #{id,jdbcType=INTEGER}
    </update>
</mapper>
```
```java
@Data
public class Count implements Serializable {
    /**
     * 账户id
     */
    private Integer id;
    /**
     * 账户名
     */
    private String name;
    /**
     * 余额
     */
    private Integer money;
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
        Count other = (Count) that;
        return (this.getId() == null ? other.getId() == null : this.getId().equals(other.getId()))
            && (this.getName() == null ? other.getName() == null : this.getName().equals(other.getName()))
            && (this.getMoney() == null ? other.getMoney() == null : this.getMoney().equals(other.getMoney()));
    }
    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + ((getId() == null) ? 0 : getId().hashCode());
        result = prime * result + ((getName() == null) ? 0 : getName().hashCode());
        result = prime * result + ((getMoney() == null) ? 0 : getMoney().hashCode());
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
        sb.append(", money=").append(money);
        sb.append(", serialVersionUID=").append(serialVersionUID);
        sb.append("]");
        return sb.toString();
    }
}
```
```java
public interface CountMapper {
    int deleteByPrimaryKey(Long id);
    int insert(Count record);
    int insertSelective(Count record);
    Count selectByPrimaryKey(Long id);
    int updateByPrimaryKeySelective(Count record);
    int updateByPrimaryKey(Count record);
    // 增加和减少都是update
    void updateMoney(@Param("id") Integer id,@Param("updateMoney") Integer updateMoney);
}
```
```java
public interface CountService {
    void transfer(Integer outUserId, Integer inUserId, Integer money);
}
```
```java
@Service
public class CountServiceImpl implements CountService {
    @Autowired
    private CountMapper countMapper;
    @Override
    public void transfer(Integer outUserId, Integer inUserId, Integer money) {
        // 增加
        countMapper.updateMoney(inUserId, money);
        // 减少
        countMapper.updateMoney(outUserId, -money);
    }
}
```
> 测试方法
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
public class MyTest {
    @Autowired
    private CountService countService;
    @Test
    public void test1() {
        countService.transfer(1,2,10);
    }
}
```
```log
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@6f8e8894] will not be managed by Spring
==>  Preparing: update count set money = money + ? where id = ?
==> Parameters: 10(Integer), 2(Integer)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@65f8f5ae]
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@6f3187b0] was not registered for synchronization because synchronization is not active
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@6f8e8894] will not be managed by Spring
==>  Preparing: update count set money = money + ? where id = ?
==> Parameters: -10(Integer), 1(Integer)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@6f3187b0]
```
![image-20230120181517927](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201815097.png)
