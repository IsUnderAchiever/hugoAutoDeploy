---
title: Thymeleaf
description: Thymeleaf
date: 2023-04-15
slug: Thymeleaf
image: 202412211939238.png
categories:
    - Java
---
## thymeleaf学习
### 依赖
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
### thymeleaf默认模板
```html
<!DOCTYPE html>
<html lang="ch" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>默认的标题</title>
    <meta name="description" content="默认的描述">
    <meta name="keywords" content="默认的关键字">
<body>
</body>
</html>
```
### thymeleaf配置
```yaml
server:
  # 应用服务 WEB 访问端口
  port: 10010
spring:
  # 应用名称
  application:
    name: thymeleaf-demo
  # 配置thymeleaf
  thymeleaf:
    cache: false
    mode: HTML5
    encoding: UTF-8
    prefix: classpath:/templates/
    #名称的后缀
    suffix: .html
    servlet:
      content-type: text/html
  datasource:
    # 数据库驱动：
    driver-class-name: com.mysql.cj.jdbc.Driver
    # 数据源名称
    name: defaultDataSource
    # 数据库连接地址
    url: jdbc:mysql://localhost:3306/test?characterEncoding=UTF-8
    # 数据库用户名&密码：
    username: root
    password: 123456
mybatis-plus:
  # 指定Mybatis的Mapper文件
  mapper-locations: classpath:mapper/*xml
  # 指定Mybatis的实体目录
  type-aliases-package: com.example.plus.domain
  # 日志打印
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  # 逻辑删除列配置
  global-config:
    db-config:
      # 全局逻辑删除的实体字段名(since 3.3.0,配置后可以忽略不配置步骤2)
      logic-delete-field: delFlag
      # 逻辑已删除值(默认为 1)
      logic-delete-value: 1
      # 逻辑未删除值(默认为 0)
      logic-not-delete-value: 0
logging:
  pattern:
    dateformat: MM-ddHH:mm:ss:SSS
```
![数据库](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304151432559.png)
### 注意
> 本项目采用`thymeleaf+mybatis plus+mysql+springboot`，项目结构如下
> 其中`templates`用来存放html，`templates`下的`component`内用于存放页面中可复用的组件
> `static`下存放静态文件，如js、css、img...
thymeleaf:.
    └─src
        └─main
            ├─java
            │ └─com
            │   └─example
            │       ├─controller
            │       ├─domain
            │       ├─mapper
            │       └─service
            │           └─impl
            └─resources
                ├─mapper
                ├─static
                │ ├─css
                │ ├─js
                │ └─json
                └─templates
                    └─component
### 示例1
> 语法:`${}`获取数据，`||`拼接(当然可也用单引号+双引号拼接)
>
> 访问`localhost:10010/test`可观察到页面标题的变化
```html
<!DOCTYPE html>
<html lang="ch" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title th:text="|拼接+${title}|">默认的标题</title>
    <meta name="description" content="默认的描述">
    <meta name="keywords" content="默认的关键字">
</head>
<body>
Hello World!
</body>
</html>
```
```java
@GetMapping("/test")
public ModelAndView indexTest(){
        ModelAndView modelAndView=new ModelAndView();
        modelAndView.addObject("title","传递的标题");
        modelAndView.setViewName("index");
        return modelAndView;
        }
```
### 语法
```java
@GetMapping("/basic-train")
public ModelAndView basicTest(){
        ModelAndView modelAndView=new ModelAndView();
        modelAndView.addObject("userList",userService.list());
        modelAndView.addObject("user",userService.getById(2));
        modelAndView.addObject("title","basic页面");
        modelAndView.setViewName("basic");
        return modelAndView;
        }
```
> 1. 链接变表达式：可引入外部文件，如js、css等
   ```html
   <link rel="stylesheet" th:href="@{css/test.css}">
   <script th:src="@{js/jquery-3.0.0.min.js}"></script>
   <a th:href="@{index.html}">超链接</a>
   ```
> 2. th:text
>
> `这里有个细节请注意`:mysql表中的字段为nick_name，这里的属性和java实体类对应，所以是nickName
   ```html
   <!-- th:text -->
   <span th:text="${user.id}"></span><br>
   <span th:text="${user.username}"></span><br>
   <span th:text="${user.password}"></span><br>
   <span th:text="${user.nickName}"></span><br>
   <!-- 换一种方式 -->
   <div th:object="${user}">
       <span th:text="*{id}"></span><br>
       <span th:text="*{username}"></span><br>
       <span th:text="*{password}"></span><br>
       <span th:text="*{nickName}"></span><br>
   </div>
   ```
> 3. th:if
```html
<!-- th:if -->
<div th:if="${user.id}==1">id为1</div>
```
> 4. th:each `循环`
```html
<!-- th:each -->
<table>
    <tr>
        <th>编号</th>
        <th>账号</th>
        <th>密码</th>
        <th>昵称</th>
    </tr>
    <tr th:each="item,index : ${userList}">
        <td th:text="${item.id}"></td>
        <td th:text="${item.username}"></td>
        <td th:text="${item.password}"></td>
        <td th:text="${item.nickName}"></td>
    </tr>
</table>
```
> 5. th:classappend `添加class`
```html
<table>
    <tr>
        <th>编号</th>
        <th>账号</th>
        <th>密码</th>
        <th>昵称</th>
    </tr>
    
    <tr th:each="item,index : ${userList}">
        <!-- 追加class -->
        <!-- 如果是最后一个，则添加test为class -->
        <td th:text="${item.id}" th:classappend="${index.last}?test"></td>
        <td th:text="${item.username}"></td>
        <td th:text="${item.password}"></td>
        <td th:text="${item.nickName}"></td>
    </tr>
</table>
```
> 6. th:switch
```html
<!-- th:switch -->
<div th:switch="${user.username}">
    <p th:case="admin">用户是admin</p>
    <p th:case="aaaaa">用户是aaaaa</p>
    <p th:case="asdas">用户是asdas</p>
    <!-- 若都不是其他case，则显示默认值 -->
    <p th:case="*">默认值</p>
</div>
```
> 7. 如何动态获取th:text里的内容？--`th:inline`
```html
<!-- 需要动态渲染的javascript -->
<!-- 同理，css也可以这样渲染 -->
<script th:inline="javascript">
    /* 若user没有传值，则使用后面的默认值;若user传了值，使用注释里的user值 */
    let user =/*[[${user}]]*/{}
    console.log("thymeleaf-user:",user);
</script>
```
> 8. 组件(碎片) th:replace
**component/com.html**
```html
<!DOCTYPE html>
<html lang="ch" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <!-- 本页面的style样式在组件被其它页面引入后是不会显示的 -->
    <style>
        #com1 {
            width: 300px;
            height: 20px;
            background-color: greenyellow;
        }
        #com2 {
            width: 300px;
            height: 20px;
            background-color: yellow;
        }
    </style>
</head>
<body>
<div th:fragment="com1" id="com1">
    com1
</div>
<div th:fragment="com2" id="com2">
    com2
</div>
</body>
</html>
```
**basic.html**
```html
<!-- th:replace -->
<!-- 替换碎片 -->
<!-- 将div标签也替换成com1内的标签 -->
<div th:replace="~{component/com::com1}"></div>
<!-- 如果不希望div标签被替换，可以使用insert -->
<div th:insert="~{component/com::com2}"></div>
<!-- 也可以通过类似jquery选择器的方式获取 -->
<div th:insert="~{component/com::#com2}"></div>
```
### ajax格式
```js
$.ajax({
    //请求的url地址
    url: "",
    //返回格式为json
    dataType: "json",
    //请求是否异步，默认为异步，这也是ajax重要特性
    async: true,
    //参数值
    data: {},
    //请求方式
    type: "GET",
    beforeSend: function () {
        //请求前的处理
    },
    success: function (data) {
        //请求成功时处理
        console.log("data:", data)
    },
    complete: function () {
        //请求完成的处理
    },
    error: function () {
        //请求出错处理
    }
});
```
### 项目地址
[点击](https://www.123pan.com/s/tMU0Vv-VtiUd.html)
