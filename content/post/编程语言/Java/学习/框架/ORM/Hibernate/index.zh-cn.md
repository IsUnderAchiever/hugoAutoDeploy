---
title: Hibernate
description: Hibernate
date: 2023-04-02
slug: Hibernate
image: 202412212123233.png
categories:
    - Hibernate
---

Hibernate学习01
==========================================================================
> [参考博客](https://blog.csdn.net/Grace_QC/article/details/86530594)
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!-- 在实体类所在的包下，创建一个xml文件。名称为【实体类+.hbm.xml】 -->
<!-- 导入约束:dtd约束 -->
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
```
![导入dtd约束](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152332308.png)
> 复制`hibernate-mapping-3.0.dtd`内约束指实体类xml
```sql
create table tb_user(
  id number primary key ,
  username varchar2(20),
  password varchar2(20),
  nick_name varchar2(20)
);
comment on table tb_user is '用户表';
comment on column tb_user.id is '主键';
comment on column tb_user.username is '用户名';
comment on column tb_user.password is '密码';
comment on column tb_user.nick_name is '昵称';
commit;
```
```java
public class User {
    private int uids;
    private String username;
    private String password;
    private String nickName;
    public User() {
    }
    public User(int uids, String username, String password, String nickName) {
        this.uids = uids;
        this.username = username;
        this.password = password;
        this.nickName = nickName;
    }
    public int getUids() {
        return uids;
    }
    public void setUids(int uids) {
        this.uids = uids;
    }
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
    public String getNickName() {
        return nickName;
    }
    public void setNickName(String nickName) {
        this.nickName = nickName;
    }
}
```
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!-- 在实体类所在的包下，创建一个xml文件。名称为【实体类+.hbm.xml】 -->
<!-- 导入约束:dtd约束 -->
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.example.domain">
    <class name="User" table="TB_USER">
        <id name="uids" column="id">
            <!-- 主键的生成方式 native是使用本地数据库的自动增长能力-->
            <generator class="native"/>
        </id>
        <property name="username" column="username"/>
        <property name="password" column="password"/>
        <property name="nickName" column="nick_name"/>
    </class>
</hibernate-mapping>
```
> 在根目录下创建`hibernate.cfg.xml`，一般是src，但是我这里设置的是java
![image-20230402221742657](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152332620.png)
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!-- 在项目的根路径下创建名为hibernate.cfg.xml的配置文件 -->
<!-- 导入约束:dtd约束 -->
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <!-- 配置sessionFactory -->
    <!-- sessionFactory的作用是用来创建session对象，session对象就是hibernate中操作数据库的核心对象-->
    <!-- 创建sessionFactory的三步 -->
    <!--
        1.连接数据库的信息
        2.hibernate的可选配置
        3.映射文件位置
     -->
    <session-factory>
        <!-- 1.连接数据库的信息 -->
        <property name="hibernate.connection.driver_class">
            oracle.jdbc.driver.OracleDriver
        </property>
        <property name="hibernate.connection.url">
            jdbc:oracle:thin:@localhost:1521:orcl
        </property>
        <property name="hibernate.connection.username">
            test
        </property>
        <property name="hibernate.connection.password">
            123456
        </property>
        <!-- 数据库的方言 -->
        <property name="hibernate.dialect">org.hibernate.dialect.Oracle8iDialect</property>
        <!-- 2.hibernate的可选配置 -->
        <!-- 显示hibernate生成的sql语句 -->
        <property name="hibernate.show_sql">true</property>
        <!-- 是否使用格式化输出sql语句 -->
        <property name="hibernate.format_sql">true</property>
        <!-- 配置hibernate采用何种方式生成DDL语句 -->
        <!-- update表示检测实体类的映射配置与数据库结构是否一致，如果不一致，更新表结构;如果没有该表，则创建该表 -->
        <!-- SQL结构化查询语言: 一共分为6个部分
             DDL: Data Definition Language
             DML: Data Manipulatiok Language
             DQL: Data Query Language
             DCL: Data Control Language 数据控制语言
             CCL: Cursor Control Language 游标控制语言
             TPL: Transaction Processing Language 事务处理语言 -->
        <property name="hibernate.hbm2ddl.auto">update</property>
        <!-- 3.映射配置文件的路径 -->
        <!-- 多个配置文件就在这里写多次 -->
        <mapping resource="com/example/domain/User.hbm.xml"/>
<!--        <mapping resource="com/example/domain/User.hbm.xml"/>-->
    </session-factory>
</hibernate-configuration>
```
