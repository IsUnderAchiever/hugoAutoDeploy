---
title: Elasticsearct复习
description: Elasticsearct复习
date: 2024-04-01
slug: Elasticsearct复习
image: 202412212115512.png
categories:
    - Elasticsearch
---

# Elasticsearch
## 基础
> 以下具体内容参考自黑马程序员课程`微服务开发框架SpringCloud+RabbitMQ+Docker+Redis+搜索+分布式微服务全技术栈课程`以及大佬的[掘金博客](https://juejin.cn/post/7046759829255225351)
### 概念
> **文档**
>
> es的数据存储会被序列化成json的格式
> es是面向文档存储数据的，`文档`可以理解成一条条的数据
>
> **索引**
>
> 索引是指相同类型文档的集合，`索引`可以理解成表
>
> **映射**
>
> 文档与文档的结构会有些许差异，比如有的文档有`name`，有的有`age`
> `索引`中`文档`的字段约束信息类似于`表结构`约束
![概念对比](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/%E6%A6%82%E5%BF%B5%E5%AF%B9%E6%AF%94.png)
> es主要用于搜索，写入操作写入MySQL，可以将MySQL的数据同步到es
>
> **当使用分词搜索数据的时候，必须是通过分词器分析的数据才能搜索出来，否则无法搜索出数据**
#### **标准分词器**
```sql
POST /_analyze
{
  "analyzer":"standard",
  "text":"我爱中国"
}
```
> 分词结果如下
>
> `特点`是逐字一分，对中文不太友好
```json
{
  "tokens": [
    {
      "token": "我",
      "start_offset": 0,
      "end_offset": 1,
      "type": "<IDEOGRAPHIC>",
      "position": 0
    },
    {
      "token": "爱",
      "start_offset": 1,
      "end_offset": 2,
      "type": "<IDEOGRAPHIC>",
      "position": 1
    },
    {
      "token": "中",
      "start_offset": 2,
      "end_offset": 3,
      "type": "<IDEOGRAPHIC>",
      "position": 2
    },
    {
      "token": "国",
      "start_offset": 3,
      "end_offset": 4,
      "type": "<IDEOGRAPHIC>",
      "position": 3
    }
  ]
}
```
#### **ik分词器**
```sql
POST /_analyze
{
  "analyzer":"ik_max_word",
  "text":"我爱中国"
}
```
> 一般使用`ik分词器`和`pinyin分词器`对中文分词
```json
{
  "tokens": [
    {
      "token": "我",
      "start_offset": 0,
      "end_offset": 1,
      "type": "CN_CHAR",
      "position": 0
    },
    {
      "token": "爱",
      "start_offset": 1,
      "end_offset": 2,
      "type": "CN_CHAR",
      "position": 1
    },
    {
      "token": "中国",
      "start_offset": 2,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 2
    }
  ]
}
```
#### **pinyin分词器**
```sql
POST /_analyze
{
  "analyzer":"pinyin",
  "text":"我爱中国"
}
```
> `pinyin分词器`可以将中文分词成拼音，一般结合`ik分词器`一起使用
```json
{
  "tokens": [
    {
      "token": "wo",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 0
    },
    {
      "token": "wazg",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 0
    },
    {
      "token": "ai",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 1
    },
    {
      "token": "zhong",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 2
    },
    {
      "token": "guo",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 3
    }
  ]
}
```
#### 配置ik分词器结合pinyin分词器
```sql
# 新建索引库并指定自定义分词器
PUT /greatom
{
   "settings": {
        "analysis": {
            "analyzer": {
                "ik_smart_pinyin": {
                    "type": "custom",
                    "tokenizer": "ik_smart",
                    "filter": ["my_pinyin", "word_delimiter"]
                },
                "ik_max_word_pinyin": {
                    "type": "custom",
                    "tokenizer": "ik_max_word",
                    "filter": ["my_pinyin", "word_delimiter"]
                }
            },
            "filter": {
                "my_pinyin": {
                    "type" : "pinyin",
                    "keep_separate_first_letter" : true,
                    "keep_full_pinyin" : true,
                    "keep_original" : true,
                    "limit_first_letter_length" : 16,
                    "lowercase" : true,
                    "remove_duplicated_term" : true
                }
            }
        }
  }
}
```
> 执行结果
```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "greatom"
}
```
**ik分词器、pinyin分词器结合**
```sql
POST /greatom/_analyze
{
  "analyzer":"ik_smart_pinyin",
  "text":"我爱中国"
}
```
> 执行结果
```json
{
  "tokens": [
    {
      "token": "w",
      "start_offset": 0,
      "end_offset": 1,
      "type": "CN_CHAR",
      "position": 0
    },
    {
      "token": "wo",
      "start_offset": 0,
      "end_offset": 1,
      "type": "CN_CHAR",
      "position": 0
    },
    {
      "token": "我",
      "start_offset": 0,
      "end_offset": 1,
      "type": "CN_CHAR",
      "position": 0
    },
    {
      "token": "a",
      "start_offset": 1,
      "end_offset": 2,
      "type": "CN_CHAR",
      "position": 1
    },
    {
      "token": "ai",
      "start_offset": 1,
      "end_offset": 2,
      "type": "CN_CHAR",
      "position": 1
    },
    {
      "token": "爱",
      "start_offset": 1,
      "end_offset": 2,
      "type": "CN_CHAR",
      "position": 1
    },
    {
      "token": "z",
      "start_offset": 2,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 2
    },
    {
      "token": "zhong",
      "start_offset": 2,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 2
    },
    {
      "token": "g",
      "start_offset": 2,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 3
    },
    {
      "token": "guo",
      "start_offset": 2,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 3
    },
    {
      "token": "中国",
      "start_offset": 2,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 3
    },
    {
      "token": "zg",
      "start_offset": 2,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 3
    }
  ]
}
```
#### ik分词器拓展词库
> 配置文件在`config`下，为`IKAnalyzer.cfg.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict">ext.dict</entry>
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords">ext_stopwords.dict</entry>
	<!--用户可以在这里配置远程扩展字典 -->
	<!--<entry key="remote_ext_dict">words_location</entry> -->
	<!--用户可以在这里配置远程扩展停止词字典-->
	<!--<entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
```
在`config`下新建`ext.dict`文件和`ext_stopwords.dict`文件
> 可以利用这个功能实现一些最新的`流行词`或者`敏感词屏蔽`
![](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/typora-icon2.png)
![](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240326215129144.png)
> 需要重启生效，但是现在先不重启测试一下
```sql
POST /_analyze
{
  "analyzer":"ik_max_word",
  "text":"兄弟们，坚持了，奥里给"
}
```
> 执行结果
```json
{
  "tokens": [
    {
      "token": "兄弟们",
      "start_offset": 0,
      "end_offset": 3,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "兄弟",
      "start_offset": 0,
      "end_offset": 2,
      "type": "CN_WORD",
      "position": 1
    },
    {
      "token": "们",
      "start_offset": 2,
      "end_offset": 3,
      "type": "CN_CHAR",
      "position": 2
    },
    {
      "token": "坚持",
      "start_offset": 4,
      "end_offset": 6,
      "type": "CN_WORD",
      "position": 3
    },
    {
      "token": "了",
      "start_offset": 6,
      "end_offset": 7,
      "type": "CN_CHAR",
      "position": 4
    },
    {
      "token": "奥",
      "start_offset": 8,
      "end_offset": 9,
      "type": "CN_CHAR",
      "position": 5
    },
    {
      "token": "里",
      "start_offset": 9,
      "end_offset": 10,
      "type": "CN_CHAR",
      "position": 6
    },
    {
      "token": "给",
      "start_offset": 10,
      "end_offset": 11,
      "type": "CN_CHAR",
      "position": 7
    }
  ]
}
```
**重启es服务**
> 执行结果
```json
{
  "tokens": [
    {
      "token": "兄弟们",
      "start_offset": 0,
      "end_offset": 3,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "兄弟",
      "start_offset": 0,
      "end_offset": 2,
      "type": "CN_WORD",
      "position": 1
    },
    {
      "token": "们",
      "start_offset": 2,
      "end_offset": 3,
      "type": "CN_CHAR",
      "position": 2
    },
    {
      "token": "坚持",
      "start_offset": 4,
      "end_offset": 6,
      "type": "CN_WORD",
      "position": 3
    },
    {
      "token": "奥里给",
      "start_offset": 8,
      "end_offset": 11,
      "type": "CN_WORD",
      "position": 4
    }
  ]
}
```
### 索引库操作
> 常见的mapping属性
**type**字段类型属性
1. text 可分词的文本
2. keyword 精确值，比如国家、品牌、ip地址、email，只有整体才有意义
3. 数值 long、integer、short、byte、double、float
4. 布尔 boolean
5. 日期 date
6. 对象 object，如人名分为`姓`和`名`两部分
需要注意的是，es没有数组类型，但是支持某个类型多个值，比如`double`支持99.1或者[99.1,98.1]（这可不是数组）
**index**是否创建索引，默认true
**analyzer**使用哪种分词器
**properties**该字段的子字段
#### 创建索引库
```sql
PUT /tb_user
{
  "mappings":{
    "properties":{
      "name":{
        "type":"object",
        "properties":{
          "first_name":{
            "type":"keyword"
          },
          "last_name":{
            "type":"keyword"
          }
        }
      },
      "info":{
        "type":"text",
        "analyzer":"ik_smart"
      }
    }
  }
}
```
> 想使用`ik+pinyin分词`
```sql
PUT /tb_user
{
   "settings": {
        "analysis": {
            "analyzer": {
                "ik_smart_pinyin": {
                    "type": "custom",
                    "tokenizer": "ik_smart",
                    "filter": ["my_pinyin", "word_delimiter"]
                },
                "ik_max_word_pinyin": {
                    "type": "custom",
                    "tokenizer": "ik_max_word",
                    "filter": ["my_pinyin", "word_delimiter"]
                }
            },
            "filter": {
                "my_pinyin": {
                    "type" : "pinyin",
                    "keep_separate_first_letter" : true,
                    "keep_full_pinyin" : true,
                    "keep_original" : true,
                    "limit_first_letter_length" : 16,
                    "lowercase" : true,
                    "remove_duplicated_term" : true
                }
            }
        }
  },
  "mappings":{
    "properties":{
      "name":{
        "type":"object",
        "properties":{
          "first_name":{
            "type":"keyword"
          },
          "last_name":{
            "type":"keyword"
          }
        }
      },
      "info":{
        "type":"text",
        "analyzer":"ik_smart_pinyin"
      }
    }
  }
}
```
#### 更新索引库
> 索引库一旦创建无法修改`(原有的字段)`，但是可以新增新的字段
```sql
PUT /tb_user/_mapping
{
  "properties":{
    "age":{
      "type":"keyword",
      "index":false
    }
  }
}
```
#### 查看、删除索引库
```SQL
# 查看
GET /tb_user
# 删除
DELETE /tb_user
```
### 文档操作
#### 新增文档
**格式如下**
```sql
POST /索引库/_doc/文档id
{
	"字段1":"值1",
	"字段2":"值2",
	"字段3":{
		"字段4":"值4",
		"字段5":"值5",
	}
}
```
```sql
POST /tb_user/_doc/1
{
  "info":"小花的个人信息",
  "name":{
    "first_name":"小",
    "last_name":"花"
  }
}
```
#### 查询文档
```sql
GET /tb_user/_doc/1
```
#### 删除文档
```sql
DELETE /tb_user/_doc/1
```
#### 修改文档
> 修改文档有些特殊，有两种方法
1. 全量修改，先删除文档，然后新增（如果指定的`文档不存在`，则删除操作无效，但新增不受影响 => 新增）
`post`请求变为`put`
```sql
PUT /tb_user/_doc/1
{
  "info":"小花的个人信息",
  "name":{
    "first_name":"小",
    "last_name":"花"
  }
}
```
2. 增量修改
**格式如下**
```sql
POST /索引库/_update/文档id
{
  "doc":{
    "字段名":"新的值"
  }
}
```
> 例如
```sql
POST /tb_user/_update/1
{
  "doc":{
    "name":{
      "last_name":"花花"
    }
  }
}
```
### 练习
> 详细使用方法
>
> [`推介博客`](https://juejin.cn/post/7046759829255225351)
```sql
create table tb_hotel
(
    id        bigint       not null comment '酒店id'
        primary key,
    name      varchar(255) not null comment '酒店名称',
    address   varchar(255) not null comment '酒店地址',
    price     int(10)      not null comment '酒店价格',
    score     int(2)       not null comment '酒店评分',
    brand     varchar(32)  not null comment '酒店品牌',
    city      varchar(32)  not null comment '所在城市',
    star_name varchar(16)  null comment '酒店星级，1星到5星，1钻到5钻',
    business  varchar(255) null comment '商圈',
    latitude  varchar(32)  not null comment '纬度',
    longitude varchar(32)  not null comment '经度',
    pic       varchar(255) null comment '酒店图片'
)
    collate = utf8mb4_general_ci
    row_format = COMPACT;
```
[数据下载](https://wwm.lanzoue.com/iMmuu1sqipfe)
> 在es中，我不仅希望根据酒店名称能搜索，而且还希望可以和其他几个字段一同搜索
>
> `copy_to`的功能，将其他几个字段全部拷贝到一个字段
>
> 如下`all`同时具备了多个字段的搜索功能
```json
{
	...
	"all":{
        "type":"text",
        "analyzer":"ik_max_word"
    },
    "brand":{
        "type":"keyword",
        "copy_to":"all"
    }
	...
}
```
> 构建es索引库
```sql
# 酒店
PUT /hotel
{
  "mappings":{
    "properties":{
      "id":{
        "type":"keyword"
      },
      "name":{
        "type":"text",
        "analyzer":"ik_max_word",
        "copy_to":"all"
      },
      "address":{
        "type":"keyword"
      },
      "price":{
        "type":"integer"
      },
      "score":{
        "type":"integer"
      },
      "brand":{
        "type":"keyword",
        "copy_to":"all"
      },
      "city":{
        "type":"keyword"
      },
      "starName":{
        "type":"keyword"
      },
      "bussiness":{
        "type":"keyword",
        "copy_to":"all"
      },
      "location":{
        "type":"geo_point"
      },
      "pic":{
        "type":"keyword",
        "index":"false"
      },
      "all":{
        "type":"text",
        "analyzer":"ik_max_word"
      }
    }
  }
}
```
#### 创建项目
##### 配置
```xml
    <properties>
        <java.version>17</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>3.0.2</spring-boot.version>
    </properties>
    <dependencies>
        <!--FastJson-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.71</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>8.3.0</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.5</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
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
```
```java
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;
@Data
@Slf4j
@Configuration
@ConfigurationProperties(prefix = "elasticsearch")
public class ElasticSearchConfig {
    /**
     * 协议
     */
    private String schema;
    /**
     * 集群地址，如果有多个用“,”隔开
     */
    private String address;
    /**
     * 连接超时时间
     */
    private int connectTimeout;
    /**
     * Socket 连接超时时间
     */
    private int socketTimeout;
    /**
     * 获取连接的超时时间
     */
    private int connectionRequestTimeout;
    /**
     * 最大连接数
     */
    private int maxConnectNum;
    /**
     * 最大路由连接数
     */
    private int maxConnectPerRoute;
    /**
     * 连接ES的用户名
     */
    private String username;
    /**
     * 数据查询的索引
     */
    private String index;
    /**
     * 密码
     */
    private String passwd;
}
```
```properties
# 应用名称
spring.application.name=demo
# 应用服务 WEB 访问端口
server.port=8080
# 查看es日志
spring.jpa.show-sql=true
logging.level.tracer=TRACE
elasticsearch.schema=http
elasticsearch.address=localhost:9200
elasticsearch.connectTimeout=10000
elasticsearch.socketTimeout=15000
elasticsearch.connectionRequestTimeout=20000
elasticsearch.maxConnectNum=100
elasticsearch.maxConnectPerRoute=100
elasticsearch.index="aha"
# 数据库驱动：
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 数据源名称
spring.datasource.name=defaultDataSource
# 数据库连接地址
spring.datasource.url=jdbc:mysql://localhost:3306/es?serverTimezone=UTC
# 数据库用户名&密码：
spring.datasource.username=root
spring.datasource.password=123456
# 配置mybatis-plus 打印sql日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
# xml文件路径
mybatis-plus.mapper-locations=classpath:/mapper/**/*.xml
# 配置mybatis-plus 包路径
mybatis-plus.type-aliases-package=com.example.domain
# mybatis-plus下划线转驼峰配置，默认为true
mybatis-plus.configuration.map-underscore-to-camel-case=true
# 配置全局默认主键类型，实体类不用加@TableId(value ="id",type = IdType.AUTO)
mybatis-plus.global-config.db-config.id-type=auto
```
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.json.jackson.JacksonJsonpMapper;
import co.elastic.clients.transport.ElasticsearchTransport;
import co.elastic.clients.transport.rest_client.RestClientTransport;
import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestClientBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.ArrayList;
import java.util.List;
/**
 * es java client
 */
@Configuration
public class ElasticsearchClientConfig {
    private final ElasticSearchConfig elasticSearchConfig;
    public ElasticsearchClientConfig (ElasticSearchConfig elasticSearchConfig) {
        this.elasticSearchConfig = elasticSearchConfig;
    }
    @Bean
    public RestClient restClient() {
        // 拆分地址
        List<HttpHost> hostLists = new ArrayList<>();
        String[] hostArray = elasticSearchConfig.getAddress().split(",");
        for (String temp : hostArray) {
            String host = temp.split(":")[0];
            String port = temp.split(":")[1];
            hostLists.add(new HttpHost(host, Integer.parseInt(port), elasticSearchConfig.getSchema()));
        }
        // 转换成 HttpHost 数组
        HttpHost[] httpHost = hostLists.toArray(new HttpHost[]{});
        // 构建连接对象
        RestClientBuilder builder = RestClient.builder(httpHost);
        // 异步连接延时配置
        builder.setRequestConfigCallback(requestConfigBuilder -> {
            requestConfigBuilder.setConnectTimeout(elasticSearchConfig.getConnectTimeout());
            requestConfigBuilder.setSocketTimeout(elasticSearchConfig.getSocketTimeout());
            requestConfigBuilder.setConnectionRequestTimeout(elasticSearchConfig.getConnectionRequestTimeout());
            return requestConfigBuilder;
        });
        // 异步连接数配置
        builder.setHttpClientConfigCallback(httpClientBuilder -> {
            httpClientBuilder.setMaxConnTotal(elasticSearchConfig.getMaxConnectNum());
            httpClientBuilder.setMaxConnPerRoute(elasticSearchConfig.getMaxConnectPerRoute());
            return httpClientBuilder;
        });
        return builder.build();
    }
    @Bean
    public ElasticsearchTransport elasticsearchTransport (RestClient restClient) {
        return new RestClientTransport(
                restClient, new JacksonJsonpMapper());
    }
    @Bean
    public ElasticsearchClient elasticsearchClient (ElasticsearchTransport transport) {
        return new ElasticsearchClient(transport);
    }
}
```
```java
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;
import java.io.Serializable;
/**
 * 
 * @TableName tb_hotel
 */
@TableName(value ="tb_hotel")
@Data
public class Hotel implements Serializable {
    /**
     * 酒店id
     */
    @TableId
    private Long id;
    /**
     * 酒店名称
     */
    private String name;
    /**
     * 酒店地址
     */
    private String address;
    /**
     * 酒店价格
     */
    private Integer price;
    /**
     * 酒店评分
     */
    private Integer score;
    /**
     * 酒店品牌
     */
    private String brand;
    /**
     * 所在城市
     */
    private String city;
    /**
     * 酒店星级，1星到5星，1钻到5钻
     */
    private String starName;
    /**
     * 商圈
     */
    private String business;
    /**
     * 纬度
     */
    private String latitude;
    /**
     * 经度
     */
    private String longitude;
    /**
     * 酒店图片
     */
    private String pic;
    @TableField(exist = false)
    private static final long serialVersionUID = 1L;
}
```
```java
import com.example.domain.Hotel;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;
import org.springframework.data.elasticsearch.annotations.GeoPointField;
import org.springframework.data.elasticsearch.core.geo.GeoPoint;
/**
 * @auther: 不是菜狗爱编程
 * @Date: 2024/03/27/7:45
 * @Description:
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
@Document(indexName = "hotel",createIndex = true)
public class HotelDoc {
    @Id
    @Field(type = FieldType.Keyword)
    private Long id;
    @Field(type = FieldType.Text)
    private String name;
    @Field(type = FieldType.Keyword)
    private String address;
    @Field(type = FieldType.Integer)
    private Integer price;
    @Field(type = FieldType.Integer)
    private Integer score;
    @Field(type = FieldType.Keyword)
    private String brand;
    @Field(type = FieldType.Keyword)
    private String city;
    @Field(type = FieldType.Keyword)
    private String starName;
    @Field(type = FieldType.Keyword)
    private String business;
    /**
     * 位置
     */
    @GeoPointField
    private GeoPoint location;
    @Field(type = FieldType.Keyword)
    private String pic;
    public HotelDoc(Hotel hotel) {
        this.id = hotel.getId();
        this.name = hotel.getName();
        this.address = hotel.getAddress();
        this.price = hotel.getPrice();
        this.score = hotel.getScore();
        this.brand = hotel.getBrand();
        this.city = hotel.getCity();
        this.starName = hotel.getStarName();
        this.business = hotel.getBusiness();
        this.location=new GeoPoint(Double.parseDouble(hotel.getLatitude()),Double.parseDouble(hotel.getLongitude()));
        this.pic = hotel.getPic();
    }
}
```
##### 操作索引
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.mapping.Property;
import co.elastic.clients.elasticsearch.indices.CreateIndexResponse;
import co.elastic.clients.elasticsearch.indices.DeleteIndexResponse;
import co.elastic.clients.elasticsearch.indices.GetIndexResponse;
import co.elastic.clients.transport.endpoints.BooleanResponse;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
import java.util.Map;
/**
 * 索引操作
 */
@Slf4j
@SpringBootTest
class IndexTest {
    @Autowired
    private ElasticsearchClient elasticsearchClient;
    /**
     * 创建索引 不使用mapping
     */
    @Test
    void createIndexWithoutMappingTest() throws IOException {
        CreateIndexResponse createIndexResponse = elasticsearchClient.indices()
                .create(createIndexRequest -> createIndexRequest.index("elasticsearch-client"));
        log.info("== {} 索引创建是否成功: {}", "elasticsearch-client", createIndexResponse.acknowledged());
    }
    /**
     * 创建索引 使用mapping
     */
    @Test
    void createIndexWithMappingTest() throws IOException {
        CreateIndexResponse createIndexResponse = elasticsearchClient.indices()
                .create(createIndexRequest -> createIndexRequest.index("elasticsearch-client")
                        .mappings(typeMapping -> typeMapping.properties("name",
                                        objectBuild -> objectBuild.text(
                                                textProperty -> textProperty.index(true)))
                                .properties("age",objectBuild -> objectBuild.integer(
                                        textProperty -> textProperty.index(false)))
                        )
                );
        log.info("== {} 索引创建是否成功: {}", "elasticsearch-client", createIndexResponse.acknowledged());
    }
    /**
     * 删除索引
     */
    @Test
    void deleteIndexTest() throws IOException {
        DeleteIndexResponse deleteIndexResponse = elasticsearchClient.indices()
                .delete(deleteIndexRequest -> deleteIndexRequest.index("elasticsearch-client"));
        log.info("== {} 索引删除是否成功: {}", "elasticsearch-client", deleteIndexResponse.acknowledged());
    }
    /**
     * 判断索引是否存在
     */
    @Test
    void checkIndexExistTest() throws IOException {
        BooleanResponse exists = elasticsearchClient.indices()
                .exists(existRequest -> existRequest.index("elasticsearch-client"));
        log.info("== {} 索引是否存在: {}", "elasticsearch-client", exists.value()?"存在":"不存在");
    }
    /**
     * 查询索引详细信息
     */
    @Test
    void getIndexTest() throws IOException {
        GetIndexResponse getIndexResponse = elasticsearchClient.indices()
                .get(getIndexRequest -> getIndexRequest.index("elasticsearch-client"));
        Map<String, Property> properties = getIndexResponse.get("elasticsearch-client").mappings().properties();
        for (String key : properties.keySet()) {
            log.info("== {} 索引的详细信息为: == key: {}, Property: {}", "elasticsearch-client", key, properties.get(key)._kind());
        }
    }
}
```
##### 文档操作
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.*;
import co.elastic.clients.elasticsearch.core.bulk.BulkOperation;
import com.example.domain.Hotel;
import com.example.domain.doc.HotelDoc;
import com.example.service.HotelService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
/**
 * 文档操作
 */
@Slf4j
@SpringBootTest
class DocTest {
    private static final String INDEX_NAME = "hotel";
    @Autowired
    private ElasticsearchClient elasticsearchClient;
    @Autowired
    private HotelService hotelService;
    /**
     * 添加文档
     * GET /hotel/_doc/61083
     */
    @Test
    void addDocTest() throws IOException {
        Hotel hotel = hotelService.getById(61083);
        HotelDoc hotelDoc = new HotelDoc(hotel);
        IndexResponse index = elasticsearchClient.index(
                indexRequest -> indexRequest.index(INDEX_NAME).id(hotelDoc.getId().toString()).document(hotelDoc));
        log.info("== response: {}, responseStatus: {}", index, index.result());
    }
    /**
     * 获取文档信息
     */
    @Test
    public void getDocTest () throws IOException {
        GetResponse<HotelDoc> getResponse = elasticsearchClient.get(getRequest ->
                getRequest.index(INDEX_NAME).id("61083"), HotelDoc.class
        );
        log.info("== document source: {}, response: {}", getResponse.source(), getResponse);
    }
    /**
     * 更新文档信息
     */
    @Test
    public void updateDocTest () throws IOException {
        Hotel hotel = hotelService.getById(61083);
        HotelDoc hotelDoc = new HotelDoc(hotel);
        hotelDoc.setName(hotelDoc.getName()+"(更新后)");
        UpdateResponse<HotelDoc> updateResponse = elasticsearchClient.update(updateRequest ->
                        updateRequest.index(INDEX_NAME).id("61083").doc(hotelDoc), HotelDoc.class
        );
        log.info("== response: {}, responseStatus: {}", updateResponse, updateResponse.result());
    }
    /**
     * 删除文档信息
     */
    @Test
    public void deleteDocTest () throws IOException {
        DeleteResponse deleteResponse = elasticsearchClient.delete(deleteRequest ->
                deleteRequest.index(INDEX_NAME).id("1")
        );
        log.info("== response: {}, result:{}", deleteResponse, deleteResponse.result());
    }
    /**
     * 批量插入文档
     * GET /hotel/_doc/36934
     * GET /hotel/_doc/38609
     */
    @Test
    public void batchInsertTest () throws IOException {
        List<Hotel> hotels = hotelService.list();
        List<BulkOperation> bulkOperationList = new ArrayList<>();
        for (Hotel hotel : hotels) {
            HotelDoc hotelDoc = new HotelDoc(hotel);
            bulkOperationList.add(new BulkOperation.Builder().create(e -> e.document(hotelDoc).id(hotel.getId().toString())).build());
        }
        BulkResponse bulkResponse = elasticsearchClient.bulk(bulkRequest ->
                bulkRequest.index(INDEX_NAME).operations(bulkOperationList)
        );
        // 这边插入成功的话显示的是 false
        log.info("== errors: {}", bulkResponse.errors());
    }
}
```
## DSL语法
1. 查询所有:查询出所有数据，一般测试用。例如: match_all
2. 全文检索(full text)查询:利用分词器对用户输入内容分词，然后去倒排索引库中匹配。例如:match_query、multi_match_query
3. 精确查询:根据精确词条值查找数据，一般是查找keyword、数值、日期、boolean等类型字段。例如:ids、range、term
4. 地理(geo)查询:根据经纬度查询。例如:geo_distance、geo_bounding_box
5. 复合(compound)查询:复合查询可以将上述各种查询条件组合起来，合并查询条件。例如:bool、function_score
### 查询所有
#### match_all
> 基本查询语法
```sql
GET /indexName/_search
{
	"query":{
		"查询类型":{
			"查询条件":"条件值"
		}
	}
}
# 例如
GET /indexName/_search
{
	"query":{
		"match_all":{
		}
	}
}
```
### 全文检索查询
> `全文检索查询`会对用户输入内容分词，常用于`搜索框查询`
#### match
```sql
GET /indexName/_search
{
	"query":{
		"match":{
			"FIELD":"TEXT"
		}
	}
}
# 例如
# 这个all字段在前文的copy_to那里
GET /hotel/_search
{
	"query":{
		"match":{
			"all":"上海五"
		}
	}
}
```
> 查询结果
```json
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 83,
      "relation": "eq"
    },
    "max_score": 4.671594,
    "hits": [
      {
        "_index": "hotel",
        "_id": "1557997004",
        "_score": 4.671594,
        "_source": {
          "id": 1557997004,
          "name": "上海五角场凯悦酒店",
          "address": "国定东路88号",
          "price": 1104,
          "score": 46,
          "brand": "凯悦",
          "city": "上海",
          "starName": "五钻",
          "business": "江湾/五角场商业区",
          "location": {
            "lat": 31.300645,
            "lon": 121.51918
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/3a3Zz9cDgbJEEJ1GcXzKhTh21YqK_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "56977",
        "_score": 4.1990623,
        "_source": {
          "id": 56977,
          "name": "上海五角场华美达大酒店",
          "address": "黄兴路1888号",
          "price": 499,
          "score": 40,
          "brand": "华美达",
          "city": "上海",
          "starName": "三钻",
          "business": "江湾/五角场商业区",
          "location": {
            "lat": 31.292932,
            "lon": 121.519759
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/26VREqAQdaGFvJdAJALVtjxcNMpL_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "39141",
        "_score": 3.700049,
        "_source": {
          "id": 39141,
          "name": "7天连锁酒店(上海五角场复旦同济大学店)",
          "address": "杨浦国权路315号",
          "price": 349,
          "score": 38,
          "brand": "7天酒店",
          "city": "上海",
          "starName": "二钻",
          "business": "江湾、五角场商业区",
          "location": {
            "lat": 31.290057,
            "lon": 121.508804
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G2/M00/C7/E3/Cii-T1knFXCIJzNYAAFB8-uFNAEAAKYkQPcw1IAAUIL012_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "2048671293",
        "_score": 3.644754,
        "_source": {
          "id": 2048671293,
          "name": "汉庭酒店(深圳观澜五和大道店)",
          "address": "观湖街道五和大道327号",
          "price": 234,
          "score": 43,
          "brand": "汉庭",
          "city": "深圳",
          "starName": "二钻",
          "business": "观澜",
          "location": {
            "lat": 22.684459,
            "lon": 114.07708
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2JrQi83S9qgDEkXqWpe5iyi44Uh2_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "339777429",
        "_score": 1.1486411,
        "_source": {
          "id": 339777429,
          "name": "上海嘉定喜来登酒店",
          "address": "菊园新区嘉唐公路66号",
          "price": 1286,
          "score": 44,
          "brand": "喜来登",
          "city": "上海",
          "starName": "五钻",
          "business": "嘉定新城",
          "location": {
            "lat": 31.394595,
            "lon": 121.245773
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2v2fKuo5bzhunSBC1n1E42cLTkZV_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "2022598930",
        "_score": 1.095608,
        "_source": {
          "id": 2022598930,
          "name": "上海宝华喜来登酒店",
          "address": "南奉公路3111弄228号",
          "price": 2899,
          "score": 46,
          "brand": "喜来登",
          "city": "上海",
          "starName": "五钻",
          "business": "奉贤开发区",
          "location": {
            "lat": 30.921659,
            "lon": 121.575572
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/45/BD/Cii-TF3ZaBmIStrbAASnoOyg7FoAAFpYwEoz9oABKe4992_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "46829",
        "_score": 1.0472558,
        "_source": {
          "id": 46829,
          "name": "上海浦西万怡酒店",
          "address": "恒丰路338号",
          "price": 726,
          "score": 46,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "上海火车站地区",
          "location": {
            "lat": 31.242977,
            "lon": 121.455864
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/x87VCoyaR8cTuYFZmKHe8VC6Wk1_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "644417",
        "_score": 1.0472558,
        "_source": {
          "id": 644417,
          "name": "上海外高桥喜来登酒店",
          "address": "自由贸易试验区基隆路28号（二号门内）",
          "price": 2419,
          "score": 46,
          "brand": "喜来登",
          "city": "上海",
          "starName": "五钻",
          "business": "浦东外高桥地区",
          "location": {
            "lat": 31.350989,
            "lon": 121.588751
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/1Rrtg9n7PdMEivVDhsehbJBrEre_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "1463484295",
        "_score": 1.0472558,
        "_source": {
          "id": 1463484295,
          "name": "上海和平豪生酒店",
          "address": "沪南公路2653-2号",
          "price": 650,
          "score": 41,
          "brand": "豪生",
          "city": "上海",
          "starName": "四钻",
          "business": "周浦康桥地区",
          "location": {
            "lat": 31.146478,
            "lon": 121.568218
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/ZxM9gWHqj657ndRsHw4j4p3CQ5k_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "1880614409",
        "_score": 1.0472558,
        "_source": {
          "id": 1880614409,
          "name": "上海崇明由由喜来登酒店",
          "address": "揽海路2888号",
          "price": 2198,
          "score": 45,
          "brand": "喜来登",
          "city": "上海",
          "starName": "五钻",
          "business": "崇明岛/长兴岛/横沙岛",
          "location": {
            "lat": 31.462167,
            "lon": 121.823103
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/21gDCGgRT3xFqCd3FxBh633j6Qsu_w200_h200_c1_t0.jpg"
        }
      }
    ]
  }
}
```
#### multi_match
> `multi_match`与match查询类似，允许同时查询多个字段
>
> 需要注意：参与搜索的字段越多，效率越低，建议使用`copy_to`将多个字段拷贝到一个字段中，采用`match`查询
```sql
GET /hotel/_search
{
	"query":{
		"multi_match":{
			"query":"TEXT1",
			"fields": ["FIELD1","FIELD2"]
		}
	}
}
# 例如
GET /hotel/_search
{
	"query":{
		"multi_match":{
			"query":"上海酒店",
			"fields": ["city","name"]
		}
	}
}
```
> 查询结果
```json
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 200,
      "relation": "eq"
    },
    "max_score": 1.1692147,
    "hits": [
      {
        "_index": "hotel",
        "_id": "339777429",
        "_score": 1.1692147,
        "_source": {
          "id": 339777429,
          "name": "上海嘉定喜来登酒店",
          "address": "菊园新区嘉唐公路66号",
          "price": 1286,
          "score": 44,
          "brand": "喜来登",
          "city": "上海",
          "starName": "五钻",
          "business": "嘉定新城",
          "location": {
            "lat": 31.394595,
            "lon": 121.245773
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2v2fKuo5bzhunSBC1n1E42cLTkZV_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "46829",
        "_score": 1.1032845,
        "_source": {
          "id": 46829,
          "name": "上海浦西万怡酒店",
          "address": "恒丰路338号",
          "price": 726,
          "score": 46,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "上海火车站地区",
          "location": {
            "lat": 31.242977,
            "lon": 121.455864
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/x87VCoyaR8cTuYFZmKHe8VC6Wk1_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "60223",
        "_score": 1.1032845,
        "_source": {
          "id": 60223,
          "name": "上海希尔顿酒店",
          "address": "静安华山路250号",
          "price": 2688,
          "score": 37,
          "brand": "希尔顿",
          "city": "上海",
          "starName": "五星级",
          "business": "静安寺地区",
          "location": {
            "lat": 31.219306,
            "lon": 121.445427
          },
          "pic": "https://m.tuniucdn.com/filebroker/cdn/res/92/10/9210e74442aceceaf6e196d61fc3b6b1_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "1463484295",
        "_score": 1.1032845,
        "_source": {
          "id": 1463484295,
          "name": "上海和平豪生酒店",
          "address": "沪南公路2653-2号",
          "price": 650,
          "score": 41,
          "brand": "豪生",
          "city": "上海",
          "starName": "四钻",
          "business": "周浦康桥地区",
          "location": {
            "lat": 31.146478,
            "lon": 121.568218
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/ZxM9gWHqj657ndRsHw4j4p3CQ5k_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "1942992995",
        "_score": 1.1032845,
        "_source": {
          "id": 1942992995,
          "name": "上海嘉定凯悦酒店",
          "address": "裕民南路1366号",
          "price": 758,
          "score": 46,
          "brand": "凯悦",
          "city": "上海",
          "starName": "五钻",
          "business": "嘉定新城",
          "location": {
            "lat": 31.352298,
            "lon": 121.263314
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/53/2D/Cii-U13edkqIfZhLAAJEW25WIF4AAGVxQIg38sAAkRz517_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "1996823660",
        "_score": 1.1032845,
        "_source": {
          "id": 1996823660,
          "name": "上海紫竹万怡酒店",
          "address": "紫星路588号3幢",
          "price": 642,
          "score": 46,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "交大/闵行经济开发区",
          "location": {
            "lat": 31.02118,
            "lon": 121.465186
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/53/2F/Cii-TF3edraIPzK9AAH_p8vdHKoAAGV3AJgSVEAAf-_019_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "2022598930",
        "_score": 1.1032845,
        "_source": {
          "id": 2022598930,
          "name": "上海宝华喜来登酒店",
          "address": "南奉公路3111弄228号",
          "price": 2899,
          "score": 46,
          "brand": "喜来登",
          "city": "上海",
          "starName": "五钻",
          "business": "奉贤开发区",
          "location": {
            "lat": 30.921659,
            "lon": 121.575572
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/45/BD/Cii-TF3ZaBmIStrbAASnoOyg7FoAAFpYwEoz9oABKe4992_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "56201",
        "_score": 1.0443928,
        "_source": {
          "id": 56201,
          "name": "上海齐鲁万怡大酒店",
          "address": "东方路838号",
          "price": 873,
          "score": 44,
          "brand": "万怡",
          "city": "上海",
          "starName": "四星级",
          "business": "浦东陆家嘴金融贸易区",
          "location": {
            "lat": 31.226031,
            "lon": 121.525801
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/52/B6/Cii-TF3eXKeIJeN7AASiKHbTtx4AAGRegDSBzMABKJA111_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "56227",
        "_score": 1.0443928,
        "_source": {
          "id": 56227,
          "name": "上海圣淘沙万怡酒店",
          "address": "南桥镇南桥路1号",
          "price": 899,
          "score": 45,
          "brand": "万怡",
          "city": "上海",
          "starName": "四星级",
          "business": "奉贤开发区",
          "location": {
            "lat": 30.910917,
            "lon": 121.456525
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/52/B9/Cii-U13eXSiIdJjXAARSA6FywFYAAGRnwHvy1AABFIb158_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "56852",
        "_score": 1.0443928,
        "_source": {
          "id": 56852,
          "name": "上海财大豪生大酒店",
          "address": "武东路188号",
          "price": 592,
          "score": 46,
          "brand": "豪生",
          "city": "上海",
          "starName": "五钻",
          "business": "江湾/五角场商业区",
          "location": {
            "lat": 31.304182,
            "lon": 121.492936
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2jGHezLZvPZqC9cBGesbP5vAhCXi_w200_h200_c1_t0.jpg"
        }
      }
    ]
  }
}
```
### 精确查询
> 一般是查找keyword、数值、日期、boolean等类型字段，`不做分词`
#### term
```sql
GET /indexName/_search
{
	"query":{
		"term":{
			"FIELD":{
				"value":"VALUE"
			}
		}
	}
}
# 例如
GET /hotel/_search
{
  "query":{
    "term":{
      "starName":{
        "value":"四钻"
      }
    }
  }
}
```
> 查询结果
```json
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 28,
      "relation": "eq"
    },
    "max_score": 1.9583635,
    "hits": [
      {
        "_index": "hotel",
        "_id": "45845",
        "_score": 1.9583635,
        "_source": {
          "id": 45845,
          "name": "上海西藏大厦万怡酒店",
          "address": "虹桥路100号",
          "price": 589,
          "score": 45,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "徐家汇地区",
          "location": {
            "lat": 31.192714,
            "lon": 121.434717
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/48GNb9GZpJDCejVAcQHYWwYyU8T_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "46829",
        "_score": 1.9583635,
        "_source": {
          "id": 46829,
          "name": "上海浦西万怡酒店",
          "address": "恒丰路338号",
          "price": 726,
          "score": 46,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "上海火车站地区",
          "location": {
            "lat": 31.242977,
            "lon": 121.455864
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/x87VCoyaR8cTuYFZmKHe8VC6Wk1_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "47066",
        "_score": 1.9583635,
        "_source": {
          "id": 47066,
          "name": "上海浦东东站华美达酒店",
          "address": "施新路958号",
          "price": 408,
          "score": 46,
          "brand": "华美达",
          "city": "上海",
          "starName": "四钻",
          "business": "浦东机场核心区",
          "location": {
            "lat": 31.147989,
            "lon": 121.759199
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2pNujAVaQbXACzkHp8bQMm6zqwhp_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "56912",
        "_score": 1.9583635,
        "_source": {
          "id": 56912,
          "name": "上海华凯华美达广场酒店",
          "address": "月华路9号",
          "price": 747,
          "score": 40,
          "brand": "华美达",
          "city": "上海",
          "starName": "四钻",
          "business": "奉贤开发区",
          "location": {
            "lat": 30.814382,
            "lon": 121.464521
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/45iaCNCuZavJTxwTLskhVKzwynLD_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "60522",
        "_score": 1.9583635,
        "_source": {
          "id": 60522,
          "name": "上海嘉豪淮海国际豪生酒店",
          "address": "汾阳路1号",
          "price": 425,
          "score": 45,
          "brand": "豪生",
          "city": "上海",
          "starName": "四钻",
          "business": "淮海路/新天地地区",
          "location": {
            "lat": 31.215497,
            "lon": 121.456297
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/38UBi4QYuaF8jN94CxQ7tb7tjtmZ_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "60916",
        "_score": 1.9583635,
        "_source": {
          "id": 60916,
          "name": "上海绿地万怡酒店",
          "address": "沪宜公路3101号",
          "price": 328,
          "score": 45,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "嘉定新城",
          "location": {
            "lat": 31.368523,
            "lon": 121.258567
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/3VLwG9tTQQnp3M3MTeMTdx9nas9B_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "395815",
        "_score": 1.9583635,
        "_source": {
          "id": 395815,
          "name": "北京明豪华美达酒店",
          "address": "天竺镇府前一街13号",
          "price": 558,
          "score": 46,
          "brand": "华美达",
          "city": "北京",
          "starName": "四钻",
          "business": "首都机场/新国展地区",
          "location": {
            "lat": 40.062832,
            "lon": 116.580678
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/52/13/Cii-U13eP2mIKCwvAAODTZXT-fAAAGKVAA9taIAA4Nl245_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "598591",
        "_score": 1.9583635,
        "_source": {
          "id": 598591,
          "name": "上海丽昂豪生大酒店",
          "address": "金新路99号",
          "price": 529,
          "score": 47,
          "brand": "豪生",
          "city": "上海",
          "starName": "四钻",
          "business": "浦东金桥地区",
          "location": {
            "lat": 31.252496,
            "lon": 121.600085
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2KfPPyPx9rWyVXif2CUuxv61Nryc_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "609372",
        "_score": 1.9583635,
        "_source": {
          "id": 609372,
          "name": "豪派特华美达广场酒店(深圳北站店)",
          "address": "民治街道梅龙路与民旺路交汇处",
          "price": 498,
          "score": 45,
          "brand": "华美达",
          "city": "深圳",
          "starName": "四钻",
          "business": "深圳北站地区",
          "location": {
            "lat": 22.620501,
            "lon": 114.033874
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/3G5TnUCPbjGYHAVWfvuixw8bs69t_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "629023",
        "_score": 1.9583635,
        "_source": {
          "id": 629023,
          "name": "和颐酒店(北京十里河欢乐谷店)",
          "address": "十八里店乡周家庄288号",
          "price": 390,
          "score": 47,
          "brand": "和颐",
          "city": "北京",
          "starName": "四钻",
          "business": "劲松/潘家园地区",
          "location": {
            "lat": 39.853354,
            "lon": 116.483437
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/28hnDdqn5uzuzCKYkw2x4pYmunXM_w200_h200_c1_t0.jpg"
        }
      }
    ]
  }
}
```
#### range
> 一般查询某个范围内的值
>
> 如下查找`"大于等于10"且"小于等于20"`的值
>
> `gte` 大于等于
> `gt`  大于
> `lte` 小于等于
> `lt`  小于
```sql
GET /indexName/_search
{
	"query":{
		"range":{
			"FIELD":{
				"gte":10,
				"lte":20
			}
		}
	}
}
# 例如
GET /hotel/_search
{
	"query":{
		"range":{
			"price":{
				"gte":100,
				"lte":200
			}
		}
	}
}
```
> 查询结果
```json
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 17,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "hotel",
        "_id": "485775",
        "_score": 1,
        "_source": {
          "id": 485775,
          "name": "如家酒店(上海闵行华东师范大学吴泾店)",
          "address": "吴泾镇宝秀路977号",
          "price": 161,
          "score": 45,
          "brand": "如家",
          "city": "上海",
          "starName": "二钻",
          "business": "交大/闵行经济开发区",
          "location": {
            "lat": 31.047135,
            "lon": 121.46224
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/V8pz15CkiMX5xYJRmbbp5zkKWJ8_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "517915",
        "_score": 1,
        "_source": {
          "id": 517915,
          "name": "如家酒店·neo(深圳草埔地铁站店)",
          "address": "布吉路1036号",
          "price": 159,
          "score": 44,
          "brand": "如家",
          "city": "深圳",
          "starName": "二钻",
          "business": "田贝/水贝珠宝城",
          "location": {
            "lat": 22.583191,
            "lon": 114.118499
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/228vhBCQmFRFWQBYX1cgoFQb6x58_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "541619",
        "_score": 1,
        "_source": {
          "id": 541619,
          "name": "如家酒店(上海莘庄地铁站龙之梦商业广场店)",
          "address": "莘庄镇莘浜路172号",
          "price": 149,
          "score": 44,
          "brand": "如家",
          "city": "上海",
          "starName": "二钻",
          "business": "莘庄工业区",
          "location": {
            "lat": 31.105797,
            "lon": 121.37755
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/3mKs3jETvJDj3dDdkRB9UyLLvPna_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "608374",
        "_score": 1,
        "_source": {
          "id": 608374,
          "name": "如家酒店(上海浦东机场龙东大道合庆店)",
          "address": "东川公路5863号",
          "price": 160,
          "score": 45,
          "brand": "如家",
          "city": "上海",
          "starName": "二钻",
          "business": "浦东机场核心区",
          "location": {
            "lat": 31.237662,
            "lon": 121.718556
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/LUYxGGV4pzjKeN5a69K4deU8JD8_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "728180",
        "_score": 1,
        "_source": {
          "id": 728180,
          "name": "如家酒店(深圳宝安西乡地铁站店)",
          "address": "西乡大道298-7号（富通城二期公交站旁）",
          "price": 184,
          "score": 43,
          "brand": "如家",
          "city": "深圳",
          "starName": "二钻",
          "business": "宝安体育中心商圈",
          "location": {
            "lat": 22.569693,
            "lon": 113.860186
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/FHdugqgUgYLPMoC4u4rdTbAPrVF_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "728415",
        "_score": 1,
        "_source": {
          "id": 728415,
          "name": "如家酒店·neo(深圳东门步行街晒布地铁站店)",
          "address": "晒布路67号",
          "price": 152,
          "score": 46,
          "brand": "如家",
          "city": "深圳",
          "starName": "二钻",
          "business": "东门商业区",
          "location": {
            "lat": 22.550183,
            "lon": 114.120771
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/25/57/Cii-U13PFNWISSnQAAEpTtoilsQAAEVWgEvur8AASlm647_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "728604",
        "_score": 1,
        "_source": {
          "id": 728604,
          "name": "如家酒店·neo(深圳南山地铁站南山市场店)",
          "address": "南新路顺富街18号化州大厦",
          "price": 198,
          "score": 43,
          "brand": "如家",
          "city": "深圳",
          "starName": "二钻",
          "business": "科技园",
          "location": {
            "lat": 22.525561,
            "lon": 113.920058
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/25/57/Cii-TF3PFLmIDGWiAAPHkaNTuOIAAEVVQBGazAAA8ep611_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "2316304",
        "_score": 1,
        "_source": {
          "id": 2316304,
          "name": "如家酒店(深圳双龙地铁站店)",
          "address": "龙岗街道龙岗墟社区龙平东路62号",
          "price": 135,
          "score": 45,
          "brand": "如家",
          "city": "深圳",
          "starName": "二钻",
          "business": "龙岗中心区/大运新城",
          "location": {
            "lat": 22.730828,
            "lon": 114.278337
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/4AzEoQ44awd1D2g95a6XDtJf3dkw_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "5873072",
        "_score": 1,
        "_source": {
          "id": 5873072,
          "name": "速8酒店（上海火车站北广场店）",
          "address": "闸北芷江西路796号",
          "price": 190,
          "score": 41,
          "brand": "速8",
          "city": "上海",
          "starName": "二钻",
          "business": "上海火车站地区",
          "location": {
            "lat": 31.255579,
            "lon": 121.452903
          },
          "pic": "https://m2.tuniucdn.com/filebroker/cdn/res/96/6d/966d6596e6cb7b48c9cc1d7da79b57c8_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "197837109",
        "_score": 1,
        "_source": {
          "id": 197837109,
          "name": "如家酒店·neo(深圳龙岗大道布吉地铁站店)",
          "address": "布吉镇深惠路龙珠商城",
          "price": 127,
          "score": 43,
          "brand": "如家",
          "city": "深圳",
          "starName": "二钻",
          "business": "布吉/深圳东站",
          "location": {
            "lat": 22.602482,
            "lon": 114.123284
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/25/58/Cii-TF3PFZOIA7jwAAKInGFN4xgAAEVbAGeP4AAAoi0485_w200_h200_c1_t0.jpg"
        }
      }
    ]
  }
}
```
### 地理查询
> lat指`纬度`、lon指`经度`
#### geo_bounding_box
> 查询`geo_point`值落在某个矩形范围内的所有文档
![image-20240328073926045](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240328073926045.png)
```sql
GET /indexName/_search
{
	"query":{
		"geo_bounding_box":{
			"FIELD":{
				"top_left":{
					"lat":31.1,
					"lon":121.5
				},
				"bottom_right":{
					"lat":30.9,
					"lon":121.7
				}
			}
		}
	}
}
# 例如
GET /hotel/_search
{
	"query":{
		"geo_bounding_box":{
			"location":{
				"top_left":{
					"lat":31.1,
					"lon":121.5
				},
				"bottom_right":{
					"lat":30.9,
					"lon":121.7
				}
			}
		}
	}
}
```
> 查询结果
```json
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "hotel",
        "_id": "2022598930",
        "_score": 1,
        "_source": {
          "id": 2022598930,
          "name": "上海宝华喜来登酒店",
          "address": "南奉公路3111弄228号",
          "price": 2899,
          "score": 46,
          "brand": "喜来登",
          "city": "上海",
          "starName": "五钻",
          "business": "奉贤开发区",
          "location": {
            "lat": 30.921659,
            "lon": 121.575572
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/45/BD/Cii-TF3ZaBmIStrbAASnoOyg7FoAAFpYwEoz9oABKe4992_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "2056298828",
        "_score": 1,
        "_source": {
          "id": 2056298828,
          "name": "上海中优城市万豪酒店",
          "address": "沪南公路7688弄1号",
          "price": 1200,
          "score": 45,
          "brand": "万豪",
          "city": "上海",
          "starName": "五钻",
          "business": "南汇/野生动物园",
          "location": {
            "lat": 31.030053,
            "lon": 121.662943
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2gBATEyysyQWmw3wZL863HGdqjaq_w200_h200_c1_t0.jpg"
        }
      }
    ]
  }
}
```
#### geo_distance
> 查询到指定中心点小于某个距离值的所有文档
![image-20240328074328371](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240328074328371.png)
```sql
GET /indexName/_search
{
	"query":{
		"geo_distance":{
			"distance":"15km",
			"FIELD":"31.21,121.5"
		}
	}
}
# 例如
GET /hotel/_search
{
	"query":{
		"geo_distance":{
			"distance":"15km",
			"location":"31.21,121.5"
		}
	}
}
```
> 查询结果
```json
{
  "took": 31,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 47,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "hotel",
        "_id": "36934",
        "_score": 1,
        "_source": {
          "id": 36934,
          "name": "7天连锁酒店(上海宝山路地铁站店)",
          "address": "静安交通路40号",
          "price": 336,
          "score": 37,
          "brand": "7天酒店",
          "city": "上海",
          "starName": "二钻",
          "business": "四川北路商业区",
          "location": {
            "lat": 31.251433,
            "lon": 121.47522
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G1/M00/3E/40/Cii9EVkyLrKIXo1vAAHgrxo_pUcAALcKQLD688AAeDH564_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "38609",
        "_score": 1,
        "_source": {
          "id": 38609,
          "name": "速8酒店(上海赤峰路店)",
          "address": "广灵二路126号",
          "price": 249,
          "score": 35,
          "brand": "速8",
          "city": "上海",
          "starName": "二钻",
          "business": "四川北路商业区",
          "location": {
            "lat": 31.282444,
            "lon": 121.479385
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G2/M00/DF/96/Cii-TFkx0ImIQZeiAAITil0LM7cAALCYwKXHQ4AAhOi377_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "38665",
        "_score": 1,
        "_source": {
          "id": 38665,
          "name": "速8酒店上海中山北路兰田路店",
          "address": "兰田路38号",
          "price": 226,
          "score": 35,
          "brand": "速8",
          "city": "上海",
          "starName": "二钻",
          "business": "长风公园地区",
          "location": {
            "lat": 31.244288,
            "lon": 121.422419
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G2/M00/EF/86/Cii-Tlk2mV2IMZ-_AAEucgG3dx4AALaawEjiycAAS6K083_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "38812",
        "_score": 1,
        "_source": {
          "id": 38812,
          "name": "7天连锁酒店(上海漕溪路地铁站店)",
          "address": "徐汇龙华西路315弄58号",
          "price": 298,
          "score": 37,
          "brand": "7天酒店",
          "city": "上海",
          "starName": "二钻",
          "business": "八万人体育场地区",
          "location": {
            "lat": 31.174377,
            "lon": 121.442875
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G2/M00/E0/0E/Cii-TlkyIr2IEWNoAAHQYv7i5CkAALD-QP2iJwAAdB6245_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "39141",
        "_score": 1,
        "_source": {
          "id": 39141,
          "name": "7天连锁酒店(上海五角场复旦同济大学店)",
          "address": "杨浦国权路315号",
          "price": 349,
          "score": 38,
          "brand": "7天酒店",
          "city": "上海",
          "starName": "二钻",
          "business": "江湾、五角场商业区",
          "location": {
            "lat": 31.290057,
            "lon": 121.508804
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G2/M00/C7/E3/Cii-T1knFXCIJzNYAAFB8-uFNAEAAKYkQPcw1IAAUIL012_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "45845",
        "_score": 1,
        "_source": {
          "id": 45845,
          "name": "上海西藏大厦万怡酒店",
          "address": "虹桥路100号",
          "price": 589,
          "score": 45,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "徐家汇地区",
          "location": {
            "lat": 31.192714,
            "lon": 121.434717
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/48GNb9GZpJDCejVAcQHYWwYyU8T_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "46829",
        "_score": 1,
        "_source": {
          "id": 46829,
          "name": "上海浦西万怡酒店",
          "address": "恒丰路338号",
          "price": 726,
          "score": 46,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "上海火车站地区",
          "location": {
            "lat": 31.242977,
            "lon": 121.455864
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/x87VCoyaR8cTuYFZmKHe8VC6Wk1_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "56201",
        "_score": 1,
        "_source": {
          "id": 56201,
          "name": "上海齐鲁万怡大酒店",
          "address": "东方路838号",
          "price": 873,
          "score": 44,
          "brand": "万怡",
          "city": "上海",
          "starName": "四星级",
          "business": "浦东陆家嘴金融贸易区",
          "location": {
            "lat": 31.226031,
            "lon": 121.525801
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/52/B6/Cii-TF3eXKeIJeN7AASiKHbTtx4AAGRegDSBzMABKJA111_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "56214",
        "_score": 1,
        "_source": {
          "id": 56214,
          "name": "上海浦东华美达大酒店",
          "address": "新金桥路18号",
          "price": 830,
          "score": 45,
          "brand": "华美达",
          "city": "上海",
          "starName": "四星级",
          "business": "浦东金桥地区",
          "location": {
            "lat": 31.244916,
            "lon": 121.590752
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/3jtXiuMKZEXJAuKuAkc47yLCjUBt_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "56392",
        "_score": 1,
        "_source": {
          "id": 56392,
          "name": "上海银星皇冠假日酒店",
          "address": "番禺路400号",
          "price": 809,
          "score": 47,
          "brand": "皇冠假日",
          "city": "上海",
          "starName": "五星级",
          "business": "徐家汇地区",
          "location": {
            "lat": 31.202768,
            "lon": 121.429524
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/37ucQ38K3UFdcRqntJ8M5dt884HR_w200_h200_c1_t0.jpg"
        }
      }
    ]
  }
}
```
### 复合查询
> 复合查询:复合查询可以将其它简单查询组合起来，实现更复杂的搜索逻辑，例如:
> fuction score:算分函数查询，可以控制文档相关性算分,控制文档排名。
#### function_score
![image-20240328075710185](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240328075710185.png)
```sql
GET /indexName/_search
{
	"query":{
		"function_score":{
			"query":{
              	"match":{
                    "all":"上海"
                }
			},
			"functions":[
                {
                	"filter":{
                		"term":{
                			"city":""
                		}
                	},
                	"weight":10
                }
            ],
            "boost_mode":"sum"
		}
	}
}
# 例如
GET /hotel/_search
{
  "query": {
    "function_score": {
      "query": {
        "match": {
          "all": "上海"
        }
      },
      "functions": [
        {
          "filter": {
            "term": {
              "city": "上海"
            }
          },
          "weight": 10
        }
      ],
      "boost_mode": "sum"
    }
  }
}
```
> 查询结果
```json
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 82,
      "relation": "eq"
    },
    "max_score": 11.148642,
    "hits": [
      {
        "_index": "hotel",
        "_id": "339777429",
        "_score": 11.148642,
        "_source": {
          "id": 339777429,
          "name": "上海嘉定喜来登酒店",
          "address": "菊园新区嘉唐公路66号",
          "price": 1286,
          "score": 44,
          "brand": "喜来登",
          "city": "上海",
          "starName": "五钻",
          "business": "嘉定新城",
          "location": {
            "lat": 31.394595,
            "lon": 121.245773
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2v2fKuo5bzhunSBC1n1E42cLTkZV_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "2022598930",
        "_score": 11.095608,
        "_source": {
          "id": 2022598930,
          "name": "上海宝华喜来登酒店",
          "address": "南奉公路3111弄228号",
          "price": 2899,
          "score": 46,
          "brand": "喜来登",
          "city": "上海",
          "starName": "五钻",
          "business": "奉贤开发区",
          "location": {
            "lat": 30.921659,
            "lon": 121.575572
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/45/BD/Cii-TF3ZaBmIStrbAASnoOyg7FoAAFpYwEoz9oABKe4992_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "46829",
        "_score": 11.0472555,
        "_source": {
          "id": 46829,
          "name": "上海浦西万怡酒店",
          "address": "恒丰路338号",
          "price": 726,
          "score": 46,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "上海火车站地区",
          "location": {
            "lat": 31.242977,
            "lon": 121.455864
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/x87VCoyaR8cTuYFZmKHe8VC6Wk1_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "644417",
        "_score": 11.0472555,
        "_source": {
          "id": 644417,
          "name": "上海外高桥喜来登酒店",
          "address": "自由贸易试验区基隆路28号（二号门内）",
          "price": 2419,
          "score": 46,
          "brand": "喜来登",
          "city": "上海",
          "starName": "五钻",
          "business": "浦东外高桥地区",
          "location": {
            "lat": 31.350989,
            "lon": 121.588751
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/1Rrtg9n7PdMEivVDhsehbJBrEre_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "1463484295",
        "_score": 11.0472555,
        "_source": {
          "id": 1463484295,
          "name": "上海和平豪生酒店",
          "address": "沪南公路2653-2号",
          "price": 650,
          "score": 41,
          "brand": "豪生",
          "city": "上海",
          "starName": "四钻",
          "business": "周浦康桥地区",
          "location": {
            "lat": 31.146478,
            "lon": 121.568218
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/ZxM9gWHqj657ndRsHw4j4p3CQ5k_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "1880614409",
        "_score": 11.0472555,
        "_source": {
          "id": 1880614409,
          "name": "上海崇明由由喜来登酒店",
          "address": "揽海路2888号",
          "price": 2198,
          "score": 45,
          "brand": "喜来登",
          "city": "上海",
          "starName": "五钻",
          "business": "崇明岛/长兴岛/横沙岛",
          "location": {
            "lat": 31.462167,
            "lon": 121.823103
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/21gDCGgRT3xFqCd3FxBh633j6Qsu_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "1942992995",
        "_score": 11.0472555,
        "_source": {
          "id": 1942992995,
          "name": "上海嘉定凯悦酒店",
          "address": "裕民南路1366号",
          "price": 758,
          "score": 46,
          "brand": "凯悦",
          "city": "上海",
          "starName": "五钻",
          "business": "嘉定新城",
          "location": {
            "lat": 31.352298,
            "lon": 121.263314
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/53/2D/Cii-U13edkqIfZhLAAJEW25WIF4AAGVxQIg38sAAkRz517_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "1996823660",
        "_score": 11.0472555,
        "_source": {
          "id": 1996823660,
          "name": "上海紫竹万怡酒店",
          "address": "紫星路588号3幢",
          "price": 642,
          "score": 46,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "交大/闵行经济开发区",
          "location": {
            "lat": 31.02118,
            "lon": 121.465186
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/53/2F/Cii-TF3edraIPzK9AAH_p8vdHKoAAGV3AJgSVEAAf-_019_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "56201",
        "_score": 11.002991,
        "_source": {
          "id": 56201,
          "name": "上海齐鲁万怡大酒店",
          "address": "东方路838号",
          "price": 873,
          "score": 44,
          "brand": "万怡",
          "city": "上海",
          "starName": "四星级",
          "business": "浦东陆家嘴金融贸易区",
          "location": {
            "lat": 31.226031,
            "lon": 121.525801
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/52/B6/Cii-TF3eXKeIJeN7AASiKHbTtx4AAGRegDSBzMABKJA111_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "56227",
        "_score": 11.002991,
        "_source": {
          "id": 56227,
          "name": "上海圣淘沙万怡酒店",
          "address": "南桥镇南桥路1号",
          "price": 899,
          "score": 45,
          "brand": "万怡",
          "city": "上海",
          "starName": "四星级",
          "business": "奉贤开发区",
          "location": {
            "lat": 30.910917,
            "lon": 121.456525
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/52/B9/Cii-U13eXSiIdJjXAARSA6FywFYAAGRnwHvy1AABFIb158_w200_h200_c1_t0.jpg"
        }
      }
    ]
  }
}
```
#### bool
布尔查询是一个或多个查询子句的组合。子查询的组合方式有:
1. must:必须匹配每个子查询，类似“与”
2. should:选择性匹配子查询，类似“或”
3. must_not:必须不匹配，`不参与算分`，类似“非”
4. filter:必须匹配，`不参与算分`
```sql
GET /hotel/_search
{
	"query":{
		"bool":{
			"must":{
				"term":{
					"city":"上海"
				}
			},
			"should":{
				"match":{
					"name":"上海"
				}
			},
			"must_not":{
				"range":{
					"price":{
						"gt":500
					}
				}
			},
			"filter":{
				"range":{
					"price":{
						"gte":100
					}
				}
			}
		}
	}
}
```
> 执行结果
```json
{
  "took": 8,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 30,
      "relation": "eq"
    },
    "max_score": 1.9134885,
    "hits": [
      {
        "_index": "hotel",
        "_id": "60916",
        "_score": 1.9134885,
        "_source": {
          "id": 60916,
          "name": "上海绿地万怡酒店",
          "address": "沪宜公路3101号",
          "price": 328,
          "score": 45,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "嘉定新城",
          "location": {
            "lat": 31.368523,
            "lon": 121.258567
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/3VLwG9tTQQnp3M3MTeMTdx9nas9B_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "38609",
        "_score": 1.861291,
        "_source": {
          "id": 38609,
          "name": "速8酒店(上海赤峰路店)",
          "address": "广灵二路126号",
          "price": 249,
          "score": 35,
          "brand": "速8",
          "city": "上海",
          "starName": "二钻",
          "business": "四川北路商业区",
          "location": {
            "lat": 31.282444,
            "lon": 121.479385
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G2/M00/DF/96/Cii-TFkx0ImIQZeiAAITil0LM7cAALCYwKXHQ4AAhOi377_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "1649956165",
        "_score": 1.861291,
        "_source": {
          "id": 1649956165,
          "name": "上海南青华美达酒店",
          "address": "华夏东路811号",
          "price": 299,
          "score": 47,
          "brand": "华美达",
          "city": "上海",
          "starName": "四钻",
          "business": "迪士尼度假区",
          "location": {
            "lat": 31.195206,
            "lon": 121.664791
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2RHmQgTpte3UVSDJ5KbqobbZGRnE_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "233036941",
        "_score": 1.8141286,
        "_source": {
          "id": 233036941,
          "name": "7天连锁酒店(上海东林寺店)",
          "address": "朱泾镇文商路79号",
          "price": 218,
          "score": 37,
          "brand": "7天酒店",
          "city": "上海",
          "starName": "二钻",
          "business": "金山枫泾古镇地区",
          "location": {
            "lat": 30.895912,
            "lon": 121.160238
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G4/M00/35/13/Cii_J1zr5PyIY3acAAFCnHJPxLUAAGX-ABvcIMAAUK0087_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "47066",
        "_score": 1.7713062,
        "_source": {
          "id": 47066,
          "name": "上海浦东东站华美达酒店",
          "address": "施新路958号",
          "price": 408,
          "score": 46,
          "brand": "华美达",
          "city": "上海",
          "starName": "四钻",
          "business": "浦东机场核心区",
          "location": {
            "lat": 31.147989,
            "lon": 121.759199
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2pNujAVaQbXACzkHp8bQMm6zqwhp_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "60522",
        "_score": 1.7713062,
        "_source": {
          "id": 60522,
          "name": "上海嘉豪淮海国际豪生酒店",
          "address": "汾阳路1号",
          "price": 425,
          "score": 45,
          "brand": "豪生",
          "city": "上海",
          "starName": "四钻",
          "business": "淮海路/新天地地区",
          "location": {
            "lat": 31.215497,
            "lon": 121.456297
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/38UBi4QYuaF8jN94CxQ7tb7tjtmZ_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "629729",
        "_score": 1.7713062,
        "_source": {
          "id": 629729,
          "name": "7天连锁酒店(上海张江高科园区店)",
          "address": "浦东新区蔡伦路103号",
          "price": 267,
          "score": 36,
          "brand": "7天酒店",
          "city": "上海",
          "starName": "二钻",
          "business": "浦东张江地区",
          "location": {
            "lat": 31.196154,
            "lon": 121.62071
          },
          "pic": "https://m2.tuniucdn.com/filebroker/cdn/res/d9/61/d961508a10865b9b29c033064f31b913_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "47478",
        "_score": 1.732251,
        "_source": {
          "id": 47478,
          "name": "速8酒店(上海松江中心店)",
          "address": "松江荣乐东路677号",
          "price": 428,
          "score": 35,
          "brand": "速8",
          "city": "上海",
          "starName": "二钻",
          "business": "佘山、松江大学城",
          "location": {
            "lat": 31.016712,
            "lon": 121.261606
          },
          "pic": "https://m.tuniucdn.com/filebroker/cdn/res/07/36/073662e1718fccefb7130a9da44ddf5c_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "5873072",
        "_score": 1.732251,
        "_source": {
          "id": 5873072,
          "name": "速8酒店（上海火车站北广场店）",
          "address": "闸北芷江西路796号",
          "price": 190,
          "score": 41,
          "brand": "速8",
          "city": "上海",
          "starName": "二钻",
          "business": "上海火车站地区",
          "location": {
            "lat": 31.255579,
            "lon": 121.452903
          },
          "pic": "https://m2.tuniucdn.com/filebroker/cdn/res/96/6d/966d6596e6cb7b48c9cc1d7da79b57c8_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "368343863",
        "_score": 1.732251,
        "_source": {
          "id": 368343863,
          "name": "如家酒店(上海金桥博兴路地铁站店)",
          "address": "博兴路1119号",
          "price": 218,
          "score": 45,
          "brand": "如家",
          "city": "上海",
          "starName": "二钻",
          "business": "浦东金桥地区",
          "location": {
            "lat": 31.266272,
            "lon": 121.593829
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/w5ERtGJEmdgdgy5qtLPatR1xfm4_w200_h200_c1_t0.jpg"
        }
      }
    ]
  }
}
```
#### 练习
> 搜索名字包含“如家”，价格不高于400,在坐标31.21,121.5周围10km范围内的酒店。
```sql
GET /hotel/_search
{
  "query": {
    "bool": {
      "must":{
        "match":{
          "name":"如家"
        }
      },
      "must_not": {
        "range": {
          "price": {
            "gt": 400
          }
        }
      },
      "filter": {
        "geo_distance": {
          "distance": "10km",
          "location": "31.21,121.5"
        }
      }
    }
  }
}
```
> 执行结果
```json
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": 1.716568,
    "hits": [
      {
        "_index": "hotel",
        "_id": "433576",
        "_score": 1.716568,
        "_source": {
          "id": 433576,
          "name": "如家酒店(上海南京路步行街店)",
          "address": "南京东路480号保安坊内",
          "price": 379,
          "score": 44,
          "brand": "如家",
          "city": "上海",
          "starName": "二钻",
          "business": "人民广场地区",
          "location": {
            "lat": 31.236454,
            "lon": 121.480948
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/52/BA/Cii-U13eXVaIQmdaAAWxgzdXXxEAAGRrgNIOkoABbGb143_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "434082",
        "_score": 1.4689932,
        "_source": {
          "id": 434082,
          "name": "如家酒店·neo(上海外滩城隍庙小南门地铁站店)",
          "address": "复兴东路260号",
          "price": 392,
          "score": 44,
          "brand": "如家",
          "city": "上海",
          "starName": "二钻",
          "business": "豫园地区",
          "location": {
            "lat": 31.220706,
            "lon": 121.498769
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/52/B6/Cii-U13eXLGIdHFzAAIG-5cEwDEAAGRfQNNIV0AAgcT627_w200_h200_c1_t0.jpg"
        }
      },
      {
        "_index": "hotel",
        "_id": "1584362548",
        "_score": 1.4178693,
        "_source": {
          "id": 1584362548,
          "name": "如家酒店(上海浦东国际旅游度假区御桥地铁站店)",
          "address": "御青路315-317号",
          "price": 339,
          "score": 44,
          "brand": "如家",
          "city": "上海",
          "starName": "二钻",
          "business": "周浦康桥地区",
          "location": {
            "lat": 31.15719,
            "lon": 121.572392
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2ybd3wqdoBtBeKcPxmyso9y1hNXa_w200_h200_c1_t0.jpg"
        }
      }
    ]
  }
}
```
## 搜索结果处理
### 排序
> 默认根据相关度算分排序，可以排序字段类型有: keyword类型、数值类型、地理坐标类型(比如 距离最近的酒店)、日期类型等。
```sql
GET /indexName/_search
{
	"query":{
		"match_all":{}
	},
	"sort":{
		"FIELD":"desc"
	}
}
# 例如
# 查询名字里有上海的酒店，按照评分降序、价格升序排列
GET /hotel/_search
{
  "query": {
    "match": {
      "name": "上海"
    }
  },
  "sort": [
    {
      "score": "desc"
    },
    {
      "price": "asc"
    }
  ]
}
```
> 执行结果
```json
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 82,
      "relation": "eq"
    },
    "max_score": null,
    "hits": [
      {
        "_index": "hotel",
        "_id": "2056126831",
        "_score": null,
        "_source": {
          "id": 2056126831,
          "name": "上海虹桥金臣皇冠假日酒店",
          "address": "申长路630弄1-3 号",
          "price": 2488,
          "score": 48,
          "brand": "皇冠假日",
          "city": "上海",
          "starName": "五钻",
          "business": "虹桥机场/国家会展中心",
          "location": {
            "lat": 31.19036,
            "lon": 121.31535
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/PvFh4Vzc84xXhm5N41F6AqdAqyJ_w200_h200_c1_t0.jpg"
        },
        "sort": [
          48,
          2488
        ]
      },
      {
        "_index": "hotel",
        "_id": "1649956165",
        "_score": null,
        "_source": {
          "id": 1649956165,
          "name": "上海南青华美达酒店",
          "address": "华夏东路811号",
          "price": 299,
          "score": 47,
          "brand": "华美达",
          "city": "上海",
          "starName": "四钻",
          "business": "迪士尼度假区",
          "location": {
            "lat": 31.195206,
            "lon": 121.664791
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2RHmQgTpte3UVSDJ5KbqobbZGRnE_w200_h200_c1_t0.jpg"
        },
        "sort": [
          47,
          299
        ]
      },
      {
        "_index": "hotel",
        "_id": "598591",
        "_score": null,
        "_source": {
          "id": 598591,
          "name": "上海丽昂豪生大酒店",
          "address": "金新路99号",
          "price": 529,
          "score": 47,
          "brand": "豪生",
          "city": "上海",
          "starName": "四钻",
          "business": "浦东金桥地区",
          "location": {
            "lat": 31.252496,
            "lon": 121.600085
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2KfPPyPx9rWyVXif2CUuxv61Nryc_w200_h200_c1_t0.jpg"
        },
        "sort": [
          47,
          529
        ]
      },
      {
        "_index": "hotel",
        "_id": "56392",
        "_score": null,
        "_source": {
          "id": 56392,
          "name": "上海银星皇冠假日酒店",
          "address": "番禺路400号",
          "price": 809,
          "score": 47,
          "brand": "皇冠假日",
          "city": "上海",
          "starName": "五星级",
          "business": "徐家汇地区",
          "location": {
            "lat": 31.202768,
            "lon": 121.429524
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/37ucQ38K3UFdcRqntJ8M5dt884HR_w200_h200_c1_t0.jpg"
        },
        "sort": [
          47,
          809
        ]
      },
      {
        "_index": "hotel",
        "_id": "1913922369",
        "_score": null,
        "_source": {
          "id": 1913922369,
          "name": "上海中建万怡酒店",
          "address": "蟠文路333号",
          "price": 889,
          "score": 47,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "虹桥机场/国家会展中心",
          "location": {
            "lat": 31.185504,
            "lon": 121.287709
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/39Afm5Bxgd784eMeFB5DrcsPnhT_w200_h200_c1_t0.jpg"
        },
        "sort": [
          47,
          889
        ]
      },
      {
        "_index": "hotel",
        "_id": "648219",
        "_score": null,
        "_source": {
          "id": 648219,
          "name": "上海金桥红枫万豪酒店",
          "address": "新金桥路15号",
          "price": 891,
          "score": 47,
          "brand": "万豪",
          "city": "上海",
          "starName": "五钻",
          "business": "浦东金桥地区",
          "location": {
            "lat": 31.244061,
            "lon": 121.591153
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/52/B6/Cii-TF3eXKuIR_a0AAUx-Xd2JLQAAGRfACSpvUABTIR560_w200_h200_c1_t0.jpg"
        },
        "sort": [
          47,
          891
        ]
      },
      {
        "_index": "hotel",
        "_id": "5870456",
        "_score": null,
        "_source": {
          "id": 5870456,
          "name": "上海宝华万豪酒店",
          "address": "广中西路333号",
          "price": 922,
          "score": 47,
          "brand": "万豪",
          "city": "上海",
          "starName": "五钻",
          "business": "大宁国际商业区",
          "location": {
            "lat": 31.279371,
            "lon": 121.446327
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/52/BA/Cii-U13eXVqIZXDFAAUC_xbrQDAAAGRrwPRyOcABQMX057_w200_h200_c1_t0.jpg"
        },
        "sort": [
          47,
          922
        ]
      },
      {
        "_index": "hotel",
        "_id": "60398",
        "_score": null,
        "_source": {
          "id": 60398,
          "name": "上海复旦皇冠假日酒店",
          "address": "邯郸路199号",
          "price": 924,
          "score": 47,
          "brand": "皇冠假日",
          "city": "上海",
          "starName": "五星级",
          "business": "江湾/五角场商业区",
          "location": {
            "lat": 31.295382,
            "lon": 121.502537
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2H1Gk8LHaBWZfYvR6NYYcGTvACmL_w200_h200_c1_t0.jpg"
        },
        "sort": [
          47,
          924
        ]
      },
      {
        "_index": "hotel",
        "_id": "47066",
        "_score": null,
        "_source": {
          "id": 47066,
          "name": "上海浦东东站华美达酒店",
          "address": "施新路958号",
          "price": 408,
          "score": 46,
          "brand": "华美达",
          "city": "上海",
          "starName": "四钻",
          "business": "浦东机场核心区",
          "location": {
            "lat": 31.147989,
            "lon": 121.759199
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2pNujAVaQbXACzkHp8bQMm6zqwhp_w200_h200_c1_t0.jpg"
        },
        "sort": [
          46,
          408
        ]
      },
      {
        "_index": "hotel",
        "_id": "56852",
        "_score": null,
        "_source": {
          "id": 56852,
          "name": "上海财大豪生大酒店",
          "address": "武东路188号",
          "price": 592,
          "score": 46,
          "brand": "豪生",
          "city": "上海",
          "starName": "五钻",
          "business": "江湾/五角场商业区",
          "location": {
            "lat": 31.304182,
            "lon": 121.492936
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2jGHezLZvPZqC9cBGesbP5vAhCXi_w200_h200_c1_t0.jpg"
        },
        "sort": [
          46,
          592
        ]
      }
    ]
  }
}
```
### 练习
>实现对酒店数据按照到你的位置坐标的距离升序排序
>
>有一些可以在线获取经纬度的网站，如[网站1](https://www.lddgo.net/convert/position#:~:text=%E5%9C%A8%E7%BA%BF%E7%BB%8F%E7%BA%AC%E5%BA%A6%E6%9F%A5%E8%AF%A2%E5%AE%9A%E4%BD%8D%E5%B7%A5%E5%85%B7%EF%BC%8C%E6%94%AF%E6%8C%81%E4%BD%BF%E7%94%A8%E5%9C%A8%E7%BA%BF%E5%9C%B0%E5%9B%BE%E6%8B%BE%E5%8F%96%E7%BB%8F%E7%BA%AC%E5%BA%A6%E5%9D%90%E6%A0%87%EF%BC%8C%E6%88%96%E8%80%85%E9%80%9A%E8%BF%87%E5%9C%B0%E5%9D%80%E5%8F%8D%E6%9F%A5%E7%BB%8F%E7%BA%AC%E5%9D%90%E6%A0%87%E3%80%82%20%E6%9C%AC%E5%B7%A5%E5%85%B7%E6%94%AF%E6%8C%81%E7%9A%84%E5%9C%A8%E7%BA%BF%E5%9C%B0%E5%9B%BE%E6%9C%89%EF%BC%9A,%E7%99%BE%E5%BA%A6%E5%9C%B0%E5%9B%BE%E3%80%81%E9%AB%98%E5%BE%B7%E5%9C%B0%E5%9B%BE%E3%80%81%E8%85%BE%E8%AE%AF%E5%9C%B0%E5%9B%BE%E5%92%8C%E8%B0%B7%E6%AD%8C%E5%9C%B0%E5%9B%BE%E3%80%82%20%E7%99%BE%E5%BA%A6%E5%9C%B0%E5%9B%BE%E6%8B%BE%E5%8F%96%E7%9A%84%E5%9D%90%E6%A0%87%E4%B8%BABD09%E5%9D%90%E6%A0%87%E7%B3%BB%EF%BC%8C%E9%AB%98%E5%BE%B7%E5%9C%B0%E5%9B%BE%E5%92%8C%E8%85%BE%E8%AE%AF%E5%9C%B0%E5%9B%BE%E6%8B%BE%E5%8F%96%E7%9A%84%E4%B8%BA%E7%81%AB%E6%98%9F%E5%9D%90%E6%A0%87%E7%B3%BB%20%28GCJ02%E5%9D%90%E6%A0%87%E7%B3%BB%29%E3%80%82)
>
>`当前位置为 120.643542,31.258042`
```sql
GET /hotel/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "_geo_distance": {
        "location": {
          "lat": "31.258042",
          "lon": "120.643542"
        },
        "order": "asc",
        "unit": "km"
      }
    }
  ]
}
```
> 查询结果，可查看`sort`字段距离
```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 201,
      "relation": "eq"
    },
    "max_score": null,
    "hits": [
      {
        "_index": "hotel",
        "_id": "200215226",
        "_score": null,
        "_source": {
          "id": 200215226,
          "name": "上海颖奕皇冠假日酒店",
          "address": "博园路6555号",
          "price": 907,
          "score": 45,
          "brand": "皇冠假日",
          "city": "上海",
          "starName": "五钻",
          "business": "嘉定新城",
          "location": {
            "lat": 31.272533,
            "lon": 121.19179
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/3Uyfi2aBRETE1K5PChiLVZCwtDLF_w200_h200_c1_t0.jpg"
        },
        "sort": [
          52.13395383160579
        ]
      },
      {
        "_index": "hotel",
        "_id": "339777429",
        "_score": null,
        "_source": {
          "id": 339777429,
          "name": "上海嘉定喜来登酒店",
          "address": "菊园新区嘉唐公路66号",
          "price": 1286,
          "score": 44,
          "brand": "喜来登",
          "city": "上海",
          "starName": "五钻",
          "business": "嘉定新城",
          "location": {
            "lat": 31.394595,
            "lon": 121.245773
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2v2fKuo5bzhunSBC1n1E42cLTkZV_w200_h200_c1_t0.jpg"
        },
        "sort": [
          59.18378970615253
        ]
      },
      {
        "_index": "hotel",
        "_id": "2003479905",
        "_score": null,
        "_source": {
          "id": 2003479905,
          "name": "上海榕港万怡酒店",
          "address": "新松江路1277号",
          "price": 798,
          "score": 46,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "佘山/松江大学城",
          "location": {
            "lat": 31.038198,
            "lon": 121.210178
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2GM761BYH8k15qkNrJrja3cwfr2D_w200_h200_c1_t0.jpg"
        },
        "sort": [
          59.205785086643566
        ]
      },
      {
        "_index": "hotel",
        "_id": "60916",
        "_score": null,
        "_source": {
          "id": 60916,
          "name": "上海绿地万怡酒店",
          "address": "沪宜公路3101号",
          "price": 328,
          "score": 45,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "嘉定新城",
          "location": {
            "lat": 31.368523,
            "lon": 121.258567
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/3VLwG9tTQQnp3M3MTeMTdx9nas9B_w200_h200_c1_t0.jpg"
        },
        "sort": [
          59.703757750943225
        ]
      },
      {
        "_index": "hotel",
        "_id": "1942992995",
        "_score": null,
        "_source": {
          "id": 1942992995,
          "name": "上海嘉定凯悦酒店",
          "address": "裕民南路1366号",
          "price": 758,
          "score": 46,
          "brand": "凯悦",
          "city": "上海",
          "starName": "五钻",
          "business": "嘉定新城",
          "location": {
            "lat": 31.352298,
            "lon": 121.263314
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/53/2D/Cii-U13edkqIfZhLAAJEW25WIF4AAGVxQIg38sAAkRz517_w200_h200_c1_t0.jpg"
        },
        "sort": [
          59.807711496975394
        ]
      },
      {
        "_index": "hotel",
        "_id": "1725781423",
        "_score": null,
        "_source": {
          "id": 1725781423,
          "name": "上海三迪华美达酒店",
          "address": "广富林路600弄7号",
          "price": 690,
          "score": 43,
          "brand": "华美达",
          "city": "上海",
          "starName": "四钻",
          "business": "佘山/松江大学城",
          "location": {
            "lat": 31.058023,
            "lon": 121.246536
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/NoHym6tuKwVazxy33wRNTNuQWd2_w200_h200_c1_t0.jpg"
        },
        "sort": [
          61.53729226590151
        ]
      },
      {
        "_index": "hotel",
        "_id": "1913922369",
        "_score": null,
        "_source": {
          "id": 1913922369,
          "name": "上海中建万怡酒店",
          "address": "蟠文路333号",
          "price": 889,
          "score": 47,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "虹桥机场/国家会展中心",
          "location": {
            "lat": 31.185504,
            "lon": 121.287709
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/39Afm5Bxgd784eMeFB5DrcsPnhT_w200_h200_c1_t0.jpg"
        },
        "sort": [
          61.78277329609102
        ]
      },
      {
        "_index": "hotel",
        "_id": "233036941",
        "_score": null,
        "_source": {
          "id": 233036941,
          "name": "7天连锁酒店(上海东林寺店)",
          "address": "朱泾镇文商路79号",
          "price": 218,
          "score": 37,
          "brand": "7天酒店",
          "city": "上海",
          "starName": "二钻",
          "business": "金山枫泾古镇地区",
          "location": {
            "lat": 30.895912,
            "lon": 121.160238
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G4/M00/35/13/Cii_J1zr5PyIY3acAAFCnHJPxLUAAGX-ABvcIMAAUK0087_w200_h200_c1_t0.jpg"
        },
        "sort": [
          63.58330395384682
        ]
      },
      {
        "_index": "hotel",
        "_id": "2056126831",
        "_score": null,
        "_source": {
          "id": 2056126831,
          "name": "上海虹桥金臣皇冠假日酒店",
          "address": "申长路630弄1-3 号",
          "price": 2488,
          "score": 48,
          "brand": "皇冠假日",
          "city": "上海",
          "starName": "五钻",
          "business": "虹桥机场/国家会展中心",
          "location": {
            "lat": 31.19036,
            "lon": 121.31535
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/PvFh4Vzc84xXhm5N41F6AqdAqyJ_w200_h200_c1_t0.jpg"
        },
        "sort": [
          64.32253706416857
        ]
      },
      {
        "_index": "hotel",
        "_id": "47478",
        "_score": null,
        "_source": {
          "id": 47478,
          "name": "速8酒店(上海松江中心店)",
          "address": "松江荣乐东路677号",
          "price": 428,
          "score": 35,
          "brand": "速8",
          "city": "上海",
          "starName": "二钻",
          "business": "佘山、松江大学城",
          "location": {
            "lat": 31.016712,
            "lon": 121.261606
          },
          "pic": "https://m.tuniucdn.com/filebroker/cdn/res/07/36/073662e1718fccefb7130a9da44ddf5c_w200_h200_c1_t0.jpg"
        },
        "sort": [
          64.65591776725215
        ]
      }
    ]
  }
}
```
### 分页
> elasticsearch默认情况下只返回top10的数据。而如果要查询更多数据就需要修改分页参数了。
> elasticsearch中通过修改from、size参数来控制要返回的分页结果
```sql
GET /indexName/_search
{
	"query":{
		"match_all":{}
	},
	// 分页开始的位置，默认为0
	"from":990,
	// 期望获取的文档总数
	"size":10,
	"sort":{
		"FIELD":"desc"
	}
}
```
> 需要`注意`的是：如果配置es集群，此时需要查询前100条数据的情况，应该要去查找每个es分片的前100条，然后聚合这些结果，重新排序选取前100条
![image-20240329072648677](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240329072648677.png)
> 目前解决方案有两种
>
> search after: 分页时需要排序,原理是从上一次的排序值开始,查询下一页数据。官方推荐使用的方式。`适用于手机翻页`
> scroll: 原理将排序数据形成快照，保存在内存。官方已经不推荐使用。
from + size:
1. 优点:支持随机翻页
2. 缺点:深度分页问题，默认查询上限(from + size)是10000
3. 场景:百度、京东、谷歌、淘宝这样的随机翻页搜索
after search:
1. 优点:没有查询上限(单次查询的size不超过10000)
2. 缺点:只能向后逐页查询，不支持随机翻页
3. 场景:没有随机翻页需求的搜索，例如手机向下滚动翻页.
scroll:
1. 优点:没有查询上限(单次查询的size不超过10000)
2. 缺点:会有额外内存消耗，并且搜索结果是非实时的
3. 场景:海量数据的获取和迁移。从ES7.1开始不推荐,建议用aftersearch方案。
### 高亮
> 将用户搜索的关键字`突出显示`
>
> 原理是将搜索的关键字用标签标记，通过前端的css标记标签来显示高亮
>
> `注意`高亮显示的搜索不能使用`match_all`，因为需要对关键字高亮显示
```sql
GET /indexName/_search
{
	"query":{
		"match":{
			"FIELD":"TEXT"
		}
	},
	"highlight":{
		"fields":{
			// 指定高亮的字段
			"FIELD":{
				// 高亮字段的前置标签
				"pre_tags":"<em>",
				// 高亮字段的后置标签
				"post_tags":"</ems>"
			}
		}
	}
}
# 例如
GET /hotel/_search
{
	"query":{
		"match":{
			"all":"上海"
		}
	},
	"highlight":{
		"fields":{
			"name":{
			  // 默认情况下搜索的字段和高亮的字段需要一样
			  "require_field_match": "false", 
			  "pre_tags":"<em>",
				"post_tags":"</em>"
			}
		}
	}
}
```
> 执行结果请查看`highlight`
```json
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 82,
      "relation": "eq"
    },
    "max_score": 1.1486411,
    "hits": [
      {
        "_index": "hotel",
        "_id": "339777429",
        "_score": 1.1486411,
        "_source": {
          "id": 339777429,
          "name": "上海嘉定喜来登酒店",
          "address": "菊园新区嘉唐公路66号",
          "price": 1286,
          "score": 44,
          "brand": "喜来登",
          "city": "上海",
          "starName": "五钻",
          "business": "嘉定新城",
          "location": {
            "lat": 31.394595,
            "lon": 121.245773
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2v2fKuo5bzhunSBC1n1E42cLTkZV_w200_h200_c1_t0.jpg"
        },
        "highlight": {
          "name": [
            "<em>上海</em>嘉定喜来登酒店"
          ]
        }
      },
      {
        "_index": "hotel",
        "_id": "2022598930",
        "_score": 1.095608,
        "_source": {
          "id": 2022598930,
          "name": "上海宝华喜来登酒店",
          "address": "南奉公路3111弄228号",
          "price": 2899,
          "score": 46,
          "brand": "喜来登",
          "city": "上海",
          "starName": "五钻",
          "business": "奉贤开发区",
          "location": {
            "lat": 30.921659,
            "lon": 121.575572
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/45/BD/Cii-TF3ZaBmIStrbAASnoOyg7FoAAFpYwEoz9oABKe4992_w200_h200_c1_t0.jpg"
        },
        "highlight": {
          "name": [
            "<em>上海</em>宝华喜来登酒店"
          ]
        }
      },
      {
        "_index": "hotel",
        "_id": "46829",
        "_score": 1.0472558,
        "_source": {
          "id": 46829,
          "name": "上海浦西万怡酒店",
          "address": "恒丰路338号",
          "price": 726,
          "score": 46,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "上海火车站地区",
          "location": {
            "lat": 31.242977,
            "lon": 121.455864
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/x87VCoyaR8cTuYFZmKHe8VC6Wk1_w200_h200_c1_t0.jpg"
        },
        "highlight": {
          "name": [
            "<em>上海</em>浦西万怡酒店"
          ]
        }
      },
      {
        "_index": "hotel",
        "_id": "644417",
        "_score": 1.0472558,
        "_source": {
          "id": 644417,
          "name": "上海外高桥喜来登酒店",
          "address": "自由贸易试验区基隆路28号（二号门内）",
          "price": 2419,
          "score": 46,
          "brand": "喜来登",
          "city": "上海",
          "starName": "五钻",
          "business": "浦东外高桥地区",
          "location": {
            "lat": 31.350989,
            "lon": 121.588751
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/1Rrtg9n7PdMEivVDhsehbJBrEre_w200_h200_c1_t0.jpg"
        },
        "highlight": {
          "name": [
            "<em>上海</em>外高桥喜来登酒店"
          ]
        }
      },
      {
        "_index": "hotel",
        "_id": "1463484295",
        "_score": 1.0472558,
        "_source": {
          "id": 1463484295,
          "name": "上海和平豪生酒店",
          "address": "沪南公路2653-2号",
          "price": 650,
          "score": 41,
          "brand": "豪生",
          "city": "上海",
          "starName": "四钻",
          "business": "周浦康桥地区",
          "location": {
            "lat": 31.146478,
            "lon": 121.568218
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/ZxM9gWHqj657ndRsHw4j4p3CQ5k_w200_h200_c1_t0.jpg"
        },
        "highlight": {
          "name": [
            "<em>上海</em>和平豪生酒店"
          ]
        }
      },
      {
        "_index": "hotel",
        "_id": "1880614409",
        "_score": 1.0472558,
        "_source": {
          "id": 1880614409,
          "name": "上海崇明由由喜来登酒店",
          "address": "揽海路2888号",
          "price": 2198,
          "score": 45,
          "brand": "喜来登",
          "city": "上海",
          "starName": "五钻",
          "business": "崇明岛/长兴岛/横沙岛",
          "location": {
            "lat": 31.462167,
            "lon": 121.823103
          },
          "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/21gDCGgRT3xFqCd3FxBh633j6Qsu_w200_h200_c1_t0.jpg"
        },
        "highlight": {
          "name": [
            "<em>上海</em>崇明由由喜来登酒店"
          ]
        }
      },
      {
        "_index": "hotel",
        "_id": "1942992995",
        "_score": 1.0472558,
        "_source": {
          "id": 1942992995,
          "name": "上海嘉定凯悦酒店",
          "address": "裕民南路1366号",
          "price": 758,
          "score": 46,
          "brand": "凯悦",
          "city": "上海",
          "starName": "五钻",
          "business": "嘉定新城",
          "location": {
            "lat": 31.352298,
            "lon": 121.263314
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/53/2D/Cii-U13edkqIfZhLAAJEW25WIF4AAGVxQIg38sAAkRz517_w200_h200_c1_t0.jpg"
        },
        "highlight": {
          "name": [
            "<em>上海</em>嘉定凯悦酒店"
          ]
        }
      },
      {
        "_index": "hotel",
        "_id": "1996823660",
        "_score": 1.0472558,
        "_source": {
          "id": 1996823660,
          "name": "上海紫竹万怡酒店",
          "address": "紫星路588号3幢",
          "price": 642,
          "score": 46,
          "brand": "万怡",
          "city": "上海",
          "starName": "四钻",
          "business": "交大/闵行经济开发区",
          "location": {
            "lat": 31.02118,
            "lon": 121.465186
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/53/2F/Cii-TF3edraIPzK9AAH_p8vdHKoAAGV3AJgSVEAAf-_019_w200_h200_c1_t0.jpg"
        },
        "highlight": {
          "name": [
            "<em>上海</em>紫竹万怡酒店"
          ]
        }
      },
      {
        "_index": "hotel",
        "_id": "56201",
        "_score": 1.0029912,
        "_source": {
          "id": 56201,
          "name": "上海齐鲁万怡大酒店",
          "address": "东方路838号",
          "price": 873,
          "score": 44,
          "brand": "万怡",
          "city": "上海",
          "starName": "四星级",
          "business": "浦东陆家嘴金融贸易区",
          "location": {
            "lat": 31.226031,
            "lon": 121.525801
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/52/B6/Cii-TF3eXKeIJeN7AASiKHbTtx4AAGRegDSBzMABKJA111_w200_h200_c1_t0.jpg"
        },
        "highlight": {
          "name": [
            "<em>上海</em>齐鲁万怡大酒店"
          ]
        }
      },
      {
        "_index": "hotel",
        "_id": "56227",
        "_score": 1.0029912,
        "_source": {
          "id": 56227,
          "name": "上海圣淘沙万怡酒店",
          "address": "南桥镇南桥路1号",
          "price": 899,
          "score": 45,
          "brand": "万怡",
          "city": "上海",
          "starName": "四星级",
          "business": "奉贤开发区",
          "location": {
            "lat": 30.910917,
            "lon": 121.456525
          },
          "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/52/B9/Cii-U13eXSiIdJjXAARSA6FywFYAAGRnwHvy1AABFIb158_w200_h200_c1_t0.jpg"
        },
        "highlight": {
          "name": [
            "<em>上海</em>圣淘沙万怡酒店"
          ]
        }
      }
    ]
  }
}
```
## `ElasticsearchClient`查询文档
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.SortOrder;
import co.elastic.clients.elasticsearch._types.query_dsl.FunctionBoostMode;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import co.elastic.clients.elasticsearch.core.search.Hit;
import co.elastic.clients.json.JsonData;
import com.example.domain.doc.HotelDoc;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
/**
 * 查询语法
 *
 * @Auther: 不是菜狗爱编程
 * @Date: 2024/03/27/22:00
 * @Description:
 */
@Slf4j
@SpringBootTest
class MatchTest {
    private static final String INDEX_NAME = "hotel";
    @Autowired
    private ElasticsearchClient elasticsearchClient;
    /**
     * 根据 city 使用term查询获取相应的文档， search api 才是 elasticsearch-client 的优势，可以看出使用 lambda 大大简化了代码量，
     * 可以与 restHighLevelClient 形成鲜明的对比，但是也有可读性较差的问题，所以 lambda 的基础要扎实
     */
    @Test
    void testRestClient() throws IOException {
        SearchResponse<HotelDoc> search = elasticsearchClient.search(s -> s.index(INDEX_NAME)
                        .query(q ->
                                q.term(t ->
                                        t.field("city").value(v -> v.stringValue("上海"))
                                )
                        ),
                HotelDoc.class);
        for (Hit<HotelDoc> hit : search.hits().hits()) {
            log.info("== hit: source: {}, id: {}", hit.source(), hit.id());
        }
    }
    /**
     * match_all
     */
    @Test
    void matchAllTest() throws IOException {
        SearchResponse<HotelDoc> search = elasticsearchClient.search(s -> s.index(INDEX_NAME)
                        .query(q -> q.matchAll(matchAll -> matchAll))
                , HotelDoc.class);
        for (Hit<HotelDoc> hit : search.hits().hits()) {
            log.info("== hit: source: {}, id: {}", hit.source(), hit.id());
        }
    }
    /**
     * match
     */
    @Test
    void matchTest() throws IOException {
        SearchResponse<HotelDoc> search = elasticsearchClient.search(s -> s.index(INDEX_NAME)
                        .query(q -> q.match(matchQuery -> matchQuery.field("all").query("上海")))
                , HotelDoc.class);
        for (Hit<HotelDoc> hit : search.hits().hits()) {
            log.info("== hit: source: {}, id: {}", hit.source(), hit.id());
        }
    }
    /**
     * multi_match
     */
    @Test
    void multiMatchTest() throws IOException {
        SearchResponse<HotelDoc> search = elasticsearchClient.search(s -> s.index(INDEX_NAME)
                        .query(q -> q.multiMatch(multiMatchQuery -> multiMatchQuery.fields("city", "name").query("上海")))
                , HotelDoc.class);
        for (Hit<HotelDoc> hit : search.hits().hits()) {
            log.info("== hit: source: {}, id: {}", hit.source(), hit.id());
        }
    }
    /**
     * term
     */
    @Test
    void termTest() throws IOException {
        SearchResponse<HotelDoc> search = elasticsearchClient.search(s -> s.index(INDEX_NAME)
                        .query(q ->
                                q.term(t ->
                                        t.field("city").value(v -> v.stringValue("上海"))
                                )
                        ),
                HotelDoc.class);
        for (Hit<HotelDoc> hit : search.hits().hits()) {
            log.info("== hit: source: {}, id: {}", hit.source(), hit.id());
        }
    }
    /**
     * range
     * 查找价格大于等于100，小于等于500
     */
    @Test
    void rangeTest() throws IOException {
        SearchResponse<HotelDoc> search = elasticsearchClient.search(s -> s.index(INDEX_NAME)
                        .query(q ->
                                q.range(rangeQuery -> rangeQuery.field("price").gte(JsonData.of(100)).lte(JsonData.of(500)))
                        ),
                HotelDoc.class);
        for (Hit<HotelDoc> hit : search.hits().hits()) {
            log.info("== hit: source: {}, id: {}", hit.source(), hit.id());
        }
    }
    /**
     * geo_bounding_box
     */
    @Test
    void geoBoundingBoxTest() throws IOException {
        SearchResponse<HotelDoc> search = elasticsearchClient.search(s -> s.index(INDEX_NAME)
                        .query(q ->
                                q.geoBoundingBox(geoBoundingBoxQuery -> geoBoundingBoxQuery.field("location")
                                        .boundingBox(geoBoundings ->
                                                geoBoundings.tlbr(topLeftBottomRightGeoBounds -> topLeftBottomRightGeoBounds
                                                        // 设置左上角的纬度、经度
                                                        .topLeft(geoLocation -> geoLocation.latlon(latLonGeoLocation -> latLonGeoLocation.lat(31.1).lon(121.5)))
                                                        // 设置右下角的纬度、经度
                                                        .bottomRight(geoLocation -> geoLocation.latlon(latLonGeoLocation -> latLonGeoLocation.lat(30.9).lon(121.7)))
                                                )
                                        )
                                )
                        ),
                HotelDoc.class);
        for (Hit<HotelDoc> hit : search.hits().hits()) {
            log.info("== hit: source: {}, id: {}", hit.source(), hit.id());
        }
    }
    /**
     * geo_distance
     */
    @Test
    void geoDistanceTest() throws IOException {
        SearchResponse<HotelDoc> search = elasticsearchClient.search(s -> s.index(INDEX_NAME)
                        .query(q ->
                                q.geoDistance(geoDistanceQuery -> geoDistanceQuery.distance("15km")
                                        .field("location")
                                        .location(geoLocation -> geoLocation
                                                .latlon(latLonGeoLocation -> latLonGeoLocation
                                                        .lat(31.1)
                                                        .lon(121.5))))
                        ),
                HotelDoc.class);
        for (Hit<HotelDoc> hit : search.hits().hits()) {
            log.info("== hit: source: {}, id: {}", hit.source(), hit.id());
        }
    }
    /**
     * function_score
     */
    @Test
    void functionScoreTest() throws IOException {
        SearchResponse<HotelDoc> search = elasticsearchClient.search(s -> s.index(INDEX_NAME)
                        .query(q -> q.functionScore(functionScoreQuery -> functionScoreQuery
                                        // 匹配all字段中 上海的关键字
                                        .query(query -> query.match(matchQuery -> matchQuery.field("all").query("上海")))
                                        // 给city为上海的数据加分
                                        .functions(functionScore -> functionScore.filter(termQuery -> termQuery.term(t -> t.field("city").value(v -> v.stringValue("上海"))))
                                                // 权重为10，加权方式为求和
                                                .weight(10.0)).boostMode(FunctionBoostMode.Sum)
                                )
                        ),
                HotelDoc.class);
        for (Hit<HotelDoc> hit : search.hits().hits()) {
            log.info("== hit: source: {}, id: {}", hit.source(), hit.id());
        }
    }
    /**
     * bool
     */
    @Test
    void boolTest() throws IOException {
        SearchResponse<HotelDoc> search = elasticsearchClient.search(s -> s.index(INDEX_NAME)
                        .query(q -> q.bool(
                                boolQuery -> boolQuery
                                        // city 必须匹配 上海
                                        .must(query -> query.term(termQuery -> termQuery.field("city").value("上海")))
                                        // name 选择性匹配 上海
                                        .should(shouldMatchQuery -> shouldMatchQuery.match(matchQuery -> matchQuery.field("name").query("上海")))
                                        // price 必须不大于500
                                        .mustNot(mustNotRangeQuery -> mustNotRangeQuery.range(rangeQuery -> rangeQuery.field("price").gt(JsonData.of(500))))
                                        // price 必须大于等于100
                                        .filter(filterRangeQuery -> filterRangeQuery.range(rangeQuery -> rangeQuery.field("price").gte(JsonData.of(100))))
                        )),
                HotelDoc.class);
        for (Hit<HotelDoc> hit : search.hits().hits()) {
            log.info("== hit: source: {}, id: {}", hit.source(), hit.id());
        }
    }
    /**
     * 排序
     */
    @Test
    void sortTest() throws IOException {
        SearchResponse<HotelDoc> search = elasticsearchClient.search(s -> s.index(INDEX_NAME)
                        .query(q -> q.match(query -> query.field("name").query("上海")))
                        .sort(sortOption -> sortOption.field(fieldSort -> fieldSort
                                // 根据score降序
                                .field("score").order(SortOrder.Desc)
                                // 根据price升序
                                .field("price").order(SortOrder.Asc))),
                HotelDoc.class);
        for (Hit<HotelDoc> hit : search.hits().hits()) {
            log.info("== hit: source: {}, id: {}", hit.source(), hit.id());
        }
    }
    /**
     * 分页
     */
    @Test
    void pageTest() throws IOException {
        SearchResponse<HotelDoc> search = elasticsearchClient.search(s -> s.index(INDEX_NAME)
                        .query(q -> q.matchAll(matchAll -> matchAll))
                        .from(10)
                        .size(10)
                        .sort(sortOption -> sortOption.field(fieldSort -> fieldSort
                                // 根据score降序
                                .field("score").order(SortOrder.Desc))),
                HotelDoc.class);
        for (Hit<HotelDoc> hit : search.hits().hits()) {
            log.info("== hit: source: {}, id: {}", hit.source(), hit.id());
        }
    }
    /**
     * 高亮
     */
    @Test
    void hightLightTest() throws IOException {
        SearchResponse<HotelDoc> search = elasticsearchClient.search(s -> s.index(INDEX_NAME)
                        .query(q -> q.match(matchQuery -> matchQuery.field("all").query("上海")))
                        .highlight(highLight -> highLight.fields("name",
                                // 默认情况下搜索的字段和高亮的字段需要一样
                                highLightField -> highLightField.requireFieldMatch(false)
                                        .preTags("em")
                                        .postTags("em"))),
                HotelDoc.class);
        for (Hit<HotelDoc> hit : search.hits().hits()) {
            log.info("== hit: source: {}, id: {}", hit.source(), hit.id());
        }
    }
}
```
## 黑马旅游案例
> 前端搭建，[下载链接](https://wwm.lanzoue.com/iLisV1t5jfje)
### 酒店搜索和分页
#### 搜索
> 这是前端请求的格式
> `key`关键字
> `page`页码
> `size`每页大小
> `sortBy`排序字段，比如根据`评价`、`价格`排序
>
> `POST`请求，路径是`/hotel/list`
```json
{
    "key": "",
    "page": 1,
    "size": 5,
    "sortBy": "default"
}
```
```java
/**
 * 搜索参数
 *
 * @author: 不是菜狗爱编程
 * @date: 2024/03/30/9:35
 * @description:
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class SearchParams {
    private String key;
    private Integer page;
    private Integer size;
    private String sortBy;
}
```
```java
/**
 * 分页实体
 *
 * @author: 不是菜狗爱编程
 * @date: 2024/03/30/9:48
 * @description:
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class PageEntity {
    private Long total;
    private List<HotelDoc> hotels;
}
```
```java
@RestController
@RequestMapping("hotel")
public class SearchController {
    @Autowired
    private HotelService hotelService;
    /**
     * 搜索
     *
     * @param params 参数
     * @return {@link PageEntity}
     */
    @PostMapping("/list")
    public PageEntity search(@RequestBody SearchParams params){
        return hotelService.search(params);
    }
}
```
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.SortOrder;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import co.elastic.clients.elasticsearch.core.search.Hit;
import co.elastic.clients.elasticsearch.core.search.HitsMetadata;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.example.domain.Hotel;
import com.example.domain.PageEntity;
import com.example.domain.SearchParams;
import com.example.domain.doc.HotelDoc;
import com.example.mapper.HotelMapper;
import com.example.service.HotelService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
/**
 * @author 不是菜狗爱编程
 * @description 针对表【tb_hotel】的数据库操作Service实现
 * @createDate 2024-03-27 07:42:58
 */
@Service
public class HotelServiceImpl extends ServiceImpl<HotelMapper, Hotel>
        implements HotelService {
    private static final String INDEX_NAME = "hotel";
    @Autowired
    private ElasticsearchClient elasticsearchClient;
    /**
     * 搜索
     *
     * @param params 参数
     * @return {@link PageEntity}
     */
    @Override
    public PageEntity search(SearchParams params) {
        // 关键字
        String key = params.getKey();
        Integer page = params.getPage();
        Integer size = params.getSize();
        try {
            SearchResponse<HotelDoc> search = elasticsearchClient.search(s -> s.index(INDEX_NAME)
                            .query(q -> {
                                // 没有查询条件，查询全部
                                if (key == null || key.isEmpty()) {
                                    return q.matchAll(matchAll -> matchAll);
                                } else {
                                    // 有查询条件，查询关键字
                                    return q.match(matchQuery -> matchQuery.field("all").query(key));
                                }
                            })
                            .from((page - 1) * size)
                            .size(size),
                    HotelDoc.class);
            return handleResponse(search);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
    private PageEntity handleResponse(SearchResponse<HotelDoc> response){
        // 解析响应
        HitsMetadata<HotelDoc> hits = response.hits();
        assert hits.total() != null;
        // 总条数
        long value = hits.total().value();
        List<Hit<HotelDoc>> hotels = hits.hits();
        List<HotelDoc> result = new ArrayList<>();
        hotels.forEach(hotelDocHit -> {
            result.add(hotelDocHit.source());
        });
        return new PageEntity(value,result);
    }
}
```
> 当前接口返回的数据
```json
{
    "total": 82,
    "hotels": [
        {
            "id": 2056126831,
            "name": "上海虹桥金臣皇冠假日酒店",
            "address": "申长路630弄1-3 号",
            "price": 2488,
            "score": 48,
            "brand": "皇冠假日",
            "city": "上海",
            "starName": "五钻",
            "business": "虹桥机场/国家会展中心",
            "location": {
                "lat": 31.19036,
                "lon": 121.31535
            },
            "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/PvFh4Vzc84xXhm5N41F6AqdAqyJ_w200_h200_c1_t0.jpg"
        },
        {
            "id": 56392,
            "name": "上海银星皇冠假日酒店",
            "address": "番禺路400号",
            "price": 809,
            "score": 47,
            "brand": "皇冠假日",
            "city": "上海",
            "starName": "五星级",
            "business": "徐家汇地区",
            "location": {
                "lat": 31.202768,
                "lon": 121.429524
            },
            "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/37ucQ38K3UFdcRqntJ8M5dt884HR_w200_h200_c1_t0.jpg"
        },
        {
            "id": 60398,
            "name": "上海复旦皇冠假日酒店",
            "address": "邯郸路199号",
            "price": 924,
            "score": 47,
            "brand": "皇冠假日",
            "city": "上海",
            "starName": "五星级",
            "business": "江湾/五角场商业区",
            "location": {
                "lat": 31.295382,
                "lon": 121.502537
            },
            "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2H1Gk8LHaBWZfYvR6NYYcGTvACmL_w200_h200_c1_t0.jpg"
        },
        {
            "id": 598591,
            "name": "上海丽昂豪生大酒店",
            "address": "金新路99号",
            "price": 529,
            "score": 47,
            "brand": "豪生",
            "city": "上海",
            "starName": "四钻",
            "business": "浦东金桥地区",
            "location": {
                "lat": 31.252496,
                "lon": 121.600085
            },
            "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2KfPPyPx9rWyVXif2CUuxv61Nryc_w200_h200_c1_t0.jpg"
        },
        {
            "id": 648219,
            "name": "上海金桥红枫万豪酒店",
            "address": "新金桥路15号",
            "price": 891,
            "score": 47,
            "brand": "万豪",
            "city": "上海",
            "starName": "五钻",
            "business": "浦东金桥地区",
            "location": {
                "lat": 31.244061,
                "lon": 121.591153
            },
            "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/52/B6/Cii-TF3eXKuIR_a0AAUx-Xd2JLQAAGRfACSpvUABTIR560_w200_h200_c1_t0.jpg"
        }
    ]
}
```
### 酒店结果过滤
> 添加参数
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class SearchParams {
    private String key;
    private Integer page;
    private Integer size;
    private String sortBy;
    private String city;
    private String brand ;
    private String starName ;
    private Integer minPrice;
    private Integer maxPrice;
}
```
> 过滤条件包括
>
> 1. city精确匹配
> 2. brand精确匹配
> 3. starName精确匹配
> 4. price范围过滤
>
> 此时有多个条件，应该用`bool`查询
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.SortOrder;
import co.elastic.clients.elasticsearch._types.query_dsl.BoolQuery;
import co.elastic.clients.elasticsearch._types.query_dsl.QueryBuilders;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import co.elastic.clients.elasticsearch.core.search.Hit;
import co.elastic.clients.elasticsearch.core.search.HitsMetadata;
import co.elastic.clients.json.JsonData;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.example.domain.Hotel;
import com.example.domain.PageEntity;
import com.example.domain.SearchParams;
import com.example.domain.doc.HotelDoc;
import com.example.mapper.HotelMapper;
import com.example.service.HotelService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
/**
 * @author 不是菜狗爱编程
 * @description 针对表【tb_hotel】的数据库操作Service实现
 * @createDate 2024-03-27 07:42:58
 */
@Service
public class HotelServiceImpl extends ServiceImpl<HotelMapper, Hotel>
        implements HotelService {
    private static final String INDEX_NAME = "hotel";
    @Autowired
    private ElasticsearchClient elasticsearchClient;
    /**
     * 搜索
     *
     * @param params 参数
     * @return {@link PageEntity}
     */
    @Override
    public PageEntity search(SearchParams params) {
        // 关键字
        Integer page = params.getPage();
        Integer size = params.getSize();
        try {
            BoolQuery boolQuery = buildBasicQuery(params);
            SearchResponse<HotelDoc> search = elasticsearchClient.search(s -> s.index(INDEX_NAME)
                            .query(q->q.bool(boolQuery))
                            .from((page - 1) * size)
                            .size(size),
                    HotelDoc.class);
            return handleResponse(search);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
    private static BoolQuery buildBasicQuery(SearchParams params) {
        BoolQuery.Builder bool = QueryBuilders.bool();
        String key = params.getKey();
        String city = params.getCity();
        String brand = params.getBrand();
        Integer minPrice = params.getMinPrice();
        Integer maxPrice = params.getMaxPrice();
        // 没有查询条件，查询全部
        if (key == null || key.isEmpty()) {
            bool.must(
                    query->query.matchAll(
                            matchAll -> matchAll)
            );
        } else {
            // 有查询条件，查询关键字
            bool.must(
                    query->query.match(
                            matchQuery -> matchQuery.field("all").query(key)
                    )
            );
        }
        // 城市条件
        if(city !=null&& !city.isEmpty()){
            bool.filter(
                    filterQuery->filterQuery.term(
                            termQuery->termQuery.field("city").value(city)
                    )
            );
        }
        // 品牌条件
        if(brand !=null&& !brand.isEmpty()){
            bool.filter(
                    filterQuery->filterQuery.term(
                            termQuery->termQuery.field("brand").value(brand)
                    )
            );
        }
        // 价格条件
        if(minPrice !=null&& maxPrice !=null){
            bool.filter(
                    filterQuery->filterQuery.range(
                            rangeQuery->rangeQuery.field("price").gte(JsonData.of(minPrice)).lte(JsonData.of(maxPrice))
                    )
            );
        }
        return bool.build();
    }
    private PageEntity handleResponse(SearchResponse<HotelDoc> response){
        // 解析响应
        HitsMetadata<HotelDoc> hits = response.hits();
        assert hits.total() != null;
        // 总条数
        long value = hits.total().value();
        List<Hit<HotelDoc>> hotels = hits.hits();
        List<HotelDoc> result = new ArrayList<>();
        hotels.forEach(hotelDocHit -> {
            result.add(hotelDocHit.source());
        });
        return new PageEntity(value,result);
    }
}
```
### 我周边的酒店
> 根据当前定位查找附近的酒店，按照距离升序排序
```json
{
    "key": "",
    "page": 1,
    "size": 5,
    "sortBy": "default",
    "city": "北京",
    "brand": "速8",
    "location": "31.258042,120.643542"
}
```
> 新增`location`字段
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class SearchParams {
    private String key;
    private Integer page;
    private Integer size;
    private String sortBy;
    private String city;
    private String brand ;
    private String starName ;
    private Integer minPrice;
    private Integer maxPrice;
    private String location;
}
```
> 新增`distance`字段
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Document(indexName = "hotel",createIndex = true)
public class HotelDoc {
    @Id
    @Field(type = FieldType.Keyword)
    private Long id;
    @Field(type = FieldType.Text)
    private String name;
    @Field(type = FieldType.Keyword)
    private String address;
    @Field(type = FieldType.Integer)
    private Integer price;
    @Field(type = FieldType.Integer)
    private Integer score;
    @Field(type = FieldType.Keyword)
    private String brand;
    @Field(type = FieldType.Keyword)
    private String city;
    @Field(type = FieldType.Keyword)
    private String starName;
    @Field(type = FieldType.Keyword)
    private String business;
    /**
     * 位置
     */
    @GeoPointField
    private GeoPoint location;
    @Field(type = FieldType.Keyword)
    private String pic;
    /**
     * 距离
     */
    private Double distance;
    public HotelDoc(Hotel hotel) {
        this.id = hotel.getId();
        this.name = hotel.getName();
        this.address = hotel.getAddress();
        this.price = hotel.getPrice();
        this.score = hotel.getScore();
        this.brand = hotel.getBrand();
        this.city = hotel.getCity();
        this.starName = hotel.getStarName();
        this.business = hotel.getBusiness();
        this.location=new GeoPoint(Double.parseDouble(hotel.getLatitude()),Double.parseDouble(hotel.getLongitude()));
        this.pic = hotel.getPic();
    }
}
```
> `handleResponse`方法添加排序功能
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.DistanceUnit;
import co.elastic.clients.elasticsearch._types.FieldValue;
import co.elastic.clients.elasticsearch._types.SortOrder;
import co.elastic.clients.elasticsearch._types.query_dsl.BoolQuery;
import co.elastic.clients.elasticsearch._types.query_dsl.QueryBuilders;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import co.elastic.clients.elasticsearch.core.search.Hit;
import co.elastic.clients.elasticsearch.core.search.HitsMetadata;
import co.elastic.clients.json.JsonData;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.example.domain.Hotel;
import com.example.domain.PageEntity;
import com.example.domain.SearchParams;
import com.example.domain.doc.HotelDoc;
import com.example.mapper.HotelMapper;
import com.example.service.HotelService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.util.CollectionUtils;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
/**
 * @author 不是菜狗爱编程
 * @description 针对表【tb_hotel】的数据库操作Service实现
 * @createDate 2024-03-27 07:42:58
 */
@Service
public class HotelServiceImpl extends ServiceImpl<HotelMapper, Hotel>
        implements HotelService {
    private static final String INDEX_NAME = "hotel";
    @Autowired
    private ElasticsearchClient elasticsearchClient;
    /**
     * 搜索
     *
     * @param params 参数
     * @return {@link PageEntity}
     */
    @Override
    public PageEntity search(SearchParams params) {
        // 关键字
        Integer page = params.getPage();
        Integer size = params.getSize();
        String location = params.getLocation();
        try {
            BoolQuery boolQuery = buildBasicQuery(params);
            SearchResponse<HotelDoc> search = elasticsearchClient.search(s -> s.index(INDEX_NAME)
                            .query(q -> q.bool(boolQuery))
                            .from((page - 1) * size)
                            .size(size)
                            .sort(sortOption -> sortOption.geoDistance(
                                    geoDistanceSort -> geoDistanceSort.field("location").location(
                                                    geoLocation -> geoLocation.latlon(
                                                            latLonGeoLocation -> latLonGeoLocation.lat(31.258042).lon(120.643542)
                                                    )
                                            )
                                            // 距离升序排序
                                            .order(SortOrder.Asc)
                                            // 单位是km
                                            .unit(DistanceUnit.Kilometers)
                            )),
                    HotelDoc.class);
            return handleResponse(search);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
    private static BoolQuery buildBasicQuery(SearchParams params) {
        BoolQuery.Builder bool = QueryBuilders.bool();
        String key = params.getKey();
        String city = params.getCity();
        String brand = params.getBrand();
        Integer minPrice = params.getMinPrice();
        Integer maxPrice = params.getMaxPrice();
        // 没有查询条件，查询全部
        if (key == null || key.isEmpty()) {
            bool.must(
                    query -> query.matchAll(
                            matchAll -> matchAll)
            );
        } else {
            // 有查询条件，查询关键字
            bool.must(
                    query -> query.match(
                            matchQuery -> matchQuery.field("all").query(key)
                    )
            );
        }
        // 城市条件
        if (city != null && !city.isEmpty()) {
            bool.filter(
                    filterQuery -> filterQuery.term(
                            termQuery -> termQuery.field("city").value(city)
                    )
            );
        }
        // 品牌条件
        if (brand != null && !brand.isEmpty()) {
            bool.filter(
                    filterQuery -> filterQuery.term(
                            termQuery -> termQuery.field("brand").value(brand)
                    )
            );
        }
        // 价格条件
        if (minPrice != null && maxPrice != null) {
            bool.filter(
                    filterQuery -> filterQuery.range(
                            rangeQuery -> rangeQuery.field("price").gte(JsonData.of(minPrice)).lte(JsonData.of(maxPrice))
                    )
            );
        }
        return bool.build();
    }
    private PageEntity handleResponse(SearchResponse<HotelDoc> response) {
        // 解析响应
        HitsMetadata<HotelDoc> hits = response.hits();
        assert hits.total() != null;
        // 总条数
        long value = hits.total().value();
        List<Hit<HotelDoc>> hotels = hits.hits();
        List<HotelDoc> result = new ArrayList<>();
        hotels.forEach(hotelDocHit -> {
            HotelDoc source = hotelDocHit.source();
            // 获取排序值
            List<FieldValue> sort = hotelDocHit.sort();
            if(!CollectionUtils.isEmpty(sort)){
                Double distance = sort.get(0).doubleValue();
                assert source != null;
                source.setDistance(distance);
            }
            result.add(source);
        });
        return new PageEntity(value, result);
    }
}
```
### 酒店竞价排名
>1. 给HotelDoc类添加isAd字段，Boolean类型，表示`是否是广告`
>2. 挑选几个你喜欢的酒店，给它的文档数据添加isAd字段，值为true
>3. 修改search方法，添加function score功能,给isAd值为true的酒店增加权重
```sql
POST /hotel/_update/46829
{
  "doc":{
    "isAd":true
  }
}
POST /hotel/_update/47066
{
  "doc":{
    "isAd":true
  }
}
```
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Document(indexName = "hotel",createIndex = true)
public class HotelDoc {
    @Id
    @Field(type = FieldType.Keyword)
    private Long id;
    @Field(type = FieldType.Text)
    private String name;
    @Field(type = FieldType.Keyword)
    private String address;
    @Field(type = FieldType.Integer)
    private Integer price;
    @Field(type = FieldType.Integer)
    private Integer score;
    @Field(type = FieldType.Keyword)
    private String brand;
    @Field(type = FieldType.Keyword)
    private String city;
    @Field(type = FieldType.Keyword)
    private String starName;
    @Field(type = FieldType.Keyword)
    private String business;
    /**
     * 位置
     */
    @GeoPointField
    private GeoPoint location;
    @Field(type = FieldType.Keyword)
    private String pic;
    /**
     * 距离
     */
    private Double distance;
    /**
     * 是广告
     */
    private Boolean isAd;
    public HotelDoc(Hotel hotel) {
        this.id = hotel.getId();
        this.name = hotel.getName();
        this.address = hotel.getAddress();
        this.price = hotel.getPrice();
        this.score = hotel.getScore();
        this.brand = hotel.getBrand();
        this.city = hotel.getCity();
        this.starName = hotel.getStarName();
        this.business = hotel.getBusiness();
        this.location=new GeoPoint(Double.parseDouble(hotel.getLatitude()),Double.parseDouble(hotel.getLongitude()));
        this.pic = hotel.getPic();
    }
}
```
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.DistanceUnit;
import co.elastic.clients.elasticsearch._types.FieldValue;
import co.elastic.clients.elasticsearch._types.SortOrder;
import co.elastic.clients.elasticsearch._types.query_dsl.BoolQuery;
import co.elastic.clients.elasticsearch._types.query_dsl.FunctionScoreMode;
import co.elastic.clients.elasticsearch._types.query_dsl.FunctionScoreQuery;
import co.elastic.clients.elasticsearch._types.query_dsl.QueryBuilders;
import co.elastic.clients.elasticsearch.core.SearchRequest;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import co.elastic.clients.elasticsearch.core.search.Hit;
import co.elastic.clients.elasticsearch.core.search.HitsMetadata;
import co.elastic.clients.json.JsonData;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.example.domain.Hotel;
import com.example.domain.PageEntity;
import com.example.domain.SearchParams;
import com.example.domain.doc.HotelDoc;
import com.example.mapper.HotelMapper;
import com.example.service.HotelService;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.util.CollectionUtils;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
/**
 * @author 不是菜狗爱编程
 * @description 针对表【tb_hotel】的数据库操作Service实现
 * @createDate 2024-03-27 07:42:58
 */
@Service
public class HotelServiceImpl extends ServiceImpl<HotelMapper, Hotel>
        implements HotelService {
    private static final String INDEX_NAME = "hotel";
    @Autowired
    private ElasticsearchClient elasticsearchClient;
    /**
     * 搜索
     *
     * @param params 参数
     * @return {@link PageEntity}
     */
    @Override
    public PageEntity search(SearchParams params) {
        // 关键字
        Integer page = params.getPage();
        Integer size = params.getSize();
        String location = params.getLocation();
        try {
            FunctionScoreQuery functionScoreQuery = buildBasicQuery(params);
            SearchRequest.Builder builder = new SearchRequest.Builder();
            builder.index(INDEX_NAME)
                    .query(q -> q.functionScore(functionScoreQuery))
                    .from((page - 1) * size)
                    .size(size);
            // 距离排序
            if (StringUtils.isNotBlank(location)) {
                builder.sort(s->s.geoDistance(
                        geoDistanceSort -> geoDistanceSort.field("location").location(
                                        geoLocation -> geoLocation.latlon(
                                                latLonGeoLocation -> latLonGeoLocation.lat(31.258042).lon(120.643542)
                                        )
                                )
                                // 距离升序排序
                                .order(SortOrder.Asc)
                                // 单位是km
                                .unit(DistanceUnit.Kilometers)));
            }
            SearchResponse<HotelDoc> search = elasticsearchClient.search(builder.build(), HotelDoc.class);
            return handleResponse(search);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
    private static FunctionScoreQuery buildBasicQuery(SearchParams params) {
        BoolQuery.Builder bool = QueryBuilders.bool();
        String key = params.getKey();
        String city = params.getCity();
        String brand = params.getBrand();
        Integer minPrice = params.getMinPrice();
        Integer maxPrice = params.getMaxPrice();
        // 没有查询条件，查询全部
        if (key == null || key.isEmpty()) {
            bool.must(
                    query -> query.matchAll(
                            matchAll -> matchAll)
            );
        } else {
            // 有查询条件，查询关键字
            bool.must(
                    query -> query.match(
                            matchQuery -> matchQuery.field("all").query(key)
                    )
            );
        }
        // 城市条件
        if (city != null && !city.isEmpty()) {
            bool.filter(
                    filterQuery -> filterQuery.term(
                            termQuery -> termQuery.field("city").value(city)
                    )
            );
        }
        // 品牌条件
        if (brand != null && !brand.isEmpty()) {
            bool.filter(
                    filterQuery -> filterQuery.term(
                            termQuery -> termQuery.field("brand").value(brand)
                    )
            );
        }
        // 价格条件
        if (minPrice != null && maxPrice != null) {
            bool.filter(
                    filterQuery -> filterQuery.range(
                            rangeQuery -> rangeQuery.field("price").gte(JsonData.of(minPrice)).lte(JsonData.of(maxPrice))
                    )
            );
        }
        FunctionScoreQuery.Builder functionScoreBuilder = QueryBuilders.functionScore();
        functionScoreBuilder.query(
                functionScore -> functionScore.functionScore(
                        q -> q.query(
                                        boolQuery -> boolQuery.bool(bool.build())
                                ).functions(
                                        f -> f.filter(
                                                t -> t.term(termQuery -> termQuery.field("isAd").value(true))
                                        ).weight(10.0)
                                )
                                .scoreMode(FunctionScoreMode.Sum)
                )
        );
        return functionScoreBuilder.build();
    }
    private PageEntity handleResponse(SearchResponse<HotelDoc> response) {
        // 解析响应
        HitsMetadata<HotelDoc> hits = response.hits();
        assert hits.total() != null;
        // 总条数
        long value = hits.total().value();
        List<Hit<HotelDoc>> hotels = hits.hits();
        List<HotelDoc> result = new ArrayList<>();
        hotels.forEach(hotelDocHit -> {
            HotelDoc source = hotelDocHit.source();
            // 获取排序值
            List<FieldValue> sort = hotelDocHit.sort();
            if (!CollectionUtils.isEmpty(sort)) {
                Double distance = sort.get(0).doubleValue();
                assert source != null;
                source.setDistance(distance);
            }
            result.add(source);
        });
        return new PageEntity(value, result);
    }
}
```
## 数据聚合
### 概念介绍
> 聚合可以实现对文档数据的统计、分析、运算。聚合常见的有三类:
>
> `桶聚合`(Bucket)
> `度量聚合`(Metric)
> `管道聚合`(Pipeline)
1. 桶聚合:用来对文档做分组
   - TermAggregation:按照文档字段值分组
   - Date Histogram:按照日期阶梯分组，例如一周为一组，或者一月为一组
2. 度量聚合:用以计算一些值， 比如:最大值、最小值、平均值等
   - Avg
   - Max
   - Min
   - Stats：同时求max、min、avg、sum等
3. 管道聚合:其它聚合的结果为基础做聚合
参与`聚合`的`字段类型`必须是:
1. keyword
2. 数值
3. 日期
4. 布尔
### 桶聚合
```sql
# 桶聚合
GET /hotel/_search
{
  // 设置size为0，结果中不包含文档，只包含聚合结果
  "size":0,
  // 定义聚合
  "aggs":{
    // 自己给聚合起个名字
    "brandAgg":{
      // 聚合的类型，按照品牌值聚合，所以选择term ,
      "terms": {
        // 参与聚合的字段
        "field": "brand",
        // 希望获取的聚合结果数量
        "size": 10
      }      
    }
  }
}
# 聚合结果排序
GET /hotel/_search
{
  "size":0,
  "aggs":{
    "brandAgg":{
      "terms": {
        "field": "brand",
        "order": {
        // 升序排序
          "_count": "asc"
        }, 
        "size": 10
      }      
    }
  }
}
# 限定聚合文档的范围，只对某一类文档做聚合
GET /hotel/_search
{
  // 对价格小于等于200的文档做聚合
  "query":{
    "range":{
      "price":{
        "lte": 200
      }
    }
  },
  "size":0,
  "aggs":{
    "brandAgg":{
      "terms": {
        "field": "brand",
        "size": 10
      }      
    }
  }
}
```
> 执行结果
```json
{
  "took": 21,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 201,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "brandAgg": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 39,
      "buckets": [
        {
          "key": "7天酒店",
          "doc_count": 30
        },
        {
          "key": "如家",
          "doc_count": 30
        },
        {
          "key": "皇冠假日",
          "doc_count": 17
        },
        {
          "key": "速8",
          "doc_count": 15
        },
        {
          "key": "万怡",
          "doc_count": 13
        },
        {
          "key": "华美达",
          "doc_count": 13
        },
        {
          "key": "和颐",
          "doc_count": 12
        },
        {
          "key": "万豪",
          "doc_count": 11
        },
        {
          "key": "喜来登",
          "doc_count": 11
        },
        {
          "key": "希尔顿",
          "doc_count": 10
        }
      ]
    }
  }
}
```
**总结**
aggs代表聚合，与query同级，此时query的作用是?
答：`限定聚合的的文档范围`
聚合必须的三要素:
1. 聚合名称
2. 聚合类型
3. 聚合字段
聚合可配置属性有:
1. size:指定聚合结果数量
2. order:指定聚合结果排序方式
3. field:指定聚合字段
### 度量聚合
> 例如，我们要求获取每个品牌的用户评分的min、max、avg等值
>
> 根据上面概念介绍，我们需要获取多个值，单一的`Avg`、`Min`、`Max`查询已经无法满足需求，所以需要使用`Stats`查询
> 大概思路应该是`先对品牌做桶聚合`，然后再对该聚合结果继续做`度量聚合`
```sql
GET /hotel/_search
{
  "size":0,
  "aggs":{
    "brandAgg":{
      "terms": {
        "field": "brand",
        "size": 10
      },
      // brand聚合的子聚合，分组后对每组分别计算
      "aggs":{
        // 聚合名称
        "scoreAgg":{
          // 聚合类型，这里的stats可以计算min、max、avg等
          "stats": {
            // 聚合字段，这里是score
            "field": "score"
          }
        }
      }
    }
  }
}
# 结果评分的平均分按照升序排序
GET /hotel/_search
{
  "size":0,
  "aggs":{
    "brandAgg":{
      "terms": {
        "field": "brand",
        "size": 10,
        "order": {
        // 按照平均分升序排序
          "scoreAgg.avg": "asc"
        }
      },
      "aggs":{
        "scoreAgg":{
          "stats": {
            "field": "score"
          }
        }
      }
    }
  }
}
```
> 执行结果
```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 201,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "brandAgg": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 39,
      "buckets": [
        {
          "key": "7天酒店",
          "doc_count": 30,
          "scoreAgg": {
            "count": 30,
            "min": 35,
            "max": 43,
            "avg": 37.86666666666667,
            "sum": 1136
          }
        },
        {
          "key": "如家",
          "doc_count": 30,
          "scoreAgg": {
            "count": 30,
            "min": 43,
            "max": 47,
            "avg": 44.833333333333336,
            "sum": 1345
          }
        },
        {
          "key": "皇冠假日",
          "doc_count": 17,
          "scoreAgg": {
            "count": 17,
            "min": 44,
            "max": 48,
            "avg": 46,
            "sum": 782
          }
        },
        {
          "key": "速8",
          "doc_count": 15,
          "scoreAgg": {
            "count": 15,
            "min": 35,
            "max": 47,
            "avg": 38.733333333333334,
            "sum": 581
          }
        },
        {
          "key": "万怡",
          "doc_count": 13,
          "scoreAgg": {
            "count": 13,
            "min": 44,
            "max": 48,
            "avg": 45.69230769230769,
            "sum": 594
          }
        },
        {
          "key": "华美达",
          "doc_count": 13,
          "scoreAgg": {
            "count": 13,
            "min": 40,
            "max": 47,
            "avg": 44,
            "sum": 572
          }
        },
        {
          "key": "和颐",
          "doc_count": 12,
          "scoreAgg": {
            "count": 12,
            "min": 44,
            "max": 47,
            "avg": 46.083333333333336,
            "sum": 553
          }
        },
        {
          "key": "万豪",
          "doc_count": 11,
          "scoreAgg": {
            "count": 11,
            "min": 43,
            "max": 47,
            "avg": 45.81818181818182,
            "sum": 504
          }
        },
        {
          "key": "喜来登",
          "doc_count": 11,
          "scoreAgg": {
            "count": 11,
            "min": 44,
            "max": 48,
            "avg": 46,
            "sum": 506
          }
        },
        {
          "key": "希尔顿",
          "doc_count": 10,
          "scoreAgg": {
            "count": 10,
            "min": 37,
            "max": 48,
            "avg": 45.4,
            "sum": 454
          }
        }
      ]
    }
  }
}
```
### RestApi实现聚合
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.SortOrder;
import co.elastic.clients.elasticsearch._types.aggregations.Aggregate;
import co.elastic.clients.elasticsearch._types.aggregations.StringTermsBucket;
import co.elastic.clients.elasticsearch.core.SearchRequest;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import co.elastic.clients.json.JsonData;
import co.elastic.clients.util.NamedValue;
import com.example.domain.doc.HotelDoc;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
import java.util.List;
import java.util.Map;
/**
 * 聚合测试
 *
 * @author: 不是菜狗爱编程
 * @date: 2024/03/31/16:32
 * @description:
 */
@SpringBootTest
class AggregationTest {
    private static final String INDEX_NAME = "hotel";
    @Autowired
    private ElasticsearchClient elasticsearchClient;
    /**
     * 桶聚合
     *
     * @throws IOException io异常
     */
    @Test
    void bucketTest() throws IOException {
        SearchRequest.Builder searchRequestBuilder = new SearchRequest.Builder();
        searchRequestBuilder.index(INDEX_NAME)
                .size(0)
                .aggregations("brandAgg",agg->agg.terms(
                        termsAggregation->termsAggregation.field("brand").size(10)
                ));
        SearchResponse<HotelDoc> search = elasticsearchClient.search(searchRequestBuilder.build(), HotelDoc.class);
        Map<String, Aggregate> map = search.aggregations();
        for (Map.Entry<String, Aggregate> entry : map.entrySet()) {
            System.out.println("entry.getValue() = " + entry.getValue());
        }
        System.out.println("============================");
        // 或者根据聚合名称来获取聚合结果
        Aggregate brandAgg = search.aggregations().get("brandAgg");
        List<StringTermsBucket> array = brandAgg.sterms().buckets().array();
        for (StringTermsBucket stringTermsBucket : array) {
            System.out.printf("酒店品牌:%s 数量%d\n",stringTermsBucket.key().stringValue(),stringTermsBucket.docCount());
        }
    }
    /**
     * 桶聚合排序
     *
     * @throws IOException io异常
     */
    @Test
    void bucketWithSortTest() throws IOException {
        SearchRequest.Builder searchRequestBuilder = new SearchRequest.Builder();
        searchRequestBuilder.index(INDEX_NAME)
                .size(0)
                .aggregations("brandAgg",agg->agg.terms(
                        termsAggregation->termsAggregation.field("brand")
                                .order(NamedValue.of("_count", SortOrder.Asc))
                                .size(10)
                ));
        SearchResponse<HotelDoc> search = elasticsearchClient.search(searchRequestBuilder.build(), HotelDoc.class);
        Map<String, Aggregate> map = search.aggregations();
        for (Map.Entry<String, Aggregate> entry : map.entrySet()) {
            System.out.println("entry.getValue() = " + entry.getValue());
        }
    }
    /**
     * 限定聚合文档的范围
     *
     * @throws IOException io异常
     */
    @Test
    void bucketWithQualifiedDocument() throws IOException {
        SearchRequest.Builder searchRequestBuilder = new SearchRequest.Builder();
        searchRequestBuilder.index(INDEX_NAME)
                .query(query->query.range(
                        rangeQuery->rangeQuery.field("price").lte(JsonData.of(200))
                ))
                .size(0)
                .aggregations("brandAgg",agg->agg.terms(
                        termsAggregation->termsAggregation.field("brand")
                                .size(10)
                ));
        SearchResponse<HotelDoc> search = elasticsearchClient.search(searchRequestBuilder.build(), HotelDoc.class);
        Map<String, Aggregate> map = search.aggregations();
        for (Map.Entry<String, Aggregate> entry : map.entrySet()) {
            System.out.println("entry.getValue() = " + entry.getValue());
        }
    }
}
```
### 黑马旅游实现品牌、城市、星级的聚合
#### 基本实现
>搜索页面的品牌、城市等信息不应该是在页面写死，而是通过聚合索引库中的酒店数据得来的
![image-20240331171911025](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240331171911025.png)
> 实现service中的`filters`方法
```java
public interface HotelService extends IService<Hotel> {
    /**
     * 搜索
     *
     * @param params 参数
     * @return {@link PageEntity}
     */
    PageEntity search(SearchParams params);
    /**
     * 实现品牌、城市、星级的聚合
     *
     * @return {@link Map}<{@link String}, {@link List}<{@link String}>>
     */
    Map<String, List<String>> filters();
}
```
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.DistanceUnit;
import co.elastic.clients.elasticsearch._types.FieldValue;
import co.elastic.clients.elasticsearch._types.SortOrder;
import co.elastic.clients.elasticsearch._types.aggregations.Aggregate;
import co.elastic.clients.elasticsearch._types.aggregations.Aggregation;
import co.elastic.clients.elasticsearch._types.aggregations.AggregationBuilders;
import co.elastic.clients.elasticsearch._types.aggregations.StringTermsBucket;
import co.elastic.clients.elasticsearch._types.query_dsl.BoolQuery;
import co.elastic.clients.elasticsearch._types.query_dsl.FunctionScoreMode;
import co.elastic.clients.elasticsearch._types.query_dsl.FunctionScoreQuery;
import co.elastic.clients.elasticsearch._types.query_dsl.QueryBuilders;
import co.elastic.clients.elasticsearch.core.SearchRequest;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import co.elastic.clients.elasticsearch.core.search.Hit;
import co.elastic.clients.elasticsearch.core.search.HitsMetadata;
import co.elastic.clients.json.JsonData;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.example.domain.Hotel;
import com.example.domain.PageEntity;
import com.example.domain.SearchParams;
import com.example.domain.doc.HotelDoc;
import com.example.mapper.HotelMapper;
import com.example.service.HotelService;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.util.CollectionUtils;
import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
/**
 * @author 不是菜狗爱编程
 * @description 针对表【tb_hotel】的数据库操作Service实现
 * @createDate 2024-03-27 07:42:58
 */
@Service
public class HotelServiceImpl extends ServiceImpl<HotelMapper, Hotel>
        implements HotelService {
    private static final String INDEX_NAME = "hotel";
    /**
     * 品牌聚合名称
     */
    private static final String BRAND_AGGREGATION="brandAgg";
    /**
     * 城市聚合名称
     */
    private static final String CITY_AGGREGATION="cityAgg";
    /**
     * 星级聚合名称
     */
    private static final String STAR_AGGREGATION="starAgg";
    @Autowired
    private ElasticsearchClient elasticsearchClient;
    @Override
    public Map<String, List<String>> filters() {
        Map<String, List<String>> result = new HashMap<>();
        try {
            SearchRequest.Builder searchRequestBuilder = new SearchRequest.Builder();
            searchRequestBuilder.index(INDEX_NAME)
                    .size(0)
                    .aggregations(buildAggregation());
            SearchResponse<HotelDoc> search = elasticsearchClient.search(searchRequestBuilder.build(), HotelDoc.class);
            // 根据聚合名称来获取聚合结果
            Map<String, Aggregate> aggregations = search.aggregations();
            List<String> brandList=getAggByName(aggregations,BRAND_AGGREGATION);
            List<String> cityList=getAggByName(aggregations,CITY_AGGREGATION);
            List<String> starList=getAggByName(aggregations,STAR_AGGREGATION);
            result.put("brand",brandList);
            result.put("city",cityList);
            result.put("star",starList);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        return result;
    }
    /**
     * 按名称获取聚合结果
     *
     * @param aggregations    聚合
     * @param aggregationName 聚合名称
     * @return {@link List}<{@link String}>
     */
    private List<String> getAggByName(Map<String, Aggregate> aggregations, String aggregationName) {
        List<String> list = new ArrayList<>();
        Aggregate brandAgg = aggregations.get(aggregationName);
        List<StringTermsBucket> array = brandAgg.sterms().buckets().array();
        for (StringTermsBucket stringTermsBucket : array) {
            list.add(stringTermsBucket.key().stringValue());
        }
        return list;
    }
    public Map<String, Aggregation> buildAggregation(){
        Map<String, Aggregation> map = new HashMap<>();
        // 品牌聚合
        Aggregation brandAgg = AggregationBuilders.terms(termsAggregation -> termsAggregation.field("brand").size(10));
        // 城市聚合
        Aggregation cityAgg = AggregationBuilders.terms(termsAggregation -> termsAggregation.field("city").size(10));
        // 星级聚合
        Aggregation starAgg = AggregationBuilders.terms(termsAggregation -> termsAggregation.field("starName").size(10));
        map.put(BRAND_AGGREGATION,brandAgg);
        map.put(CITY_AGGREGATION,cityAgg);
        map.put(STAR_AGGREGATION,starAgg);
        return map;
    }
}
```
> 测试
```java
@SpringBootTest
class HotelServiceImplTest {
    @Autowired
    private HotelService hotelService;
    @Test
    void filters() {
        Map<String, List<String>> filters = hotelService.filters();
        for (Map.Entry<String, List<String>> entry : filters.entrySet()) {
            System.out.println(entry.getKey() + "= " + entry.getValue());
        }
    }
}
```
> 日志打印如下
```
2024-03-31T17:46:47.091+08:00 TRACE 17560 --- [           main] tracer                                   : curl -iX POST 'http://localhost:9200/hotel/_search?typed_keys=true' -d '{"aggregations":{"starAgg":{"terms":{"field":"starName","size":10}},"brandAgg":{"terms":{"field":"brand","size":10}},"cityAgg":{"terms":{"field":"city","size":10}}},"size":0}'
# HTTP/1.1 200 OK
# X-elastic-product: Elasticsearch
# content-type: application/vnd.elasticsearch+json;compatible-with=8
# content-length: 1072
#
# {"took":0,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":201,"relation":"eq"},"max_score":null,"hits":[]},"aggregations":{"sterms#starAgg":{"doc_count_error_upper_bound":0,"sum_other_doc_count":0,"buckets":[{"key":"二钻","doc_count":84},{"key":"五钻","doc_count":49},{"key":"四钻","doc_count":28},{"key":"五星级","doc_count":20},{"key":"三钻","doc_count":13},{"key":"四星级","doc_count":7}]},"sterms#cityAgg":{"doc_count_error_upper_bound":0,"sum_other_doc_count":0,"buckets":[{"key":"上海","doc_count":83},{"key":"北京","doc_count":62},{"key":"深圳","doc_count":56}]},"sterms#brandAgg":{"doc_count_error_upper_bound":0,"sum_other_doc_count":39,"buckets":[{"key":"7天酒店","doc_count":30},{"key":"如家","doc_count":30},{"key":"皇冠假日","doc_count":17},{"key":"速8","doc_count":15},{"key":"万怡","doc_count":13},{"key":"华美达","doc_count":13},{"key":"和颐","doc_count":12},{"key":"万豪","doc_count":11},{"key":"喜来登","doc_count":11},{"key":"希尔顿","doc_count":10}]}}}
star= [二钻, 五钻, 四钻, 五星级, 三钻, 四星级]
city= [上海, 北京, 深圳]
brand= [7天酒店, 如家, 皇冠假日, 速8, 万怡, 华美达, 和颐, 万豪, 喜来登, 希尔顿]
```
#### 添加过滤条件
```java
/**
 * @author: 不是菜狗爱编程
 * @date: 2024/03/30/9:47
 * @description:
 */
@RestController
@RequestMapping("hotel")
public class SearchController {
    @Autowired
    private HotelService hotelService;
    @PostMapping("/filters")
    public Map<String, List<String>> filters(@RequestBody SearchParams params){
        return hotelService.filters(params);
    }
}
```
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.DistanceUnit;
import co.elastic.clients.elasticsearch._types.FieldValue;
import co.elastic.clients.elasticsearch._types.SortOrder;
import co.elastic.clients.elasticsearch._types.aggregations.Aggregate;
import co.elastic.clients.elasticsearch._types.aggregations.Aggregation;
import co.elastic.clients.elasticsearch._types.aggregations.AggregationBuilders;
import co.elastic.clients.elasticsearch._types.aggregations.StringTermsBucket;
import co.elastic.clients.elasticsearch._types.query_dsl.BoolQuery;
import co.elastic.clients.elasticsearch._types.query_dsl.FunctionScoreMode;
import co.elastic.clients.elasticsearch._types.query_dsl.FunctionScoreQuery;
import co.elastic.clients.elasticsearch._types.query_dsl.QueryBuilders;
import co.elastic.clients.elasticsearch.core.SearchRequest;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import co.elastic.clients.elasticsearch.core.search.Hit;
import co.elastic.clients.elasticsearch.core.search.HitsMetadata;
import co.elastic.clients.json.JsonData;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.example.domain.Hotel;
import com.example.domain.PageEntity;
import com.example.domain.SearchParams;
import com.example.domain.doc.HotelDoc;
import com.example.mapper.HotelMapper;
import com.example.service.HotelService;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.util.CollectionUtils;
import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
/**
 * @author 不是菜狗爱编程
 * @description 针对表【tb_hotel】的数据库操作Service实现
 * @createDate 2024-03-27 07:42:58
 */
@Service
public class HotelServiceImpl extends ServiceImpl<HotelMapper, Hotel>
        implements HotelService {
    private static final String INDEX_NAME = "hotel";
    /**
     * 品牌聚合名称
     */
    private static final String BRAND_AGGREGATION="brandAgg";
    /**
     * 城市聚合名称
     */
    private static final String CITY_AGGREGATION="cityAgg";
    /**
     * 星级聚合名称
     */
    private static final String STAR_AGGREGATION="starAgg";
    @Autowired
    private ElasticsearchClient elasticsearchClient;
    @Override
    public Map<String, List<String>> filters(SearchParams params) {
        Map<String, List<String>> result = new HashMap<>();
        try {
            FunctionScoreQuery functionScoreQuery = buildBasicQuery(params);
            SearchRequest.Builder searchRequestBuilder = new SearchRequest.Builder();
            searchRequestBuilder.index(INDEX_NAME)
                	// 添加过滤条件
                    .query(q -> q.functionScore(functionScoreQuery))
                    .size(0)
                    .aggregations(buildAggregation());
            SearchResponse<HotelDoc> search = elasticsearchClient.search(searchRequestBuilder.build(), HotelDoc.class);
            // 根据聚合名称来获取聚合结果
            Map<String, Aggregate> aggregations = search.aggregations();
            List<String> brandList=getAggByName(aggregations,BRAND_AGGREGATION);
            List<String> cityList=getAggByName(aggregations,CITY_AGGREGATION);
            List<String> starList=getAggByName(aggregations,STAR_AGGREGATION);
            result.put("brand",brandList);
            result.put("city",cityList);
            result.put("star",starList);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        return result;
    }
    /**
     * 按名称获取聚合结果
     *
     * @param aggregations    聚合
     * @param aggregationName 聚合名称
     * @return {@link List}<{@link String}>
     */
    private List<String> getAggByName(Map<String, Aggregate> aggregations, String aggregationName) {
        List<String> list = new ArrayList<>();
        Aggregate brandAgg = aggregations.get(aggregationName);
        List<StringTermsBucket> array = brandAgg.sterms().buckets().array();
        for (StringTermsBucket stringTermsBucket : array) {
            list.add(stringTermsBucket.key().stringValue());
        }
        return list;
    }
    public Map<String, Aggregation> buildAggregation(){
        Map<String, Aggregation> map = new HashMap<>();
        // 品牌聚合
        Aggregation brandAgg = AggregationBuilders.terms(termsAggregation -> termsAggregation.field("brand").size(10));
        // 城市聚合
        Aggregation cityAgg = AggregationBuilders.terms(termsAggregation -> termsAggregation.field("city").size(10));
        // 星级聚合
        Aggregation starAgg = AggregationBuilders.terms(termsAggregation -> termsAggregation.field("starName").size(10));
        map.put(BRAND_AGGREGATION,brandAgg);
        map.put(CITY_AGGREGATION,cityAgg);
        map.put(STAR_AGGREGATION,starAgg);
        return map;
    }
    private static FunctionScoreQuery buildBasicQuery(SearchParams params) {
        BoolQuery.Builder bool = QueryBuilders.bool();
        String key = params.getKey();
        String city = params.getCity();
        String brand = params.getBrand();
        Integer minPrice = params.getMinPrice();
        Integer maxPrice = params.getMaxPrice();
        // 没有查询条件，查询全部
        if (key == null || key.isEmpty()) {
            bool.must(
                    query -> query.matchAll(
                            matchAll -> matchAll)
            );
        } else {
            // 有查询条件，查询关键字
            bool.must(
                    query -> query.match(
                            matchQuery -> matchQuery.field("all").query(key)
                    )
            );
        }
        // 城市条件
        if (city != null && !city.isEmpty()) {
            bool.filter(
                    filterQuery -> filterQuery.term(
                            termQuery -> termQuery.field("city").value(city)
                    )
            );
        }
        // 品牌条件
        if (brand != null && !brand.isEmpty()) {
            bool.filter(
                    filterQuery -> filterQuery.term(
                            termQuery -> termQuery.field("brand").value(brand)
                    )
            );
        }
        // 价格条件
        if (minPrice != null && maxPrice != null) {
            bool.filter(
                    filterQuery -> filterQuery.range(
                            rangeQuery -> rangeQuery.field("price").gte(JsonData.of(minPrice)).lte(JsonData.of(maxPrice))
                    )
            );
        }
        FunctionScoreQuery.Builder functionScoreBuilder = QueryBuilders.functionScore();
        functionScoreBuilder.query(
                functionScore -> functionScore.functionScore(
                        q -> q.query(
                                        boolQuery -> boolQuery.bool(bool.build())
                                ).functions(
                                        f -> f.filter(
                                                t -> t.term(termQuery -> termQuery.field("isAd").value(true))
                                        ).weight(10.0)
                                )
                                .scoreMode(FunctionScoreMode.Sum)
                )
        );
        return functionScoreBuilder.build();
    }
}
```
## 自动补全
### 自定义分词器
> elasticsearch中分词器(analyzer) 的组成包含三部分:
1. character filters:在tokenizer之 前对文本进行处理。例如删除字符、替换字符
2. tokenizer:将文本按照-定的规则切割成词条(term) 。例如keyword,就是不分词;还有ik_smart
3. tokenizer filter:将tokenizer输 出的词条做进一步 处理。例如大小写转换、同义词处理、拼音处理等
![image-20240331211025082](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240331211025082.png)
我们可以在**创建索引库时**，通过settings来配置自定义的analyzer(分词器)
这个分词器只对`当前索引库`有效
这里没有 `character filters`，这里并不需要对特殊字符进行处理
```sql
PUT /test
{
	"settings":{
		"analysis":{
			// 自定义分词器
			"analyzer":{
				// 分词器名称
				"my_analyzer":{
					"tokenizer":"ik_max_word",
					"filter":"pinyin"
				}
			}
		}
	}
}
```
> 这里的`pinyin`分词器其实还有些问题，会一个字一个字地转成拼音，而且把中文删除，需要进行进一步的定制
```sql
PUT /test
{
  "settings":{
    "analysis":{
      // 自定义分词器
      "analyzer":{
        // 分词器名称
        "my_analyzer":{
          "tokenizer":"ik_max_word",
          "filter":"py"
        }
      },
      // 自定义tokenizer filter
      "filter":{
        // 过滤器名称
        "py":{
          // 过滤器类型，这里是pinyin
          "type" : "pinyin",
          "keep_full_pinyin" : false,
          "keep_joined_full_pinyin" : true,
          // 是否要保留中文
          "keep_original" : true,
          "limit_first_letter_length" : 16,
          "remove_duplicated_term" : true,
          "none_chinese_pinyin_tokenize" : false
        }
      }
    }
  }
}
# 例如
PUT /test
{
  "settings":{
    "analysis":{
      // 自定义分词器
      "analyzer":{
        // 分词器名称
        "my_analyzer":{
          "tokenizer":"ik_max_word",
          "filter":"py"
        }
      },
      // 自定义tokenizer filter
      "filter":{
        // 过滤器名称
        "py":{
          // 过滤器类型，这里是pinyin
          "type" : "pinyin",
          "keep_full_pinyin" : false,
          "keep_joined_full_pinyin" : true,
          // 是否要保留中文
          "keep_original" : true,
          "limit_first_letter_length" : 16,
          "remove_duplicated_term" : true,
          "none_chinese_pinyin_tokenize" : false
        }
      }
    }
  },
  "mappings":{
    "properties":{
      "name":{
        "type":"text",
        "analyzer":"my_analyzer"
      }
    }
  }
}
```
> 测试方法1
```sql
POST /test/_analyze
{
  "text":["建设祖国"],
  "analyzer":"my_analyzer"
}
```
> 执行结果
```json
{
  "tokens": [
    {
      "token": "今天天气",
      "start_offset": 0,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "jintiantianqi",
      "start_offset": 0,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "jttq",
      "start_offset": 0,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "今天",
      "start_offset": 0,
      "end_offset": 2,
      "type": "CN_WORD",
      "position": 1
    },
    {
      "token": "jintian",
      "start_offset": 0,
      "end_offset": 2,
      "type": "CN_WORD",
      "position": 1
    },
    {
      "token": "jt",
      "start_offset": 0,
      "end_offset": 2,
      "type": "CN_WORD",
      "position": 1
    },
    {
      "token": "天天",
      "start_offset": 1,
      "end_offset": 3,
      "type": "CN_WORD",
      "position": 2
    },
    {
      "token": "tiantian",
      "start_offset": 1,
      "end_offset": 3,
      "type": "CN_WORD",
      "position": 2
    },
    {
      "token": "tt",
      "start_offset": 1,
      "end_offset": 3,
      "type": "CN_WORD",
      "position": 2
    },
    {
      "token": "天气",
      "start_offset": 2,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 3
    },
    {
      "token": "tianqi",
      "start_offset": 2,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 3
    },
    {
      "token": "tq",
      "start_offset": 2,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 3
    }
  ]
}
```
> 测试方法2
```sql
# 插入两条测试数据
POST /test/_doc/1
{
  "name":"狮子"
}
POST /test/_doc/2
{
  "name":"虱子"
}
# 查看插入数据
GET /test/_search
{
  "query":{
    "match_all": {}
  }
}
GET /test/_search
{
  "query":{
    "match": {
      "name":"狮子很猛"
    }
  }
}
```
> 执行结果
```json
{
  "took": 8,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.33425623,
    "hits": [
      {
        "_index": "test",
        "_id": "1",
        "_score": 0.33425623,
        "_source": {
          "name": "狮子"
        }
      },
      {
        "_index": "test",
        "_id": "2",
        "_score": 0.3085442,
        "_source": {
          "name": "虱子"
        }
      }
    ]
  }
}
```
### 问题
> 可以看到这里搜的是`狮子`，但是结果把`虱子`也搜到了
拼音分词器适合在创建倒排索引的时候使用，但不能在搜索的时候使用。
![image-20240331214256951](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240331214256951.png)
> 用户更希望 输入中文的时候，用`中文`进行分词搜索，而输入拼音的时候，用`拼音`来搜索，排除掉输入中文，使用`拼音`来搜，这样会搜索到同音字
因此字段在创建倒排索引时应该用my_analyzer分词器;字段在搜索时应该使用ik_smart分词器
为什么这么说呢？可以根据上图多揣摩揣摩
> 在插入文档`狮子`和`虱子`的时候，倒排索引中就出现了这四个词条，`狮子`、`shizi`、`sz`、`虱子`
>
> 在搜索的时候，使用ik分词器，输入中文则会对应的中文词条，输入英文则会搜索到对应的英文词条
```sql
PUT /test
{
  "settings":{
    "analysis":{
      "analyzer":{
        "my_analyzer":{
          "tokenizer":"ik_max_word",
          "filter":"py"
        }
      },
      "filter":{
        "py":{
          "type" : "pinyin",
          "keep_full_pinyin" : false,
          "keep_joined_full_pinyin" : true,
          "keep_original" : true,
          "limit_first_letter_length" : 16,
          "remove_duplicated_term" : true,
          "none_chinese_pinyin_tokenize" : false
        }
      }
    }
  },
  "mappings":{
    "properties":{
      "name":{
        "type":"text",
        "analyzer":"my_analyzer",
        // 指定ik_smart分词器
        "search_analyzer":"ik_smart"
      }
    }
  }
}
```
> 继续上述测试方法，结果如下
```json
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.9530773,
    "hits": [
      {
        "_index": "test",
        "_id": "1",
        "_score": 0.9530773,
        "_source": {
          "name": "狮子"
        }
      }
    ]
  }
}
```
### completion suggester查询
elasticsearch提供了Completion Suggester查询来实现自动补全功能。这个查询会匹配以用户输入内容开头的词条并返回。为了提高补全查询的效率，对于文档中字段的类型有一些约束:
```sql
PUT /test2
{
  "mappings": {
    "properties": {
      "title": {
        "type": "completion"
      }
    }
  }
}
// 示例数据
POST /test2/_doc
{
  "title":["Sony","WH-1000XM3"]
}
POST /test2/_doc
{
  "title":["SK-II","PITERA"]
}
POST /test2/_doc
{
  "title":["Nintendo","Switch"]
}
```
> 查询语法如下
```sql
GET /test2/_search
{
  "suggest":{
    // 自定义自动补全的名称
    "title_suggest":{
      // 关键字
      "text":"s",
      "completion":{
        // 补全查询的字段
        "field":"title",
        // 跳过重复
        "skip_duplicates":true,
        // 获取前10条结果
        "size":10
      }
    }
  }
}
```
> 执行结果
```json
{
  "took": 878,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 0,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "suggest": {
    "title_suggest": [
      {
        "text": "s",
        "offset": 0,
        "length": 1,
        "options": [
          {
            "text": "SK-II",
            "_index": "test2",
            "_id": "0_HglI4BYJfEHegm4Kw4",
            "_score": 1,
            "_source": {
              "title": [
                "SK-II",
                "PITERA"
              ]
            }
          },
          {
            "text": "Sony",
            "_index": "test2",
            "_id": "0vHglI4BYJfEHegm2qwt",
            "_score": 1,
            "_source": {
              "title": [
                "Sony",
                "WH-1000XM3"
              ]
            }
          },
          {
            "text": "Switch",
            "_index": "test2",
            "_id": "1PHglI4BYJfEHegm5qyP",
            "_score": 1,
            "_source": {
              "title": [
                "Nintendo",
                "Switch"
              ]
            }
          }
        ]
      }
    ]
  }
}
```
### hotel索引库的自动补全、拼音搜索功能
实现思路如下
1. 修改hotel索引库结构，设置自定义拼音分词器
2. 修改索引库的name、all字段，使用自定义分词器
3. 索引库添加一个新字段suggestion,类型为completion类型，使用自定义的分词器
4. 给HotelDoc类添加suggestion字段,内容包含brand、business
5. 重新导入数据到hotel库
```sql
// 酒店数据索引库
PUT /hotel
{
  "settings": {
    "analysis": {
      "analyzer": {
        "text_anlyzer": {
          "tokenizer": "ik_max_word",
          "filter": "py"
        },
        // 参与自动补全的词条本来是固定的，无需进行分词
        "completion_analyzer": {
          "tokenizer": "keyword",
          "filter": "py"
        }
      },
      "filter": {
        "py": {
          "type": "pinyin",
          "keep_full_pinyin": false,
          "keep_joined_full_pinyin": true,
          "keep_original": true,
          "limit_first_letter_length": 16,
          "remove_duplicated_term": true,
          "none_chinese_pinyin_tokenize": false
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "id":{
        "type": "keyword"
      },
      "name":{
        "type": "text",
        "analyzer": "text_anlyzer",
        "search_analyzer": "ik_smart",
        "copy_to": "all"
      },
      "address":{
        "type": "keyword",
        "index": false
      },
      "price":{
        "type": "integer"
      },
      "score":{
        "type": "integer"
      },
      "brand":{
        "type": "keyword",
        "copy_to": "all"
      },
      "city":{
        "type": "keyword"
      },
      "starName":{
        "type": "keyword"
      },
      "business":{
        "type": "keyword",
        "copy_to": "all"
      },
      "location":{
        "type": "geo_point"
      },
      "pic":{
        "type": "keyword",
        "index": false
      },
      "all":{
        "type": "text",
        // 创建索引时，使用拼音
        "analyzer": "text_anlyzer",
        // 搜索时使用ik_smart，而不需要拼音
        "search_analyzer": "ik_smart"
      },
      "suggestion":{
          "type": "completion",
          "analyzer": "completion_analyzer"
      }
    }
  }
}
```
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Document(indexName = "hotel",createIndex = true)
public class HotelDoc {
    @Id
    @Field(type = FieldType.Keyword)
    private Long id;
    @Field(type = FieldType.Text)
    private String name;
    @Field(type = FieldType.Keyword)
    private String address;
    @Field(type = FieldType.Integer)
    private Integer price;
    @Field(type = FieldType.Integer)
    private Integer score;
    @Field(type = FieldType.Keyword)
    private String brand;
    @Field(type = FieldType.Keyword)
    private String city;
    @Field(type = FieldType.Keyword)
    private String starName;
    @Field(type = FieldType.Keyword)
    private String business;
    /**
     * 位置
     */
    @GeoPointField
    private GeoPoint location;
    @Field(type = FieldType.Keyword)
    private String pic;
    /**
     * 距离
     */
    private Double distance;
    /**
     * 是广告
     */
    private Boolean isAd;
    /**
     * 自动补全
     */
    @Field(type = FieldType.Keyword)
    private List<String> suggestion;
    public HotelDoc(Hotel hotel) {
        this.id = hotel.getId();
        this.name = hotel.getName();
        this.address = hotel.getAddress();
        this.price = hotel.getPrice();
        this.score = hotel.getScore();
        this.brand = hotel.getBrand();
        this.city = hotel.getCity();
        this.starName = hotel.getStarName();
        this.business = hotel.getBusiness();
        this.location=new GeoPoint(Double.parseDouble(hotel.getLatitude()),Double.parseDouble(hotel.getLongitude()));
        this.pic = hotel.getPic();
        // 判断是否有多个值
        if(this.business.contains("/")){
            // 商圈是否被"/"分割
            String[] arr = this.business.split("/");
            // 添加元素
            this.suggestion=new ArrayList<>();
            Collections.addAll(this.suggestion,arr);
        }else {
            this.suggestion= Arrays.asList(this.brand,this.business);
        }
    }
}
```
> 导入数据
```java
    /**
     * 批量插入文档
     * GET /hotel/_doc/36934
     * GET /hotel/_doc/38609
     */
    @Test
    public void batchInsertTest () throws IOException {
        List<Hotel> hotels = hotelService.list();
        List<BulkOperation> bulkOperationList = new ArrayList<>();
        for (Hotel hotel : hotels) {
            HotelDoc hotelDoc = new HotelDoc(hotel);
            bulkOperationList.add(new BulkOperation.Builder().create(e -> e.document(hotelDoc).id(hotel.getId().toString())).build());
        }
        BulkResponse bulkResponse = elasticsearchClient.bulk(bulkRequest ->
                bulkRequest.index(INDEX_NAME).operations(bulkOperationList)
        );
        // 这边插入成功的话显示的是 false
        log.info("== errors: {}", bulkResponse.errors());
    }
```
> 在`kibana`中测试一下
```sql
GET /hotel/_search
{
  "suggest":{
    "suggestions":{
      "text":"sd",
      "completion":{
        "field":"suggestion",
        "skip_duplicates":true,
        "size":10
      }
    }
  }
}
```
> 查询结果
```json
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 0,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "suggest": {
    "suggestions": [
      {
        "text": "sd",
        "offset": 0,
        "length": 2,
        "options": [
          {
            "text": "上地产业园/西三旗",
            "_index": "hotel",
            "_id": "2359697",
            "_score": 1,
            "_source": {
              "id": 2359697,
              "name": "如家酒店(北京上地安宁庄东路店)",
              "address": "清河小营安宁庄东路18号20号楼",
              "price": 420,
              "score": 46,
              "brand": "如家",
              "city": "北京",
              "starName": "二钻",
              "business": "上地产业园/西三旗",
              "location": {
                "lat": 40.041322,
                "lon": 116.333316
              },
              "pic": "https://m.tuniucdn.com/fb3/s1/2n9c/2wj2f8mo9WZQCmzm51cwkZ9zvyp8_w200_h200_c1_t0.jpg",
              "suggestion": [
                "如家",
                "上地产业园/西三旗"
              ]
            }
          },
          {
            "text": "首都机场/新国展地区",
            "_index": "hotel",
            "_id": "395702",
            "_score": 1,
            "_source": {
              "id": 395702,
              "name": "北京首都机场希尔顿酒店",
              "address": "首都机场3号航站楼三经路1号",
              "price": 222,
              "score": 46,
              "brand": "希尔顿",
              "city": "北京",
              "starName": "五钻",
              "business": "首都机场/新国展地区",
              "location": {
                "lat": 40.048969,
                "lon": 116.619566
              },
              "pic": "https://m.tuniucdn.com/fb2/t1/G6/M00/52/10/Cii-U13ePtuIMRSjAAFZ58NGQrMAAGKMgADZ1QAAVn_167_w200_h200_c1_t0.jpg",
              "suggestion": [
                "希尔顿",
                "首都机场/新国展地区"
              ]
            }
          }
        ]
      }
    ]
  }
}
```
### RestApi实现
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.SearchRequest;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import co.elastic.clients.elasticsearch.core.search.CompletionSuggest;
import co.elastic.clients.elasticsearch.core.search.CompletionSuggestOption;
import co.elastic.clients.elasticsearch.core.search.Suggestion;
import com.example.domain.doc.HotelDoc;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
import java.util.List;
/**
 * 自动补全测试
 *
 * @author: 不是菜狗爱编程
 * @date: 2024/04/01/7:13
 * @description:
 */
@SpringBootTest
class CompletionTest {
    private static final String INDEX_NAME = "hotel";
    @Autowired
    private ElasticsearchClient elasticsearchClient;
    /**
     * 自动补全测试
     */
    @Test
    void completionTest() throws IOException {
        SearchRequest.Builder searchBuilder = new SearchRequest.Builder();
        // 设置自动补全名称
        searchBuilder.index(INDEX_NAME).suggest(suggester -> suggester.suggesters("suggestions",
                // 搜索关键字
                suggesters -> suggesters.text("sh").completion(
                        // 补全字段
                        completionSuggester -> completionSuggester.field("suggestion")
                                // 跳过重复项
                                .skipDuplicates(true)
                                // 条数
                                .size(10)
                )
        ));
        SearchResponse<HotelDoc> search = elasticsearchClient.search(searchBuilder.build(), HotelDoc.class);
        // 根据自动补全名称来获取
        List<Suggestion<HotelDoc>> suggestions = search.suggest().get("suggestions");
        for (Suggestion<HotelDoc> suggestion : suggestions) {
            CompletionSuggest<HotelDoc> completion = suggestion.completion();
            for (CompletionSuggestOption<HotelDoc> option : completion.options()) {
                String text = option.text();
                System.out.println("text = " + text);
            }
        }
    }
}
```
### 黑马旅游实现输入框自动补全
> 请求url为`/hotel/suggestion?key=xxx`
```java
@RestController
@RequestMapping("hotel")
public class SearchController {
    @Autowired
    private HotelService hotelService;
    @GetMapping("/suggestion")
    public List<String> suggestion(@RequestParam("key") String prefix){
        return hotelService.getSuggestions(prefix);
    }
}
```
```java
package com.example.service.impl;
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.DistanceUnit;
import co.elastic.clients.elasticsearch._types.FieldValue;
import co.elastic.clients.elasticsearch._types.SortOrder;
import co.elastic.clients.elasticsearch._types.aggregations.Aggregate;
import co.elastic.clients.elasticsearch._types.aggregations.Aggregation;
import co.elastic.clients.elasticsearch._types.aggregations.AggregationBuilders;
import co.elastic.clients.elasticsearch._types.aggregations.StringTermsBucket;
import co.elastic.clients.elasticsearch._types.query_dsl.BoolQuery;
import co.elastic.clients.elasticsearch._types.query_dsl.FunctionScoreMode;
import co.elastic.clients.elasticsearch._types.query_dsl.FunctionScoreQuery;
import co.elastic.clients.elasticsearch._types.query_dsl.QueryBuilders;
import co.elastic.clients.elasticsearch.core.SearchRequest;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import co.elastic.clients.elasticsearch.core.search.*;
import co.elastic.clients.json.JsonData;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.example.domain.Hotel;
import com.example.domain.PageEntity;
import com.example.domain.SearchParams;
import com.example.domain.doc.HotelDoc;
import com.example.mapper.HotelMapper;
import com.example.service.HotelService;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.util.CollectionUtils;
import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
/**
 * @author 不是菜狗爱编程
 * @description 针对表【tb_hotel】的数据库操作Service实现
 * @createDate 2024-03-27 07:42:58
 */
@Service
public class HotelServiceImpl extends ServiceImpl<HotelMapper, Hotel>
        implements HotelService {
    private static final String INDEX_NAME = "hotel";
    @Autowired
    private ElasticsearchClient elasticsearchClient;
    @Override
    public List<String> getSuggestions(String prefix) {
        try {
            // 返回自动补全结果
            List<String> list = new ArrayList<>();
            SearchRequest.Builder searchBuilder = new SearchRequest.Builder();
            // 设置自动补全名称
            searchBuilder.index(INDEX_NAME).suggest(suggester -> suggester.suggesters("suggestions",
                    // 搜索关键字
                    suggesters -> suggesters.text(prefix).completion(
                            // 补全字段
                            completionSuggester -> completionSuggester.field("suggestion")
                                    // 跳过重复项
                                    .skipDuplicates(true)
                                    // 条数
                                    .size(10)
                    )
            ));
            SearchResponse<HotelDoc> search = elasticsearchClient.search(searchBuilder.build(), HotelDoc.class);
            // 根据自动补全名称来获取
            List<Suggestion<HotelDoc>> suggestions = search.suggest().get("suggestions");
            for (Suggestion<HotelDoc> suggestion : suggestions) {
                CompletionSuggest<HotelDoc> completion = suggestion.completion();
                for (CompletionSuggestOption<HotelDoc> option : completion.options()) {
                    String text = option.text();
                    list.add(text);
                }
            }
            return list;
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```
## 数据同步
### 方案选择
>elasticsearch中的酒店数据来自于mysql数据库，因此mysql数据发生改变时，elasticsearch也必须跟着改变,这个就是elasticsearch与mysql之间的`数据同步`。
1. 同步调用
   ![image-20240401081103639](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240401081103639.png)
   > 存在问题：`业务耦合`对性能有影响
2. 异步通知
   ![image-20240401081220463](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240401081220463.png)
   > 消除耦合，提升了性能
3. 监听binlog
   ![image-20240401081305533](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240401081305533.png)
   > 对mysql的压力增加了
   **总结**
   方式一:同步调用
   优点:实现简单,粗暴
   缺点:业务耦合度高
   方式二:异步通知
   优点:低耦合,实现难度一般
   缺点:依赖mq的可靠性
   方式三:监听binlog
   优点:完全解除服务间耦合
   缺点:开启binlog增加数据库负担、实现复杂度高
### MQ实现数据同步
## 集群
