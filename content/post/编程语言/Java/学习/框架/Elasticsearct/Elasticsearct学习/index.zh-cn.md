---
title: Elasticsearct学习
description: Elasticsearct学习
date: 2023-05-30
slug: Elasticsearct学习
image: 202412212115512.png
categories:
    - Elasticsearch
---

# elasticsearch学习
## 索引库操作
### 下载sql文件，执行sql
> #### 执行sql，创建表
[sql文件](https://www.123pan.com/s/tMU0Vv-OvzUd.html)
### 进入kibana
> # **进入[`kibana`](http://localhost:5601/app/dev_tools#/console)**
> 在 Elasticsearch 中，创建文档、索引和 mapping 的顺序如下：
> 1. 创建索引：在 Elasticsearch 中，索引是存储文档的容器。要创建索引，可以使用 PUT 请求，指定索引名称和一些可选的设置。
>
> 2. 创建 mapping：mapping 定义了索引中的字段和它们的数据类型。要创建 mapping，可以使用 PUT 请求，指定索引名称和 mapping 定义。
>
> 3. 创建文档：文档是索引中的数据单元。要创建文档，可以使用 POST 请求，指定索引名称、文档 ID 和文档内容。
>
>   `需要注意的是`，如果在创建文档时指定的索引不存在，Elasticsearch 会自动创建该索引，并使用默认的 mapping。因此，为了确保索引中的字段和数据类型符合预期，最好在创建索引之前先定义好 mapping。
![image-20230525145544098](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132141026.png)
<img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132141404.png" alt="image-20230525150351576" style="zoom:80%;" />
> ### 酒店定义如下，但是`注意一个问题`
>
> name、brand、business等字段都要参与搜索，也就是说用户输入的关键词可能是多个关键字（查询条件不是一个值，而是多个值）
>
> 可以使用`copy_to`属性，将当前字段拷贝到指定字段
```sql
# 酒店的mapping映射（不完美）
PUT /hotel
{
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"
      },
      "name": {
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "address": {
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
        "type": "keyword"
      },
      "city":{
        "type": "keyword"
      },
      "starName":{
        "type": "keyword"
      },
      "business":{
        "type":"keyword"
      },
      "location":{
        "type":"geo_point"
      },
      "pic":{
        "type":"keyword",
        "index":false
      }
    }
  }
}
```
> # 更改后的mapping如下
```sql
# 酒店的mapping映射
PUT /hotel
{
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"
      },
      "name": {
        "type": "text",
        "analyzer": "ik_max_word",
        "copy_to":"all"
      },
      "address": {
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
        "copy_to":"all"
      },
      "city":{
        "type": "keyword"
      },
      "starName":{
        "type": "keyword"
      },
      "business":{
        "type":"keyword",
        "copy_to":"all"
      },
      "location":{
        "type":"geo_point"
      },
      "pic":{
        "type":"keyword",
        "index":false
      },
      "all": {
        "type": "text",
        "analyzer": "ik_max_word"
      }
    }
  }
}
```
![image-20230525152524846](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132141241.png)
### 创建项目
> # 可以看到我这里的版本是`8.6.2`
![image-20230525152822182](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132141441.png)
> pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>demo</description>
    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.3.12.RELEASE</spring-boot.version>
    </properties>
    <dependencies>
        <!--elasticsearch-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
        <!--mybatis plus-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.2</version>
        </dependency>
        <!--fastjson-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.58</version>
        </dependency>
        <!--web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!--test-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
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
                    <mainClass>com.example.demo.DemoApplication</mainClass>
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
</project>
```
### 编写测试类
```java
import lombok.extern.slf4j.Slf4j;
import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
class HotelDocumentTest {
    private RestHighLevelClient client;
    @BeforeEach
    void initClient(){
        this.client=new RestHighLevelClient(RestClient.builder(
                HttpHost.create("http://localhost:9200")
                // 若是集群，可指定多个地址如下
                //HttpHost.create("http://localhost:9200"),
                //HttpHost.create("http://localhost:9200")
        ));
    }
    @AfterEach
    void destroyClient(){
        try {
            this.client.close();
        } catch (IOException e) {
            log.error("关闭资源出错,{}",e.getMessage());
            throw new RuntimeException(e);
        }
    }
    @Test
    void contextLoads() {
        System.out.println(client);
    }
}
```
![image-20230525155822926](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132141743.png)
> 存放静态常量
```java
public class HotelContants {
    public static final String MAPPING_TEMPLATE="{\n" +
            "  \"mappings\": {\n" +
            "    \"properties\": {\n" +
            "      \"id\": {\n" +
            "        \"type\": \"keyword\"\n" +
            "      },\n" +
            "      \"name\": {\n" +
            "        \"type\": \"text\",\n" +
            "        \"analyzer\": \"ik_max_word\",\n" +
            "        \"copy_to\":\"all\"\n" +
            "      },\n" +
            "      \"address\": {\n" +
            "        \"type\": \"keyword\",\n" +
            "        \"index\": false\n" +
            "      },\n" +
            "      \"price\":{\n" +
            "        \"type\": \"integer\"\n" +
            "      },\n" +
            "      \"score\":{\n" +
            "        \"type\": \"integer\"\n" +
            "      },\n" +
            "      \"brand\":{\n" +
            "        \"type\": \"keyword\",\n" +
            "        \"copy_to\":\"all\"\n" +
            "      },\n" +
            "      \"city\":{\n" +
            "        \"type\": \"keyword\"\n" +
            "      },\n" +
            "      \"starName\":{\n" +
            "        \"type\": \"keyword\"\n" +
            "      },\n" +
            "      \"business\":{\n" +
            "        \"type\":\"keyword\",\n" +
            "        \"copy_to\":\"all\"\n" +
            "      },\n" +
            "      \"location\":{\n" +
            "        \"type\":\"geo_point\"\n" +
            "      },\n" +
            "      \"pic\":{\n" +
            "        \"type\":\"keyword\",\n" +
            "        \"index\":false\n" +
            "      },\n" +
            "      \"all\": {\n" +
            "        \"type\": \"text\",\n" +
            "        \"analyzer\": \"ik_max_word\"\n" +
            "      }\n" +
            "    }\n" +
            "  }\n" +
            "}";
}
```
> 编写测试方法
```java
import com.example.demo.constants.HotelContants;
import lombok.extern.slf4j.Slf4j;
import org.apache.http.HttpHost;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.client.indices.CreateIndexRequest;
import org.elasticsearch.common.xcontent.XContentType;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
class HotelDocumentTest {
    private RestHighLevelClient client;
    @BeforeEach
    void initClient(){
        this.client=new RestHighLevelClient(RestClient.builder(
                HttpHost.create("http://localhost:9200")
                // 若是集群，可指定多个地址如下
                //HttpHost.create("http://localhost:9200"),
                //HttpHost.create("http://localhost:9200")
        ));
    }
    @AfterEach
    void destroyClient(){
        try {
            this.client.close();
        } catch (IOException e) {
            log.error("关闭资源出错,{}",e.getMessage());
            throw new RuntimeException(e);
        }
    }
    /**
     * 创建索引库
     */
    @Test
    void createHotelIndexTest(){
        // 1.创建Request对象
        CreateIndexRequest request = new CreateIndexRequest("hotel");
        // 2.请求参数，MAPPING_TEMPLATE是静态常量字符串，内容是创建索引库的DSL语句
        request.source(HotelContants.MAPPING_TEMPLATE, XContentType.JSON);
        // 3.发起请求
        try {
            client.indices().create(request, RequestOptions.DEFAULT);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```
> 运行成功，前往kibana查询索引库
```sql
GET /hotel
```
![image-20230525160904452](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132141049.png)
> 删除索引库、判断索引库是否存在
```java
import com.example.demo.constants.HotelContants;
import lombok.extern.slf4j.Slf4j;
import org.apache.http.HttpHost;
import org.elasticsearch.action.admin.indices.delete.DeleteIndexRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.client.indices.CreateIndexRequest;
import org.elasticsearch.client.indices.GetIndexRequest;
import org.elasticsearch.common.xcontent.XContentType;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
class HotelDocumentTest {
    private RestHighLevelClient client;
    @BeforeEach
    void initClient() {
        this.client = new RestHighLevelClient(RestClient.builder(
                HttpHost.create("http://localhost:9200")
                // 若是集群，可指定多个地址如下
                //HttpHost.create("http://localhost:9200"),
                //HttpHost.create("http://localhost:9200")
        ));
    }
    @AfterEach
    void destroyClient() {
        try {
            this.client.close();
        } catch (IOException e) {
            log.error("关闭资源出错,{}", e.getMessage());
            throw new RuntimeException(e);
        }
    }
    /**
     * 创建索引库
     */
    @Test
    void createHotelIndexTest() {
        // 1.创建Request对象
        CreateIndexRequest request = new CreateIndexRequest("hotel");
        // 2.请求参数，MAPPING_TEMPLATE是静态常量字符串，内容是创建索引库的DSL语句
        request.source(HotelContants.MAPPING_TEMPLATE, XContentType.JSON);
        // 3.发起请求
        try {
            client.indices().create(request, RequestOptions.DEFAULT);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
    /**
     * 删除索引库
     */
    @Test
    void deleteHotelIndexTest() {
        // 1.创建Request对象
        DeleteIndexRequest request = new DeleteIndexRequest("hotel");
        // 2.发起请求
        try {
            client.indices().delete(request, RequestOptions.DEFAULT);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
    /**
     * 判断索引库是否存在
     */
    @Test
    void existsHotelIndexTest() {
        // 1.创建Request对象
        GetIndexRequest request = new GetIndexRequest("hotel");
        // 2.发起请求
        boolean exists = false;
        try {
            exists = client.indices().exists(request, RequestOptions.DEFAULT);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        System.out.println("索引库"+(exists?"存在":"不存在"));
    }
}
```
### 索引库总结
索引库操作的基本步骤
1. 初始化RestHighLevelClient
2. 创建XXXIndexRequest（XXX是create、get、delete）
3. 准备DSL（create时需要语句）
4. 发送请求，调用RestHighLevelClient对象的indices().xxx()方法（xxx是create、exists、delete）
## 文档操作
### 插入数据
```java
import com.alibaba.fastjson.JSON;
import com.example.demo.domain.Hotel;
import com.example.demo.domain.HotelDoc;
import com.example.demo.service.HotelService;
import lombok.extern.slf4j.Slf4j;
import org.apache.http.HttpHost;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
@SpringBootTest
class HotelIndexTest {
    @Autowired
    private HotelService hotelService;
    private RestHighLevelClient client;
    @BeforeEach
    void initClient() {
        this.client = new RestHighLevelClient(RestClient.builder(
                HttpHost.create("http://localhost:9200")
                // 若是集群，可指定多个地址如下
                //HttpHost.create("http://localhost:9200"),
                //HttpHost.create("http://localhost:9200")
        ));
    }
    @AfterEach
    void destroyClient() {
        try {
            this.client.close();
        } catch (IOException e) {
            log.error("关闭资源出错,{}", e.getMessage());
            throw new RuntimeException(e);
        }
    }
    @Test
    void addIndexTest() {
        // hotel对象与索引库不符，经纬度是分开的两个字段，索引库则是一个字段
        // 所以新建一个HotelDoc类与索引库对应
        Hotel hotel = hotelService.getById(61083L);
        HotelDoc hotelDoc = new HotelDoc(hotel);
        // 1.准备request对象
        IndexRequest request = new IndexRequest("hotel").id(hotel.getId().toString());
        // 2.准备json文档
        request.source(JSON.toJSONString(hotelDoc), XContentType.JSON);
        // 3.发送请求
        try {
            client.index(request, RequestOptions.DEFAULT);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```
```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
@Data
@AllArgsConstructor
@NoArgsConstructor
public class HotelDoc {
    private Long id;
    private String name;
    private String address;
    private Integer price;
    private Integer score;
    private String brand;
    private String city;
    private String starName;
    private String business;
    private String location;
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
        this.location = hotel.getLatitude() + "," + hotel.getLongitude();
        this.pic = hotel.getPic();
    }
}
```
> ### 这里运行报错了，不过不影响，插入数据已经成功
>
> kibana使用`GET /hotel/_doc/61083`查询
>
> [参考博客](https://blog.csdn.net/qq_34263207/article/details/127788269)
![image-20230525173745593](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132141135.png)
查看日志打印，发现
![image-20230525174026003](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132141829.png)
es版本>8 推介查看[博客](https://blog.csdn.net/qq_34263207/article/details/127847033)
> # 添加依赖和配置
```xml
        <dependency>
            <groupId>co.elastic.clients</groupId>
            <artifactId>elasticsearch-java</artifactId>
            <version>8.1.1</version>
        </dependency>
        <dependency>
            <groupId>jakarta.json</groupId>
            <artifactId>jakarta.json-api</artifactId>
            <version>2.0.1</version>
        </dependency>
```
```yaml
spring:
  es:
    address: localhost
    port: 9200
    scheme: http
    username: elastic
    password: 123456
```
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.json.jackson.JacksonJsonpMapper;
import co.elastic.clients.transport.ElasticsearchTransport;
import co.elastic.clients.transport.rest_client.RestClientTransport;
import org.apache.http.HttpHost;
import org.apache.http.auth.AuthScope;
import org.apache.http.auth.UsernamePasswordCredentials;
import org.apache.http.client.CredentialsProvider;
import org.apache.http.impl.client.BasicCredentialsProvider;
import org.apache.http.impl.nio.client.HttpAsyncClientBuilder;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestClientBuilder;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class ElasticSearchConfig {
    @Value("${spring.es.address}")
    String address;
    @Value("${spring.es.port}")
    Integer port;
    @Value("${spring.es.scheme}")
    String scheme;
    @Value("${spring.es.username}")
    String username;
    @Value("${spring.es.password}")
    String password;
    @Bean
    public ElasticsearchClient esRestClientWithCred(){
        final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
        // 配置连接ES的用户名和密码，如果没有用户名和密码可以不加这一行
        credentialsProvider.setCredentials(AuthScope.ANY, new UsernamePasswordCredentials(username, password));
        RestClientBuilder restClientBuilder = RestClient.builder(new HttpHost(address, port, scheme))
                .setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
                    @Override
                    public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpAsyncClientBuilder) {
                        return httpAsyncClientBuilder.setDefaultCredentialsProvider(credentialsProvider);
                    }
                });
        RestClient restClient = restClientBuilder.build();
        ElasticsearchTransport transport = new RestClientTransport(
                restClient, new JacksonJsonpMapper());
        return new ElasticsearchClient(transport);
    }
}
```
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.GetResponse;
import co.elastic.clients.elasticsearch.core.IndexRequest;
import com.alibaba.fastjson.JSON;
import com.example.demo.domain.Hotel;
import com.example.demo.domain.HotelDoc;
import com.example.demo.service.HotelService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
@SpringBootTest
class HotelIndexElasticTest {
    @Autowired
    private HotelService hotelService;
    @Autowired
    ElasticsearchClient client;
    @Test
    void addIndexTest() {
        // 创建要插入的实体
        // hotel对象与索引库不符，经纬度是分开的两个字段，索引库则是一个字段
        // 所以新建一个HotelDoc类与索引库对应
        Hotel hotel = hotelService.getById(61083);
        HotelDoc hotelDoc = new HotelDoc(hotel);
        // 方法一
        IndexRequest<Object> indexRequest = new IndexRequest.Builder<>()
                // 博客中并未添加这句话，需要自己指定id，否则 GET /hotel/_doc/61083 在kibana中无法查询到相关数据
                .id(hotel.getId().toString())
                .index("hotel")
                .document(hotelDoc)
                .build();
        try {
            client.index(indexRequest);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```
> ### 在上述代码中遇到过一个`问题`
>
> 正常运行结束，但是在kibana中无法查询到数据
>
> 是因为没有手动指定id，具体查看上述代码注释
> #### 采用lambda表达式的写法
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.GetResponse;
import co.elastic.clients.elasticsearch.core.IndexRequest;
import com.alibaba.fastjson.JSON;
import com.example.demo.domain.Hotel;
import com.example.demo.domain.HotelDoc;
import com.example.demo.service.HotelService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
@SpringBootTest
class HotelIndexElasticTest {
    @Autowired
    private HotelService hotelService;
    @Autowired
    ElasticsearchClient client;
    @Test
    void addIndexTest2() {
        // 创建要插入的实体
        // hotel对象与索引库不符，经纬度是分开的两个字段，索引库则是一个字段
        // 所以新建一个HotelDoc类与索引库对应
        Hotel hotel = hotelService.getById(61083);
        HotelDoc hotelDoc = new HotelDoc(hotel);
        // 方法二
        try {
            client.index(item -> item
                    .id(hotel.getId().toString())
                    .index("hotel")
                    .document(hotelDoc));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```
### 查询数据
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.GetResponse;
import co.elastic.clients.elasticsearch.core.IndexRequest;
import com.alibaba.fastjson.JSON;
import com.example.demo.domain.Hotel;
import com.example.demo.domain.HotelDoc;
import com.example.demo.service.HotelService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
@SpringBootTest
class HotelIndexElasticTest {
    @Autowired
    private HotelService hotelService;
    @Autowired
    ElasticsearchClient client;
    /**
     * 获取索引
     */
    @Test
    void getIndexTest(){
        String id = "61083";
        GetResponse<HotelDoc> response = null;
        try {
            response = client.get(g -> g
                            .index("hotel")
                            .id(id),
                    HotelDoc.class
            );
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        if (response.found()) {
            HotelDoc hotelDoc = response.source();
            log.info("返回结果 " + JSON.toJSONString(hotelDoc));
        } else {
            log.info("数据不存在");
        }
    }
}
```
### 修改数据
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.GetResponse;
import co.elastic.clients.elasticsearch.core.IndexRequest;
import com.alibaba.fastjson.JSON;
import com.example.demo.domain.Hotel;
import com.example.demo.domain.HotelDoc;
import com.example.demo.service.HotelService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
@SpringBootTest
class HotelIndexElasticTest {
    @Autowired
    private HotelService hotelService;
    @Autowired
    ElasticsearchClient client;
    
    @Test
    void updateIndexTest() {
        Hotel hotel = hotelService.getById(61083);
        HotelDoc hotelDoc = new HotelDoc(hotel);
        // 将城市从“上海”变为“不在上海”
        hotelDoc.setCity("不在上海");
        try {
            client.update(item -> item
                            .index("hotel")
                            .doc(hotelDoc)
                            .id(hotel.getId().toString()),
                    HotelDoc.class
            );
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```
![image-20230525183341843](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132141653.png)
### 删除数据
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.GetResponse;
import co.elastic.clients.elasticsearch.core.IndexRequest;
import com.alibaba.fastjson.JSON;
import com.example.demo.domain.Hotel;
import com.example.demo.domain.HotelDoc;
import com.example.demo.service.HotelService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
@SpringBootTest
class HotelIndexElasticTest {
    @Autowired
    private HotelService hotelService;
    @Autowired
    ElasticsearchClient client;
    /**
     * 删除索引测试
     */
    @Test
    void deleteIndexTest() {
        Hotel hotel = hotelService.getById(61083);
        try {
            client.delete(item -> item
                    .id(hotel.getId().toString())
                    .index("hotel"));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```
![image-20230525183714877](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132142573.png)
### 批量导入数据
> # 利用JavaRestClient批量导入酒店数据到ES
>
> 需求:批量查询酒店数据，然后批量导入索引库中
>
> 思路:
>
> 1. 利用mybatis-plus查询酒店数据
> 2. 将查询到的酒店数据(Hotel) 转换为文档类型数据( HotelDoc )
> 3. 利用JavaRestClient中 的Bulk批处理，实现批量新增文档
<img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132142688.png" alt="image-20230525211027361" style="zoom:80%;" />
> ### 黑马程序员里的写法如上图所示
>
> 但是由于我在windows安装的elasticsearch的版本问题，所以采用如下写法
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.BulkRequest;
import co.elastic.clients.elasticsearch.core.GetResponse;
import co.elastic.clients.elasticsearch.core.IndexRequest;
import co.elastic.clients.elasticsearch.core.bulk.BulkOperation;
import co.elastic.clients.elasticsearch.core.bulk.IndexOperation;
import com.alibaba.fastjson.JSON;
import com.example.demo.domain.Hotel;
import com.example.demo.domain.HotelDoc;
import com.example.demo.service.HotelService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
@Slf4j
@SpringBootTest
class HotelIndexElasticTest {
    @Autowired
    private HotelService hotelService;
    @Autowired
    ElasticsearchClient client;
    /**
     * 批量导入数据测试
     */
    @Test
    void bulkIndexListTest() {
        // 1.创建Bulk请求
        BulkRequest bulkRequest = null;
        List<BulkOperation> bulkOperations = new ArrayList<>();
        for (Hotel hotel : hotelService.list()) {
            HotelDoc hotelDoc = new HotelDoc(hotel);
            IndexOperation<Object> build = new IndexOperation.Builder<>()
                    .id(hotelDoc.getId().toString())
                    .index("hotel")
                    .document(hotelDoc).build();
            bulkOperations.add(build._toBulkOperation());
        }
        // 2.添加要批量提交的请求
        bulkRequest = new BulkRequest.Builder()
                .index("hotel")
                .operations(bulkOperations)
                .build();
        System.out.printf("一共插入了%s条数据%n",bulkRequest.operations().size());
        // 3.发起bulk请求
        try {
            client.bulk(bulkRequest);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```
> # 批量查询
```sql
GET /hotel/_search
```
### 文档总结
文档操作的基本步骤
1. 初始化RestHighLevelClient
2. 创建XXXRequest。（XXX是Index、Get、Update、Delete）
3. 准备参数（Index和Update时需要）
4. 发送请求，调用RestHighLevelClient对象的xxx()方法（xxx是index、get、update、delete）
5. 解析结果（get时需要）
## DSL查询及案例
### DSL查询语法
#### 基本语法
```sql
# 查询所有
GET /hotel/_search
{
  "query":{
    "match_all": {}
  }
}
```
#### 全文检索查询
```sql
# match查询
GET /hotel/_search
{
  "query":{
    "match":{
      "all":"外滩"
    }
  }
}
```
>## 如下查询与上面的查询效果一样，但是由于检索多个字段，效率更低
>
>推介使用上面的查询`（copy_to）`
```sql
# multi_match查询
GET /hotel/_search
{
  "query":{
    "multi_match": {
      "query": "外滩如家",
      "fields": ["brand","name","business"]
    }
  }
}
```
#### 精确查询
> ## 精确匹配不会对输入的内容进行分词，`必须完全一致`
>
> 假如:输入的是`上海杭州`，则查询结果为0
```sql
# term查询
GET /hotel/_search
{
  "query":{
    "term":{
      "city":{
        "value":"上海"
      }
    }
  }
}
```
> ## 价格大于等于100，小于等于300
>
> `gt`为大于，`gte`为大于等于；`lte和lt同理`
```sql
# range查询
GET /hotel/_search
{
  "query":{
    "range":{
      "price": {
        "gte": 100,
        "lte": 300
      }
    }
  }
}
```
#### 地理查询
> ## 查询在一个矩形范围内的地点
>
> `top_left(左上角)`和`bottom_right(右下角)`分别确定一个点，两个点水平作两条直线得到的`矩形范围`
<img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132142348.png" alt="image-20230525222759408" style="zoom: 67%;" />
```sql
# geo_bounding_box查询
GET /hotel/_search
{
  "query":{
    "geo_bounding_box": {
      "location": {
        "top_left": {
          "lat": 31.1,
          "lon": 121.5
        },
        "bottom_right": {
          "lat": 30.9,
          "lon": 121.7
        }
      }
    }
  }
}
```
> ## 查询距离一个`点`一定范围内的数据
<img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132142284.png" alt="image-20230525222958773" style="zoom:67%;" />
```sql
# geo_distance查询
GET /hotel/_search
{
  "query":{
    "geo_distance": {
      "distance": "15km",
      "location": {
        "lat": 31.21,
        "lon": 121.5
      }
    }
  }
}
```
> 也可以使用下面这种写法
```sql
# geo_distance查询
GET /hotel/_search
{
  "query":{
    "geo_distance": {
      "distance": "15km",
      "location": "31.21,121.5"
    }
  }
}
```
#### Function score query
![image-20230525224121984](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132142645.png)
> # 案例
>
> 让“如家”这个品牌的酒店排名靠前一些
```sql
# 让“如家”这个品牌的酒店排名靠前一些
# function score查询
GET /hotel/_search
{
  "query":{
    "function_score": {
      "query": {
        "match": {
          "all": "如家"
        }
      },
      "functions": [
        {
          "filter":{
            "term":{
              "brand":"如家"
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
#### 复合查询Boolean Query
>布尔查询是-个或多个查询子句的组合。子查询的组合方式有:
>1. must:必须匹配每个子查询，类似“与”
>2. should:选择性匹配子查询，类似“或"
>3. must_ not:必须不匹配，`不参与算分`，类似“非”
>4. filter:必须匹配，`不参与算分`
```sql
# city在上海、brand是“皇冠假日”和“华美达”中的一个、price不能小于等于500、score大于等于45
GET /hotel/_search
{
  "query":{
    "bool": {
      "must": [
        {
          "term": {
            "city": {
              "value": "上海"
            }
          }
        }
      ],
      "should": [
        {
          "term": {
            "brand": {
              "value": "皇冠假日"
            }
          }
        },
        {
          "term": {
            "brand": {
              "value": "华美达"
            }
          }
        }
      ],
      "must_not": [
        {
          "range": {
            "price": {
              "lte": 500
            }
          }
        },
        {
          "range": {
            "score": {
              "gte": 45
            }
          }
        }
      ]
    }
  }
}
```
> # 案例练习
>
> 需求：搜索名字包含`如家`，价格不高于400，在坐标31.21、121.5周围10km范围内的酒店
```sql
# 搜索名字包含`如家`，价格不高于400，在坐标31.21、121.5周围10km范围内的酒店
GET /hotel/_search
{
  "query":{
    "bool": {
      "must": [
        {
          "match": {
            "name": "如家"
          }
        }
      ],
      "must_not": [
        {
          "range": {
            "price": {
              "gt": 400
            }
          }
        }
      ],
      "filter": [
        {
          "geo_distance": {
            "distance": "10km",
            "location": {
              "lat": 31.21,
              "lon": 121.5
            }
          }
        }
      ]
    }
  }
}
```
### 搜索结果处理
#### 排序
```sql
# 排序，排序字段和排序方式 asc、desc
GET /hotel/_search
{
  "query":{
    "match_all": {}
  },
  "sort":{
    "price":"asc"
  }
}
```
> # 地理位置排序
```sql
# 排序，排序字段和排序方式 asc、desc
GET /hotel/_search
{
  "query":{
    "match_all": {}
  },
  "sort":{
    "_geo_distance":{
      "location":{
        "lat": 31.21,
        "lon": 121.5
      },
      "order":"asc",
      "unit":"km"
    }
  }
}
```
> 也可以采用如下写法
```sql
# 排序，排序字段和排序方式 asc、desc
GET /hotel/_search
{
  "query":{
    "match_all": {}
  },
  "sort":{
    "_geo_distance":{
      "location":"31.21,121.5",
      "order":"asc",
      "unit":"km"
    }
  }
}
```
> # 案例：对酒店数据按照用户评价降序排序，评价相同的按照价格升序排序
```sql
# 对酒店数据按照用户评价降序排序，评价相同的按照价格升序排序
GET /hotel/_search
{
  "query":{
    "match_all": {}
  },
  "sort":[
    {
      "score":"desc"
    },
    {
      "price":"asc"
    }
  ]
}
```
> # 案例：实现对酒店数据按照位置坐标的距离升序排序
```sql
# 实现对酒店数据按照位置坐标的距离升序排序
GET /hotel/_search
{
  "query":{
    "match_all": {}
  },
  "sort":{
    "_geo_distance":{
      "location":"31.21,121.5",
      "order":"asc",
      "unit":"km"
    }
  }
}
# 或者采用如下地理位置写法
# 实现对酒店数据按照位置坐标的距离升序排序
GET /hotel/_search
{
  "query":{
    "match_all": {}
  },
  "sort":{
    "_geo_distance":{
      "location":{
        "lat":31.21,
        "lon":121.5
      },
      "order":"asc",
      "unit":"km"
    }
  }
}
```
#### 分页
>elasticsearch默认情况下只返回top10的数据。而如果要查询更多数据就需要修改分页参数了。
>
>elasticsearch中通过修改`from`、`size`参 数来控制要返回的分页结果
```sql
# 分页查询
GET /hotel/_search
{
  "query":{
    "match_all": {}
  },
  "sort":[
    {
      "price":"asc"
    }
  ],
  "from":0,
  "size":5
}
```
> ## `深度分页问题`
>
> ES是分布式的，所以会面临深度分页问题。例如按price排序后，获取from = 990，size =10的数据:
<img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132142794.png" alt="image-20230526095625293" style="zoom:67%;" />
> #### 如果希望查找前1000条数据，`正确`的做法是取每个片上的前1000条数据，然后将这些数据重新排序
>
> 1. 首先在每个数据分片.上都排序并查询前1000条文档。
> 2. 然后将所有节点的结果聚合，在内存中重新排序选出前1000条文档
> 3. 最后从这1000条中，选取从990开始的10条文档
>
> `注意`：如果搜索页数过深，或者结果集(from + size)越大，对内存和CPU的消耗也越高。因此ES设定结果集查询的.上限是10000
>
> 针对深度分页，ES提供了两种`解决方案`,官方文档:
>
> search after:分页时需要排序，原理是从上一-次的排序值开始，查询下一页数据。官方推荐使用的方式。
>
> scroll:原理将排序数据形成快照，保存在内存。官方已经不推荐使用。
`from + size`:
优点:支持随机翻页
缺点:深度分页问题，默认查询上限(from + size)是10000
场景:百度、京东、谷歌、淘宝这样的`随机翻页搜索`
`after search`:
优点:没有查询上限(单次查询的size不超过10000 )
缺点:只能向后逐页查询，不支持随机翻页
场景:没有随机翻页需求的搜索，例如`手机向下滚动翻页`
`scroll`:
优点:没有查询.上限(单次查询的size不超过10000)
缺点:会有额外内存消耗，并且搜索结果是非实时的
场景:海量数据的获取和迁移。从ES7.1开始不推荐，建议用aftersearch方案
#### 高亮
```sql
# 高亮查询
GET /hotel/_search
{
  "query":{
    "match": {
      "all": "如家"
    }
  },
  "highlight":{
    // 指定要高亮的字段
    "fields": {
      "all": {
        // 用来标记高亮字段的前置标签
        "pre_tags": "<em>",
        // 用来标记高亮字段的后置标签
        "post_tags": "</em>"
      }
    }
  }
}
```
> ### 注意这里不能使用`match_all`，因为需要指定搜索的关键字
>
> 同时，默认情况下 ES搜索字段必须与高亮字段一致，上述案例中都是`all`，请看下面案例
```sql
# 高亮查询
GET /hotel/_search
{
  "query":{
    "match": {
      "all": "如家"
    }
  },
  "highlight":{
    // 指定要高亮的字段
    "fields": {
      "name": {
      	// 搜索字段和高亮字段可以不匹配
        "require_field_match": "false", 
        // 用来标记高亮字段的前置标签
        "pre_tags": "<em>",
        // 用来标记高亮字段的后置标签
        "post_tags": "</em>"
      }
    }
  }
}
```
### RestClient查询
#### match_all
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.SearchRequest;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import com.example.demo.domain.HotelDoc;
import com.example.demo.service.HotelService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
@SpringBootTest
class HotelSearchTest {
    @Autowired
    private HotelService hotelService;
    @Autowired
    ElasticsearchClient client;
    @Test
    void matchAllTest() throws IOException {
        SearchRequest request = new SearchRequest.Builder()
            	.index("hotel")
                .query(q -> q.matchAll(matchAllQuery -> matchAllQuery))
                .build();
        SearchResponse<HotelDoc> matchSearch = client.search(request, HotelDoc.class);
        log.info("共搜索到:{}条数据", (matchSearch.hits().total() == null ? 0 : matchSearch.hits().total().value()));
        matchSearch.hits().hits().forEach(item -> {
            System.out.println(item.source());
        });
    }
}
```
#### match
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.SearchRequest;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import com.example.demo.domain.HotelDoc;
import com.example.demo.service.HotelService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
@SpringBootTest
class HotelSearchTest {
    @Autowired
    private HotelService hotelService;
    @Autowired
    ElasticsearchClient client;
    @Test
    void matchTest() throws IOException {
        SearchRequest request = new SearchRequest.Builder()
                .index("hotel")
                .query(query->query.match(
                        matchQuery->matchQuery.field("brand").query("如家")
                ))
                .build();
        SearchResponse<HotelDoc> matchSearch = client.search(request, HotelDoc.class);
        matchSearch.hits().hits().forEach(item -> {
            System.out.println(item.source());
        });
    }
}
```
#### term
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.SearchRequest;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import com.example.demo.domain.HotelDoc;
import com.example.demo.service.HotelService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
@SpringBootTest
class HotelSearchTest {
    @Autowired
    private HotelService hotelService;
    @Autowired
    ElasticsearchClient client;
	
    @Test
    void termQueryTest() throws IOException {
        SearchRequest request = new SearchRequest.Builder()
                .index("hotel")
                .query(query -> query.term(
                        termQuery -> termQuery.field("city").value("北京")
                ))
                .build();
        SearchResponse<HotelDoc> matchSearch = client.search(request, HotelDoc.class);
        matchSearch.hits().hits().forEach(item -> {
            System.out.println(item.source());
        });
    }
}
```
#### range
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.SearchRequest;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import com.example.demo.domain.HotelDoc;
import com.example.demo.service.HotelService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
@SpringBootTest
class HotelSearchTest {
    @Autowired
    private HotelService hotelService;
    @Autowired
    ElasticsearchClient client;
    @Test
    void termQueryTest() throws IOException {
        SearchRequest request = new SearchRequest.Builder()
                .index("hotel")
                .query(query -> query.range(
                        rangeQuery->rangeQuery
                                .field("price")
                                .gte(JsonData.of(100)).lte(JsonData.of(300))
                ))
                .build();
        SearchResponse<HotelDoc> matchSearch = client.search(request, HotelDoc.class);
        matchSearch.hits().hits().forEach(item -> {
            System.out.println(item.source());
        });
    }
}
```
#### bool
> # 搜索名字包含`如家`，价格不高于400，在坐标31.21、121.5周围10km范围内的酒店（案例在之前boolean query部分出现过）
```sql
# 搜索名字包含`如家`，价格不高于400，在坐标31.21、121.5周围10km范围内的酒店
GET /hotel/_search
{
  "query":{
    "bool": {
      "must": [
        {
          "match": {
            "name": "如家"
          }
        }
      ],
      "must_not": [
        {
          "range": {
            "price": {
              "gt": 400
            }
          }
        }
      ],
      "filter": [
        {
          "geo_distance": {
            "distance": "10km",
            "location": {
              "lat": 31.21,
              "lon": 121.5
            }
          }
        }
      ]
    }
  }
}
```
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.SearchRequest;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import com.example.demo.domain.HotelDoc;
import com.example.demo.service.HotelService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
@SpringBootTest
class HotelSearchTest {
    @Autowired
    private HotelService hotelService;
    @Autowired
    ElasticsearchClient client;
    @Test
    void boolQueryTest() throws IOException {
        SearchRequest request = new SearchRequest.Builder()
                .index("hotel")
                .query(query -> query.bool(
                        boolQuery -> boolQuery
                                .must(mustQuery -> mustQuery
                                        .match(matchQuery -> matchQuery
                                                .field("name").query("如家"))
                                )
                                .mustNot(mustNotQuery -> mustNotQuery
                                        .range(rangeQuery -> rangeQuery
                                                .field("price")
                                                .gt(JsonData.of(400))
                                        )
                                )
                                .filter(filterQuery -> filterQuery
                                        .geoDistance(geoDistanceQuery ->
                                                geoDistanceQuery
                                                        .distance("10km")
                                                        .field("location")
                                                        .location(geoLocation -> geoLocation
                                                                .latlon(new LatLonGeoLocation.Builder()
                                                                        .lat(31.21)
                                                                        .lon(121.5)
                                                                        .build()
                                                                )
                                                        )
                                        )
                                )
                ))
                .build();
        SearchResponse<HotelDoc> matchSearch = client.search(request, HotelDoc.class);
        matchSearch.hits().hits().forEach(item -> {
            System.out.println(item.source());
        });
    }	
}
```
> # 也可以使用下面的写法
>
> `注意`：QueryBuilders的包别导错了
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.LatLonGeoLocation;
import co.elastic.clients.elasticsearch._types.query_dsl.BoolQuery;
import co.elastic.clients.elasticsearch._types.query_dsl.QueryBuilders;
import co.elastic.clients.elasticsearch.core.SearchRequest;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import co.elastic.clients.json.JsonData;
import com.example.demo.domain.HotelDoc;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
@SpringBootTest
class HotelSearchTest {
    @Autowired
    ElasticsearchClient client;
    @Test
    void boolQueryTest() throws IOException {
        BoolQuery build = new BoolQuery.Builder()
                .must(QueryBuilders.term().field("name").value("如家").build()._toQuery())
                .mustNot(QueryBuilders.range().field("price").gt(JsonData.of(400)).build()._toQuery())
                .filter(QueryBuilders.geoDistance().field("location")
                        .distance("10km")
                        .location(item -> item.latlon(new LatLonGeoLocation.Builder().lat(31.21).lon(121.5).build()))
                        .build()._toQuery())
                .build();
        SearchRequest request = new SearchRequest.Builder()
                .index("hotel")
                .query(build._toQuery())
                .build();
        SearchResponse<HotelDoc> matchSearch = client.search(request, HotelDoc.class);
        matchSearch.hits().hits().forEach(item -> {
            System.out.println(item.source());
        });
    }
}
```
fuzzy
>模糊查询（fuzzy query）是一种搜索技术，它允许在搜索时使用模糊匹配，以便在搜索结果中包含与搜索词项相似但不完全匹配的项。模糊查询通常用于处理拼写错误、同义词、缩写词、音近字等情况。在实现模糊查询时，常用的算法包括编辑距离算法、n-gram算法、Soundex算法等。在搜索引擎、数据库、信息检索等领域中，模糊查询是一种常见的技术手段。
```sql
GET /hotel/_search
{
  "query":{
    "fuzzy": {
      "brand": {
        "fuzziness": "2",
        "value": "儒家"
      }
    }
  }
}
```
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.LatLonGeoLocation;
import co.elastic.clients.elasticsearch._types.query_dsl.BoolQuery;
import co.elastic.clients.elasticsearch._types.query_dsl.QueryBuilders;
import co.elastic.clients.elasticsearch.core.SearchRequest;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import co.elastic.clients.json.JsonData;
import com.example.demo.domain.HotelDoc;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
@SpringBootTest
class HotelSearchTest {
    @Autowired
    ElasticsearchClient client;
    @Test
    void fuzzyQueryTest() throws IOException {
        SearchResponse<HotelDoc> request = client.search(searchRequest -> searchRequest
                .index("hotel")
                .query(query -> query
                        .fuzzy(fuzzyQuery -> fuzzyQuery
                                .field("brand")
                                .fuzziness("auto")
                                .value("儒家")
                        )
                ), HotelDoc.class);
        log.info("共搜索到:{}条数据", (request.hits().total() == null ? 0 : request.hits().total().value()));
        request.hits().hits().forEach(item -> {
            System.out.println(item.source());
        });
    }
}
```
#### 排序和分页
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.LatLonGeoLocation;
import co.elastic.clients.elasticsearch._types.SortOrder;
import co.elastic.clients.elasticsearch._types.query_dsl.BoolQuery;
import co.elastic.clients.elasticsearch._types.query_dsl.QueryBuilders;
import co.elastic.clients.elasticsearch.core.SearchRequest;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import co.elastic.clients.json.JsonData;
import com.example.demo.domain.HotelDoc;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
@SpringBootTest
class HotelSearchTest {
    @Autowired
    ElasticsearchClient client;
    /**
     * 分页、排序测试
     */
    @Test
    void pageAndSortTest() throws IOException {
        // 页码、每页信息条数
        int page = 1, size = 10;
        SearchRequest request = new SearchRequest.Builder()
                .index("hotel")
                .query(builder -> builder.match(matchAllQuery -> matchAllQuery.field("brand").query("如家")))
                .from((page - 1) * size)
                .size(size)
                .sort(sortOptions -> sortOptions.field(fieldSort -> fieldSort.field("price").order(SortOrder.Asc)))
                .build();
        SearchResponse<HotelDoc> search = client.search(request, HotelDoc.class);
        log.info("共搜索到:{}条数据", (search.hits().total() == null ? 0 : search.hits().total().value()));
        search.hits().hits().forEach(item -> {
            System.out.println(item.source());
        });
    }
}
```
#### 高亮
> # 总体来说不算太难，对应sql语句使用idea的提示基本就可以凑出来
```sql
GET /hotel/_search
{
  "query":{
    "match": {
      "all": "如家"
    }
  },
  "highlight":{
    "fields": {
      "name": {
        "require_field_match": "false", 
        "pre_tags":"<em>",
        "post_tags": "</em>"
      }
    }
  }
}
```
```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.LatLonGeoLocation;
import co.elastic.clients.elasticsearch._types.SortOrder;
import co.elastic.clients.elasticsearch._types.query_dsl.BoolQuery;
import co.elastic.clients.elasticsearch._types.query_dsl.QueryBuilders;
import co.elastic.clients.elasticsearch.core.SearchRequest;
import co.elastic.clients.elasticsearch.core.SearchResponse;
import co.elastic.clients.json.JsonData;
import com.example.demo.domain.HotelDoc;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
@Slf4j
@SpringBootTest
class HotelSearchTest {
    @Autowired
    ElasticsearchClient client;
    /**
     * 高亮测试
     */
    @Test
    void highLightTest() throws IOException {
        SearchRequest request = new SearchRequest.Builder()
                .index("hotel")
                .query(query->query.match(matchQuery->matchQuery.field("all").query("如家")))
                .highlight(highLight->highLight.fields("name",builder -> builder.preTags("<em>").postTags("</em>")).requireFieldMatch(false))
                .build();
        SearchResponse<HotelDoc> search = client.search(request, HotelDoc.class);
        log.info("共搜索到:{}条数据", (search.hits().total() == null ? 0 : search.hits().total().value()));
        search.hits().hits().forEach(item -> {
            item.highlight().values().forEach(System.out::println);
        });
    }
}
```
