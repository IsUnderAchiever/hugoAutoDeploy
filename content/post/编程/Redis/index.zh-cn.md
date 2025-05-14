---
title: Redis
description: Redis
date: 2024-04-24
slug: Redis
image: 202412212127078.png
categories:
    - Redis
---
# Redis
## 单点Redis的问题
1. 数据丢失问题
   Redis是内存存储，服务重启可能会丢失数据
2. 并发能力问题
   单节点Redis并发能力虽然不错，但也无法满足如618这样的高并发场景
3. 故障恢复问题
   如果Redis宕机，则服务不可用，需要一种自动的故障恢复手段
4. 存储能力问题
   Redis基于内存，单节点能存储的数据量难以满足海量数据需求
> **解决方案**
1. 数据丢失问题
   实现Redis数据持久化
2. 并发能力问题
   搭建主从集群，实现读写分离(提高并发能力，实现高可用，进一步避免宕机导致的数据丢失)
3. 故障恢复问题
   利用Redis哨兵，实现健康检测和自动恢复
4. 存储能力问题
   搭建分片集群，利用插槽机制实现动态扩容
## Redis持久化
### RDB
> RDB全称Redis Database Backup file ( Redis数据备份文件)，也被叫做Redis数据快照。简单来说就是把内存中的所有数据都记录到`磁盘`中。当Redis实例故障重启后，从磁盘读取快照文件,恢复数据。
>
> 快照文件称为RDB文件，默认是保存在当前运行目录。
```sh
redis-cli
# 由Redis主进程来执行RDB，会阻塞其他命令
# 而将数据备份到磁盘中的速度又很慢，不推介这种方法
save
# 子进程在后台异步执行RDB，避免主进程受到影响
bgsave
```
**Redis停机时，会自动进行一次RDB备份**
```sh
C:\Users\Administrator>redis-cli
# 测试redis连接
127.0.0.1:6379> ping
PONG
# 存数据
127.0.0.1:6379> set num 123
OK
# 取数据
127.0.0.1:6379> get num
"123"
127.0.0.1:6379>
```
> 停止Redis，日志打印如下
```
D:\Redis>redis-server.exe redis.windows.conf
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 3.0.504 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 3204
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'
[3204] 15 Apr 21:11:30.427 # Server started, Redis version 3.0.504
[3204] 15 Apr 21:11:30.443 * DB loaded from disk: 0.000 seconds
[3204] 15 Apr 21:11:30.443 * The server is now ready to accept connections on port 6379
[3204] 15 Apr 21:12:39.581 # User requested shutdown...
[3204] 15 Apr 21:12:39.581 * Saving the final RDB snapshot before exiting.
[3204] 15 Apr 21:12:39.581 * DB saved on disk
[3204] 15 Apr 21:12:39.581 # Redis is now ready to exit, bye bye...
```
> 这里有一句`Saving the final RDB snapshot before exiting.`
>
> Redis将文件保存在运行目录中
>
> 我这是`windows`系统，如果是`linux`则使用`ls`、`ll`
![image-20240415211506657](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240415211506657.png)
> 但是我们担心如果突然宕机还没来得及持久化怎么办，我们更加希望隔一段时间持久化一次
>
> Redis内部有触发RDB的机制，可以在redis.conf文件中找到， 格式如下:
```properties
# 900秒内，如果至少一个key被修改，则执行bgsave
# 如果是save "" 则表示禁用Redis
save 900 1
save 300 10
save 60 10000
```
> RDB的其他配置也可以在配置文件中设置
```properties
# 是否压缩，建议不开启，压缩也会消耗CPU，磁盘不值钱
rdbcompression yes
# RDB文件名称
dbfilename dump.rdb
# 文件保存的路径
dir ./
```
> bgsave开始时会`fork`主进程得到子进程，子进程共享主进程的内存数据。完成fork后读取内存数据并写入RDB文件。
![image-20240415213015095](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240415213015095.png)
> fork采用的是copy-on-write技术:
> 当主进程执行读操作时,访问共享内存;
> 当主进程执行写操作时，则会拷贝一份数据，执行写操作。
![image-20240415213600960](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240415213600960.png)
#### 总结
RDB方式bgsave的基本流程?
1. fork主进程得到一-个子进程，共享内存空间
2. 子进程读取内存数据并写入新的RDB文件
3. 用新RDB文件替换旧的RDB文件。
RDB会在什么时候执行? save 60 1000代表什么含义?
1. 默认是服务停止时。
2. 代表60秒内至少执行1000次修改则触发RDB
RDB的缺点?
1. RDB执行间隔时间长，两次RDB之间写入数据有丟失的风险
2. fork子进程、压缩、写出RDB文件都比较耗时
### AOF
> AOF全称为Append Only File ( 追加文件)。Redis处理的每-一个写命令都会记录在AOF文件，可以看做是命令日志文件。
>
> 如果需要恢复数据，将日志文件里的命令全部执行一遍即可
AOF默认是关闭的，需要修改redis.conf配置文件来开启AOF:
```properties
# 是否开启A0F功能，默认是no 
appendonly no
# AOF文件的名称
appendfilename "appendonly.aof"
```
> AOF的命令记录的频率也可以通过redis.conf文件来配:
```properties
# 表示每执行一次写命令，立即记录到AOF文件
appendfsync always
# 写命令执行完先放入AOF缓冲区，然后表示每隔1秒将缓冲区数据写到AOF文件，是默认方案
appendfsync everysec
# 写命令执行完先放入AOF缓冲区，由操作系统决定何时将缓冲区内容写回磁盘
appendfsync no
```
![image-20240415214253132](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240415214253132.png)
由于AOF记录的是命令操作，而不是直接记录值，所以会比RDB文件大很多
因为是记录命令, AOF文件会比RDB文件大的多。而且AOF会记录对同一个key的多次写操作，但只有最后一-次写操作才有意义。通过执行bgrewriteaof命令，可以让AOF文件执行重写功能，用最少的命令达到相同效果。
![image-20240415214743582](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240415214743582.png)
Redis也会在`触发阈值`时自动去重写AOF文件。阈值也可以在redis.conf中配置: 
```properties
# AOF文件比上次文件增长超过多少百分比则触发重写
auto-aof-rewrite-percentage 100
# AOF文件体积最小多大以上才触发重写
auto-aof-rewrite-min-size 64mb
```
`RDB`和`AOF`各有自己的优缺点，如果对数据安全性要求较高,在实际开发中往往会结合两者来使用。
![image-20240415215054692](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240415215054692.png)
## Redis主从
>单节点Redis的并发能力是有上限的，要近一步提高Redis的并发能力，就需要搭建主从集群，实现读写分离。
![image-20240416063931474](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240416063931474.png)
> 开启RDB，关闭AOF
>
> 一主二从 A（B、C） 一个master两个slave/replica
```properties
# 禁用RDB
# save ""
save 900 1
save 300 10
save 60 10000
appendonly no
```
### windows
windows配置redis集群查看[该博客](https://blog.csdn.net/weixin_41846320/article/details/83753667)
> 主节点
![image-20240416070837220](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240416070837220.png)
> 从节点
![image-20240416070901240](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240416070901240.png)
![image-20240416070908369](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240416070908369.png)
> 从节点只能读数据，不能写数据
![image-20240416071004714](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240416071004714.png)
> 关闭主节点后，从节点的状态
![image-20240416071252667](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240416071252667.png)
> 重启主节点后，从节点的状态
![image-20240416071420341](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240416071420341.png)
### linux
> 接下来演示`docker配置redis集群`，参考[博客链接](https://blog.csdn.net/weixin_43025151/article/details/134205482)
![image-20240421125052157](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240421125052157.png)
> 可以看到本地已经拉取了`redis`镜像
在`/root`目录下新建一个`redis.conf`文件和`data`目录
`data`用以存储redis持久化文件，`redis.conf`作为redis的配置文件，这里配置`主从集群`，所以需要新建三个`conf`和`data`，目录及文件如下
![image-20240421135026402](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240421135026402.png)
>`redis-7001.conf`下载[链接](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/redis-7001.conf)
> 
>启动容器，这里使用`7001`作为端口号
```sh
docker run --name redis-master \
-v /root/redis/redis-7001.conf:/etc/redis/redis.conf \
-v /root/redis/data:/data \
-dp 7001:6379 \
redis:latest \
redis-server /etc/redis/redis.conf
```
![image-20240421133144513](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240421133144513.png)
> 配置文件直接复制7001即可，反正将来也是以容器为单位启动，不会造成端口冲突，但是虚拟机(宿主机)的端口映射则不能都写7001了
>
> 7001端口已经被`master`占据了，剩下两个`slave`使用`7002、7003`
>
> 修改对应的持久化目录
![image-20240421140154283](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240421140154283.png)
`redis-7001.conf`文件中添加如下配置
```sh
slave-announce-ip 192.168.56.10
slave-announce-port 7001
```
`redis-7002.conf、redis-7003.conf`也是一样
```sh
slave-announce-ip 192.168.56.10
slave-announce-port 7002
```
```sh
slave-announce-ip 192.168.56.10
slave-announce-port 7003
```
> 启动`slave`
>
> 在启动 slave 的`命令`中需要指出其 `slaveof` 于谁，这种是`临时方案`，重启失效
>
> `永久方案`是在conf文件中配置`slaveof <master_ip> <master_port>`
>
> redis5.0之后新增`replicaof`效果和`slaveof` 一样
```sh
docker run --name redis-slave1 \
-v /root/redis/redis-7002.conf:/etc/redis/redis.conf \
-v /root/redis/data:/data \
-dp 7002:6379 \
redis:latest \
redis-server /etc/redis/redis.conf --slaveof 192.168.56.10 7001
```
```sh
docker run --name redis-slave2 \
-v /root/redis/redis-7003.conf:/etc/redis/redis.conf \
-v /root/redis/data:/data \
-dp 7003:6379 \
redis:latest \
redis-server /etc/redis/redis.conf --slaveof 192.168.56.10 7001
```
> 关系查看
```sh
docker exec -it redis-master redis-cli info replication
```
![image-20240421142455620](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240421142455620.png)
```sh
docker exec -it redis-slave1 redis-cli info replication
```
```sh
docker exec -it redis-slave2 redis-cli info replication
```
> slave1
![image-20240421142858831](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240421142858831.png)
> slave2
![image-20240421142916435](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240421142916435.png)
> 此时停止master
```sh
docker stop redis-master
```
![image-20240421143048582](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240421143048582.png)
>尝试向slave中写入数据
```sh
docker exec -it redis-slave1 /bin/bash
redis-cli
set name tigerhhzz11
```
![image-20240421143606124](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240421143606124.png)
> 向master中写入数据，在slave中读出
![image-20240421143752145](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240421143752145.png)
![image-20240421143738686](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240421143738686.png)
## 数据同步原理
### 步骤
主从第一次同步是`全量同步`
![image-20240421144659573](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240421144659573.png)
> master怎么知道slave是否是第一次请求数据呢？这里会用到两个很重要的概念:
>
> `Replication id`:简称replid, 是数据集的标记，id一致则说明是同一数据集。每一个master都有唯一的replid,slave则会`继承`master节点的replid
>
> `offset`:偏移量,随着记录在repl_baklog 中的数据增多而逐渐增大。slave完成同步时也会记录当前同步的offset。如果slave的offset小于master的offset,说明slave数据落后于master,需要更新。
>
> 因此slave做数据同步,必须向master声明自己的`replication id`和`offset`, master才可以判断到底需要同步哪些数据
>
> master需要根据`replid`判断slave节点是否是第一次来做数据同步，`replid`不一致就是第一次做数据同步
**查看容器日志**
> 可以查看容器日志，数据同步步骤请看总结
```sh
docker logs redis-master 
```
```sh
docker logs redis-slave1
```
```sh
docker logs redis-slave2
```
### 总结
1. slave节点请求增量同步
2. master节点判断replid,发现不一致, 拒绝增量同步
3. master将完整内存数据生成RDB，发送RDB到slave
4. slave清空本地数据，加载master的RDB
5. master将RDB期间的命令记录在repl_baklog,并持续将log中的命令发送给slave
6. slave执行接收到的命令，保持与master之间的同步
### 增量同步
主从第一次同步是`全量同步`,但如果slave重启后同步，则执行`增量同步`
![image-20240421150448468](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240421150448468.png)
> `repl_baklog`本质是一个`环形数组`，slave与master数据之间的差异=master-slave（图中红色线段的部分）
>
> 当数据记完一圈后，会覆盖掉上次的内容继续往下记载，slave也会继续跟着master往后同步数据
>
> 但是这样就存在`无法进行增量同步`的情况
>
> `repl_baklog`大小有上限，写满后会覆盖最早的数据。如果slave断开时间过久，导致数据被覆盖,则无法实现增量同步，只能再次全量同步。
>
> 如果master已经覆盖掉上一次的内容，开始记入第二圈的数据，而slave由于某种原因宕机，导致master覆盖的数据并没有同步完成
![image-20240421151021414](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240421151021414.png)
> 此时就只能做全量同步了，无法进行增量同步
>
> 应当尽可能的减少全量同步，因为它的性能较差。或许可以进一步优化全量同步的性能
>
> 可以从以下几个方面来优化Redis主从就集群:
>
> 1. 在master中配置repl-diskless-sync yes启用无磁盘复制，避免全量同步时的磁盘IO。
>
>    (避免磁盘读写，采用网络传输，适用于磁盘较慢、网络较快的情况)
>
> 2. Redis单节点上的内存占用不要太大，减少RDB导致的过多磁盘IO
>
> 3. 适当提高repl_baklog的大小，发现slave宕机时尽快实现故障恢复,尽可能避免全量同步
>
> 4. 限制一个master上的slave节点数量，如果实在是太多slave,则可以采用`主-从-从`链式结构，减少master压力
![image-20240421151554098](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240421151554098.png)
### 总结
简述全量同步和增量同步区别?
1. 全量同步: master将完整 内存数据生成RDB，发送RDB到slave。后续命令则记录在repl_ baklog, 逐个发送给slave。
2. 增量同步: slave提交自己的offset到master, master获取repl_ baklog中 从offset之后的命令给slave
什么时候执行全量同步?
1. slave节点第一次连接master节点时
2. slave节点断开时间太久，repl_baklog中的offset已经被覆盖时
什么时候执行增量同步?
slave节点断开又恢复,并且在repl_baklog中 能找到offset时
## Redis哨兵
### 哨兵的作用和工作原理
> slave节点宕机恢复后可以找master节点同步数据，如果`master`节点`宕机`怎么办?
>
> master节点如果宕机，那么此时用户无法进行`redis数据写入`的操作，可用性下降了
>
> 我们需要去监控集群中的节点状态，在master节点挂掉之后，选一个新的master，增进挂掉的master重启后，作为slave
Redis提供了哨兵(Sentinel) 机制来实现主从集群的自动故障恢复。哨兵的结构和作用如下:
**监控**: Sentinel 会不断检查您的master和slave是否按预期工作
**自动故障恢复**:如果master故障，Sentinel会将一个slave提升为master。当故障实例恢复后也以新的master为主
**通知**: Sentinel充当Redis客户端的服务发现来源，当集群发生故障转移时，会将最新信息推送给Redis的客户端
![image-20240421195306145](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404212010655.png)
#### 服务状态监控
Sentinel基于心跳机制监测服务状态,每隔1秒向集群的每个实例发送ping命令:
- 主观下线:如果某sentinel节点发现某实例未在规定时间响应，则认为该实例主观下线。
- 客观下线:若超过指定数量( quorum)的sentinel都认为该实例主观下线，则该实例客观下线。quorum值最好超过Sentinel实例数量的一半。
![image-20240421200042664](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404212010983.png)
#### 选举新的master
一旦发现master故障，sentinel需 要在salve中选择- -一个作为新的master,选择依据是这样的:
1. 首先会判断slave节点与master节点断开时间长短，如果超过指定值( down-after-milliseconds * 10)则会排除该slave节点
2. 然后判断slave节点的slave-priority值, 越小优先级越高，如果是0则永不参与选举
3. 如果slave-prority一样，则判断slave节点的effset值， 越大说明数据越新，优先级越高
4. 最后是判断slave节点的运行id大小，越小优先级越高。
#### 实现故障转移
当选中了其中一个slave为新的master后(例如slave1)，故障的转移的步骤如下:
1. sentinel给 备选的slave1节点发送`slaveof no one`命令,让该节点成为master
2. sentinel给所有其它slave发送`slaveof 192.168.150.101 7002`命令，让这些slave成为新master的从节点，开始从新的master上同步数据。
3. 最后，sentinel将故障节点标记为slave,当故障节点恢复后会自动成为新的master的slave节点
![image-20240421201712016](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404212017069.png)
#### 总结
Sentinel的三个作用是什么?
1. 监控
2. 故障转移
3. 通知
Sentinel如何判断一个redis实例是否健康?
1. 每隔1秒发送一次ping命令，如果超过一定时间没有 相向则认为是主观下线
2. 如果大多数sentinel都认为实例主观下线，则判定服务下线
故障转移步骤有哪些?
1. 首先选定一个slave作为新的master,执行slaveof no one
2. 然后让所有节点都执行slaveof 新master
3. 修改故障节点配置，添加slaveof 新master
### 搭建哨兵集群
> 手动创建`sentinel`下的`s1、s2、s3`，然后配置`sentinel.conf`文件
![image-20240421210126992](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404212101080.png)
```
port 27001
sentinel announce-ip 192.168.56.10
sentinel monitor redis-master 192.168.56.10 7001 2
sentinel down-after-milliseconds redis-master 5000
sentinel failover-timeout redis-master 60000
```
解读：
- `port 27001`：是当前sentinel实例的端口
- `sentinel monitor redis-master192.168.56.10 7001 2`：指定主节点信息
  - `redis-master`：主节点名称，自定义，任意写
  - `192.168.56.10 7001`：主节点的ip和端口
  - `2`：选举master时的quorum值
> 启动`sentinel`的`jar`包
![image-20240421210646154](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404212106197.png)
> 启动三个sentinel容器
```sh
docker run --name sentinel-1 \
-v /root/redis/sentinel/sentinel1.conf:/etc/redis/sentinel.conf \
-dp 27001:26379 \
redis:latest \
redis-sentinel /etc/redis/sentinel.conf
```
```sh
docker run --name sentinel-2 \
-v /root/redis/sentinel/sentinel2.conf:/etc/redis/sentinel.conf \
-dp 27002:26379 \
redis:latest \
redis-sentinel /etc/redis/sentinel.conf
```
```sh
docker run --name sentinel-3 \
-v /root/redis/sentinel/sentinel3.conf:/etc/redis/sentinel.conf \
-dp 27003:26379 \
redis:latest \
redis-sentinel /etc/redis/sentinel.conf
```
> 关系查看
```sh
docker exec -it sentinel-1 redis-cli -h 192.168.56.10 -p 27001 info sentinel
```
![image-20240421222424683](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404212224751.png)
> 注意目录对应关系，如果在运行容器时，将`conf`路径写错了，则可能出现如下情况
![image-20240421222751214](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404212227284.png)
> 此时停止`redis-master`节点，查看`sentinel`日志
```sh
docker stop redis-master 
docker logs sentinel-1
```
![image-20240422064431835](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404220644926.png)
> 解决办法，直接挂载整个目录，而不是只挂载某个conf文件，注意`权限`设置为777，目录和文件都要设置
>
> `chmod 777 -R <file>`
![image-20240422073450817](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404220734941.png)
```sh
docker run --name sentinel-1 \
-v /root/redis/sentinel/s1:/etc/redis \
-dp 27001:26379 \
redis:latest \
redis-sentinel /etc/redis/sentinel.conf
```
```sh
docker run --name sentinel-2 \
-v /root/redis/sentinel/s2:/etc/redis \
-dp 27002:26379 \
redis:latest \
redis-sentinel /etc/redis/sentinel.conf
```
```sh
docker run --name sentinel-3 \
-v /root/redis/sentinel/s3:/etc/redis \
-dp 27003:26379 \
redis:latest \
redis-sentinel /etc/redis/sentinel.conf
```
> 但是日志打印还有一个警告
**1:X 21 Apr 2024 23:33:24.072 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.**
![image-20240422073645324](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404220736397.png)
> 解决办法
```sh
vim /etc/sysctl.conf
```
![image-20240422074046273](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404220740344.png)
> 文件新增如下内容
```conf
net.core.somaxconn = 1024
vm.overcommit_memory = 1
```
>添加完成后，刷新内核参数，立即生效
```sh
/sbin/sysctl -p
```
> 但是我这里并没有生效，所以采用如下方法
```sh
docker run --name sentinel-1 --sysctl net.core.somaxconn=1024 \
-v /root/redis/sentinel/s1:/etc/redis \
-dp 27001:26379 \
redis:latest \
redis-sentinel /etc/redis/sentinel.conf
```
```sh
docker run --name sentinel-2 --sysctl net.core.somaxconn=1024 \
-v /root/redis/sentinel/s2:/etc/redis \
-dp 27002:26379 \
redis:latest \
redis-sentinel /etc/redis/sentinel.conf
```
```sh
docker run --name sentinel-3 --sysctl net.core.somaxconn=1024 \
-v /root/redis/sentinel/s3:/etc/redis \
-dp 27003:26379 \
redis:latest \
redis-sentinel /etc/redis/sentinel.conf
```
> 问题已经解决
![image-20240422075110021](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404220751093.png)
> 停掉`redis-master`
>
> 查看sentinel日志
```
docker logs sentinel-3
```
![image-20240422080023345](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404220800499.png)
> 查看redis节点状态，发现`7003`确实成为`master`了
```sh
docker exec -it redis-slave2 redis-cli info replication
```
![image-20240422075815065](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404220758218.png)
> 重启`7001`，发现原来的`master`变成`slave`了
![image-20240422080253175](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404220802251.png)
### RedisTemplate的哨兵模式
> 新建项目，导入依赖
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```
```yaml
server:
    port: 8080
logging:
    level:
      io.lettuce.core: debug
    pattern:
        dateformat: yyyy-MM-dd HH:mm:ss:SSS
spring:
    data:
        redis:
            sentinel:
                # 指定master 节点名称
                master: mymaster
                # 指定redis-sentinel集群信息
                nodes:
                    - 192.168.56.10:27001
                    - 192.168.56.10:27002
                    - 192.168.56.10:27003
```
```java
@RestController
public class TestController {
    private StringRedisTemplate redisTemplate;
    @Autowired
    public TestController(StringRedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }
    @GetMapping("/get/{key}")
    public String get(@PathVariable String key) {
        return redisTemplate.opsForValue().get(key);
    }
    @GetMapping("/set/{key}/{value}")
    public String set(@PathVariable String key,@PathVariable String value) {
        redisTemplate.opsForValue().set(key,value);
        return "success";
    }
}
```
> 配置`Redis`读写分离
```java
@SpringBootApplication
public class RedisDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(RedisDemoApplication.class, args);
    }
    /**
     * 配置Redis的读写策略，实现master写入、slave读取(读写分离
     * MASTER:从主节点读取
     * MASTER_PREFERRED: 优先从master节点读取，master不可用才读取replica
     * REPLICA:从slave (replica) 节点读取
     * REPLICA_PREFERRED: 优先从slave (replica) 节点读取，所有的slave都不可用才读取master)
     *
     * @return {@link LettuceClientConfigurationBuilderCustomizer}
     */
    @Bean
    public LettuceClientConfigurationBuilderCustomizer configurationBuilderCustomizer(){
        return clientConfigurationBuilder -> clientConfigurationBuilder.readFrom(ReadFrom.REPLICA_PREFERRED);
    }
}
```
![image-20240422082247840](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404220822902.png)
> 报错`io.lettuce.core.RedisCommandExecutionException: ERR No such master with that name`
>
> 名称改为`redis-master`，与`sentinel.conf`文件中的配置保持一致
![image-20240422082422321](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404220824380.png)
```yaml
server:
    port: 8080
logging:
    level:
      io.lettuce.core: debug
    pattern:
        dateformat: yyyy-MM-dd HH:mm:ss:SSS
spring:
    data:
        redis:
            sentinel:
                # 指定master 节点名称
                master: redis-master
                # 指定redis-sentinel集群信息
                nodes:
                    - 192.168.56.10:27001
                    - 192.168.56.10:27002
                    - 192.168.56.10:27003
```
> 访问`http://localhost:8080/get/name`(得有一个叫name的数据)
![image-20240422082543326](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404220825414.png)
**可以看到这里通过7002来获取数据**
> 访问`http://localhost:8080/set/name/333`
![image-20240422082647557](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404220826733.png)
**可以看到这里通过7003来设置数据**
> 重启`redis-slave2`
```
docker restart redis-slave2
```
> 此时`slave1(7002)`被选为master
![image-20240422083027509](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404220830568.png)
![image-20240422083109475](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404220831656.png)
> `7003`为`replica`
![image-20240422083125383](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404220831464.png)
> 使用idea连接虚拟机
>
> `Deployment->Configuration`
![image-20240422201104329](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404222011412.png)
> 新建`SFTP`
![image-20240422201138710](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404222011833.png)
![image-20240422201502055](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404222015126.png)
> 查看虚拟机文件
>
> `Deployment->Browse Remote Host`
![image-20240422201551614](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404222015689.png)
> 右侧就可以看到虚拟机的文件信息了
![image-20240422201634100](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404222016266.png)
> 打开命令窗口
>
> 点击`Tools->Start SSH Session`
![image-20240422201705287](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404222017371.png)
![image-20240422201727483](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404222017656.png)
## Redis分片集群
> 我们知道单节点的Redis内存不要设置过高，在内存的持久化或者数据同步的时候会导致大量的IO，性能会下降
>
> 但是如果内存降低之后无法存储海量的数据怎么办？
>
> 解决了高并发读，高并发写怎么解决？
### 分片集群结构
主从和哨兵可以解决高可用、高并发读的问题。但是依然有两个问题没有解决:
- 海量数据存储问题
- 高并发写的问题
使用分片集群可以解决.上述问题，分片集群特征:
- 集群中有多个master,每个master保存不同数据
- 每个master都可以有多个slave节点
- master之间通过ping监测彼此健康状态
- 客户端请求可以访问集群任意节点，最终都会被转发到正确节点
> 此时不再需要哨兵机制了，`master`节点之间可以互相监控各自的状态
>
> 集群结构如下表所示
| 序号 | 角色   | 容器名称         | 网络模式 | 地址及端口         |
| ---- | ------ | ---------------- | -------- | ------------------ |
| 1    | master | master-cluster-1 | host     | 192.168.56.10:8001 |
| 2    | master | master-cluster-2 | host     | 192.168.56.10:8002 |
| 3    | master | master-cluster-3 | host     | 192.168.56.10:8003 |
| 4    | slave  | slave-cluster-1  | host     | 192.168.56.10:9001 |
| 5    | slave  | slave-cluster-2  | host     | 192.168.56.10:9002 |
| 6    | slave  | slave-cluster-3  | host     | 192.168.56.10:9003 |
> 在`/root/redis-cluster`下新建配置文件`redis.conf`
>
> 并打开以下两处的注释
```properties
# 开启 cluster 功能，即分布式系统功能
cluster-enabled yes
# 指定其需要的配置文件名称
cluster-config-file nodes-6379.conf
```
![image-20240422204825169](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404222048262.png)
> 重命名`redis.conf`为`redis1.conf`，并复制成其他五份`redis2.conf、redis3.conf、redis4.conf、redis5.conf、redis6.conf`
>
> 这6份配置文件内容完全相同
>
> 更改文件之后记得上传
![image-20240422205154155](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404222051246.png)
> 停止之前运行的`redis、sentinel`容器
```sh
docker stop redis-master redis-slave1 redis-slave2 sentinel-1 sentinel-2 sentinel-3
```
![image-20240422205342819](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404222053908.png)
```sh
docker run --name master-cluster-1 \
--network host \
-v /root/redis-cluster/redis1.conf:/etc/redis/redis.conf \
-v /root/redis-cluster/data/8001:/data \
-d redis:latest redis-server /etc/redis/redis.conf \
--port 8001
```
```sh
docker run --name master-cluster-2 \
--network host \
-v /root/redis-cluster/redis2.conf:/etc/redis/redis.conf \
-v /root/redis-cluster/data/8002:/data \
-d redis:latest redis-server /etc/redis/redis.conf \
--port 8002
```
```sh
docker run --name master-cluster-3 \
--network host \
-v /root/redis-cluster/redis3.conf:/etc/redis/redis.conf \
-v /root/redis-cluster/data/8003:/data \
-d redis:latest redis-server /etc/redis/redis.conf \
--port 8003
```
```sh
docker run --name slave-cluster-1 \
--network host \
-v /root/redis-cluster/redis4.conf:/etc/redis/redis.conf \
-v /root/redis-cluster/data/9001:/data \
-d redis:latest redis-server /etc/redis/redis.conf \
--port 9001
```
```sh
docker run --name slave-cluster-2 \
--network host \
-v /root/redis-cluster/redis5.conf:/etc/redis/redis.conf \
-v /root/redis-cluster/data/9002:/data \
-d redis:latest redis-server /etc/redis/redis.conf \
--port 9002
```
```sh
docker run --name slave-cluster-3 \
--network host \
-v /root/redis-cluster/redis6.conf:/etc/redis/redis.conf \
-v /root/redis-cluster/data/9003:/data \
-d redis:latest redis-server /etc/redis/redis.conf \
--port 9003
```
> 6 个节点启动后，它们仍是 6 个独立的 Redis，通过 `redis-cli --cluster create` 命令可将 6个节点创建为一个分布式系统。–cluster replicas 1 指定每个 master 会带有一个slave 副本。
>
> 所以前三个是`主`，后三个是`从`
```sh
docker exec -it master-cluster-1 /bin/bash
redis-cli --cluster create --cluster-replicas 1 192.168.56.10:8001 192.168.56.10:8002 192.168.56.10:8003 192.168.56.10:9001 192.168.56.10:9002 192.168.56.10:9003
```
![image-20240422210937770](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404222109918.png)
> 查看节点信息
```sh
redis-cli -c -p 8003 cluster nodes
```
![image-20240422211623156](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404222116235.png)
> 这样就可以查看到节点的信息了
### 散列插槽
Redis会把每一个master节 点映射到0~16383共16384个插槽(hashslot) 上，查看集群信息时就能看到:
![image-20240423215451539](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404232154657.png)
> 我们这里插槽分布如下
| 端口 | 插槽范围    |
| ---- | ----------- |
| 8001 | 0-5460      |
| 8002 | 5461-10922  |
| 8003 | 10923-16383 |
数据key不是与节点绑定,而是与插槽绑定。redis会根据key的有效部分计算插槽值，分两种情况:
- key中包含"{}",且"{}"中至少包含1个字符， "{}"中的部分是有效部分
- key中不包含"{}" ，整个key都是有效部分
例如: key是num,那么就根据num计算,如果是{itcast}num,则根据itcast计算。计算方式是利用CRC16算法得到一个hash值，然后对16384取余,得到的结果就是slot值。
> 连接redis
>
> 注意这里一定要加`-c`，代表`集群模式`
```sh
redis-cli -c -p 8001
```
![image-20240423221028770](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404232210854.png)
> redis会先算出插槽的值，最后找到对应的ip及端口，存储(获取)对应redis实例里的数据
Redis如何判断某个key应该在哪个实例?
1. 将16384个插槽分配到不同的实例
2. 根据key的有 效部分计算哈希值，对16384取余余数作为插槽，寻找插槽所在实例即可
如何将同一类数据固定的保存在同一个Redis实例?
这一类数据使用相同的有效部分，例如key都以{typeld}为前缀，这样{a}name、{a}gender就能都存储在8003端口上
### 集群伸缩
redis-cli --cluster提供了很多操作集群的命令，可以通过下面方式查看:
```sh
docker exec -it master-cluster-1 /bin/bash
redis-cli --cluster help
```
![image-20240424073841796](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404240738901.png)
> 向集群中添加一个新的master节点,并向其中存储num = 10
**需求**
启动一个新的redis实例，端口为`8004`
添加8004到之前的集群，并作为一个master节点
给8004节点分配插槽，使得num这个key可以存储到8004实例
> 分析，这里有几个关键点
>
> 1. 新增redis(master)实例到集群
> 2. 分配插槽，使其能够存储`num`作为key(hashkey为`2765`)
>
> 参考[博客](https://www.cnblogs.com/zhoujinyi/p/11606935.html)
**复制一份`redis7.conf`文件**
```sh
docker run --name master-cluster-4 \
--network host \
-v /root/redis-cluster/redis7.conf:/etc/redis/redis.conf \
-v /root/redis-cluster/data/8004:/data \
-d redis:latest redis-server /etc/redis/redis.conf \
--port 8004
```
![image-20240424080947181](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404240809279.png)
> 可以看到集群节点里并没有新建的`8004`
```sh
docker exec -it master-cluster-1 /bin/bash
redis-cli -c -p 8003 cluster nodes
```
![image-20240424082942970](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242013465.png)
> 查看帮助文档
```sh
redis-cli --cluster help
```
![image-20240424082917801](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404240829903.png)
> 添加节点到集群
```sh
redis-cli --cluster add-node 192.168.56.10:8004 192.168.56.10:8003
```
![image-20240424202128104](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242021196.png)
> 再次查看集群信息，可以看到`8004`是作为master的，但是并没有给它分配插槽
```sh
redis-cli -c -p 8003 cluster nodes
```
![image-20240424202157821](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242021909.png)
> 接下来需要为`8004`分配至少`0~2765`的插槽，这里就直接分配前`3000`个了
![image-20240424204432417](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242044685.png)
> 输入`yes`
>
> 再次查看节点信息，可以看到已经分配了`0-2999`的插槽
```sh
redis-cli -p 8004 cluster nodes
```
![image-20240424204721576](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242047718.png)
> 试一下获取`num`，可以看到是从8004上获取的数据
```sh
redis-cli -c -p 8001
```
![image-20240424204902473](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242049543.png)
> **练习**
>
> 删除8004节点
![image-20240424205045856](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242050050.png)
> 转移插槽
![image-20240424205333760](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242053831.png)
> 删除节点，可以看到`8004`节点已经被移除
```sh
redis-cli --cluster del-node 192.168.56.10:8004 969b40741ba2c93557ff9013a774f98322f77ba0
```
![image-20240424205522582](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242055720.png)
> 删除容器
![image-20240424205605167](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242056356.png)
```sh
docker rm -f master-cluster-4
docker ps
```
### 故障转移
#### 自动故障转移
> 当集群中有一个master宕机会发生什么呢?
![image-20240424205957402](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242059554.png)
> 重新启动8002
![image-20240424210155393](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242101468.png)
> 可以看到`8002`变成了`master`，而`9002`变成了`slaver`，实现了自动故障转移
#### 手动故障转移
利用`cluster failover`命令可以手动让集群中的某个master宕机，切换到执行cluster failover命令的这个slave节点，实现无感知的数据迁移。其流程如下:
![image-20240424210353844](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242103921.png)
手动的Failover支持三种不同模式:
1. 缺省:默认的流程，如图1~6步
2. force:省略了对offset的一致性校验
3. takeover:直接执行第5步，忽略数据一致性、忽略master状态和其它master的意见
> **练习**
>
> 手动故障转移，让`8002`重新变成master
![image-20240424210932755](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242109850.png)
```sh
CLUSTER FAILOVER
```
> 看到`8002`重新变成master
![image-20240424211012127](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242110235.png)
### RedisTemplate
RedisTemplate底层同样基于lettuce实现了分片集群的支持，而使用的步骤与哨兵模式基本一致:
1. 引入redis的starter依赖
2. 配置分片集群地址
3. 配置读写分离
与哨兵模式相比，其中只有分片集群的配置方式略有差异，如下:
```yaml
# 配置分片集群
spring:
    data:
        redis:
            cluster:
                # 指定分片集群的每一个节点信息
                nodes:
                    - 192.168.56.10:8001
                    - 192.168.56.10:8002
                    - 192.168.56.10:8003
                    - 192.168.56.10:9001
                    - 192.168.56.10:9002
                    - 192.168.56.10:9003
```
> 访问`http://localhost:8080/get/a`
>
> 查看日志，通过`9003`获取到`a`
![image-20240424212554166](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242125244.png)
> 查看节点信息
>
> `9003`的master是`8003`
```sh
redis-cli -c -p 8003 cluster nodes
```
![image-20240424212728958](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242127034.png)
> 访问`http://localhost:8080/set/a/666`
>
> 通过`8003`去set值
![image-20240424212846737](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202404242128817.png)
