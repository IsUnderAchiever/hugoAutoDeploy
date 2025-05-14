---
title: MyBatis
description: MyBatis
date: 2023-03-19
slug: MyBatis
image: mybatis-logo.png
categories:
    - MyBatis
---

MyBatis
=======
> 使用以下sql新建表`(我使用的是oracle数据库，而非mysql)`
>
> 使用之前提过的自动生成方式生成对应的实体类和mapper
```sql
-- 创建用户表
create table tb_people
(
    id     varchar2(50) primary key,
    name   varchar2(30),
    gender number(1),
    age    number(3) check ( age > 0 )
);
-- 创建手机表
create table tb_phone
(
    id    varchar2(50) primary key,
    name  varchar2(20),
    brand varchar2(20),
    price number(10, 2)
);
-- 创建 用户-手机详情表
create table tb_people_phone_detail
(
    people_id varchar2(50),
    phone_id  varchar2(50),
    primary key (people_id, phone_id)
);
commit;
select *
from tb_people;
select *
from tb_phone;
select *
from tb_people_phone_detail;
-- 添加表注释
comment on table tb_people is '用户表';
comment on table tb_phone is '手机表';
comment on table tb_people_phone_detail is '用户-手机详情表';
-- 添加字段注释
comment on column tb_people.id is '用户主键';
comment on column tb_people.name is '用户姓名';
comment on column tb_people.gender is '用户性别';
comment on column tb_people.age is '用户年龄';
comment on column tb_phone.id is '手机主键';
comment on column tb_phone.name is '手机名称';
comment on column tb_phone.brand is '手机品牌';
comment on column tb_phone.price is '手机价格';
comment on column tb_people_phone_detail.people_id is '用户主键';
comment on column tb_people_phone_detail.phone_id is '手机主键';
```
> 添加依赖
```xml
<!--mybatis-->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.10</version>
</dependency>
<!--oracle-->
<dependency>
    <groupId>com.oracle.database.jdbc</groupId>
    <artifactId>ojdbc6</artifactId>
    <version>11.2.0.4</version>
</dependency>
<!--log4j-->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
<!--junit-->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
<!-- hutool -->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.8.15</version>
</dependency>
```
```properties
# 数据库驱动：
jdbc.driver=oracle.jdbc.driver.OracleDriver
# 数据库连接地址
jdbc.url=jdbc:oracle:thin:@localhost:1521:orcl
# 数据库用户名&密码：
jdbc.username=root
jdbc.password=123456
```
```properties
# 控制台打印
log4j.rootLogger=debug, stdout
log4j.logger.org.mybatis.example.BlogMapper=TRACE
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n
# 输出到文件
log4j.appender.file=org.apache.log4j.ConsoleAppender
log4j.appender.file.File=C:\Users\tong\Desktop\log4j
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n
```
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC
        "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--设置文件所在路径-->
    <properties resource="jdbc.properties"/>
    <settings>
        <!-- 打印sql日志 -->
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
    <!--别名配置-->
    <typeAliases>
        <package name="com.example.domain"/>
    </typeAliases>
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
![项目目录结构](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152332452.png)
```java
public class ApplicationTest {
    private SqlSession session;
    @Before
    public void init() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        session=sqlSessionFactory.openSession();
    }
    @After
    public void destory(){
        // 提交事务，并释放资源
        session.commit();
        session.close();
    }
}
```
#{}和${}的区别
----------------------------------------------------------------------------
如果使用#{}，他是预编译的sq可以防止SQL注入攻击 
如果使用${}，他是直接把参数值拿来进行拼接，这样会有SQL注入的危险
SQL片段抽取
------------------------------------------------------------------------
![image-20230319165154581](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152332762.png)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.PeopleMapper">
    <resultMap id="BaseResultMap" type="com.example.domain.People">
        <id column="ID" jdbcType="VARCHAR" property="id"/>
        <result column="NAME" jdbcType="VARCHAR" property="name"/>
        <result column="GENDER" jdbcType="DECIMAL" property="gender"/>
        <result column="AGE" jdbcType="DECIMAL" property="age"/>
    </resultMap>
    <sql id="Base_Column_List">
        ID
        , NAME, GENDER, AGE
    </sql>
    <!-- 查询全部用户 -->
    <select id="selectPeople" resultType="com.example.domain.People">
        select
        <include refid="Base_Column_List"/>
        from tb_people
    </select>
</mapper>
```
mapper.xml 标签
--------------------------------------------------------------------------
1.  if
2.  trim
3.  where
4.  set
5.  foreach
6.  choose
> if
```xml
<select id="selectPeople" resultType="com.example.domain.People">
    select
    <include refid="Base_Column_List"/>
    from tb_people
    where id = #{id}
    <if test="name!=null">
        and name = #{name}
    </if>
</select>
```
> trim
```xml
<!-- 清除前缀 -->
<select id="selectPeople" resultType="com.example.domain.People">
    select
    <include refid="Base_Column_List"/>
    from tb_people
    <trim prefixOverrides="and|or">
        and
    </trim>
</select>
```
```xml
<!-- 清除后缀 -->
<select id="selectPeople" resultType="com.example.domain.People">
    select
    <include refid="Base_Column_List"/>
    from tb_people
    <trim suffixOverrides="like">
        where 1=1 like
    </trim>
</select>
```
```xml
<!-- 添加前缀 -->
<select id="selectPeople" resultType="com.example.domain.People">
    select
    <include refid="Base_Column_List"/>
    from tb_people
    <trim prefix="where">
        1=1
    </trim>
</select>
```
```xml
<!-- 添加后缀 -->
<select id="selectPeople" resultType="com.example.domain.People">
    select
    <include refid="Base_Column_List"/>
    from tb_people
    <trim prefix="1=1">
        where
    </trim>
</select>
```
```xml
<!-- 综合使用 -->
<select id="selectPeople" resultType="com.example.domain.People">
    select
    <include refid="Base_Column_List"/>
    from tb_people
    <trim prefix="where" prefixOverrides="and|or">
        <if test="id!=null">
            id = #{id}
        </if>
        <if test="name!=null">
            and name = #{name}
        </if>
    </trim>
</select>
```
> where
```xml
<!--where 标签等价于 trim prefix="where" prefixOverrides="and|or"-->
<select id="selectPeople" resultType="com.example.domain.People">
select
    <include refid="Base_Column_List"/>
    from tb_people
    <where>
        <if test="id!=null">
            and id = #{id}
        </if>
        <if test="name!=null">
            and name = #{name}
        </if>
    </where>
</select>
```
> set
```xml
<!-- set 标签等价于 trim prefix="set" suffixOverrides="," -->
<update id="updatePeople">
    update people
    <set>
        <if test="name!=null">
            name = #{name},
        </if>
        <if test="gender!=null">
            gender = #{gender},
        </if>
    </set>
    where id = #{id}
</update>
```
> foreach
```sql
select * from people where id in (?,?,?,?);
```
```xml
<!--ids 为接口内接收的参数名称-->
<select id="selectPeople" resultType="com.example.domain.People">
    select
    <include refid="Base_Column_List"/>
    from tb_people
    <where>
        <foreach collection="ids" open="id in (" close=")" item="id" separator=",">
            #{id}
        </foreach>
    </where>
</select>
```
> choose
>
> 类似于java的switch、case、default
>
> 1.  如果id不为空，则通过id查询
> 2.  如果id为空，名称不为空，则通过名称查询
> 3.  如果id和名称都为空，则查询id为3的用户
```xml
<select id="selectPeople" resultType="com.example.domain.People">
    select
    <include refid="Base_Column_List"/>
    from tb_people
    <where>
        <choose>
            <when test="id!=null">
                id = #{id}
            </when>
            <when test="name!=null">
                name = #{name}
            </when>
            <otherwise>
                id = 3
            </otherwise>
        </choose>
    </where>
</select>
```
字段映射问题
-------------------------------------------------------------------------------------
```java
// 先插入几条数据
@Test
public void addPeople(){
    PeopleMapper peopleMapper=session.getMapper(PeopleMapper.class);
    peopleMapper.insert(new People(IdUtil.simpleUUID(),"李华",1,15));
    peopleMapper.insert(new People(IdUtil.simpleUUID(),"张三",1,25));
    peopleMapper.insert(new People(IdUtil.simpleUUID(),"王大锤",1,26));
    peopleMapper.insert(new People(IdUtil.simpleUUID(),"翠花",0,18));
}
@Test
public void addPhone(){
    PhoneMapper phoneMapper = session.getMapper(PhoneMapper.class);
    phoneMapper.insert(new Phone(IdUtil.simpleUUID(),"k30","红米",BigDecimal.valueOf(3000)));
    phoneMapper.insert(new Phone(IdUtil.simpleUUID(),"note 11t","红米",BigDecimal.valueOf(4000)));
    phoneMapper.insert(new Phone(IdUtil.simpleUUID(),"mate 50","华为",BigDecimal.valueOf(5000)));
}
@Test
public void addPeoplePhoneDetail(){
    PeoplePhoneDetailMapper peoplePhoneDetailMapper=session.getMapper(PeoplePhoneDetailMapper.class);
    peoplePhoneDetailMapper.insert(new PeoplePhoneDetail("b0e11aed0db6412ebf342dc9676b5ba6","1cff22c9409c44fd8524ea9c4d6e3385"));
    peoplePhoneDetailMapper.insert(new PeoplePhoneDetail("b0e11aed0db6412ebf342dc9676b5ba6","639153d49e0d4d33af43e4d4534b1631"));
    peoplePhoneDetailMapper.insert(new PeoplePhoneDetail("6f226961637346ec82499567122e00ea","2b1edb7c93f04d7d91fc8200378a2c94"));
    peoplePhoneDetailMapper.insert(new PeoplePhoneDetail("672b002417de41d184d7d14f31660e9f","639153d49e0d4d33af43e4d4534b1631"));
    peoplePhoneDetailMapper.insert(new PeoplePhoneDetail("de32b418f26e475aa68ef2f692e58cb6","1cff22c9409c44fd8524ea9c4d6e3385"));
}
```
> 如 数据库 people\_id 字段无法映射到 java 里peopleId，即`驼峰命名法`
> 解决方法1：在mybatis里开启驼峰命名的自动映射
```xml
<settings
    <!-- 打印sql日志 -->
    <setting name="logImpl" value="STDOUT_LOGGING"/>
    <!-- 配置驼峰映射 -->
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```
> 解决方法2：在sql里配置别名
```sql
select people_id peopleId,phone_id phoneId from tb_people_phone_detail
```
> 解决方法3：自定义映射规则`(继承映射规则)`
> 问题
![image-20230319150511655](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152333538.png)
![image-20230319150532911](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152333012.png)
> 正确展示
![image-20230319150634576](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152333925.png)
多表查询
---------------------------------------------------------------
### 一对一
![一对一查询](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152333203.png)
> 可以看到这是一个一对多的关系，但是就`以张三作为演示`，张三的关系是一对一的
>
> 首先需要明确通过什么来查什么？
>
> 我们现在需要通过使用者来查询他的手机，所以需要在使用者的属性里新建一个`手机`
>
> 别忘了生成对应属性的get、set等方法哦
![image-20230319150918664](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152333832.png)
#### 关联查询
> 先测试一下sql是否正确
```sql
select tb_people.id "id",
       tb_people.name "name",
       tb_people.gender "gender",
       tb_people.age "age",
       tb_phone.id    "phone_id",
       tb_phone.name  "phone_name",
       tb_phone.brand "phone_brand",
       tb_phone.price "phone_price"
from tb_people,
     tb_people_phone_detail,
     tb_phone
where tb_people.id = tb_people_phone_detail.people_id(+)
  and tb_people_phone_detail.phone_id = tb_phone.id(+)
  and tb_people.id = '6f226961637346ec82499567122e00ea'
```
**`写法1`**
![image-20230319153350899](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152333837.png)
**`写法2`**
![image-20230319153645683](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152333222.png)
> 以上两种写法都能查询到结果
![image-20230319153711710](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152334376.png)
### 一对多
> 接下来测试一对多，将`Phone phone` 改成`List<Phone> phones`
![image-20230319153856569](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152333231.png)
![image-20230319154549831](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152333771.png)
![image-20230319154452735](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152333000.png)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.PeopleMapper">
    <resultMap id="BaseResultMap" type="com.example.domain.People">
        <id column="ID" jdbcType="VARCHAR" property="id"/>
        <result column="NAME" jdbcType="VARCHAR" property="name"/>
        <result column="GENDER" jdbcType="DECIMAL" property="gender"/>
        <result column="AGE" jdbcType="DECIMAL" property="age"/>
    </resultMap>
    <sql id="Base_Column_List">
        ID
        , NAME, GENDER, AGE
    </sql>
    <resultMap id="PeoplePhonesResultMap" type="com.example.domain.People" extends="BaseResultMap">
        <collection property="phones" ofType="com.example.domain.Phone">
            <id column="phone_id" property="id"/>
            <result column="phone_name" property="name"/>
            <result column="phone_brand" property="brand"/>
            <result column="phone_price" property="price"/>
        </collection>
    </resultMap>
    <!-- 根据用户id查询 -->
    <select id="selectPeopleWithPhones" resultType="com.example.domain.People">
        select tb_people.id "id",
               tb_people.name "name",
               tb_people.gender "gender",
               tb_people.age "age",
               tb_phone.id    "phone_id",
               tb_phone.name  "phone_name",
               tb_phone.brand "phone_brand",
               tb_phone.price "phone_price"
        from tb_people,
             tb_people_phone_detail,
             tb_phone
        where tb_people.id = tb_people_phone_detail.people_id(+)
          and tb_people_phone_detail.phone_id = tb_phone.id(+)
          and tb_people.id = #{id}
    </select>
    <!-- 查询全部用户 -->
    <select id="selectPeople" resultType="com.example.domain.People">
        select
        <include refid="Base_Column_List"/>
        from tb_people
    </select>
</mapper>
```
### 分步查询
> 分布查询也可以完成以上操作
>
> 我们分为两个select操作
>
> 1.  查找到对应的手机id`(可能是多部)`
> 2.  根据手机id查找到对应的手机
![image-20230319160711641](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152333713.png)
![image-20230319160722486](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152334236.png)
> 在方法上右键，选择copy reference
![image-20230319155018277](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152334627.png)
![image-20230319160822156](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152334106.png)
**将id作为参数传给findRoleByUserId方法**
![image-20230319160850613](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152334214.png)
> 查看打印的日志
>
> 可以看到执行了两个sql
![image-20230319161044940](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152334856.png)
分布查询里的`按需加载`
-----------------------------------------------------------------------------------------------------------------------------------
![image-20230319161232719](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152334047.png)
![image-20230319161346819](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152334497.png)
> 这里我们只打印输出了人的id，并没有涉及到手机的信息，所以只执行了一个sql语句
分页查询
---------------------------------------------------------------
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>mybatis-test</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.10</version>
        </dependency>
        <!--oracle-->
        <dependency>
            <groupId>com.oracle.database.jdbc</groupId>
            <artifactId>ojdbc6</artifactId>
            <version>11.2.0.4</version>
        </dependency>
        <!--log4j-->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
        <!--junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
        <!--hutool-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.8.15</version>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
        </dependency>
        <!--pagehelper-->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper</artifactId>
            <version>4.0.0</version>
        </dependency>
    </dependencies>
</project>
```
> pagehelper的版本不要太高，否则匹配会出现问题
![image-20230319161600953](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152334867.png)
```java
@Test
public void test2(){
PeopleMapper peopleMapper = session.getMapper(PeopleMapper.class);
    // 设置分页查询参数
    PageHelper.startPage(1,2);
    List<People> peopleList = peopleMapper.selectPeople();
    PageInfo<People> pageInfo = new PageInfo<>(peopleList);
    // 总条数
    long total = pageInfo.getTotal();
    // 总页数
    int pages = pageInfo.getPages();
    // 当前页
    int pageNum = pageInfo.getPageNum();
    // 每页显示长度
    int pageSize = pageInfo.getPageSize();
    System.out.println("总条数:"+total);
    System.out.println("总页数:"+pages);
    System.out.println("当前页:"+pageNum);
    System.out.println("每页显示长度:"+pageSize);
}
```
分页查询可能出现的问题
--------------------------------------------------------------------------------------------------------------------------------------------
> 在一对多的多表查询中，pagehelper可能会出现数据显示不全的问题，这种情况使用分布查询即可解决
MyBatis缓存
--------------------------------------------------------------
### 一级缓存
> 一级缓存是默认开启的
几种不会使用一级缓存的情况
1.  调用相同方法但是传入的参数不同
2.  调用相同方法参数也相同，但是使用的是另外一个SqISession
3.  如果查询完后，对同一个表进行了增，删改的操作，都会清空这sqlSession上的缓存
4.  如果手动调用SqlSession的clearCache方法清除缓存了，后面也使用不了缓存
### 二级缓存
注意:只有close或者commit后的数据才会进入二级缓存。
#### 开启二级缓存
1.  全局开启
    
    > 在mybatis配置文件中配置
    
    ```xml
    <settings>
        <setting name="cacheEnabled" value="true"/>
    </settings>
    ```
    
    
    
2.  局部开启
    
    > 在mapper.xml里配置cache标签
    
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.mapper.PeopleMapper">
        <cache/>
    </mapper>
    ```
    
    
需要注意的问题
------------------------------------------------------------------------------------------------
1.  一般不用二级缓存，一级缓存是session级，而二级缓存是mapper级，即便是一个新的sqlSession，也可以获取到之前缓存里的数据，可能导致数据更改后未更新的情况
    
2.  mapper里的注释不要写在sql里
    
    ![image-20230319162702656](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152334703.png)
