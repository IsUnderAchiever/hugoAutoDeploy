---
title: 新建springboot项目
description: 新建springboot项目
date: 2022-11-27
slug: 新建springboot项目
image: 202412212133331.png
categories:
    - Spring
---

### 新建SpringBoot项目
下文包括使用sts、idea新建SpringBoot项目，并编写登录逻辑以及测试
1. 新建SpringBoot项目
  - 使用Spring Tool Suite
    右键新建，选择Spring Starter Project
    ![202301201930984.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201930984.png)
    ![202301201931798.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201931798.png)
    注意：其实不安装lombok插件也没什么，手动给get、set方法 和 有参、无参的构造方法即可
    
    sts 的lombok插件需要自己安装jar包，安装过程上网查找，这里不再多说；idea的插件安装 同
  - 使用IDEA
    ![202301201931023.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201931023.png)
    ![202301201931066.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201931066.png)
    ![202301201931675.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201931675.png)
    
    右键新建，选择Spring Initializr (无法访问spring官网可以选择custom，https://start.aliyun.com)
    
> 更正：
>
> 这里我有mybatis-plus，我也不知道为啥，据说正常情况是没有的，重装系统之后又没有了
>
> 如果没有mybatis-plus选择，可直接在pom.xml导入如下依赖即可
```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.1</version>
</dependency>
```
1. 进入pom.xml，导入mybatis-plus自动生成的依赖  导入依赖后保存，sts保存后自动下载，idea需要点击右上角的 “M” 按钮，Load Maven Changes
  ```xml
  <!--MyBatis-plus自动生成-->
  <dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.1</version>
  </dependency>
  <!--velocity-->
  <dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.0</version>
  </dependency>
  ```
2. 编写自动生成类，可自动生成简易版的（domain、mapper、mapper_xml、service、serviceimpl、controller）
   如果想要生成比较完全的，可以去gitee上搜索“人人开源”，找到renren-generator，这个可以生成更完全，甚至可以生成前端的代码
   ```java
   import com.baomidou.mybatisplus.annotation.DbType;
   import com.baomidou.mybatisplus.annotation.IdType;
   import com.baomidou.mybatisplus.generator.AutoGenerator;
   import com.baomidou.mybatisplus.generator.config.DataSourceConfig;
   import com.baomidou.mybatisplus.generator.config.GlobalConfig;
   import com.baomidou.mybatisplus.generator.config.PackageConfig;
   import com.baomidou.mybatisplus.generator.config.StrategyConfig;
   import com.baomidou.mybatisplus.generator.config.rules.DateType;
   import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
   
   public class MyBatisPlusGenerator {
       public static void main(String[] args) {
           //1. 全局配置
           GlobalConfig config = new GlobalConfig();
           // 是否支持AR模式
           config.setActiveRecord(true)
                   // 生成路径，最好使用绝对路径，window路径是不一样的
                   //TODO  TODO  TODO  TODO
                   .setOutputDir("D:\demo\src\main\java")
                   // 文件覆盖
                   .setFileOverride(true)
                   // 主键策略
                   .setIdType(IdType.AUTO)
                   .setDateType(DateType.ONLY_DATE)
                   // 设置生成的service接口的名字的首字母是否为I，默认Service是以I开头的
                   .setServiceName("%sService")
                   //实体类结尾名称
                   .setEntityName("%s")
                   //生成基本的resultMap
                   .setBaseResultMap(true)
                   //不使用AR模式
                   .setActiveRecord(false)
                   //生成基本的SQL片段
                   .setBaseColumnList(true);
           //2. 数据源配置
           DataSourceConfig dsConfig = new DataSourceConfig();
           // 设置数据库类型
           dsConfig.setDbType(DbType.MYSQL)
                   .setDriverName("com.mysql.cj.jdbc.Driver")
                   //TODO  TODO  TODO  TODO
                   .setUrl("jdbc:mysql://127.0.0.1:3306/demo?useSSL=false")
                   .setUsername("root")
                   .setPassword("123456");
           //3. 策略配置globalConfiguration中
           StrategyConfig stConfig = new StrategyConfig();
           //全局大写命名
           stConfig.setCapitalMode(true)
                   // 数据库表映射到实体的命名策略(驼峰命名)
                   .setNaming(NamingStrategy.underline_to_camel)
                   //使用lombok
                   .setEntityLombokModel(true)
                   //使用restcontroller注解
                   .setRestControllerStyle(true)
                   // 生成的表, 支持多表一起生成，以数组形式填写
                   //TODO  TODO  TODO  TODO 两个方式，直接写，或者使用命令行输入
                   .setInclude("u");
   //                .setInclude("product","product_task","banner");
           //4. 包名策略配置
           PackageConfig pkConfig = new PackageConfig();
           //TODO  TODO  TODO  TODO 包名策略配置
           pkConfig.setParent("com.example.demo")
                   .setMapper("mapper")
                   .setService("service")
                   .setServiceImpl("service.impl")
                   .setController("controller")
                   .setEntity("domain")
                   .setXml("mapper");
           //5. 整合配置
           AutoGenerator ag = new AutoGenerator();
           ag.setGlobalConfig(config)
                   .setDataSource(dsConfig)
                   .setStrategy(stConfig)
                   .setPackageInfo(pkConfig);
           //6. 执行操作
           ag.execute();
           System.out.println("======= 相关代码生成完毕  ========");
       }
   }
   ```
   有几个地方根据自己的需求改（标有 //TODO  TODO 的地方）
   1. 生成文件的路径
   2. 数据库连接的配置
   3. 生成哪几个表
   4. 包名策略配置
   运行main方法，自动生成（**需要注意**，生成之后注销掉方法，不然等你编写完springboot代码点击运行的时候，可能运行的还是自动生成的代码，导致你刚写完的代码被覆盖）
   
   5. sts 运行完后需要右键刷新一下项目
   6. 在src/main/resources下新建mapper文件夹，将生成的mapper.xml移动到mapper文件夹下
   7. 如果不用lombok插件，则需要去掉实体类的注解，自己生成get、set方法 和 有参、无参的构造方法
   **注意@Data注解，不使用lombok插件的都需要自己生成get、set方法，下面使用@Data注解的位置我不再强调了**
   
3. 配置application.properties
   ```properties
   # 应用名称
   spring.application.name=demo
   
   # 应用服务 WEB 访问端口
   server.port=8080
   
   # 数据库驱动：
   spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
   # 数据源名称
   spring.datasource.name=defaultDataSource
   # 数据库连接地址(demo是我们需要连接的数据库)
   spring.datasource.url=jdbc:mysql://localhost:3306/demo?serverTimezone=UTC
   # 数据库用户名&密码：
   spring.datasource.username=root
   spring.datasource.password=123456
   ```
   这里我的数据库连接地址这样就可以了，如果你们有问题，需要配置mysql的时区 或者 写完整 
   ```properties
   # 完整写法(demo是我们需要连接的数据库)
   spring.datasource.url=jdbc:mysql://localhost:3306/demo?useUnicode=true&characterEncoding=utf8&autoReconnect=true&serverTimezone=Asia/Shanghai
   ```
4. 在启动类上加上注解（DemoApplication.java）
   ```properties
   @MapperScan("com.example.demo.mapper")
   ```
5. 接下来编写一个简单的登录业务
   1. mapper继承了BaseMapper，有基本的增删改查方法，所以不用管
   2. 在service里编写
      ```java
      // 登录
      public Object login(U u);
      ```
   3. serviceimpl里实现该方法                 这里有个问题要说一下。
      ```java
      // 如果你不准备改service层 以及 serviceimpl 的继承关系
      public interface UService extends IService<U>{
      }
      
      @Service
      public class UServiceImpl extends ServiceImpl<UMapper, U> implements UService {
      }
      // extends IService<U> 和 extends ServiceImpl<UMapper, U> 这两处
      
      // 那么其实你在 UServiceImpl里不用再自动注入umapper了，如下
      @Autowired
      private UMapper umapper;
      
      // 在UServiceImpl内，按住ctrl点击ServiceImpl，进入他的源码，你会发现里面有一个baseMapper
      // 源码内：
      @Autowired
      protected M baseMapper;
      // 仔细看看源码你就会发现baseMapper其实就是你的mapper层，所以不用再写mapper层的自动注入了，直接用baseMapper代替
      
      // ------------------
      
      // 还有一种说法，在service 和 serviceimpl里去掉他的继承关系，就是最原始的serviceimpl 实现 service即可
      // 使用自动注入mapper的方式在serviceimpl内编写代码
      public interface UService extends IService<U>{
      }
      
      @Service
      public class UServiceImpl extends ServiceImpl<UMapper, U> implements UService {
          @Autowired
          private UMapper umapper;
      }
      // 因为继承IService 和 ServiceImpl实际上增加了耦合性，我们都知道，优秀的代码==高内聚、低耦合
      
      // 以上两种写法请自行斟酌  这里我就用第一种了
      ```
   4. 这里推介一个好用的工具包  hutool，导入依赖
      ```xml
      <dependency>
          <groupId>cn.hutool</groupId>
          <artifactId>hutool-all</artifactId>
          <version>5.8.10</version>
      </dependency>
      ```
   5. 新建一个com.example.demo.util文件夹，新建R类
      ```java
      @Data
      public class R {
      
          private int code;
      
          private String message;
      
          private String type;
      
          private Boolean success;
      
          private Object data;
      
          public static R success(String message) {
              R r = new R();
              r.setCode(200);
              r.setMessage(message);
              r.setSuccess(true);
              r.setType("success");
              r.setData(null);
              return r;
          }
      
          public static R success(String message, Object data) {
              R r = success(message);
              r.setData(data);
              return r;
          }
      
          public static R warning(String message) {
              R r = error(message);
              r.setType("warning");
              return r;
          }
      
          public static R error(String message) {
              R r = success(message);
              r.setSuccess(false);
              r.setType("error");
              return r;
          }
      
          public static R fatal(String message) {
              R r = error(message);
              r.setCode(500);
              return r;
          }
      }
      ```
   6. 编写serviceimpl
      ```java
      @Service
      public class UServiceImpl extends ServiceImpl<UMapper, U> implements UService {
      
      	@Override
      	public Object login(U u) {
      		// TODO Auto-generated method stub
      		// 登录的情况
      		// 1.账号密码为空
      		// 2.账号未注册
      		// 3.密码错误
               // 4.登陆成功
      		if(StrUtil.isEmptyIfStr(u)) {
      			return R.error("账号密码为空");
      		}
      		QueryWrapper<U> userQuery=new QueryWrapper<>();
               // 查找数据库，当数据库中name的值等于u.getName()的时候
      		userQuery.eq("name", u.getName());
               // 从数据库中查找到的user（可能为空，即未注册）
      		U user=baseMapper.selectOne(userQuery);
      		if(user==null) {
      			// 用户未注册
      			return R.error("用户未注册");
      		}else {
      			// 登录 成功 or 失败
      			if(user.getPassword().equals(u.getPassword())) {
      				return R.success("登陆成功");
      			}else {
      				return R.error("密码错误");
      			}
      		}
      	}
      
      }
      ```
   7. 编写controller
      ```java
      @RestController
      @RequestMapping("/user")
      public class UController {
      	@Autowired
      	private UService uservice;
      	
      	@PostMapping("/login")
      	public Object checkLogin(@RequestBody U u) {
      		return uservice.login(u);
      	}
      }
      ```
   8. 运行启动类（DemoApplication.java）
6. 目录结构
      <img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201931906.png" alt="目录结构" style="zoom:67%;" />
      <img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201931512.png" alt="目录结构" style="zoom:67%;" />
7.  打开postman进行测试，没有的可以去[官网下载](https://www.postman.com/downloads/?utm_source=postman-home)，我这里也提供一份
      链接：https://pan.baidu.com/s/1PfkZpMb_5zs9adP5TSRXRA?pwd=cc41 
      提取码：cc41 
      <img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201931989.png" alt="请求测试" style="zoom:67%;" />
    
8. 最后比较一下idea和sts。
   
      sts和eclipse差不多，上手难度低，需要配置的不多。相比之下idea上手难度高一些，需要配置的地方也多一些。
   
      但idea功能更多，比如安装一些插件，lombok、mybatis-plus的插件。建议用idea，编写代码真的比sts舒服很多。
