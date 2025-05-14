---
title: Seata
description: Seata
date: 2024-04-16
slug: Seata
image: 202412212130408.png
categories:
    - Seata
---

# Seata
## 概念
### 事务的ACID原则
![image-20240410224007195](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410224007195.png)
1. 原子性
   事务中的所有操作，要么全部成功,要么全部失败
2. 一致性
   要保证数据库内部完整性约束、声明性约束
3. 隔离性
   对同一资源操作的事务不能同时发生
4. 持久性
   对数据库做的一切修改将永久保存,不管是否出现故障
### 案例
微服务下单业务，在下单时会调用订单服务，创建订单并写入数据库。然后订单服务调用账户服务和库存服务:
- 账户服务负责扣减用户余额
- 库存服务负责扣减商品库存
![image-20240410224108029](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410224108029.png)
> 我们希望`创建订单`、`扣减余额`、`扣件商品库存`三个操作`同时成功或者失败`
**这里新建三个数据库**
![image-20240411202324161](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411202324161.png)
**sentinel_account**
```sql
create table account_tbl
(
    id      int auto_increment comment '账户编号'
        primary key,
    user_id varchar(255)               null comment '用户编号',
    money   int(11) unsigned default 0 null comment '余额'
)
    comment '账户表' charset = utf8
                     row_format = COMPACT;
                     
INSERT INTO sentinel_account.account_tbl (id, user_id, money) VALUES (1, 'user202103032042012', 1000);
```
**sentinel_order**
```sql
create table order_tbl
(
    id             int auto_increment comment '订单编号'
        primary key,
    user_id        varchar(255)  null comment '用户编号',
    commodity_code varchar(255)  null comment '商品编码',
    count          int default 0 null comment '数量',
    money          int default 0 null comment '金额'
)
    comment '订单表' charset = utf8
                     row_format = COMPACT;
```
**sentinel_storage**
```sql
create table storage_tbl
(
    id             int auto_increment comment '库存编号'
        primary key,
    commodity_code varchar(255)               null comment '商品编码',
    count          int(11) unsigned default 0 null comment '库存数量',
    constraint commodity_code
        unique (commodity_code)
)
    comment '库存表' charset = utf8
                     row_format = COMPACT;
                     
INSERT INTO sentinel_storage.storage_tbl (id, commodity_code, count) VALUES (1, '100202003032041', 10);
```
> 基础项目的[链接](https://gitee.com/tongstyle/seata-demo)
**调用`client`实现`扣余额`、`扣库存`**
![image-20240411214609610](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411214609610.png)
> 使用`ApiFox`测试订单创建请求
![image-20240411220655902](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411220655902.png)
> 看看数据，发现该商品还剩下10个
![image-20240411220800785](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411220800785.png)
> 我们直接买20个，正常来讲`库存不足`应该要`失败`，这次用户要跟账户表保持一致
![image-20240411221346303](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411221346303.png)
> 报错
![image-20240411221519897](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411221519897.png)
> 刷新之后发现，库存表数量没变`(扣减失败)`，但是账户表`金额`被扣了
![image-20240411221600202](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411221600202.png)
> **它们每个服务是独立的**，各自提交自己的事务，所以并没有达成事务的一致
**`总结：在分布式系统下，一个业务跨越多个服务或数据源，每个服务都是一个分支事务， 要保证所有分支事务最终状态一致,这样的事务就是分布式事务。`**
## 理论知识
### CAP定理
1998年，加州大学的计算机科学家Eric Brewer提出，分布式系统有三个指标:
1. Consistency (`一致性`)
2. Availability (`可用性`)
3. Partition tolerance (`分区容错性`)
Eric Brewer说，分布式系统无法`同时满足`这三个指标。这个结论就叫做`CAP定理`。
![image-20240411221941042](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411221941042.png)
#### 一致性
Consistency (一致性) :用户访问分布式系统中的任意节点,得到的数据必须一致
![image-20240411222047176](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411222047176.png)
#### 可用性
Availability (可用性) :用户访问集群中的任意健康节点,必须能得到响应，而不是超时或拒绝
![image-20240411222133028](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411222133028.png)
#### 分区容错
Partition (分区) :因为网络故障或其它原因导致分布式系统中的部分节点与其它节点失去连接，形成独立分区。
Tolerance (容错) :在集群出现分区时,整个系统也要持续对外提供服务
![image-20240411222243057](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411222243057.png)
#### 总结
分布式系统节点通过网络连接，一定会出现分区问题(P)，当分区出现时，系统的一致性(C)和可用性(A)就无法同时满足
#### 思考
elasticsearch集群是CP还是AP?
ES集群出现分区时，故障节点会被剔除集群，数据分片会重新分配到其它节点，保证数据一致。因此是低可用性，高一致性，属于CP
### BASE理论
BASE理论是对CAP的一种解决思路,包含三个思想:
`Basically Available`(**`基本可用`**) :分布式系统在出现故障时，允许损失部分可用性，即保证核心可用。
`SoftState` (**`软状态`**) :在一定时间内，允许出现中间状态，比如临时的不一致状态。
`Eventually Consistent` (**`最终一致性`**) :虽然无法保证强-致性,但是在软状态结束后，最终达到数据一致。
而分布式事务最大的问题是各个子事务的一致性问题，因此可以**借鉴CAP定理和BASE理论**:
`AP模式`:各子事务分别执行和提交,允许出现结果不一致,然后采用弥补措施`恢复数据`即可，实现`最终一致`。
`CP模式`:各个子事务执行后互相等待，同时提交,同时回滚,达成`强一致`。但事务等待过程中,处于弱可用状态。因为会锁定资源导致无法访问
#### 分布式事务模型
解决分布式事务，各个子系统之间必须能感知到彼此的事务状态，才能保证状态一致，因此需要一个事务协调者来协调每一个事务的参与者(子系统事务)。
![image-20240411223154817](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411223154817.png)
>这里的子系统事务，称为分支事务;有关联的各个分支事务在一起称为全局事务
## 初识Seata
### Seata架构
Seata事务管理中有三个重要的角色:
1. TC (Transaction Coordinator) -事务协调者:维护全局和分支事务的状态，协调全局事务提交或回滚。
2. TM (Transaction Manager) -事务管理器:定义全局事务的范围、开始全局事务、提交或回滚全局事务。
3. RM (Resource Manager) -资源管理器:管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。
![image-20240411223721605](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411223721605.png)
Seata提供了四种不同的分布式事务解决方案:
- XA模式:强一致性分阶段事务模式，牺牲了一定的可用性，无业务侵入
- TCC模式:最终一致的分阶段事务模式，有业务侵入
- AT模式:最终一致的分阶段事务模式，无业务侵入，也是Seata的默认模式
- SAGA模式:长事务模式，有业务侵入
### 安装Seata
#### 下载
前往Github[下载](https://github.com/apache/incubator-seata)，这里选择`seata-server-1.7.0`的版本
#### 配置nacos为注册中心和配置中心
![image-20240412070658420](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412070658420.png)
> `application.example.yml`的配置很全，我们copy需要的配置到`application.yml`即可
![image-20240412071007499](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412071007499.png)
**修改后的配置**
```yml
#  Copyright 1999-2019 Seata.io Group.
#
#  Licensed under the Apache License, Version 2.0 (the "License");
#  you may not use this file except in compliance with the License.
#  You may obtain a copy of the License at
#
#  http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
#  limitations under the License.
server:
  port: 7091
spring:
  application:
    name: seata-server
logging:
  config: classpath:logback-spring.xml
  file:
    path: ${user.home}/logs/seata
  extend:
    logstash-appender:
      destination: 127.0.0.1:4560
    kafka-appender:
      bootstrap-servers: 127.0.0.1:9092
      topic: logback_to_logstash
console:
  user:
    username: seata
    password: seata
seata:
  config:
    # support: nacos 、 consul 、 apollo 、 zk  、 etcd3
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      namespace: d2455f6d-ed00-41ba-9915-09021bc0df6c
      # group: SEATA_GROUP
      group: DEFAULT_GROUP
      username: nacos
      password: nacos
      context-path:
      ##if use MSE Nacos with auth, mutex with username/password attribute
      #access-key:
      #secret-key:
      data-id: seataServer.properties
  registry:
    # support: nacos 、 eureka 、 redis 、 zk  、 consul 、 etcd3 、 sofa
    type: nacos
    preferred-networks: 30.240.*
    nacos:
      application: seata-server
      server-addr: 127.0.0.1:8848
      # group: SEATA_GROUP
      group: DEFAULT_GROUP
      namespace: d2455f6d-ed00-41ba-9915-09021bc0df6c
      cluster: default
      username: nacos
      password: nacos
      context-path:
      ##if use MSE Nacos with auth, mutex with username/password attribute
      #access-key:
      #secret-key:
  store:
    # support: file 、 db 、 redis
    mode: db
#  server:
#    service-port: 8091 #If not configured, the default is '${server.port} + 1000'
  security:
    secretKey: SeataSecretKey0c382ef121d778043159209298fd40bf3850a017
    tokenValidityInMilliseconds: 1800000
    ignore:
      urls: /,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.jpeg,/**/*.ico,/api/v1/auth/login
```
**在`nacos`中新增配置**
![image-20240412071132954](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412071132954.png)
> 配置文件可以查看
![image-20240412071236353](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412071236353.png)
**新增配置如下**
> 下面除了`db`外的配置都是默认的，写到nacos是为了方便后期修改，`db之外的配置`不写也没事
>
> 这里的`driverClassName`我用的是`com.mysql.cj.jdbc.Driver`，注意修改成自己的数据库名、账号、密码
```properties
# 数据存储方式，db代表数据库
store.mode=db
store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.cj.jdbc.Driver
store.db.url=jdbc:mysql://127.0.0.1:3306/seata?useUnicode=true&rewriteBatchedStatements=true
store.db.user=root
store.db.password=123456
store.db.minConn=5
store.db.maxConn=30
store.db.globalTable=global_table
store.db.branchTable=branch_table
store.db.queryLimit=100
store.db.lockTable=lock_table
store.db.maxWait=5000
# 事务、日志等配置
server.recovery.committingRetryPeriod=1000
server.recovery.asynCommittingRetryPeriod=1000
server.recovery.rollbackingRetryPeriod=1000
server.recovery.timeoutRetryPeriod=1000
server.maxCommitRetryTimeout=-1
server.maxRollbackRetryTimeout=-1
server.rollbackRetryTimeoutUnlockEnable=false
server.undo.logSaveDays=7
server.undo.logDeletePeriod=86400000
# 客户端与服务端传输方式
transport.serialization=seata
transport.compressor=none
# 关闭metrics功能，提高性能
metrics.enabled=false
metrics.registryType=compact
metrics.exporterList=prometheus
metrics.exporterPrometheusPort=9898
```
![image-20240412071551150](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412071551150.png)
> db配置，注意这里新增一个`undo_log`表，该表的建表语句在sql中不存在
>
> **完整sql如下**
```sql
-- -------------------------------- The script used when storeMode is 'db' --------------------------------
-- the table to store GlobalSession data
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_status_gmt_modified` (`status` , `gmt_modified`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;
-- the table to store BranchSession data
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;
-- the table to store lock data
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(128),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `status`         TINYINT      NOT NULL DEFAULT '0' COMMENT '0:locked ,1:rollbacking',
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_status` (`status`),
    KEY `idx_branch_id` (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;
CREATE TABLE IF NOT EXISTS `distributed_lock`
(
    `lock_key`       CHAR(20) NOT NULL,
    `lock_value`     VARCHAR(20) NOT NULL,
    `expire`         BIGINT,
    primary key (`lock_key`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;
INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('AsyncCommitting', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('RetryCommitting', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('RetryRollbacking', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('TxTimeoutCheck', ' ', 0);
-- 注意此处0.3.0+ 增加唯一索引 ux_undo_log
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```
> 而且`业务数据库也需要增加该表`
![image-20240412071916161](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412071916161.png)
> 双击`seata-server.bat`启动即可
![image-20240412071937524](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412071937524.png)
> 服务已经注册到nacos中了
![image-20240412072014359](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412072014359.png)
> 注意命名空间，我们springboot的demo是`public`的，所以需要将seata的命名空间改为`public`
![image-20240412072736505](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412072736505.png)
### 基本使用
> 引入依赖
```xml
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
        </dependency>
```
> 在`微服务`中新增如下配置
```yaml
seata:
    registry:
        type: nacos
        nacos:
            server-addr: localhost:8848
            namespace: public
            group: DEFAULT_GROUP
            # tc服务在nacos的注册名称
            application: seata-server
            username: nacos
            password: nacos
    # 事务组，根据这个获取tc服务的cluster名称
    tx-service-group: seata-demo
    service:
        # 事务组与TC服务cluster的映射关系
        vgroup-mapping:
          seata-demo: SH
```
> 完整的配置如下
```yaml
server:
    port: 8070
spring:
    application:
        name: account-service
    cloud:
        nacos:
            config:
                server-addr: localhost:8848
            discovery:
                server-addr: localhost:8848
    config:
        import: nacos:nacos-config-example.properties?refresh=true
    datasource:
        # 数据库驱动：
        driver-class-name: com.mysql.cj.jdbc.Driver
        # 数据库连接地址
        name: defaultDataSource
        # 数据库用户名&密码：
        username: root
        password: 123456
        # 数据源名称
        url: jdbc:mysql://localhost:3306/sentinel_account?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
mybatis-plus:
    configuration:
        # 配置mybatis-plus 打印sql日志
        log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
        # mybatis-plus下划线转驼峰配置，默认为true
        map-underscore-to-camel-case: true
    # xml文件路径
    mapper-locations: classpath:/mapper/**/*.xml
    # 配置mybatis-plus 包路径
    type-aliases-package: com.example.account.entity
seata:
    registry:
        type: nacos
        nacos:
            server-addr: localhost:8848
            namespace: public
            group: DEFAULT_GROUP
            # tc服务在nacos的注册名称
            application: seata-server
            username: nacos
            password: nacos
    # 事务组，根据这个获取tc服务的cluster名称
    tx-service-group: seata-demo
    service:
        # 事务组与TC服务cluster的映射关系
        vgroup-mapping:
          seata-demo: SH
```
> 如果需要锁定一个服务，我们需要确定以下四个配置
>
> 1. 命名空间
> 2. 服务分组
> 3. 服务名称
> 4. 集群名称
**集群如下**
![image-20240412073342542](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412073342542.png)
> 再改一下`seata`的配置，配置`SH`集群
![image-20240412073612996](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412073612996.png)
![image-20240412073639729](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412073639729.png)
> 我们并没有在微服务的`application`文件中配置`cluster`
![image-20240412073752159](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412073752159.png)
> 并不能在`registry`下直接配置`cluster`
>
> 正确做法是配置`事务组`，将事务分组，比如`account`、`order`、`storage`将来会被同一个`tc`集群管理，所以需要将它们配置到同一个事务组下
>
> 再根据`事务组`获取到映射关系，最终找到`集群名称`
>
> 这部分涉及到了`seata`的高可用，暂时不用过多深究
**启动服务**
启动微服务后，seata控命令窗口打印日志如下
```
RM register success,message:RegisterRMRequest{resourceIds='jdbc:mysql://localhost:3306/sentinel_account', version='1.7.0-native-rc2', applicationId='account-service', transactionServiceGroup='seata-demo', extraData='null'},channel:[id: 0xc06aa8c5, L:/192.168.56.1:8091 - R:/192.168.56.1:57929],client version:1.7.0-native-rc2
```
## 动手实践
### XA模式
#### 概念
> XA规范是x/Open组织定义的分布式事务处理(DTP, Distributed Transaction Processing)标准，XA规范描述了全局的TM与局部的RM之间的接口，几乎所有主流的数据库都对XA规范提供了支持。XA模式是一种强一致性、低可用性的模式
**正常情况**
![image-20240412075025182](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412075025182.png)
**异常情况**
![image-20240412075111494](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412075111494.png)
#### Seata中的XA模式
> seata的XA模式做了一些调整,但大体相似
![image-20240412075313338](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412075313338.png)
**RM一阶段的工作**:
1. 注册分支事务到TC
2. 执行分支业务sq|但不提交
3. 报告执行状态到TC
**TC二阶段的工作**:
- TC检测各分支事务执行状态
  ​	a.如果都成功，通知所有RM提交事务
  ​	b.如果有失败，通知所有RM回滚事务
**RM二阶段的工作**:
接收TC指令，提交或回滚事务
#### 总结
XA模式的优点是什么?
1. 事务的强一致性,满足ACID原则。
2. 常用数据库都支持，实现简单，并且没有代码侵入
XA模式的缺点是什么?
1. 因为一阶段需要锁定数据库资源，等待二阶段结束才释放，性能较差
2. 依赖关系型数据库实现事务(如redis就无法实现)
#### 实现XA模式
1. 修改`application.yml`文件，开启`XA`模式
   ```yaml
   seata:
       registry:
           type: nacos
           nacos:
               server-addr: localhost:8848
               namespace: public
               group: DEFAULT_GROUP
               # tc服务在nacos的注册名称
               application: seata-server
               username: nacos
               password: nacos
       # 事务组，根据这个获取tc服务的cluster名称
       tx-service-group: seata-demo
       service:
           # 事务组与TC服务cluster的映射关系
           vgroup-mapping:
               seata-demo: SH
   	# 开启XA模式
       data-source-proxy-mode: XA
   ```
2. 给发起全局事务的入口方法添加`@GlobalTransactional`注解
   ```java
   @Slf4j
   @Service
   public class OrderServiceImpl implements OrderService {
   
       private final AccountClient accountClient;
       private final StorageClient storageClient;
       private final OrderMapper orderMapper;
   
       public OrderServiceImpl(AccountClient accountClient, StorageClient storageClient, OrderMapper orderMapper) {
           this.accountClient = accountClient;
           this.storageClient = storageClient;
           this.orderMapper = orderMapper;
       }
   
       @Override
       @GlobalTransactional
       public Long create(Order order) {
           // 创建订单
           orderMapper.insert(order);
           try {
               // 扣用户余额
               accountClient.deduct(order.getUserId(), order.getMoney());
               // 扣库存
               storageClient.deduct(order.getCommodityCode(), order.getCount());
   
           } catch (FeignException e) {
               log.error("下单失败，原因:{}", e.contentUTF8(), e);
               throw new RuntimeException(e.contentUTF8(), e);
           }
           return order.getId();
       }
   }
   ```
3. 重启服务并测试
**运行前的数据**
![image-20240412080441953](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412080441953.png)
**运行后的数据**
![image-20240412080512334](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412080512334.png)
> 可以看到实现了`全局事务`
### AT模式
#### 概念
> AT模式同样是分阶段提交的事务模型，不过缺弥补了XA模型中资源锁定周期过长的缺陷。
![image-20240412081358601](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412081358601.png)
>阶段一RM的工作:
>
>1. 注册分支事务
>2. 记录`undo-log` (数据快照)
>3. 执行业务sql并提交
>4. 报告事务状态
>
>阶段二提交时RM的工作:
>删除undo-log即可
>
>阶段二回滚时RM的工作:
>根据undo-log恢复数据到更新前
**案例如下**
一个分支业务的SQL是这样的: 
```sql
update tb account set money = money- 10 where id = 1
```
![image-20240412081715928](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412081715928.png)
#### 总结
简述AT模式与XA模式最大的区别是什么?
- XA模式一阶段不提交事务，锁定资源; AT模式一阶段直接提交，不锁定资源。
- XA模式依赖数据库机制实现回滚; AT模式利用数据快照实现数据回滚。
- `XA模式强一致`; `AT模式最终一致`
#### AT模式的脏写问题
![image-20240412210846377](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412210846377.png)
> 事务1在提交事务之后，紧接着被事务2获取到了锁，而事务1的全局事务需要回滚，则此时事务1会将`money`恢复到100，但是影响到了事务2的操作
>
> 原因就是事务没有做到隔离，解决方式就是引入`全局锁`，标记对某行数据的拥有权，在事务提交前，其他事务无法修改
![image-20240412211854987](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240412211854987.png)
> 事务1在提交事务后，释放DB锁；而DB锁马上被事务2获取到了，但全局事务锁在事务1手上，事务2拿不到
>
> 而DB锁在事务2手上，事务1拿不到，此时产生了死锁，但由于获取全局事务锁的过程有重试机制，任务超时会回滚并释放DB锁
>
> 这时事务1获取到了DB锁
**有一个问题，如果在事务1的`1.3~2.1`过程中，有一个不被`seata`管理的事务修改了值，也会产生脏写问题**
![](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/QQ%E5%9B%BE%E7%89%8720240412212956.png)
> 在保存快照时，保存两份数据，一个是修改前的，一个是修改后的
>
> 当释放全局锁时，对比修改后的数据，如果不一致，说明在这段时间有其他事务修改了数据
>
> 这就没办法了，只能人工介入，这是比较极端的情况
#### 总结
AT模式的优点:
1. 一阶段完成直接提交事务,释放数据库资源，性能比较好
2. 利用全局锁实现读写隔离
3. 没有代码侵入,框架自动完成回滚和提交
AT模式的缺点:
1. 两阶段之间属于软状态，属于最终一致
2. 框架的快照功能会影响性能,但比XA模式要好很多
#### 步骤
1. 新建两张表，其中lock_table导入到TC服务关联的数据库，undo_log表导入到微服务关联的数据库，这些操作在Seata配置阶段已经做过了
2. 修改`application.yml`，将事务模式改为`AT`即可
   ```yaml
   seata:
       registry:
           type: nacos
           nacos:
               server-addr: localhost:8848
               namespace: public
               group: DEFAULT_GROUP
               # tc服务在nacos的注册名称
               application: seata-server
               username: nacos
               password: nacos
       # 事务组，根据这个获取tc服务的cluster名称
       tx-service-group: seata-demo
       service:
           # 事务组与TC服务cluster的映射关系
           vgroup-mapping:
               seata-demo: SH
       data-source-proxy-mode: AT
   ```
   
3. 重启服务测试
### TCC模式
#### 概念
TCC模式与AT模式非常相似,每阶段都是独立事务，不同的是TCC通过人工编码来实现数据恢复，而非自动恢复(生成快照)。需要实现三个方法:
1. Try: 资源的检测和预留;
2. Confirm: 完成资源操作业务;要求Try成功Confirm一定要能成功。
3. Cancel: 预留资源释放,可以理解为try的反向操作。
![image-20240413063600514](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240413063600514.png)
![image-20240413063809543](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240413063809543.png)
#### 总结
TCC模式的每个阶段是做什么的?
1. Try:资源检查和预留
2. Confirm:业务执行和提交
3. Cancel:预留资源的释放
TCC的优点是什么?
1. 一阶段完成直接提交事务,释放数据库资源,性能好
2. 相比AT模型，无需生成快照，无需使用全局锁，性能最强
3. 不依赖数据库事务，而是依赖补偿操作，可以用于非事务型数据库
TCC的缺点是什么?
1. 有代码侵入，需要人为编写try、Confirm和Cancel接口，太麻烦
2. 软状态，事务是最终一致
3. 需要考虑Confirm和Cancel的失败情况，做好`幂等处理`。(失败后，seata会`重试`，需要考虑重复处理的情况)
#### 案例
需求如下:
1. 修改account-service,编写try、confirm、 cancel逻辑
2. try业务:添加冻结金额，扣减可用金额
3. confirm业务:删除冻结金额
4. cancel业务:删除冻结金额，恢复可用金额
5. 保证confirm、cancel接口的`幂等性`（业务接口不会因为重复调用出现问题）
6. 允许`空回滚`
7. 拒绝`业务悬挂`
> 有几个需要注意的点，`资源预留`是为了修改目标数据的值，但是新增订单是一个新增逻辑，这并`不适合TCC模式`，用`AT模式`就很好了
>
> 扣减余额和扣减库存可以使用TCC模式，在一个分布式事务当中既有`TCC模式`又有`AT模式`可以么？
>
> 答案是可以的
##### 空回滚问题
当某分支事务的try阶段阻塞时，可能导致全局事务超时而触发二阶段的cancel操作。在未执行try操作时先执行了cancel操作，这时cancel不能做回滚,就是`空回滚`。这里并没有进行`资源预留`也无法进行`回滚`，但是这里也不能进行报错，否则`Seata`会认为`cancel`失败，应该返回正常结束。
![image-20240413065417807](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240413065417807.png)
##### 业务悬挂
> 此时阻塞的业务终于畅通了，但是这个事务已经提交了，后续既没有进行`confirm`也没有执行`cancel`
>
> 对于已经空回滚的业务，如果以后继续执行try,就永远不可能confirm或cancel,这就是`业务悬挂`。应当阻止执行空回滚后的try操作，避免悬挂
![image-20240413070301607](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240413070301607.png)
> 为了避免`空回滚`，应该在`cancel`判断是否进行了`try`
>
> 为了避免`业务悬挂`，应该在`try`判断是否进行了`cancel`
>
> 这样就必须在数据库里记录当前状态，判断当前是在`try`？还是在`confirm`或者`cancel`？
```sql
-- auto-generated definition
create table accoutn_freeze_tbl_8
(
    xid          varchar(188)           not null comment '事务编号'
        primary key,
    user_id      varchar(255)           null comment '用户编号`',
    freeze_money int unsigned default 0 null comment '冻结金额',
    state        int                    null comment '事务状态(0:try、1:confirm、2:cancel)'
)
    comment '冻结金额表';
```
###### try业务
1. 记录冻结金额和事务状态到account_ freeze表
2. 扣减account表可用金额
###### confirm业务
根据xid删除account_freeze表的冻结记录
###### cancel业务
1. 修改account_ freeze表,冻结金额为0，state为2
2. 修改account表，恢复可用金额
###### 如何判断是否空回滚
cancel业务中，根据xid查询account_freeze,如果为null则说明try还没做，需要空回滚
###### 如何避免是否业务悬挂
try业务中，根据xid查询account_freeze如果已经存在则证明Cancel已经执行，拒绝执行try业务
##### 代码实现
TCC的`Try`、`Confirm`、 `Cancel`方法都需要在接口中基于注解来声明，语法如下:
> name指定`try`方法、commitMethod指定`confirm`方法、rollbackMethod指定`cancel`方法
```java
public interface AccountTCCService {
    /**
     * 从用户账户中扣款
     *
     * @param userId 用户 ID
     * @param money  钱
     */
    @TwoPhaseBusinessAction(name = "deduct", commitMethod = "confirm", rollbackMethod = "cancel")
    void deduct(@BusinessActionContextParameter(paramName = "userId") String userId,
                @BusinessActionContextParameter(paramName = "money") int money);
    boolean confirm(BusinessActionContext context);
    boolean cancel(BusinessActionContext context);
}
```
```java
@Data
@TableName("account_freeze_tbl")
public class AccountFreeze {
    @TableId(type = IdType.INPUT)
    private String xid;
    private String userId;
    private Integer freezeMoney;
    private Integer state;
    public static abstract class State {
        public final static int TRY = 0;
        public final static int CONFIRM = 1;
        public final static int CANCEL = 2;
    }
}
```
```java
public interface AccountFreezeMapper extends BaseMapper<AccountFreeze> {
}
```
```java
@Slf4j
@Service
public class AccountTCCServiceImpl implements AccountTCCService {
    private final AccountMapper accountMapper;
    private final AccountFreezeMapper accountFreezeMapper;
    @Autowired
    public AccountTCCServiceImpl(AccountMapper accountMapper, AccountFreezeMapper accountFreezeMapper) {
        this.accountMapper = accountMapper;
        this.accountFreezeMapper = accountFreezeMapper;
    }
    /**
     * 扣除金额
     * TCC try方法
     *
     * @param userId 用户 ID
     * @param money  钱
     */
    @Transactional
    @Override
    public void deduct(String userId, int money) {
        // 获取事务id
        String xid = RootContext.getXID();
        // 数据库字段设置为unsigned int，所以不能为负数
        // 1.扣减金额
        accountMapper.deduct(userId,money);
        // 2.记录冻结金额、事务状态
        AccountFreeze accountFreeze = new AccountFreeze();
        accountFreeze.setUserId(userId);
        accountFreeze.setFreezeMoney(money);
        accountFreeze.setState(AccountFreeze.State.TRY);
        accountFreeze.setXid(xid);
        accountFreezeMapper.insert(accountFreeze);
    }
    /**
     * 确认
     * TCC confirm方法
     *
     * @param context 上下文
     * @return boolean
     */
    @Override
    public boolean confirm(BusinessActionContext context) {
        // 获取事务id
        String xid = context.getXid();
        // 删除了一条数据
        return accountFreezeMapper.deleteById(xid)==1;
    }
    /**
     * 取消
     * TCC cancel方法
     *
     * @param context 上下文
     * @return boolean
     */
    @Override
    public boolean cancel(BusinessActionContext context) {
        // 查询冻结记录
        String xid = context.getXid();
        AccountFreeze accountFreeze = accountFreezeMapper.selectById(xid);
        // 1.恢复可用余额
        accountMapper.refund(accountFreeze.getUserId(),accountFreeze.getFreezeMoney());
        // 2.将冻结金额清零，状态设置为CANCEL
        accountFreeze.setFreezeMoney(0);
        accountFreeze.setState(AccountFreeze.State.CANCEL);
        int count = accountFreezeMapper.updateById(accountFreeze);
        return count==1;
    }
}
```
> 以上是基本内容
>
> 接下来增加对于`空回滚`和`业务悬挂`的判断
```java
@Slf4j
@Service
public class AccountTCCServiceImpl implements AccountTCCService {
    private final AccountMapper accountMapper;
    private final AccountFreezeMapper accountFreezeMapper;
    @Autowired
    public AccountTCCServiceImpl(AccountMapper accountMapper, AccountFreezeMapper accountFreezeMapper) {
        this.accountMapper = accountMapper;
        this.accountFreezeMapper = accountFreezeMapper;
    }
    /**
     * 扣除金额
     * TCC try方法
     *
     * @param userId 用户 ID
     * @param money  钱
     */
    @Transactional
    @Override
    public void deduct(String userId, int money) {
        // 获取事务id
        String xid = RootContext.getXID();
        // 判断业务悬挂
        AccountFreeze selectedById = accountFreezeMapper.selectById(xid);)
        if(selectedById!=null){
            // cancel已经执行过
            return;
        }
        // 数据库字段设置为unsigned int，所以不能为负数
        // 1.扣减金额
        accountMapper.deduct(userId,money);
        // 2.记录冻结金额、事务状态
        AccountFreeze accountFreeze = new AccountFreeze();
        accountFreeze.setUserId(userId);
        accountFreeze.setFreezeMoney(money);
        accountFreeze.setState(AccountFreeze.State.TRY);
        accountFreeze.setXid(xid);
        accountFreezeMapper.insert(accountFreeze);
    }
    /**
     * 确认
     * TCC confirm方法
     *
     * @param context 上下文
     * @return boolean
     */
    @Override
    public boolean confirm(BusinessActionContext context) {
        // 获取事务id
        String xid = context.getXid();
        // 删除了一条数据，删除数据本来就是幂等性的
        return accountFreezeMapper.deleteById(xid)==1;
    }
    /**
     * 取消
     * TCC cancel方法
     *
     * @param context 上下文
     * @return boolean
     */
    @Override
    public boolean cancel(BusinessActionContext context) {
        // 查询冻结记录
        String xid = context.getXid();
        String userId = context.getActionContext("userId").toString();
        AccountFreeze accountFreeze = accountFreezeMapper.selectById(xid);
        // 空回滚的判断
        if(accountFreeze==null){
            // 记录空回滚的数据
            accountFreeze.setUserId(userId);
            accountFreeze.setFreezeMoney(0);
            accountFreeze.setState(AccountFreeze.State.CANCEL);
            accountFreeze.setXid(xid);
            accountFreezeMapper.insert(accountFreeze);
            return true;
        }
        // 幂等性判断
        if(accountFreeze.getState()==AccountFreeze.State.CANCEL){
            // 已经执行过cancel方法了，无需重复执行
            return true;
        }
        // 1.恢复可用余额
        accountMapper.refund(accountFreeze.getUserId(),accountFreeze.getFreezeMoney());
        // 2.将冻结金额清零，状态设置为CANCEL
        accountFreeze.setFreezeMoney(0);
        accountFreeze.setState(AccountFreeze.State.CANCEL);
        int count = accountFreezeMapper.updateById(accountFreeze);
        return count==1;
    }
}
```
### Saga模式
Saga模式是SEATA提供的长事务解决方案。也分为两个阶段:
1. 一阶段:直接提交本地事务
2. 二阶段:成功则什么都不做;失败则通过编写补偿业务来回滚
Saga模式的缺点是没有隔离性
Saga模式优点:
1. 事务参与者可以基于事件驱动实现异步调用，吞吐高
2. 一阶段直接提交事务，无锁，性能好
3. 不用编写TCC中的三个阶段，实现简单
缺点:
1. 软状态持续时间不确定，时效性差
2. 没有锁，没有事务隔离，会有脏写
### 模式对比
![image-20240413205630731](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240413205630731.png)
