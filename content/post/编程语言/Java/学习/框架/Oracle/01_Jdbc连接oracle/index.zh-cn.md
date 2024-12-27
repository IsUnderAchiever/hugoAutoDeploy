---
title: 01_Jdbc连接oracle
description: 01_Jdbc连接oracle
date: 2023-03-10
slug: 01_Jdbc连接oracle
image: 202412212036798.png
categories:
    - Java
---

## JDBC连接Oracle实现CRUD操作
> 下面是项目的结构
>
> lib下存放的是oracle的连接包，`ojdbc6-11.2.0.4.jar`
```tree
D:.
├─.idea
│  └─inspectionProfiles
├─lib
└─src
    └─com
        └─example
            ├─config
            ├─domain
            └─mapper
                └─impl
```
```sql
-- 建表
create table tb_user
(
    id        number(10) primary key,
    nickname  varchar2(20),
    username  varchar2(20),
    password varchar2(20)
);
-- 插入测试数据
insert into tb_user (id, nickname, username, password) values (1,'哈哈','admin','123456');
insert into tb_user (id, nickname, username, password) values (2,'呵呵','abcd','456');
insert into tb_user (id, nickname, username, password) values (3,'嘿嘿','aan','1234');
commit;
-- 查询表
select * from tb_user;
```
<img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303102244438.png" alt="image-20230310224428393" style="zoom:80%;" />
```java
public class User {
    private Integer id;
    private String nickname;
    private String username;
    private String password;
    public User(Integer id, String nickname, String username, String password) {
        this.id = id;
        this.nickname = nickname;
        this.username = username;
        this.password = password;
    }
    public User() {
    }
    public Integer getId() {
        return id;
    }
    public void setId(Integer id) {
        this.id = id;
    }
    public String getNickname() {
        return nickname;
    }
    public void setNickname(String nickname) {
        this.nickname = nickname;
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
    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", nickname='" + nickname + '\'' +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```
```java
import java.sql.*;
public class OracleConfig {
    public static Connection getConnection() {
        Connection connection = null;
        try {
            Class.forName("oracle.jdbc.driver.OracleDriver");
            connection = DriverManager.getConnection("jdbc:oracle:thin:@localhost:1521/orcl", "root", "123456");
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        return connection;
    }
    public static void close(Connection connection, Statement statement, PreparedStatement preparedStatement, ResultSet resultSet){
        try {
            resultSet.close();
            preparedStatement.close();
            connection.close();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }
}
```
```java
public interface UserMapper {
    List<User> selectUsers();
}
```
```java
import src.com.example.config.OracleConfig;
import src.com.example.domain.User;
import src.com.example.mapper.UserMapper;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
public class UserMapperImpl implements UserMapper {
    @Override
    public List<User> selectUsers() {
        List<User> list = new ArrayList<>();
        // 建立连接
        Connection connection = OracleConfig.getConnection();
        PreparedStatement preparedStatement =null;
        ResultSet resultSet =null;
        try {
            preparedStatement = connection.prepareStatement("select * from tb_user");
            resultSet = preparedStatement.executeQuery();
            while (resultSet.next()){
                // 创建对象
                User user=new User(
                        resultSet.getInt("id"),
                        resultSet.getString("nickname"),
                        resultSet.getString("username"),
                        resultSet.getString("password"));
                list.add(user);
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        // 关闭资源
        OracleConfig.close(connection,null,preparedStatement,resultSet);
        return list;
    }
    static class MyTest{
        public static void main(String[] args) {
            UserMapperImpl userMapper = new UserMapperImpl();
            List<User> list = userMapper.selectUsers();
            list.forEach(System.out::println);
        }
    }
}
```
> 下图是演示结果
![image-20230310224210463](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303102242670.png)
### 关于在lib内导入jar包的操作
<img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303102248438.png" alt="image-20230310224841395" style="zoom:80%;" />
<img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303102246819.png" alt="image-20230310224654748" style="zoom:80%;" />
<img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303102247022.png" alt="image-20230310224732971" style="zoom: 80%;" />
> 如果以上操作`依然无法连接到oracle`，请查看第二张图片内选择的模块是否正确
### 可能出现的报错信息
1. 表或视图不存在
   1. sql和数据库对不上
   2. 在数据库插入数据后执行以下commit操作`(非常重要！！！)`
2. sql语句错误
   1. sql语句写错了
   2. sql语句内的末尾的分号去掉`(去掉;)`
