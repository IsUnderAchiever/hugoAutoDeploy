---
title: Activiti7
description: Activiti7
date: 2024-03-01
slug: Activiti7
image: 202412212108679.png
categories:
    - Activiti
---

# 全新Activiti7工作流讲解
# 一、Activiti7概述
官网地址：https://www.activiti.org/
&emsp;&emsp;Activiti由Alfresco软件开发，目前最高版本Activiti 7。是BPMN的一个基于java的软件实现，不过Activiti 不仅仅包括BPMN，还有DMN决策表和CMMN Case管理引擎，并且有自己的用户管理、微服务API等一系列功能，是一个服务平台。
![image-20230517195056636](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242245900.png)
# 二、Activiti7的入门案例
官方手册：http://jeecg.com/activiti5.21/
## 1.创建SpringBoot项目
&emsp;&emsp;现在开发中或者我们自己学习写案例都是通过SpringBoot脚手架工具来快速构建项目的。那么我们也就直接通过创建SpringBoot项目来给大家讲解相关的案例。创建一个普通的SpringBoot项目。指定版本为2.4.2即可
![image-20230517211943193](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242245006.png)
&emsp;&emsp;然后添加对应的依赖：Activiti7的依赖和MySQL的依赖
```xml
<properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.4.2</spring-boot.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.23</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-spring-boot-starter</artifactId>
            <version>7.0.0.GA</version>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot.version}</version>
                <configuration>
                    <mainClass>com.boge.act.PrepareDemo2Application</mainClass>
                    <skip>true</skip>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```
相关的Activiti依赖加载进来了
![image-20230517212124728](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242245918.png)
到这儿基本环境就OK了
## 2.获取ProcessEngine
### 2.1 默认的方式
&emsp;&emsp;在工作流引擎框架中，`ProcessEngine`是一个非常核心的对象，我们需要首先解决这个对象的获取。获取方式很多。先来看最简单的一个基于`activiti.cfg.xml`的XML文件的配置方式。
```java
@Test
public void test1(){
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    System.out.println(processEngine);
}
```
&emsp;&emsp;通过`getDefaultProcessEngine`方法加载会默认的从classpath路径下加载`activiti.cfg.xml`配置文件。我们添加该文件。内容如下：
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans   http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/activiti7" />
        <property name="jdbcDriver" value="com.mysql.cj.jdbc.Driver" />
        <property name="jdbcUsername" value="root" />
        <property name="jdbcPassword" value="123456" />
        <property name="databaseSchemaUpdate" value="true" />
        <property name="asyncExecutorActivate" value="false" />
        <property name="mailServerHost" value="mail.my-corp.com" />
        <property name="mailServerPort" value="5025" />
    </bean>
</beans>
```
然后我们就可以启动，但是出现了如下错误：
![image-20230517211345644](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242245332.png)
&emsp;&emsp;出现这种情况只需要在mysql的连接字符串中添加上nullCatalogMeansCurrent=true，设置为只查当前连接的schema库即可。
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans   http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/activiti7?nullCatalogMeansCurrent=true" />
        <property name="jdbcDriver" value="com.mysql.cj.jdbc.Driver" />
        <property name="jdbcUsername" value="root" />
        <property name="jdbcPassword" value="123456" />
        <property name="databaseSchemaUpdate" value="true" />
        <property name="asyncExecutorActivate" value="false" />
        <property name="mailServerHost" value="mail.my-corp.com" />
        <property name="mailServerPort" value="5025" />
    </bean>
</beans>
```
然后执行程序正确。搞定。同时在数据库中创建了相关的表结构
![image-20230517212912548](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242245982.png)
### 2.2 编程方式获取
&emsp;&emsp;上面的配置文件的方式中的配置文件其实是一个Spring的配置文件，但是这并不意味着Activiti只能用于Spring环境。我们也可以通过编程的方式来使用配置文件，从而来构建ProcessEngineConfiguration对象，具体的实现如下：
```java
@Test
public void test2(){
    ProcessEngine engine = ProcessEngineConfiguration
        	.createStandaloneInMemProcessEngineConfiguration()
            .setJdbcUrl("jdbc:mysql://localhost:3306/activiti7?nullCatalogMeansCurrent=true")
            .setJdbcDriver("com.mysql.cj.jdbc.Driver")
            .setJdbcPassword("123456")
            .setJdbcUsername("root")
            .setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE)
            .buildProcessEngine();
    System.out.println(engine);
}
```
上面讲解中的相关属性说明：
**databaseSchemaUpdate**：用于设置流程引擎启动关闭时使用的数据库表结构控制策略
- `false` (默认): 当引擎启动时，检查数据库表结构的版本是否匹配库文件版本。版本不匹配时抛出异常。
- `true`: 构建引擎时，检查并在需要时更新表结构。表结构不存在则会创建。
- `create-drop`: 引擎创建时创建表结构，并在引擎关闭时删除表结构。
### 2.3 表结构介绍
&emsp;&emsp;在Activiti7中。我们启动服务会自动维护Activiti7需要使用到的相关的表结构。在这块我们需要有个大概的了解。首先是支持的数据库有：
| Activiti数据库类型 | 示例JDBC URL                                                 | 备注                                     |
| ------------------ | ------------------------------------------------------------ | ---------------------------------------- |
| h2                 | jdbc:h2:tcp://localhost/activiti                             | 默认配置的数据库                         |
| mysql              | jdbc:mysql://localhost:3306/activiti?autoReconnect=true      | 已使用mysql-connector-java数据库驱动测试 |
| oracle             | jdbc:oracle:thin:@localhost:1521:xe                          |                                          |
| postgres           | jdbc:postgresql://localhost:5432/activiti                    |                                          |
| db2                | jdbc:db2://localhost:50000/activiti                          |                                          |
| mssql              | jdbc:sqlserver://localhost:1433;databaseName=activiti (jdbc.driver=com.microsoft.sqlserver.jdbc.SQLServerDriver) *OR* jdbc:jtds:sqlserver://localhost:1433/activiti (jdbc.driver=net.sourceforge.jtds.jdbc.Driver) | 已使用Microsoft JDBC Driver 4.0 (sqljdb  |
&emsp;&emsp;Activiti的所有数据库表都以**ACT_**开头。第二部分是说明表用途的两字符标示符。服务API的命名也大略符合这个规则。
**ACT_RE_***: `RE`代表`repository`。带有这个前缀的表包含“静态”信息，例如流程定义与流程资源（图片、规则等）。
**ACT_RU_***: `RU`代表`runtime`。这些表存储运行时信息，例如流程实例（process instance）、用户任务（user task）、变量（variable）、作业（job）等。Activiti只在流程实例运行中保存运行时数据，并在流程实例结束时删除记录。这样保证运行时表小和快。
**ACT_ID_***: `ID`代表`identity`。这些表包含身份信息，例如用户、组等。
**ACT_HI_***: `HI`代表`history`。这些表存储历史数据，例如已完成的流程实例、变量、任务等。
**ACT_GE_***: 通用数据。用于不同场景下
注意：MySQL数据库最好使用5.7及以上的版本
## 3.在线流程设计器
&emsp;&emsp;接下来我们通过官方提供的流程设计器来实现一个简单流程的设计。然后完成相关的部署和流程整体操作。
官网下载地址：https://www.activiti.org/get-started 下载下来后解压缩
![image-20230519111655829](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242245884.png)
进入到wars中。提供的有Activiti-app.war
![image-20230519111731391](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242245820.png)
把这war包拷贝到Tomcat服务器中即可。注意Tomcat的版本不要高于8.5，然后Tomcat服务。访问 http://localhost:8080/activiti-app 即可。登录的账号密码是 admin  test
![image-20230519111931446](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242245125.png)
![image-20230519112000632](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242245689.png)
![image-20230519112027397](C:\Users\dpb\AppData\Roaming\Typora\typora-user-images\image-20230519112027397.png)
点击create process 弹出窗口。录入相关的流程定义信息
![image-20230519112212684](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242245281.png)
绘制好流程图后。保存并下载对应的xml文件
![image-20230519112253385](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242246185.png)
得到的流程图的xml内容：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/processdef">
  <process id="test1" name="test1" isExecutable="true">
    <documentation>test1</documentation>
    <startEvent id="startEvent1"></startEvent>
    <userTask id="sid-470631FF-51BA-4954-96BB-346B99CA0A2C" name="人事审批" activiti:assignee="zhangsan">
      <extensionElements>
        <modeler:initiator-can-complete xmlns:modeler="http://activiti.com/modeler"><![CDATA[false]]></modeler:initiator-can-complete>
      </extensionElements>
    </userTask>
    <sequenceFlow id="sid-B53369E8-E698-4F53-AE40-97E7654BFA78" sourceRef="startEvent1" targetRef="sid-470631FF-51BA-4954-96BB-346B99CA0A2C"></sequenceFlow>
    <userTask id="sid-34454522-B109-41C9-8519-59D29B621099" name="经理审批" activiti:assignee="lisi">
      <extensionElements>
        <modeler:initiator-can-complete xmlns:modeler="http://activiti.com/modeler"><![CDATA[false]]></modeler:initiator-can-complete>
      </extensionElements>
    </userTask>
    <sequenceFlow id="sid-5AE88ADE-9FD3-48F2-81EF-528DA0C068CB" sourceRef="sid-470631FF-51BA-4954-96BB-346B99CA0A2C" targetRef="sid-34454522-B109-41C9-8519-59D29B621099"></sequenceFlow>
    <endEvent id="sid-EA0332FA-59B0-45C0-9D24-47C78051D52C"></endEvent>
    <sequenceFlow id="sid-F6C0657A-C92F-4DEA-AAB1-93750FFBD7E5" sourceRef="sid-34454522-B109-41C9-8519-59D29B621099" targetRef="sid-EA0332FA-59B0-45C0-9D24-47C78051D52C"></sequenceFlow>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_test1">
    <bpmndi:BPMNPlane bpmnElement="test1" id="BPMNPlane_test1">
      <bpmndi:BPMNShape bpmnElement="startEvent1" id="BPMNShape_startEvent1">
        <omgdc:Bounds height="30.0" width="30.0" x="100.0" y="163.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-470631FF-51BA-4954-96BB-346B99CA0A2C" id="BPMNShape_sid-470631FF-51BA-4954-96BB-346B99CA0A2C">
        <omgdc:Bounds height="80.0" width="100.0" x="175.0" y="138.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-34454522-B109-41C9-8519-59D29B621099" id="BPMNShape_sid-34454522-B109-41C9-8519-59D29B621099">
        <omgdc:Bounds height="80.0" width="100.0" x="320.0" y="138.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="sid-EA0332FA-59B0-45C0-9D24-47C78051D52C" id="BPMNShape_sid-EA0332FA-59B0-45C0-9D24-47C78051D52C">
        <omgdc:Bounds height="28.0" width="28.0" x="465.0" y="164.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="sid-5AE88ADE-9FD3-48F2-81EF-528DA0C068CB" id="BPMNEdge_sid-5AE88ADE-9FD3-48F2-81EF-528DA0C068CB">
        <omgdi:waypoint x="275.0" y="178.0"></omgdi:waypoint>
        <omgdi:waypoint x="320.0" y="178.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-F6C0657A-C92F-4DEA-AAB1-93750FFBD7E5" id="BPMNEdge_sid-F6C0657A-C92F-4DEA-AAB1-93750FFBD7E5">
        <omgdi:waypoint x="420.0" y="178.0"></omgdi:waypoint>
        <omgdi:waypoint x="465.0" y="178.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="sid-B53369E8-E698-4F53-AE40-97E7654BFA78" id="BPMNEdge_sid-B53369E8-E698-4F53-AE40-97E7654BFA78">
        <omgdi:waypoint x="130.0" y="178.0"></omgdi:waypoint>
        <omgdi:waypoint x="175.0" y="178.0"></omgdi:waypoint>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```
然后我们就可以做流程的部署操作了
## 4.流程操作
### 4.1 流程部署
&emsp;&emsp;设计好了流程图我们就可以通过如下的代码完成流程的部署。
```java
   /**
     * 流程部署操作
     */
    @Test
    public void test3(){
        // 1.获取ProcessEngine对象
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        // 2.完成流程的部署操作 需要通过RepositoryService来完成
        RepositoryService repositoryService = processEngine.getRepositoryService();
        // 3.完成部署操作
        Deployment deploy = repositoryService.createDeployment()
                .addClasspathResource("flow/test1.bpmn20.xml")
                .name("第一个流程")
                .deploy();
        System.out.println(deploy.getId());
        System.out.println(deploy.getName());
    }
```
流程部署的行为会涉及到数据库中的这两张表
![image-20230519114622514](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242246862.png)
然后我们可以通过Activiti提供的相关的API来获取流程部署和流程定义的相关信息
```java
/**
 * 查询当前部署的流程有哪些
 */
@Test
public void test4(){
    ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
    RepositoryService repositoryService = engine.getRepositoryService();
    // 查询有哪些部署的流程--》查询相关的流程定义信息
    // repositoryService.createDeploymentQuery() 查询流程部署的相关信息
    // repositoryService.createProcessDefinitionQuery() 查询部署的流程的相关的定义
    List<Deployment> list = repositoryService.createDeploymentQuery().list(); // 查询所有的部署信息
    for (Deployment deployment : list) {
        System.out.println(deployment.getId());
        System.out.println(deployment.getName());
    }
    List<ProcessDefinition> list1 = repositoryService.createProcessDefinitionQuery().list();
    for (ProcessDefinition processDefinition : list1) {
        System.out.println(processDefinition.getId());
        System.out.println(processDefinition.getName());
        System.out.println(processDefinition.getDescription());
    }
}
```
### 4.2 发起流程
&emsp;&emsp;部署流程成功后。我们就可以发起一个流程。发起流程需要通过`RuntimeService`来实现。
```java
/**
 * 发起一个流程
 */
@Test
public void test5(){
    ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
    // 发起流程 需要通过 runtimeService来实现
    RuntimeService runtimeService = engine.getRuntimeService();
    // 通过流程定义ID来启动流程  返回的是流程实例对象
    ProcessInstance processInstance = runtimeService
            .startProcessInstanceById("test1:1:3");
    System.out.println("processInstance.getId() = " + processInstance.getId());
    System.out.println("processInstance.getDeploymentId() = " + processInstance.getDeploymentId());
    System.out.println("processInstance.getDescription() = " + processInstance.getDescription());
}
```
发起流程成功后。在对应的`act_ru_task`中就有一条对应的待办记录。
![image-20230520100457878](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242246137.png)
对应的流程状态如下：
![image-20230520100513859](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242246104.png)
### 4.3 查询流程
&emsp;&emsp;用户登录后要查看待办的任务信息。我们需要通过`TaskService`来实现查询操作。具体代码如下：
```java
    /**
     * 待办查询
     */
    @Test
    public void test6(){
        ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
        // 待办查询 执行中的任务处理通过 TaskService来实现
        TaskService taskService = engine.getTaskService();
        // Task 对象对应的其实就是 act_ru_task 这张表的记录
        List<Task> list = taskService.createTaskQuery().taskAssignee("lisi").list();
        if(list != null && !list.isEmpty()){
            for (Task task : list) {
                System.out.println("task.getId() = " + task.getId());
                System.out.println("task.getName() = " + task.getName());
                System.out.println("task.getAssignee() = " + task.getAssignee());
            }
        }else{
            System.out.println("当前没有待办任务");
        }
    }
```
### 4.4 审批流程
&emsp;&emsp;当前登录用户查看到相关的待办信息后。可以做流程的审批处理。
```java
/**
 * 任务审批
 */
@Test
public void test7(){
    ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
    // 做任务申请 也需要通过 TaskService 来实现
    TaskService taskService = engine.getTaskService();
    // 根据当前登录用户查询出对应的待办信息
    List<Task> list = taskService.createTaskQuery().taskAssignee("lisi").list();
    if(list != null && list.size() > 0){
        for (Task task : list) {
            // 做对应的任务审批处理
            taskService.complete(task.getId());
        }
    }
    // 完成任务
    // taskService.complete("2505");
}
```
## 5.涉及表结构
&emsp;&emsp;上面一个审批涉及到的表结构的介绍
| 表名              | 说明                                                   |
| ----------------- | ------------------------------------------------------ |
| act_re_deployment | 部署流程的记录表：一次部署行为会产生一张表             |
| act_re_procdef    | 流程定义表：一张流程图对应的表                         |
| act_hi_procinst   | 流程实例表：发起一个流程。就会创建对应的一张表         |
| act_ru_task       | 流程待办表：当前需要审批的记录表，节点审批后就会被删除 |
| act_hi_actinst    | 历史记录：流程审批节点的记录信息                       |
![相关表结构](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242246614.png)
# 三、表达式
## 值表达式
![image-20230623091854461](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242246605.png)
> `${}`
##  方法表达式
方法表达式Method expression:调用一个方法， 可以带或不带参数。当调不带参数的方法时，要确保在方法名后添加空括号(以避免与值表达式混淆)。传递的参数可以是字面值,也可以是表达式，它们会被自动解析。例如:
```java
${printer.print()}
${myBean.getAssignee()}
${myBean.addNewOrder('orderName')}
${myBean.doSomething(myVar,execution)}
```
## 监听器
![image-20230623132637416](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242246908.png)
![image-20230623132659870](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242246049.png)
