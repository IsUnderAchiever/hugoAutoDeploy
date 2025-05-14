---
title: Beetl3
description: Beetl3
date: 2024-11-05
slug: Beetl3
image: 202412232109437.png
categories:
    - Beetl3
---
# beetl3

> [参考文档](https://static.kancloud.cn/xiandafu/beetl3_guide/1992542)

## 基础入门

### 安装

```xml
<dependency>
  <groupId>com.ibeetl</groupId>
  <artifactId>beetl</artifactId>
  <version>3.16.2.RELEASE</version>
</dependency>
```

### 快速开始

Beetl的核心是`GroupTemplate`，是一个重量级对象，实际使用的时候`建议`使用单模式创建
创建GroupTemplate需要2个参数，一个是`模板资源加载器`，一个是`配置类`
模板资源加载器Beetl内置了6种，分别是：

- **StringTemplateResourceLoader**：字符串模板加载器，用于加载字符串模板，如本例所示
- **FileResourceLoader**：文件模板加载器，需要一个根目录作为参数构造，传入getTemplate方法的String是模板文件相对于Root目录的相对路径
- **ClasspathResourceLoader**：现代web应用最常用的文件模板加载器，模板文件位于Classpath里
- **WebAppResourceLoader**：用于webapp集成，假定模板根目录就是WebRoot目录，参考web集成章
- **MapResourceLoader**：可以动态存入模板
- **CompositeResourceLoader**：混合使用多种加载方式

```java
import org.beetl.core.Configuration;
import org.beetl.core.GroupTemplate;
import org.beetl.core.Template;
import org.beetl.core.resource.StringTemplateResourceLoader;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class BeetlDemoApplicationTests {

    @Test
    void contextLoads() throws Exception {
        //初始化代码
        StringTemplateResourceLoader resourceLoader = new StringTemplateResourceLoader();
        Configuration cfg = Configuration.defaultConfiguration();
        GroupTemplate gt = new GroupTemplate(resourceLoader, cfg);
        //获取模板
        Template t = gt.getTemplate("hello,${name}");
        t.binding("name", "beetl");
        //渲染结果
        String str = t.render();
        System.out.println(str);
    }

}
```

> `t.binding("name", "beetl");`将`name`的值绑定为`beetl`
>
> `String str = t.render();`渲染模板，得到输出的结果

除此之外，还提供了多种渲染输出的方法

- **template.render()** 返回渲染结果，如本例所示
- **template.renderTo(Writer)** 渲染结果输出到Writer里，如果你的Writer是一个FilterWriter，则可把输出保存到文件里
- **template.renderTo(OutputStream)** 渲染结果输出到OutputStream里

> 1. Beetl 支持为模板自定义定界符和占位符，如本例子采用的默认占位符号 `${}`。 后面可以看到，可以定义任意其他符号，比如`#{}` 或者 `##`
> 2. 如果不想写代码直接体验 Beetl 提供的基本功能，可以使用 http://ibeetl.com/beetlonline/

### 模板基础配置

Beetl提供不但功能齐全，而且还有很多独特功能，通过简单的配置文件，就可以定义众多的功能，默认情况下，Configuration类总是会`先`加载默认的配置文件（位于`/org/beetl/core/beetl-default.properties`，作为新手，**通常只需要关注3,4,5,6行定界符的配置，以及12行模板字符集的配置就可以了**，其他配置会在后面章节陆续提到,同时，对于Spring等框架，有些配置将会被这些框架的配置覆盖，需要参考后面章节）下，beetl-default.properties其内容片断如下：

```properties
#默认配置
ENGINE=org.beetl.core.engine.FastRutimeEngine
DELIMITER_PLACEHOLDER_START=${
DELIMITER_PLACEHOLDER_END=}
DELIMITER_STATEMENT_START=<%
DELIMITER_STATEMENT_END=%>
DIRECT_BYTE_OUTPUT = FALSE
HTML_TAG_SUPPORT = true
HTML_TAG_FLAG = #
HTML_TAG_BINDING_ATTRIBUTE = var
#3.17版本以后，默认不再支持Java直接调用
NATIVE_CALL = FALSE 
TEMPLATE_CHARSET = UTF-8
ERROR_HANDLER = org.beetl.core.ConsoleErrorHandler
#3.17版本以后，使用白名单管理Java直接调用
NATIVE_SECUARTY_MANAGER= org.beetl.core.WhiteListNativeSecurityManager 
MVC_STRICT = FALSE

#资源配置，resource后的属性只限于特定ResourceLoader
RESOURCE_LOADER=org.beetl.core.resource.ClasspathResourceLoader
#classpath 根路径
RESOURCE.root= /
#是否检测文件变化,开发用true合适，但线上要改为false
RESOURCE.autoCheck= true
#自定义脚本方法文件的Root目录和后缀
RESOURCE.functionRoot = functions
RESOURCE.functionSuffix = html
#自定义标签文件Root目录和后缀
RESOURCE.tagRoot = htmltag
RESOURCE.tagSuffix = tag
#####  扩展 ##############
## 内置的方法
FN.date = org.beetl.ext.fn.DateFunction
......
##内置的功能包
FNP.strutil = org.beetl.ext.fn.StringUtil
......
##内置的默认格式化函数
FTC.java.util.Date = org.beetl.ext.format.DateFormat
.....
## 标签类
TAG.include= org.beetl.ext.tag.IncludeTag
TAG.html.include= org.beetl.ext.tag.html.IncludeResourceHtmlTag
TAG.html.foreach= org.beetl.ext.tag.html.ForeachHtmlTag
```

第3,4行指定了占位符号，默认是`${``}`，也可以指定为其他占位符。

第5,6行指定了语句的定界符号，默认是`<%``%>`，也可以指定为其他定界符号

第12行指定模板字符集是**UTF-8**

模板开发者不需要关心如上配置，可以创建一个beetl.properties的配置文件，此时，该配置文件将覆盖默认的配置文件属性，比如，你的定界符考虑是`<!--:` 和 `-->` ,则在**beetl.properties**加入一行即可,并将此配置文件放入Classpath根目录下即可。

### 模板加载器

```java
class BeetlDemoApplicationTests {
    private static final String RESOURCES_PATH="%s\\src\\main\\resources\\%s";
    @Test
    void contextLoads() throws Exception {
        // D:\workspace\IdeaProjects\beetl-demo\src\main\resources\template
        // 设置根目录
        String root = String.format(RESOURCES_PATH, System.getProperty("user.dir"), "template");
        // 指定字符集为UTF-8，也可不指定，因为配置文件默认就是UTF-8
        FileResourceLoader resourceLoader = new FileResourceLoader(root,"utf-8");
        Configuration cfg = Configuration.defaultConfiguration();
        GroupTemplate gt = new GroupTemplate(resourceLoader, cfg);
        Template t = gt.getTemplate("/s01/hello.txt");
        t.binding("name", "名字");
        t.binding("my_name", "张三");
        String str = t.render();
        System.out.println(str);
    }

}
```

> `src/main/resources/template/s01/hello.txt`

```
你好,请问你叫什么${name}，我叫${my_name}
```

### Classpath资源模板加载器

`最常用的加载器`。在springboot里，模板资源是打包到jar文件或者同Class放在一起，因此，可以使用ClasspathResourceLoader来加载模板实例

```java
class BeetlDemoApplicationTests {
    @Test
    void contextLoads() throws Exception {
        ClasspathResourceLoader resourceLoader = new ClasspathResourceLoader("org/tong/template/");
        Configuration cfg = Configuration.defaultConfiguration();
        GroupTemplate gt = new GroupTemplate(resourceLoader, cfg);
        Template t = gt.getTemplate("/hello.txt");
        t.binding("name", "beetl");
        String str = t.render();
        System.out.println(str);
    }

}
```

> `src/main/resources/org/tong/template/hello.txt`

```
你好，这是${name}模板
```

### WebApp资源模板加载器

WebAppResourceLoader 是用于Java EE web应用的资源模板加载器，默认根路径是WebRoot目录。也可以通过制定root属性来设置相对于WebRoot的的模板根路径，从安全角考虑，建议放到WEB-INF目录下

如下是Jfinal集成 里初始化GroupTemplate的方法

```java
Configuration cfg = Configuration.defaultConfiguration();
WebAppResourceLoader resourceLoader = new WebAppResourceLoader();
groupTemplate = new GroupTemplate(resourceLoader, cfg);
```

WebAppResourceLoader 假定 beetl.jar 是位于 WEB-INF/lib 目录下，因此，可以通过WebAppResourceLoader类的路径来推断出WebRoot路径从而指定模板根路径。所有线上环境一般都是如此，如果是开发环境或者其他环境不符合此假设，你需要调用resourceLoader.setRoot() 来指定模板更路径

## 语法

### 定界符与占位符号

Beetl模板语言类似JS语言和习俗，只需要将Beetl语言放入定界符号里即可，如默认的是`<% %>` ,占位符用于静态文本里嵌入占位符用于输出，如下是正确例子

```js
<%
var num1=3;
var num2=2;
var sum=num1+num2;
%>
sum:${num1+num2} 或者 ${sum}
```

**千万不要**在定界符里使用占位符号，因为占位符仅仅嵌在静态文本里，如下例子是**错误**例子

```js
<%
var num1=3;
var num2=2;
var sum=${num1}+${num2};
%>
```

### 注释

> 单行注释采用`//`，多行注释采用`/*  */`

### 变量

#### 临时变量

> 临时变量使用`var`定义

```js
<%
var name = "abc";
var sql ="""
SELECT *
FROM TH_USER
WHERE NAME = '张三'
AND AGE > 18
""";
%>
SQL:${sql} ${name}
```

#### 全局变量

> 全局变量是通过在java代码里调用`template.binding`传入的变量,这些变量能在模板的任何一个地方，包括子模板都能访问到

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer id;
    private String username;
    private String password;
}
```



```java
class BeetlDemoApplicationTests {
    @Test
    void contextLoads() throws Exception {
        ClasspathResourceLoader resourceLoader = new ClasspathResourceLoader("org/tong/template/");
        Configuration cfg = Configuration.defaultConfiguration();
        GroupTemplate gt = new GroupTemplate(resourceLoader, cfg);
        Template t = gt.getTemplate("/demo.vue");
        // 设置全局变量
        t.binding("userList", initUserList());
        String str = t.render();
        System.out.println(str);
    }

    List<User> initUserList(){
        List<User> userList = new ArrayList<>();
        userList.add(new User(1,"admin","admin"));
        userList.add(new User(2,"zhangsan","zhangsan"));
        userList.add(new User(3,"lisi","lisi"));
        return userList;
    }

}
```



```js
<%
for(user in userList) {
%>
user_name is ${user.username}
<%
}
%>
```

> 自从2.8.0版本后，有一个特殊的变量成为root变量，当模板找不到变量的时候，会寻找root变量的属性来作为变量的值，这个root变量必须绑定为"_root"

```java
class BeetlDemoApplicationTests {
    @Test
    void contextLoads() throws Exception {
        ClasspathResourceLoader resourceLoader = new ClasspathResourceLoader("org/tong/template/");
        Configuration cfg = Configuration.defaultConfiguration();
        GroupTemplate gt = new GroupTemplate(resourceLoader, cfg);
        Template t = gt.getTemplate("/demo.vue");
        //绑定"_root"变量的值
        t.binding("_root", new User(1,"admin","123456"));
        String str = t.render();
        System.out.println(str);
    }
}
```



```js
${username}
${password}
```

