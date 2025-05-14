---
title: Freemarker
description: Freemarker
date: 2024-07-26
slug: Freemarker
image: 202412212120187.png
categories:
    - Freemarker
---
# Freemarker

## 基础

### 入门

> 导入依赖

```xml
        <!-- freemarker依赖 -->
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-freemarker</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
```

> 配置`application.yml`

```yaml
spring:
    freemarker:
        # 关闭缓存
        cache: false
        charset: utf-8
        suffix: .ftl
        settings:
          # 检查模板更新延迟时间，设置为0表示立即检查
          template_update_delay: 0
server:
    # 服务端口
    port: 8881
```

> 在`resources`下新建`templates/01.basic.ftl`，内容如下
>
> 使用`${}`插值语法展示对应的数据

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>freemark-01</title>
</head>
<body>
hello,${student.name}！你的年龄是${student.age}
</body>
</html>
```

> 在对应的`controller`中设置数据

```java
@Controller
public class FreemarkController {
    @GetMapping("/basic")
    public String basic(Model model) {
        Student student = new Student("张三",23);
        model.addAttribute("student",student);
        return "01.basic";
    }
}
```

### 扩展

找到`spring-boot-autoconfigure`包，打开`spring.factories`文件，找到如下配置

```
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration
```

![image-20240725214219055](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412212058806.png)

> 进入`FreeMarkerProperties`即可查看到默认配置

![image-20240725214317571](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412212058616.png)

### 基础语法

#### 注释

```html
<#-- 这是一段注释 -->
```

#### 插值

```
${student.name}
```

#### ftl指令

ftl指令与html标记类似，名字前加`#`进行区分，freemarker会解析标签中的`表达式`或者`逻辑`

```
<# >Ftl指令</#>
```

#### 文本

剩下的不是freemarker的注释、插值、ftl指令的内容就会被freemarker忽略解析，直接输出内容

### FTL指令

#### 集合指令

##### List

> 我们在案例中实现一个表格展示的效果，类似如下

```html
<table>
    <tr>
        <th>姓名</th>
        <th>年龄</th>
        <th>地址</th>
    </tr>
    <tr>
        <td>张三</td>
        <td>23</td>
        <td>江苏</td>
    </tr>
    <tr>
        <td>李四</td>
        <td>24</td>
        <td>湖南</td>
    </tr>
</table>
```

> 前端代码如下

```html
<table>
    <tr>
        <th>序号</th>
        <th>姓名</th>
        <th>年龄</th>
    </tr>
    <#-- studentList就是在java中设置的attribute -->
    <#-- stu就是遍历之后的每一个对象 -->
    <#list studentList as stu>
        <tr>
            <#-- 使用'_index'来获取当前对象序号(从0开始) -->
            <td>${stu_index+1}</td>
            <td>${stu.name}</td>
            <td>${stu.age}</td>
        </tr>
    </#list>
</table>
```

```java
@Controller
public class FreemarkController {
    @GetMapping("/list")
    public String list(Model model) {
        List<Student> studentList = getStudentList();
        model.addAttribute("studentList",studentList);
        return "02.list";
    }

    protected List<Student> getStudentList(){
        List<Student> list = new ArrayList<>();
        Student student1 = new Student("张三",23);
        Student student2 = new Student("李四",24);
        Student student3 = new Student("王五",25);
        list.add(student1);
        list.add(student2);
        list.add(student3);
        return list;
    }
}
```

##### Map

> 方式1：通过`map['keyname'].property`来获取对象的属性
>
> 方式2：通过`map.keyname.property来获取对象的属性`
>
> 遍历`Map`使用`studentMap?keys as key`来获取对象的`key`，但是需要注意，此时只能使用`studentMap[key].name`来获取属性
>
> 如果使用`studentMap.key`的方式，它会认为studentMap中存在`key`这个键，但是实际上我们只有`stu1`、`stu2`、`stu3`，所以会报错

```html
<h2>方式1：通过map['keyname'].property</h2>
姓名:${studentMap['stu1'].name}<br>
年龄:${studentMap['stu1'].age}<br>
<h2>方式2：通过map.keyname.property</h2>
姓名:${studentMap.stu2.name}<br>
年龄:${studentMap.stu2.age}<br>
<h2>遍历map</h2>
<#-- '?keys'代表获取到集合中的每一个key -->
<#list studentMap?keys as key>
    序号:${key_index+1}
    <#-- 注意这里不能使用`studentMap.key`的方式，否则它会认为studentMap中存在key这个对象 -->
    姓名:${studentMap[key].name}
    年龄:${studentMap[key].age}<br>
</#list>
```

```java
@Controller
public class FreemarkController {
    @GetMapping("/map")
    public String map(Model model) {
        Map<String,Student> studentMap = getStudentMap();
        model.addAttribute("studentMap",studentMap);
        return "03.map";
    }

    protected Map<String,Student> getStudentMap(){
        Map<String,Student> map = new HashMap<String,Student>();
        Student student1 = new Student("张三",23);
        Student student2 = new Student("李四",24);
        Student student3 = new Student("王五",25);
        map.put("stu1",student1);
        map.put("stu2",student2);
        map.put("stu3",student3);
        return map;
    }
}
```

#### if指令

> 格式如下

```
<#if expression>
<#else>
</#if>
```

> 案例，我们需要将`李四`的数据字体显示为`红色`

```html
<table>
    <tr>
        <th>序号</th>
        <th>姓名</th>
        <th>年龄</th>
    </tr>
    <#list studentList as stu>
        <#-- 判断条件 -->
        <#if stu.name='李四'>
            <tr style="color: red">
                <td>${stu_index+1}</td>
                <td>${stu.name}</td>
                <td>${stu.age}</td>
            </tr>
        <#else >
            <tr>
                <td>${stu_index+1}</td>
                <td>${stu.name}</td>
                <td>${stu.age}</td>
            </tr>
        </#if>
    </#list>
</table>
```

> 注意，在freemarker中，`=`与`==`一样

### 运算符

#### 数学运算符

##### 加

> +

##### 减

> -

##### 乘

> *

##### 除

> /

##### 取模(求余)

> %

#### 比较运算符

1. =或者==
2. !=
3. `>`或者gt
4. `>=`或者gte
5. `<`或者lt
6. `<=`或者lte

1. =和!=可以用于字符串、数值和日期来比较是否相等
2. =和!=两边必须是相同类型的值,否则会产生错误
3. 字符串"x"、 "x"、"X"比较是不等的.因为FreeMarker是精确比较
4. gt代替>, FreeMarker会把>解释成FTL标签的结束字符,可使用括号避免这种情况,如:<#if (x>y)>

#### 逻辑运算符

与 &&

或 ||

非 !

> 比如

```html
<#if (10>20) && (10>0)>
    true
<#else>
    false
</#if>
```

### 空值处理

判断某变量是否存在使用`??`,如果该变量存在，则返回true，否则false

```js
    <#if studentList??>
        <#list studentList as stu>
        </#list>
    </#if>
```

设置缺失变量的`默认值`使用`!`

```js
${name!''}

// 如果是嵌套对象，建议使用()包裹起来
${(student.name)!''}
```

### 内建函数

内建函数语法格式:`变量 + ? + 函数名称`

> 之前使用的`?keys`就是内建函数

```html
<#-- '?keys'代表获取到集合中的每一个key -->
<#list studentMap?keys as key>
    序号:${key_index+1}
    <#-- 注意这里不能使用`studentMap.key`的方式，否则它会认为studentMap中存在key这个对象 -->
    姓名:${studentMap[key].name}
    年龄:${studentMap[key].age}<br>
</#list>
```

#### 集合大小Size

```js
集合大小：${studentList?size}
```

#### 日期格式化

```js
显示年月日:${student.birthday?date}
显示时分秒:${student.birthday?time}
显示日期、时间:${student.birthday?datetime}
自定义格式化:${student.birthday?string("yyyy-MM-dd HH:mm:ss")}
```

#### 内建函数c

```java
model.addAttribute("point",123456789);
```

point是数字型，使用`${point}`会显示这个数字的值,每三位使用逗号分隔。比如`point:123,456,789`

如果不想显示为每三位分隔的数字，可以使用c函数将数字型转成字符串输出

```javascript
${point?c}
```

#### JSON字符串转对象 eval

assign的作用是定义一个变量,`?eval`就是将当前字符串转成对象的内建函数

```js
<#assign say="{'username':'admin','password':'123456'}"/>
<#assign user=say?eval/>

<div>${user.username}</div>
<div>${user.password}</div>
```

## 输出静态文件

使用`ftl`模板以及`数据模型`，在freemarker整合就即可输出静态文件

```java
@SpringBootTest
class FreemarkTestApplicationTests {
    @Autowired
    private Configuration configuration;
    @Test
    void contextLoads() throws Exception {
        Template template = configuration.getTemplate("01.basic.ftl");
        // 第一个参数是数据模型
        // 第二个参数是输出流
        template.process(getStudent(),new FileWriter("C:\\Users\\93879\\Desktop\\map.html"));
    }

    protected Map<String,Student> getStudent(){
        Map<String, Student> map = new HashMap<>();
        Student student = new Student("张三", 23, new Date(), 4000.0f);
        map.put("student",student);
        return map;
    }
}
```

> 为什么`Configuration`可以直接被注入呢？

找到`FreeMarkerAutoConfiguration`配置类

![image-20240726090256996](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412212058540.png)

源码中相关部分如下

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({freemarker.template.Configuration.class, FreeMarkerConfigurationFactory.class})
@EnableConfigurationProperties({FreeMarkerProperties.class})
@Import({FreeMarkerServletWebConfiguration.class, FreeMarkerReactiveWebConfiguration.class, FreeMarkerNonWebConfiguration.class})
public class FreeMarkerAutoConfiguration {
    // 省略
}
```

重点在于`@Import`导入了`FreeMarkerReactiveWebConfiguration`配置，继续点进去

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnWebApplication(
    type = Type.REACTIVE
)
@AutoConfigureAfter({WebFluxAutoConfiguration.class})
class FreeMarkerReactiveWebConfiguration extends AbstractFreeMarkerConfiguration {

    @Bean
    freemarker.template.Configuration freeMarkerConfiguration(FreeMarkerConfig configurer) {
        return configurer.getConfiguration();
    }
	
    // 其余代码省略
}
```

这里声明了一个`Configuration`的Bean，所以后面可以直接注入
