---
title: 多级缓存
description: 多级缓存
date: 2024-06-01
slug: 多级缓存
image: pawel-czerwinski-8uZPynIu-rQ-unsplash_hu3425483315149503896.jpg
categories:
    - 多级缓存
---

# 多级缓存
## 传统缓存的问题
> 传统的缓存策略一般是请求到达Tomcat后，先查询Redis,如果未命中则查询数据库，存在下面的问题:
请求要经过Tomcat处理，Tomcat的性能成为整个系统的瓶颈
Redis缓存失效时，会对数据库产生冲击
![image-20240424214450555](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242144598.png)
## 多级缓存方案
多级缓存就是充分利用请求处理的每个环节,分别添加缓存，减轻Tomcat压力，提升服务性能:
![image-20240424214727874](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242147940.png)
> 减轻了`tomcat`的压力，避免了对数据库的压力，需要在nginx内部实现对于redis、tomcat访问的业务代码
>
> 将来会将`nginx`部署为集群
![image-20240424214908046](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242149087.png)
1. JVM进程缓存(Tomcat缓存)
2. Lua语法入门
3. 实现多级缓存
4. 缓存同步策略
## 环境配置
### MySQL
> 后期做数据同步需要用到MySQL的主从功能，所以需要大家在虚拟机中，利用Docker来运行一个MySQL容器。
>
> 为了方便后期配置MySQL，我们先准备两个目录，用于挂载容器的数据和配置文件目录：
```sh
# 进入/tmp目录
cd /tmp
# 创建文件夹
mkdir mysql
# 进入mysql目录
cd mysql
```
### 运行命令
> 在/tmp/mysql/conf目录添加一个my.cnf文件，作为mysql的配置文件：
```sh
# 创建文件
touch /tmp/mysql/conf/my.cnf
```
**文件内容如下**
```conf
[mysqld]
skip-name-resolve
character_set_server=utf8
datadir=/var/lib/mysql
server-id=1000
```
```sh
docker run \
 -p 3307:3306 \
 --name mysql-demo \
 -v $PWD/conf:/etc/mysql/conf.d \
 -v $PWD/logs:/logs \
 -v $PWD/data:/var/lib/mysql \
 -e MYSQL_ROOT_PASSWORD=123456 \
 --privileged \
 -d \
 mysql:5.7.25
```
**重启容器**
```sh
docker restart mysql-demo
```
navicat新建连接
![image-20240425074130881](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404250741940.png)
新建数据库
![image-20240425074220509](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404250742532.png)
执行sql
```sql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;
DROP TABLE IF EXISTS `tb_item`;
CREATE TABLE `tb_item`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '商品id',
  `title` varchar(264) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '商品标题',
  `name` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '' COMMENT '商品名称',
  `price` bigint(20) NOT NULL COMMENT '价格（分）',
  `image` varchar(200) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '商品图片',
  `category` varchar(200) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '类目名称',
  `brand` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '品牌名称',
  `spec` varchar(200) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '规格',
  `status` int(1) NULL DEFAULT 1 COMMENT '商品状态 1-正常，2-下架，3-删除',
  `create_time` datetime NULL DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime NULL DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`) USING BTREE,
  INDEX `status`(`status`) USING BTREE,
  INDEX `updated`(`update_time`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 50002 CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '商品表' ROW_FORMAT = COMPACT;
INSERT INTO `tb_item` VALUES (10001, 'RIMOWA 21寸托运箱拉杆箱 SALSA AIR系列果绿色 820.70.36.4', 'SALSA AIR', 16900, 'https://m.360buyimg.com/mobilecms/s720x720_jfs/t6934/364/1195375010/84676/e9f2c55f/597ece38N0ddcbc77.jpg!q70.jpg.webp', '拉杆箱', 'RIMOWA', '{\"颜色\": \"红色\", \"尺码\": \"26寸\"}', 1, '2019-05-01 00:00:00', '2019-05-01 00:00:00');
INSERT INTO `tb_item` VALUES (10002, '安佳脱脂牛奶 新西兰进口轻欣脱脂250ml*24整箱装*2', '脱脂牛奶', 68600, 'https://m.360buyimg.com/mobilecms/s720x720_jfs/t25552/261/1180671662/383855/33da8faa/5b8cf792Neda8550c.jpg!q70.jpg.webp', '牛奶', '安佳', '{\"数量\": 24}', 1, '2019-05-01 00:00:00', '2019-05-01 00:00:00');
INSERT INTO `tb_item` VALUES (10003, '唐狮新品牛仔裤女学生韩版宽松裤子 A款/中牛仔蓝（无绒款） 26', '韩版牛仔裤', 84600, 'https://m.360buyimg.com/mobilecms/s720x720_jfs/t26989/116/124520860/644643/173643ea/5b860864N6bfd95db.jpg!q70.jpg.webp', '牛仔裤', '唐狮', '{\"颜色\": \"蓝色\", \"尺码\": \"26\"}', 1, '2019-05-01 00:00:00', '2019-05-01 00:00:00');
INSERT INTO `tb_item` VALUES (10004, '森马(senma)休闲鞋女2019春季新款韩版系带板鞋学生百搭平底女鞋 黄色 36', '休闲板鞋', 10400, 'https://m.360buyimg.com/mobilecms/s720x720_jfs/t1/29976/8/2947/65074/5c22dad6Ef54f0505/0b5fe8c5d9bf6c47.jpg!q70.jpg.webp', '休闲鞋', '森马', '{\"颜色\": \"白色\", \"尺码\": \"36\"}', 1, '2019-05-01 00:00:00', '2019-05-01 00:00:00');
INSERT INTO `tb_item` VALUES (10005, '花王（Merries）拉拉裤 M58片 中号尿不湿（6-11kg）（日本原装进口）', '拉拉裤', 38900, 'https://m.360buyimg.com/mobilecms/s720x720_jfs/t24370/119/1282321183/267273/b4be9a80/5b595759N7d92f931.jpg!q70.jpg.webp', '拉拉裤', '花王', '{\"型号\": \"XL\"}', 1, '2019-05-01 00:00:00', '2019-05-01 00:00:00');
DROP TABLE IF EXISTS `tb_item_stock`;
CREATE TABLE `tb_item_stock`  (
  `item_id` bigint(20) NOT NULL COMMENT '商品id，关联tb_item表',
  `stock` int(10) NOT NULL DEFAULT 9999 COMMENT '商品库存',
  `sold` int(10) NOT NULL DEFAULT 0 COMMENT '商品销量',
  PRIMARY KEY (`item_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = COMPACT;
INSERT INTO `tb_item_stock` VALUES (10001, 99996, 3219);
INSERT INTO `tb_item_stock` VALUES (10002, 99999, 54981);
INSERT INTO `tb_item_stock` VALUES (10003, 99999, 189);
INSERT INTO `tb_item_stock` VALUES (10004, 99999, 974);
INSERT INTO `tb_item_stock` VALUES (10005, 99999, 18649);
SET FOREIGN_KEY_CHECKS = 1;
```
![image-20240425074417184](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404250744238.png)
[基本工程下载](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404250755125.rar)
[nginx配置](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404250758691.rar)
在nginx目录下，运行命令启动nginx
```sh
start nginx.exe
```
![image-20240425080100462](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404250801495.png)
> 访问页面`http://localhost/item.html?id=10001`
![image-20240425080135891](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404250801005.png)
现在，页面是假数据展示的。我们需要向服务器发送ajax请求，查询商品数据。
打开控制台，可以看到页面有发起`ajax`查询数据
而这个请求地址同样是80端口，所以被当前的nginx反向代理了。
查看nginx的conf目录下的`nginx.conf`文件
其中的关键配置如下：
![image-20210816114416561](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404250804293.png)
其中的192.168.150.101是我的虚拟机IP，也就是我的Nginx业务集群要部署的地方：
完整内容如下：
```nginx
#user  nobody;
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    #tcp_nopush     on;
    keepalive_timeout  65;
    upstream nginx-cluster{
        server 192.168.150.101:8081;
    }
    server {
        listen       80;
        server_name  localhost;
	location /api {
            proxy_pass http://nginx-cluster;
        }
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```
## 本地进程缓存
缓存在日常开发中启动至关重要的作用，由于是存储在内存中，数据的读取速度是非常快的，能大量减少对数据库的访问，减少数据库的压力。我们把缓存分为两类:
分布式缓存，例如Redis:
- 优点:存储容量更大、可靠性更好、可以在集群间共享
- 缺点:访问缓存有网络开销
- 场景:缓存数据量较大、可靠性要求较高、需要在集群间共享
进程本地缓存，例如HashMap、GuavaCache:
- 优点:读取本地内存,没有网络开销，速度更快
- 缺点:存储容量有限、可靠性较低、无法共享
- 场景:性能要求较高，缓存数据量较小
```java
    @Test
    void testBasicOps() {
        // 创建缓存对象
        Cache<String, String> cache = Caffeine.newBuilder().build();
        // 存数据
        cache.put("gf", "迪丽热巴");
        // 取数据，不存在则返回null
        String gf = cache.getIfPresent("gf");
        System.out.println("gf = " + gf);
        // 取数据，不存在则去数据库查询
        String defaultGF = cache.get("defaultGF", key -> {
            // 这里可以去数据库根据 key查询value
            return "柳岩";
        });
        System.out.println("defaultGF = " + defaultGF);
    }
```
Caffeine提供了三种缓存驱逐策略:
- 基于容量:设置缓存的数量上限
  ```java
          // 创建缓存对象
          Cache<String, String> cache = Caffeine.newBuilder()
                  // 设置缓存大小上限为 1
                  .maximumSize(1)
                  .build();
  ```
- 基于时间:设置缓存的有效时间
  ```java
          // 创建缓存对象
          Cache<String, String> cache = Caffeine.newBuilder()
                  // 设置缓存有效期为 10 秒
                  .expireAfterWrite(Duration.ofSeconds(1))
                  .build();
  ```
  
- 基于引用:设置缓存为软引用或弱引用,利用GC来回收缓存数据。性能较差,不建议使用。
在默认情况下，当一个缓存元素过期的时候，Caffeine不会`自动立即`将其`清理`和驱逐。而是在一次读或写操作后， 或者在空闲时间完成对失效数据的驱逐。
> 实现商品的查询的本地进程缓存
>
> 利用Caffeine实现下列需求:
>
> - 给根据id查询商品的业务添加缓存，缓存未命中时查询数据库
> - 给根据id查询商品库存的业务添加缓存，缓存未命中时查询数据库
> - 缓存初始大小为100
> - 缓存上限为10000
新建配置类
> 这里`10_000`其实是指`10000`，下划线的作用是分隔数字，方便阅读，不加也行
```java
@Configuration
public class CaffeineConfig {
    @Bean
    public Cache<Long, Item> itemCache(){
        return Caffeine.newBuilder()
                .initialCapacity(100)
                .maximumSize(10_000)
                .build();
    }
    @Bean
    public Cache<Long, ItemStock> stockCache(){
        return Caffeine.newBuilder()
                .initialCapacity(100)
                .maximumSize(10_000)
                .build();
    }
}
```
> 查询api添加缓存
>
> 1. 它首先检查给定 `id` 对应的缓存项是否存在（即 `ItemStock` 是否在缓存中）。
> 2. 如果缓存中存在该 `id` 对应的 `ItemStock`，则直接从缓存中获取并返回结果。
> 3. 如果缓存中不存在该 `id` 对应的 `ItemStock`，则执行传递给 `get` 方法的第二个参数——一个函数式接口（`key -> stockService.getById(key)`）。这个函数会被用来计算并返回需要缓存的值，即通过调用 `stockService.getById(key)` 查询数据库以获取 `ItemStock` 对象。
> 4. 在从数据库获取到 `ItemStock` 后，Caffeine 会将其放入缓存，并将该对象作为结果返回。
>
> 虽然代码片段中没有显式地进行“存入缓存”的操作，但在首次查询不存在于缓存中的 `id` 时，Caffeine 自动完成了从数据库加载数据并缓存这一过程。后续对该 `id` 的查询将会命中缓存，从而避免了对数据库的重复访问。
```java
@RestController
@RequestMapping("item")
public class ItemController {
    @Autowired
    private Cache<Long, Item> itemCache;
    @Autowired
    private Cache<Long, ItemStock> stockCache;
    // 注入Bean
    @Autowired
    private IItemService itemService;
    @Autowired
    private IItemStockService stockService;
    @GetMapping("list")
    public PageDTO queryItemPage(
            @RequestParam(value = "page", defaultValue = "1") Integer page,
            @RequestParam(value = "size", defaultValue = "5") Integer size) {
        // 分页查询商品
        Page<Item> result = itemService.query()
                .ne("status", 3)
                .page(new Page<>(page, size));
        // 查询库存
        List<Item> list = result.getRecords().stream().peek(item -> {
            ItemStock stock = stockService.getById(item.getId());
            item.setStock(stock.getStock());
            item.setSold(stock.getSold());
        }).collect(Collectors.toList());
        // 封装返回
        return new PageDTO(result.getTotal(), list);
    }
    @PostMapping
    public void saveItem(@RequestBody Item item) {
        itemService.saveItem(item);
    }
    @PutMapping
    public void updateItem(@RequestBody Item item) {
        itemService.updateById(item);
    }
    @PutMapping("stock")
    public void updateStock(@RequestBody ItemStock itemStock) {
        stockService.updateById(itemStock);
    }
    @DeleteMapping("/{id}")
    public void deleteById(@PathVariable("id") Long id) {
        itemService.update().set("status", 3).eq("id", id).update();
    }
    @GetMapping("/{id}")
    public Item findById(@PathVariable("id") Long id) {
        // 如果缓存存在，则直接返回，不存在则查数据库
        return itemCache.get(id, key -> itemService.query()
                .ne("status", 3).eq("id", key)
                .one()
        );
    }
    @GetMapping("/stock/{id}")
    public ItemStock findStockById(@PathVariable("id") Long id) {
        // 如果缓存存在，则直接返回，不存在则查数据库
        return stockCache.get(id, key -> stockService.getById(key));
    }
}
```
> 访问`http://localhost:8081/item/10001`，多次刷新，但日志只打印了一次`sql`，说明查的是缓存
>
> 同理访问`http://localhost:8081/item/stock/10001`
## Lua语法入门
### 基础
1.在Linux虚拟机的任意目录下，新建一-个hello.lua文件
```sh
touch hello.lua
```
2.添加下面的内容
```lua
print("Hello World!")
```
3.运行
```lua
lua hello.lua
```
| 数据类型 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| nil      | 这个最简单，只有值nil属于该类，表示一个无效值(在条件表达式中相当于false)。 |
| boolean  | 包含两个值: false和true                                      |
| number   | 表示双精度类型的实浮点数                                     |
| string   | 字符串由一对双引号或单引号来表示                             |
| function | 由C或Lua编写的函数                                           |
| table    | Lua中的表(table) 其实是一个"关联数组" (associative arrays)，数组的索引可以是数字、字符串或表类型。在Lua里，table的创建是通过"构造表达式"来完成，最简单构造表达式是{},用来创建一 个空表。 |
> 可以使用`type`关键字来判断变量的类型
```
print(type("hello"))
```
> 声明变量
>
> `local`代表局部变量
```lua
-- 声明字符串
local str = 'hello'
-- 声明数字
local num = 21
-- 声明布尔类型
local flag = true
-- 声明数组 key为索引的table
local arr = {'java','python','go'}
-- 声明table，类似于java中的map
local map={ name = 'jack' , gender = '男'}
-- 访问数组，lua数组的角标从1开始
print(arr[1])
-- 访问table
print(map['name'])
print(map.name)
-- 字符串拼接
local helloworld='Hello ' .. 'World!'
```
> 遍历
```lua
-- 遍历数组
local arr = {'java','python','go'}
-- ipairs代表解析数组
for key,value in ipairs(arr) do
    print(key,value)
end
-- 遍历map
local map={ name = 'jack' , gender = '男'}
-- 遍历table
for key,value in pairs(map) do
    print(key,value)
end
```
**执行结果如下**
![image-20240425210357720](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404252104807.png)
### 函数
> 定义函数的语法
```lua
function 函数名(arg1,arg2...,arg3)
    -- 函数体
    return 返回值
end
-- 例如，定义一个函数
function printArr(arr)
    for key,value in ipairs(arr) do
        print(key,value)
    end
end
printArr({'java','python','go'})
```
### 条件控制
```lua
if(布尔表达式)
then
    -- [为true时，执行的语句]
else
    -- [为false时，执行的语句]
end
```
> 逻辑操作符，`and`,`or`,`not`
如、`A and B`、`A orB`、`not B`
```lua
function printArr(arr)
    if(not arr) then
        print('参数不能为nil')
        return nil
    end
    for key,value in ipairs(arr) do
        print(key,value)
    end
end
local lang={'java','python','go'}
printArr(lang)
printArr(nil)
```
## 多级缓存
### OpenRestry
#### 安装
> [官网](https://openresty.org/cn/download.html)
OpenResty是-个基于Nginx的高性能Web平台，用于方便地搭建能够处理超高并发、扩展性极高的动态Web应用、Web服务和动态网关。具备下列特点:
- 具备Nginx的完整功能
- 基于Lua语言进行扩展，集成了大量精良的Lua库、第三方模块
- 允许使用Lua自定义业务逻辑、自定义库
首先要安装OpenResty的**依赖开发库**，执行命令：
```sh
yum install -y pcre-devel openssl-devel gcc --skip-broken
```
安装**OpenResty仓库**
```sh
yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo
```
如果提示说命令不存在，则运行：
```
yum install -y yum-utils 
```
然后再重复上面的命令
**安装OpenResty**
然后就可以像下面这样安装软件包，比如 `openresty`：
```bash
yum install -y openresty
```
**安装opm工具**
opm是OpenResty的一个管理工具，可以帮助我们安装一个第三方的Lua模块。
如果你想安装命令行工具 `opm`，那么可以像下面这样安装 `openresty-opm` 包：
```sh
yum install -y openresty-opm
```
**配置nginx**
```sh
vi /etc/profile
```
在最下面加入两行：
```sh
export NGINX_HOME=/usr/local/openresty/nginx
export PATH=${NGINX_HOME}/sbin:$PATH
```
NGINX_HOME：后面是OpenResty安装目录下的nginx的目录
然后让配置生效：
```
source /etc/profile
```
所以运行方式与nginx基本一致：
```sh
# 启动nginx
nginx
# 重新加载配置
nginx -s reload
# 停止
nginx -s stop
```
nginx的默认配置文件注释太多，影响后续我们的编辑，这里将nginx.conf中的注释部分删除，保留有效部分。
修改`/usr/local/openresty/nginx/conf/nginx.conf`文件，内容如下：
```nginx
#user  nobody;
worker_processes  1;
error_log  logs/error.log;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       8081;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```
在Linux的控制台输入命令以启动nginx：
```sh
nginx
```
然后访问页面：http://192.168.56.10:8081，注意ip地址替换为你自己的虚拟机IP：
![image-20240425215453990](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404252154083.png)
#### 入门
![image-20240427193749125](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404271937233.png)
> 访问`http://localhost/item.html?id=10001`商品详情页，发送的请求是`http://localhost/api/item/10001`
![image-20240427193945728](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404271939787.png)
nginx在捕获到`/api`请求后，转给了虚拟机`8081`，而刚刚部署的`OpenRestry`在8081，这里`OpenRestry`需要做一些业务处理
步骤一:修改nginx.conf文件
1.在nginx.conf的http 下面，添加对OpenResty的Lua模块的加载:
```
#加载lua模块
lua_package_path "/usr/local/openresty/lualib/?.lua;;";
#加载c模块
lua_package_cpath "/usr/local/openresty/lualib/?.so;;";
```
2.在nginx.conf的server 下面，添加对`/api/item`这个路径的监听:
```
location /api/item {
    #响应类型，这里返回json
    default_type application/json;
    #响应数据由lua/item.lua这个文件来决定
    content_by_lua_file lua/item.lua;
}
```
![image-20240427194735493](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404271947581.png)
> 1. 在nginx目录下新建文件夹`lua`
> 2. lua目录下新建文件`item.lua`
> 3. 编写如下内容
>
> ```lua
> -- 返回一条假数据，这里的ngx.say()函数就是把数据写入Response中
> ngx.say('{"id":10001,"name":"普通行李箱"}')
> ```
>
> 4. 重新加载配置
>
> ```
> nginx -s reload
> ```
![image-20240427200628182](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404272006323.png)
可以看到**lua脚本**已经生效了
#### 请求参数
OpenResty提供了各种API用来获取不同类型的请求参数:
![image-20240427201344237](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404272013371.png)
**案例**
获取请求路径中的商品id信息，拼接到json结果中返回
```
        location ~ /api/item/(\d+) {
            #响应类型，这里返回json
            default_type application/json;
            #响应数据由lua/item.lua这个文件来决定
            content_by_lua_file lua/item.lua;
        }
```
![image-20240427202103810](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404272021885.png)
![image-20240427202117153](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404272021225.png)
#### 封装http请求工具
![image-20240428075935605](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404280759689.png)
**案例**
这里要修改item.lua,满足下面的需求:
1. 获取请求参数中的id
2. 根据id向Tomcat服务发送请求，查询商品信息
3. 根据id向Tomcat服务发送请求，查询库存信息
4. 组装商品信息、库存信息，序列化为JSON格式并返回
> nginx发送http请求
>
> `/path`是指具体的请求路径，如`/item/10001`
>
> 返回响应的内容包括
>
> 1. resp.status 响应状态码
> 2. resp.header 响应头，是一个table
> 3. resp.body 响应体，就是响应数据
```lua
local resp=ngx.location.capture('/path',{
	-- 请求方式
	method=ngx.HTTP_GET,
	-- get方式传参
	args={a=1,b=2},
	-- post方式传参
	body="c=3&d=4"
})
```
注意:这里的path是路径，并不包含IP和端口。这个请求会被nginx内部的server监听并处理。
但是我们希望这个请求发送到Tomcat服务器，所以还需要编写一个server来对这个路径做反向代理:
```conf
        location /item {
        	proxy_pass http://192.168.56.1:8081;
        }
```
![image-20240428080836536](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404280808615.png)
> 封装通用http请求
![image-20240428081628881](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404280816959.png)
```lua
-- 封装函数，发送http请求
local function read_http(path,params)
    local resp=ngx.location.capture(path,{
            method=ngx.HTTP_GET,
            args=params,
    })
    if not resp then
        -- 记录错误信息，返回404
        ngx.log(ngx.ERR,"http not found,path: ",path,", args: ",args)
        ngx.exit(404)
    end
    return resp.body
end
-- 导出方法
local _M={
    read_http=read_http
}
return _M
```
再次请求即可
![image-20240429201037008](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404292010118.png)
> 咱先理一下请求的过程
1. 首先nginx里的html发送ajax请求`http://localhost/api/item/10001`
2. nginx反向代理给`192.168.56.10:8081`
   ![image-20240429201307731](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404292013794.png)
3. 而虚拟机的`openresty`由`item.lua`来处理这个请求
   ![image-20240429201412119](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404292014207.png)
4. `item.lua`通过封装好的请求工具发送请求
   ![image-20240429201806527](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404292018619.png)
5. 而`openresty`本身也具有nginx的功能，将请求反向代理到windows主机上
   ![image-20240429201852232](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404292018327.png)
6. 后端接收到请求，响应数据
   ![image-20240429201955918](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404292019049.png)
> 现在数据还不完整，还需要查询库存信息返回，此时只有商品信息
>
> 需要将json数据转换成lua中的table(也就是`DTO`)
>
> OpenResty提供了一个`cjson`的模块用来处理JSON的序列化和反序列化。
```lua
-- 导入cjson库
local cjson=require("cjson")
-- 序列化
local obj={
    name="jack",
    age=21
}
local json=cjson.encode(obj)
-- 反序列化
local json='{"name":"jack","age":21}'
local obj=cjson.decode(json)
print(obj.name)
```
> 编写`item.lua`
```lua
-- 导入cjson库
local cjson=require("cjson")
-- 导入请求工具common
-- 这里common.lua正好在lualib目录下，所以可以直接导入;如果不是，则需要写上路径
local common=require("common")
local read_http=common.read_http
-- 获取路径参数
local id=ngx.var[1]
-- 查询商品信息
local itemJson=read_http("/item/"..id,nil)
-- 查询库存信息
local stockJson=read_http("/item/stock/"..id,nil)
-- dto转换
-- json数据转换成lua中的table
local item=cjson.decode(itemJson)
local stock=cjson.decode(stockJson)
-- 组合数据
item.stock=stock.stock
item.sold=stock.sold
-- 返回结果
ngx.say(cjson.encode(item))
```
> 重启`OpenResty`，再次查看请求，库存信息已经出现
```sh
nginx -s reload
```
![image-20240429203042025](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404292030122.png)
### Tomcat集群的负载均衡
> 既然要配置负载均衡，那请求就不能只指向一个ip了，需要配置集群
![image-20240430074537126](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404300745207.png)
**查询缓存过程:**查询8081，如果`缓存`存在则直接返回，如果不存在，在查询数据库之后，存入`进程缓存`中，下次即可从进程缓存获取数据
`但是`，**进程缓存无法共享**，就是`8081`的进程缓存无法与`8082`共享
我们希望查询商品id为`10001`在第一次查询之后永远都有缓存，该如何做？
我们必须要让`10001`的请求每次都查询同一台服务器，这样就可以让缓存一直生效
> 修改nginx负载均衡算法
>
> 对请求uri采用hash算法，使在请求相同的商品时能走到同一台服务器
![image-20240430075210681](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404300752746.png)
**配置集群**
![image-20240430075622620](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404300756714.png)
```conf
#user  nobody;
worker_processes  1;
error_log  logs/error.log;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    # 加载lua模块
    lua_package_path "/usr/local/openresty/lualib/?.lua;;";
    # 加载c模块
    lua_package_cpath "/usr/local/openresty/lualib/?.so;;";
    upstream tomcat-cluster{
        hash $request-uri;
        server 192.168.56.1:8081;
        server 192.168.56.1:8082;
    }
    server {
        listen       8081;
        server_name  localhost;
        location /item {
        	proxy_pass http://tomcat-cluster;
        }
        location ~ /api/item/(\d+) {
            #响应类型，这里返回json
            default_type application/json;
            #响应数据由lua/item.lua这个文件来决定
            content_by_lua_file lua/item.lua;
        }
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```
> 继续复制一台Springboot实例，并启动
![image-20240430075741082](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404300757137.png)
> 访问`http://localhost/item.html?id=10002`和`http://localhost/item.html?id=10001`
>
> 负载均衡已实现，而在多次访问时，并没有mysql日志打印，说明是查询缓存得到的数据
![image-20240430080001509](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404300800606.png)
![image-20240430080008547](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404300800644.png)
### 添加Redis需求
> 但是按照多级缓存的设想，并不是优先查询Tomcat，而是优先查询Redis，在Redis缓存未命中时，再去查询Tomcat
#### 冷启动与缓存预热
冷启动:服务刚刚启动时，Redis中并没有缓存，如果所有商品数据都在第一次查询 时添加缓存，可能会给数据库带来较大压力。
缓存预热:在实际开发中，我们可以利用大数据统计用户访问的热点数据，在项目启动时将这些热点数据提前查询并保存到Redis中。
我们数据量较少，可以在启动时将所有数据都放入缓存中。
> 使用Docker安装Redis
```sh
docker run --name redis -p 6379:6379 -d redis redis-server --appendonly yes
```
> 添加Redis依赖
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```
> 配置Redis地址
```yaml
spring:
  redis:
    host: 192.168.56.10
```
>编写Redis初始化类
```java
@Component
public class RedisHandler implements InitializingBean {
    @Autowired
    private StringRedisTemplate template;
    @Override
    public void afterPropertiesSet() throws Exception {
        // 初始化缓存
    }
}
```
> 我的配置
```yaml
server:
  port: 8081
spring:
  application:
    name: itemservice
  datasource:
    url: jdbc:mysql://192.168.56.10:3307/heima?useSSL=false
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
  redis:
    host: 192.168.56.10
    port: 6379
mybatis-plus:
  type-aliases-package: com.heima.item.pojo
  configuration:
    map-underscore-to-camel-case: true
  global-config:
    db-config:
      update-strategy: not_null
      id-type: auto
logging:
  level:
    com.heima: debug
  pattern:
    dateformat: HH:mm:ss:SSS
```
```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.heima.item.pojo.Item;
import com.heima.item.pojo.ItemStock;
import com.heima.item.service.IItemService;
import com.heima.item.service.IItemStockService;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Component;
import java.util.List;
@Component
public class RedisHandler implements InitializingBean {
    @Autowired
    private StringRedisTemplate template;
    @Autowired
    private IItemService itemService;
    @Autowired
    private IItemStockService stockService;
    // Spring自带的序列化工具
    private static final ObjectMapper MAPPER = new ObjectMapper();
    @Override
    public void afterPropertiesSet() throws Exception {
        // 初始化缓存
        // 1.查询商品信息
        List<Item> itemList = itemService.list();
        // 2.存入缓存
        for (Item item : itemList) {
            // 将item序列化成json
            String json = MAPPER.writeValueAsString(item);
            // 存入缓存
            template.opsForValue().set("item:id:"+item.getId(),json);
        }
        // 3.查询库存信息
        List<ItemStock> stockList = stockService.list();
        // 2.存入缓存
        for (ItemStock stock : stockList) {
            // 将item序列化成json
            String json = MAPPER.writeValueAsString(stock);
            // 存入缓存
            template.opsForValue().set("item:stock:id:"+stock.getId(),json);
        }
    }
}
```
> 访问`http://localhost/item.html?id=10004`界面，实现了缓存预热的功能
![image-20240430082142624](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404300821701.png)
#### 查询Redis缓存
> `OpenResty`优先查询`Redis`，`Redis`缓存未命中，再查询`Tomcat`
**OpenResty的Redis模块**
OpenResty提供了操作Redis的模块，我们只要引入该模块就能直接使用:
**引入Redis模块，并初始化Redis对象**
```lua
-- 引入redis模块
local redis = requirl("resty.redis")
-- 初始化Redis对象
local red = redis:new( )
-- 设置Redis超时时间
red:set_timeouts(1000, 1000, 1000)
```
**封装函数，用来释放Redis连接，其实是放入连接池**
```lua
-- 关闭redi s连接的工具方法，其实是放入连接池
local function close_redis(red)
    -- 连接的空闲时间，单位是毫秒
    local pool_max_idle_time = 10000 
    -- 连接池大小
    local pool_size = 100 
    local ok,err = red:set_keepalive(pool_max_idle_time,pool_size)
    if not ok then
        ngx.log(ngx.ERR，"放入Redis连接池失败: ",err)
    end
end
```
**封装函数，从Redis读数据并返回**
```lua
-- 查询redis的方法ip和port是redi s地址，key是查询的key
local function read_redis(ip, port, key)
    -- 获取一个连接
    local ok，err = red:connect(ip,port)
    if not ok then
        ngx.log(ngx.ERR，"连接redis失败: ",err)
        return nil
    end
    -- 查询redis
    local resp, err = red:get(key)
    -- 查询失败处理
    if not resp then
        ngx.log(ngx.ERR，"查询Redis失败: " ,err,", key = "，key)
    end
    -- 查询成功，但是得到的数据为空
    if resp == ngx.null then
    	resp = nil
    	ngx.log(ngx.ERR，"查询Redis数据为空， key = ",key)
    end
    close_redis(red)
    return resp
end
```
> 在`common.lua`下处理如下
```lua
-- 导入redis
local redis=require("resty.redis")
-- 初始化redis
local red=redis:new()
red:set_timeouts(1000,1000,1000)
-- 关闭redis连接的工具方法，其实是放入连接池
local function close_redis(red)
    local pool_max_idle_time = 10000 -- 连接的空闲时间，单位是毫秒
    local pool_size = 100 --连接池大小
    local ok, err = red:set_keepalive(pool_max_idle_time, pool_size)
    if not ok then
        ngx.log(ngx.ERR, "放入redis连接池失败: ", err)
    end
end
-- 查询redis的方法 ip和port是redis地址，key是查询的key
local function read_redis(ip, port, key)
    -- 获取一个连接
    local ok, err = red:connect(ip, port)
    if not ok then
        ngx.log(ngx.ERR, "连接redis失败 : ", err)
        return nil
    end
    -- 查询redis
    local resp, err = red:get(key)
    -- 查询失败处理
    if not resp then
        ngx.log(ngx.ERR, "查询Redis失败: ", err, ", key = " , key)
    end
    --得到的数据为空处理
    if resp == ngx.null then
        resp = nil
        ngx.log(ngx.ERR, "查询Redis数据为空, key = ", key)
    end
    close_redis(red)
    return resp
end
-- 封装函数，发送http请求
local function read_http(path,params)
    local resp=ngx.location.capture(path,{
            method=ngx.HTTP_GET,
            args=params,
    })
    if not resp then
        -- 记录错误信息，返回404
        ngx.log(ngx.ERR,"http not found,path: ",path,", args: ",args)
        ngx.exit(404)
    end
    return resp.body
end
-- 导出方法
local _M={
    read_http=read_http,
    read_redis=read_redis
}
return _M
```
> 在`item.lua`下封装一个`read_data`函数，优先查询redis，redis未命中再查询tomcat
>
> 查询商品和库存，都调用`read_data`函数
```lua
-- 封装函数，先查询redis，再查询http
local function read_data(key, path, par ams)
    -- 查询redis
    local resp = read_redis("127.0.0.1",6379,key)
    -- 判断redis是否命中
    if not resp then
        -- Redis查询失败，查询http
        resp = read_http(path, par ams)
	end
    return resp
end
```
> `item.lua`
```lua
-- 导入cjson库
local cjson=require("cjson")
-- 导入请求工具common
-- 这里common.lua正好在lualib目录下，所以可以直接导入;如果不是，则需要写上路径
local common=require("common")
local read_http=common.read_http
local read_redis=common.read_redis
-- 封装查询函数
function read_data(key,path,params)
    -- 查询redis
    local resp=read_redis("127.0.0.1",6379,key)
    -- 判断查询结果
    if not resp then
        -- redis查询失败，尝试查询tomcat
        ngx.log("redis查询失败，尝试查询tomcat：",key)
        resp=read_http(path,params)
    end
    return resp
end
-- 获取路径参数
local id=ngx.var[1]
-- 查询商品信息
local itemJson=read_data("item:id:"..id , "/item/"..id , nil)
-- 查询库存信息
local stockJson=read_data("item:stock:id:"..id , "/item/stock/"..id , nil)
-- dto转换
-- json数据转换成lua中的table
local item=cjson.decode(itemJson)
local stock=cjson.decode(stockJson)
-- 组合数据
item.stock=stock.stock
item.sold=stock.sold
-- 返回结果
ngx.say(cjson.encode(item))
```
更新Redis数据，重新查看网页，发现已经变成`22`寸了
![image-20240506202615377](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202405062026467.png)
![image-20240506202636120](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202405062026232.png)
#### Nginx本地缓存
> 优先查询`OpenResty本地缓存`，其次是Redis，最后是Tomcat
>
> OpenResty为Nginx提供了`shard dict`的功能，可以在nginx的多个worker之间共享数据，实现缓存功能。
- 开启共享字典，在nginx.conf的http下添加配置
  ```conf
  # 共享字典，也就是本地缓存，名称叫做：item_cache，大小150m
  lua_shared_dict item_cache 150m;
  ```
- 操作共享字典
  ```lua
  -- 获取本地缓存对象
  local item_cache=ngx.shared.item_cache
  -- 存储，指定key、value、过期时间，单位s，默认为0代表永不过期
  item_cache:set('key','value',1000)
  -- 读取
  local val=item_cache:get('key')
  ```
![image-20240506204425210](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202405062044303.png)
> 修改`item.lua`，优先查询nginx本地缓存，其次是redis、tomcat、mysql
>
> 查询Redis、Tomcat成功后，将缓存信息写入本地缓存，并设置有效期
>
> 商品基本信息设置为有效期30min
>
> 库存信息设置为有效期1min
```lua
-- 导入cjson库
local cjson=require("cjson")
-- 导入请求工具common
-- 这里common.lua正好在lualib目录下，所以可以直接导入;如果不是，则需要写上路径
local common=require("common")
local read_http=common.read_http
local read_redis=common.read_redis
local item_cache=ngx.shared.item_cache
-- 封装查询函数
function read_data(key,expire,path,params)
    -- 查询本地缓存
    local val=item_cache:get(key)
    if not val then
        ngx.log(ngx.ERR,"本地缓存查询失败，尝试查询redis，key:",key)
        -- 查询redis
        val=read_redis("127.0.0.1",6379,key)
        -- 判断查询结果
        if not val then
            -- redis查询失败，尝试查询tomcat
            ngx.log(ngx.ERR,"redis查询失败，尝试查询tomcat，key:",key)
            val=read_http(path,params)
        end
    end
    -- 查询成功，把数据写入本地缓存
    item_cache:set(key,val,expire)
    return val
end
-- 获取路径参数
local id=ngx.var[1]
-- 查询商品信息
local itemJson=read_data("item:id:"..id,1800,"/item/"..id,nil)
-- 查询库存信息
local stockJson=read_data("item:stock:id:"..id,60,"/item/stock/"..id,nil)
-- dto转换
-- json数据转换成lua中的table
local item=cjson.decode(itemJson)
local stock=cjson.decode(stockJson)
-- 组合数据
item.stock=stock.stock
item.sold=stock.sold
-- 返回结果
ngx.say(cjson.encode(item))
```
## 缓存同步
缓存数据同步的常见方式有三种:
- 设置有效期:给缓存设置有效期，到期后自动删除。再次查询时更新
  - 优势:简单、方便
  - 缺点:时效性差，缓存过期之前可能不-致
  - 场景:更新频率较低，时效性要求低的业务
- 同步双写: 在修改数据库的同时，直接修改缓存
  - 优势:时效性强，缓存与数据库强一致
  - 缺点:有代码侵入,耦合度高;
  - 场景:对一致性、时效性要求较高的缓存数据
- 异步通知:修改数据库时发送事件通知，相关服务监听到通知后修改缓存数据
  - 优势:低耦合，可以同时通知多个缓存服务
  - 缺点:时效性一般，可能存在中间不一致状态
  - 场景:时效性要求-般，有多个服务需要同步
> 基于MQ的异步通知
![image-20240506212404397](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202405062124605.png)
> 基于Canal的异步通知
![image-20240506212506025](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202405062125223.png)
Canal是基于mysql的主从同步来实现的，MySQL主从同步的原理如下:
- MySQL master将数据变更写入二进制日志( binary log)，其中记录的数据叫做binary log events
- MySQL slave将master的binary log events拷贝到它的中继日志(relay log)
- MySQL slave重放relay log中事件,将数据变更反映它自己的数据
![image-20240506212739255](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202405062127347.png)
### 初识Canal
Canal就是把自己伪装成MySQL的一个slave节 点,从而监听master的binary log变化。再把得到的变化信息通知给Canal的客户端，进而完成对其它数据库的同步。
![image-20240506212852571](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202405062128659.png)
### 开启MySQL主从
> 修改配置文件
```
/tmp/mysql/conf/my.cnf
```
添加内容
```
log-bin=/var/lib/mysql/mysql-bin
binlog-do-db=heima
```
重启mysql
```sh
docker restart mysql-demo
```
![image-20240507080007082](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202405070800294.png)
### 设置用户权限
接下来添加一个用于数据同步的用户，出于安全的考虑，这里仅提供对`heima`这个库的操作权限
```sql
create user canal@'%' IDENTIFIED by 'canal';
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT,SUPER ON *.* TO 'canal'@'%' identified by 'canal';
FLUSH PRIVILEGES;
```
执行成功后，查看到该用户
![image-20240507080342715](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202405070803813.png)
![image-20240507080458864](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202405070804039.png)
> 如果从库的`position`小于主库的`position`，则证明有数据需要同步
### 创建网络
我们需要创建一个网络，将MySQL、Canal、MQ放到同一个Docker网络中：
```sh
docker network create heima
```
让mysql加入这个网络：
```sh
docker network connect heima mysql-demo
```
然后运行命令创建Canal容器：
```sh
docker run -p 11111:11111 --name canal \
-e canal.destinations=heima \
-e canal.instance.master.address=mysql-demo:3306  \
-e canal.instance.dbUsername=canal  \
-e canal.instance.dbPassword=canal  \
-e canal.instance.connectionCharset=UTF-8 \
-e canal.instance.tsdb.enable=true \
-e canal.instance.gtidon=false  \
-e canal.instance.filter.regex=heima\\..* \
--network heima \
-d canal/canal-server:v1.1.5
```
说明:
- `-p 11111:11111`：这是canal的默认监听端口
- `-e canal.instance.master.address=mysql:3306`：数据库地址和端口，如果不知道mysql容器地址，可以通过`docker inspect 容器id`来查看
- `-e canal.instance.dbUsername=canal`：数据库用户名
- `-e canal.instance.dbPassword=canal` ：数据库密码
- `-e canal.instance.filter.regex=`：要监听的表名称
表名称监听支持的语法：
```
mysql 数据解析关注的表，Perl正则表达式.
多个正则之间以逗号(,)分隔，转义符需要双斜杠(\\) 
常见例子：
1.  所有表：.*   or  .*\\..*
2.  canal schema下所有表： canal\\..*
3.  canal下的以canal打头的表：canal\\.canal.*
4.  canal schema下的一张表：canal.test1
5.  多个规则组合使用然后以逗号隔开：canal\\..*,mysql.test1,mysql.test2 
```
> 查看是否互联成功
```sh
docker exec -it canal /bin/bash
# 查看运行日志
more /home/admin/canal-server/logs/canal/canal.log
more /home/admin/canal-server/logs/heima/heima.log
```
### 监听Canal实现缓存同步
导入依赖
```xml
<dependency>
    <groupId>top.javatool</groupId>
    <artifactId>canal-spring-boot-starter</artifactId>
    <version>1.2.1-RELEASE</version>
</dependency>
```
编写配置
```yaml
canal:
	# canal实例名称，要跟canal-server运行时设置的destination一致
	destination: heima
	# canal地址
	server: 192.168.56.10:11111
```
![image-20240507082657564](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202405070826797.png)
```java
import com.heima.item.pojo.Item;
import org.springframework.stereotype.Component;
import top.javatool.canal.client.annotation.CanalTable;
import top.javatool.canal.client.handler.EntryHandler;
@Component
@CanalTable("tb_item")
public class ItemHandler implements EntryHandler<Item> {
    @Override
    public void insert(Item item) {
        // 新增数据到redis
    }
    @Override
    public void update(Item before, Item after) {
        // 更新redis数据
        // 更新本地数据
    }
    @Override
    public void delete(Item item) {
        // 删除redis数据
        // 清理本地缓存
    }
}
```
> 此时`Canal`还无法将表的字段映射到对应的`DTO`，需要添加`Canal`的注解进行指定
![image-20240507082940638](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202405070829863.png)
```java
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;
import org.springframework.data.annotation.Id;
import org.springframework.data.annotation.Transient;
import java.util.Date;
@Data
@TableName("tb_item")
public class Item {
    /**
     * 商品id
     */
    @TableId(type = IdType.AUTO)
    @Id
    private Long id;
    /**
     * 商品名称
     */
    private String name;
    /**
     * 商品标题
     */
    private String title;
    /**
     * 价格（分）
     */
    private Long price;
    /**
     * 商品图片
     */
    private String image;
    /**
     * 分类名称
     */
    private String category;
    /**
     * 品牌名称
     */
    private String brand;
    /**
     * 规格
     */
    private String spec;
    /**
     * 商品状态 1-正常，2-下架
     */
    private Integer status;
    /**
     * 创建时间
     */
    private Date createTime;
    /**
     * 更新时间
     */
    private Date updateTime;
    @Transient
    @TableField(exist = false)
    private Integer stock;
    @Transient
    @TableField(exist = false)
    private Integer sold;
}
```
```java
@Component
public class RedisHandler implements InitializingBean {
    @Autowired
    private StringRedisTemplate template;
    @Autowired
    private IItemService itemService;
    @Autowired
    private IItemStockService stockService;
    private static final ObjectMapper MAPPER = new ObjectMapper();
    @Override
    public void afterPropertiesSet() throws Exception {
        // 初始化缓存
        // 1.查询商品信息
        List<Item> itemList = itemService.list();
        // 2.存入缓存
        for (Item item : itemList) {
            // 将item序列化成json
            String json = MAPPER.writeValueAsString(item);
            // 存入缓存
            template.opsForValue().set("item:id:"+item.getId(),json);
        }
        // 3.查询库存信息
        List<ItemStock> stockList = stockService.list();
        // 2.存入缓存
        for (ItemStock stock : stockList) {
            // 将item序列化成json
            String json = MAPPER.writeValueAsString(stock);
            // 存入缓存
            template.opsForValue().set("item:stock:id:"+stock.getId(),json);
        }
    }
    /**
     * 保存项目
     *
     * @param item 项目
     */
    public void saveItem(Item item){
        // 将item序列化成json
        String json = null;
        try {
            json = MAPPER.writeValueAsString(item);
        } catch (JsonProcessingException e) {
            throw new RuntimeException(e);
        }
        template.opsForValue().set("item:id:"+item.getId(),json);
    }
    /**
     * 按 ID 删除项目
     *
     * @param id
     */
    public void removeItemById(Long id){
        template.delete("item:id:"+id);
    }
}
```
```java
import com.github.benmanes.caffeine.cache.Cache;
import com.heima.item.config.RedisHandler;
import com.heima.item.pojo.Item;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import top.javatool.canal.client.annotation.CanalTable;
import top.javatool.canal.client.handler.EntryHandler;
@Slf4j
@Component
@CanalTable("tb_item")
public class ItemHandler implements EntryHandler<Item> {
    @Autowired
    private RedisHandler redisHandler;
    @Autowired
    private Cache<Long,Item> itemCache;
    @Override
    public void insert(Item item) {
        // 新增数据到JVM进程缓存
        log.info("新增数据到JVM进程缓存");
        itemCache.put(item.getId(),item);
        // 新增数据到redis
        log.info("新增数据到redis");
        redisHandler.saveItem(item);
    }
    @Override
    public void update(Item before, Item after) {
        // 更新JVM进程缓存
        log.info("更新JVM进程缓存");
        itemCache.put(after.getId(),after);
        // 更新redis数据
        log.info("更新redis数据");
        redisHandler.saveItem(after);
    }
    @Override
    public void delete(Item item) {
        // 删除JVM进程缓存
        log.info("删除JVM进程缓存");
        itemCache.invalidate(item.getId());
        // 删除redis数据
        log.info("删除redis数据");
        redisHandler.removeItemById(item.getId());
    }
}
```
> 访问`http://localhost:8081/`进行后台管理，修改数据后，访问`http://localhost:8081/item/10001`和`redis`发现数据已经修改
