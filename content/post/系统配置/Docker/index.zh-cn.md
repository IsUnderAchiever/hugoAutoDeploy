---
title: Docker
description: Docker配置
date: 2022-12-14
slug: Docker配置
image: dbc-docker-desktop-home.webp
categories:
    - 系统配置
---

## Docker环境配置
1. 安装VirtualBox
2. 安装[vagrant](https://developer.hashicorp.com/vagrant/downloads)
3. 安装完成vagrant后重启一下系统
4. 验证vagrant是否安装成功
   - cmd输入vagrant，是否有命令提示
## 使用vagrant创建虚拟机——[官方镜像仓库](https://app.vagrantup.com/boxes/search)
![001.初始化centos](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201958195.png)
```bash
vagrant init centos/7
```
此时在E盘已经生成了一个名为“Vagrantfile”的文件	
启动虚拟环境
![002.启动虚拟环境](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201958546.png)
```bash
vagrant up
# 如果速度很慢，使用提前下载好的CentOS7镜像，速度嘎嘎快
# vagrant init centos/7 # 没初始化的，要初始化
vagrant box add centos/7 CentOS-7-x86_64-Vagrant-2004_01.VirtualBox.box
vagrant up
```
ctrl+c停掉
```
vagrant ssh
```
提示[vagrant@localhost ~]$  连接成功，此时已经可以在VirtualBox内查看到该虚拟机
若提示 vagrant@127.0.0.1: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
则输入命令set VAGRANT_PREFER_SYSTEM_BIN=0 后再重新连接即可
退出vagrant的方法：输入命令exit
以文本的方式打开Vagrantfile，找到这一行配置
![003更改网络配置](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201958434.png)
```bash
# config.vm.network "private_network", ip: "192.168.33.10"
```
所以我们需要将虚拟机改为192.168.56.XX
即更改为
```bash
# config.vm.network "private_network", ip: "192.168.56.10"
```
```
vagrant reload
```
重启虚拟机
```bash
vagrant ssh
ip addr # 查看虚拟机ip地址是否已经发生了改变
ping 192.168.xxx # 主机与虚拟机是否互相ping通
```
## 安装docker
概念：什么是docker？
虚拟化容器技术。Docker基于镜像，可以秒级启动各种容器。每一种容器都是一个完整的运行环境，容器之间互相隔离。
### [docker的镜像市场](https://hub.docker.com/)
[安装docker](https://docs.docker.com/engine/install/centos/)
```bash
vagrant ssh # 连接虚拟机
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
# 卸载旧版本的docker
```
```bash
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
# 过程需要输入y选择确认
# 启动docker
sudo systemctl start docker
# 查看docker版本
docker -v
# 查看docker安装了哪些镜像
sudo docker images
# 设置docker开机自启动
sudo systemctl enable docker
```
## 镜像加速
登录阿里云后，进入 控制台，左侧菜单找到 “产品与服务”，选择“容器镜像服务”，选择“镜像工具”，点击“镜像加速器”
```bash
# 执行以下四个命令
sudo mkdir -p /etc/docker
#----------------------------------
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://g9nk2v6o.mirror.aliyuncs.com"]
}
EOF
#----------------------------------
sudo systemctl daemon-reload
#----------------------------------
sudo systemctl restart docker
```
`/etc/docker/daemon.json`

```json
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://docker.nju.edu.cn"
  ],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
```



## 切换root用户

每次sudo很麻烦？直接切换root用户
```bash
su root
# 默认密码是vagrant
```
## 连接XShell
```bash
vagrant ssh
# 切换为root用户
sudo -i
# 修改root用户密码
passwd
sudo vi /etc/ssh/sshd_config
# 找到PasswordAuthentication yes这一行，去掉前面的#
# # To disable tunneled clear text passwords, change to no here!
# PasswordAuthentication yes
# #PermitEmptyPasswords no
# PasswordAuthentication no
# 保存后退出
# 重启sshd服务
systemctl restart sshd
# 使用ssh工具进行连接
```
详情请查看这篇[博客](https://blog.csdn.net/qq_43651260/article/details/128055181)
## 安装MySQL
连接上虚拟机之后
在docker镜像仓库搜索mysql，进入tags可查看版本信息
```bash
#------- 安装mysql5.7-----------------
# docker pull mysql # 会下载最新的mysql
# 下载指定版本的mysql，比如5.7
# docker pull mysql:5.7
sudo docker pull mysql:5.7
sudo docker run -p 3306:3306 \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql:8.0.20 \
--name mysql
# 查看mysql是否运行
docker ps
# 没有查看到容器
#  查看,发现容器是退出状态
docker ps -a
# Exited
# 运行以下命令
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql
# 若依然存在问题,可以停用并删除容器,重新启动mysql
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
# 删除镜像
docker rmi 镜像id
# docker 删除报错:
Error response from daemon: conflict: unable to delete 8e6aee9da407 (must be forced) - image is referenced in multiple repositories
# 解决办法
docker rmi -f 镜像id
# docker 服务命令
# 启动：
# systemctl start docker
# 守护进程重启：
# systemctl daemon-reload
# 重启docker服务：
# systemctl restart docker / service docker restart
# 关闭：
# docker service docker stop / docker systemctl stop docker
# 问题解决之后------
# 进入配置文件挂载的目录下
cd /mydata/mysql/conf
# 编辑配置文件my.cnf
vi my.cnf
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
# 重启mysql容器
docker restart mysql
# 设置启动docker时，mysql自启动
docker update mysql --restart=always
#------- 安装mysql8.0.20-----------------
#拉取镜像
docker pull mysql:8.0.20
#启动镜像,用于拷贝配置文件到宿主机
docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0.20
#查看是否启动成功
docker ps -a
#新建挂载目录并拷贝配置文件
mkdir -p /mysqldata/
docker cp  mysql:/etc/mysql /mysqldata/
#检查是否运行成功
docker ps -a
# 重启mysql容器
docker restart mysql
# 设置启动docker时，mysql自启动
docker update mysql --restart=always
# 使用Navicat链接mysql
# ql自启动
docker update mysql --restart=always
```
## Redis配置
```bash
# 下载最新redis镜像
docker pull redis
```
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201958499.png)
```bash
mkdir -p /mydata/redis/conf
cd /mydata/redis/conf
touch redis.conf
docker run -p 6379:6379 --name redis -v /mydata/redis/data:/data \
-v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
-d redis redis-server /etc/redis/redis.conf
```
![2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201958293.png)
![3](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201958909.png)
```bash
# 测试一下redis
docker exec -it redis redis-cli
```
![4](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201958121.png)
```bash
# redis默认配置是没有持久化的
# 如果重启redis，再获取之前的数据就获取不到了
# 具体命令可查看下图
vi redis.conf
# 【输入 i 进入插入模式】
# 开启AOF持久化
appendonly yes
```
![5](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201958706.png)
![6](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201958530.png)
##### 修改redis的密码
```bash
# 创建容器时设置密码
# docker run -itd --name redis-6379 -p 6379:6379 redis --requirepass 123456
# 为现有的redis创建密码或修改密码
#1.进入redis的容器
docker exec -it 容器ID bash
#2.进入redis目录
cd /usr/local/bin
#3.运行命令：
redis-cli
#4.查看现有的redis密码：
config get requirepass
# 这一步若报错(error) NOAUTH Authentication required.
# 是因为redis本身设置有密码，此时需要执行以下命令
auth password; # password是之前设置的redis密码
#5.设置redis密码
config set requirepass 密码
# 清空redis密码
config set requirepass ""
```
### 安装redis可视化工具
链接：https://pan.baidu.com/s/10z0S6b9mo76CWK9y3spPRg?pwd=51tz 
提取码：51tz 
--来自百度网盘超级会员V3的分享
![7](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201958101.png)
点击New Connection右边的按钮，可进入设置页面，设置语言
![8](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201959057.png)
## docker安装ElasticSearch
### 1.黑马程序员视频内安装方法：
链接：https://pan.baidu.com/s/1pGnbmkMjD8RXe5Y4xotfOA?pwd=ihe5 
提取码：ihe5 
![image-20210510165308064](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202002537.png)
将其上传到虚拟机中，然后运行命令加载即可：
```sh
# 导入数据
docker load -i es.tar
```
同理还有`kibana`的tar包也需要这样做。
运行docker命令，部署单点es：
```sh
docker run -d \
	--name es \
    -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
    -e "discovery.type=single-node" \
    -v es-data:/usr/share/elasticsearch/data \
    -v es-plugins:/usr/share/elasticsearch/plugins \
    --privileged \
    --network es-net \
    -p 9200:9200 \
    -p 9300:9300 \
    elasticsearch:7.12.1
```
```sh
docker run -d \
	--name elasticsearch \
    -v /home/tong/software/docker/elasticsearch/data:/usr/share/elasticsearch/data \
    -v /home/tong/software/docker/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
    -v /home/tong/software/docker/elasticsearch/config:/usr/share/elasticsearch/config \
    -p 9200:9200 \
    -p 9300:9300 \
    elasticsearch:8.7.0
```

```sh
docker run -d \
	--name elasticsearch \
    -p 9200:9200 \
    -p 9300:9300 \
    elasticsearch:8.7.0
```

命令解释：

- `-e "cluster.name=es-docker-cluster"`：设置集群名称
- `-e "http.host=0.0.0.0"`：监听的地址，可以外网访问
- `-e "ES_JAVA_OPTS=-Xms512m -Xmx512m"`：内存大小
- `-e "discovery.type=single-node"`：非集群模式
- `-v es-data:/usr/share/elasticsearch/data`：挂载逻辑卷，绑定es的数据目录
- `-v es-logs:/usr/share/elasticsearch/logs`：挂载逻辑卷，绑定es的日志目录
- `-v es-plugins:/usr/share/elasticsearch/plugins`：挂载逻辑卷，绑定es的插件目录
- `--privileged`：授予逻辑卷访问权
- `--network es-net` ：加入一个名为es-net的网络中
- `-p 9200:9200`：端口映射配置
运行docker命令，部署kibana
```sh
docker run -d \
--name kibana \
-e ELASTICSEARCH_HOSTS=http://es:9200 \
--network=es-net \
-p 5601:5601  \
kibana:7.12.1
```
- `--network es-net` ：加入一个名为es-net的网络中，与elasticsearch在同一个网络中
- `-e ELASTICSEARCH_HOSTS=http://es:9200"`：设置elasticsearch的地址，因为kibana已经与elasticsearch在一个网络，因此可以用容器名直接访问elasticsearch
- `-p 5601:5601`：端口映射配置
kibana启动一般比较慢，需要多等待一会，可以通过命令：
```sh
docker logs -f kibana
```
查看运行日志，当查看到下面的日志，说明成功：
![image-20210109105135812](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202010279.png)
### 2.网上的安装方法：
```bash
# 下载镜像文件
docker pull elasticsearch:7.4.2
# 创建搭载目录
mkdir -p /mydata/elasticsearch/config
mkdir -p /mydata/elasticsearch/data
echo "http.host: 0.0.0.0" > /mydata/elasticsearch/config/elasticsearch.yml
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e  "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
-v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-v  /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.4.2
docker ps# 查看容器状态 up 运行
```
若存在问题，查看这篇[博客](https://blog.csdn.net/qq_38327769/article/details/124847508?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522167101576116800182116891%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=167101576116800182116891&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-2-124847508-null-null.142^v68^control,201^v4^add_ask,213^v2^t3_control1&utm_term=docker%E5%AE%89%E8%A3%85ElasticSearch&spm=1018.2226.3001.4187)
我这里的问题是启动的时候没问题，但是过一会有自动关闭
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201959669.png)
解决问题和上方博客中写的一样
```bash
chmod -R 777 /mydata/elasticsearch/
docker start elasticsearch
```
查看运行结果
http://虚拟机ip地址:9200/ 
![2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201959677.png)
### 安装Kibana
```bash
# 下载镜像文件
docker pull kibana:7.4.2
# 安装容器
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://自己的IP:9200 -p 5601:5601 -d kibana:7.4.2
```
## 安装ik分词器
[ik分词器下载地址](https://github.com/medcl/elasticsearch-analysis-ik/releases)
### 3.1.在线安装ik插件（较慢）
```shell
# 进入容器内部
docker exec -it elasticsearch /bin/bash
# 在线下载并安装
./bin/elasticsearch-plugin  install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.12.1/elasticsearch-analysis-ik-7.12.1.zip
#退出
exit
#重启容器
docker restart elasticsearch
```
### 3.2.离线安装ik插件（推荐）
### 1）查看数据卷目录
安装插件需要知道elasticsearch的plugins目录位置，而我们用了数据卷挂载，因此需要查看elasticsearch的数据卷目录，通过下面命令查看:
```sh
docker volume inspect es-plugins
```
显示结果：
```json
[
    {
        "CreatedAt": "2022-05-06T10:06:34+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/es-plugins/_data",
        "Name": "es-plugins",
        "Options": null,
        "Scope": "local"
    }
]
```
说明plugins目录被挂载到了：`/var/lib/docker/volumes/es-plugins/_data `这个目录中。
### 2）解压缩分词器安装包
下面我们需要把课前资料中的ik分词器解压缩，重命名为ik
![image-20210506110249144](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202011441.png)
### 3）上传到es容器的插件数据卷中
也就是`/var/lib/docker/volumes/es-plugins/_data `：
![image-20210506110704293](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202011084.png)
###  4）重启容器
```shell
# 4、重启容器
docker restart es
```
```sh
# 查看es日志
docker logs -f es
```
### 5）测试：
IK分词器包含两种模式：
* `ik_smart`：最少切分
* `ik_max_word`：最细切分
```json
GET /_analyze
{
  "analyzer": "ik_max_word",
  "text": "黑马程序员学习java太棒了"
}
```
结果：
```json
{
  "tokens" : [
    {
      "token" : "黑马",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "程序员",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "程序",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "员",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "CN_CHAR",
      "position" : 3
    },
    {
      "token" : "学习",
      "start_offset" : 5,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 4
    },
    {
      "token" : "java",
      "start_offset" : 7,
      "end_offset" : 11,
      "type" : "ENGLISH",
      "position" : 5
    },
    {
      "token" : "太棒了",
      "start_offset" : 11,
      "end_offset" : 14,
      "type" : "CN_WORD",
      "position" : 6
    },
    {
      "token" : "太棒",
      "start_offset" : 11,
      "end_offset" : 13,
      "type" : "CN_WORD",
      "position" : 7
    },
    {
      "token" : "了",
      "start_offset" : 13,
      "end_offset" : 14,
      "type" : "CN_CHAR",
      "position" : 8
    }
  ]
}
```
## 3.3 扩展词词典
随着互联网的发展，“造词运动”也越发的频繁。出现了很多新的词语，在原有的词汇列表中并不存在。比如：“奥力给”，“传智播客” 等。
所以我们的词汇也需要不断的更新，IK分词器提供了扩展词汇的功能。
1）打开IK分词器config目录：
![image-20210506112225508](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202012285.png)
2）在IKAnalyzer.cfg.xml配置文件内容添加：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 *** 添加扩展词典-->
        <entry key="ext_dict">ext.dic</entry>
</properties>
```
3）新建一个 ext.dic，可以参考config目录下复制一个配置文件进行修改
```properties
传智播客
奥力给
```
4）重启elasticsearch 
```sh
docker restart es
# 查看 日志
docker logs -f elasticsearch
```
![image-20201115230900504](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202012676.png)
日志中已经成功加载ext.dic配置文件
5）测试效果：
```json
GET /_analyze
{
  "analyzer": "ik_max_word",
  "text": "传智播客Java就业超过90%,奥力给！"
}
```
> 注意当前文件的编码必须是 UTF-8 格式，严禁使用Windows记事本编辑
