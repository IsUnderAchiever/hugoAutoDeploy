---
title: Nexus搭建Maven私服
description: Nexus搭建Maven私服
date: 2025-06-02
slug: Nexus搭建Maven私服
image: 202506081432952.png
categories:
    - 系统配置
---
# Nexus搭建Maven私服

参考[Nexus私服搭建与Maven仓库](https://developer.aliyun.com/article/1328445)、[MAVEN 私有仓库搭建与配置](https://blog.csdn.net/z562743237/article/details/108852509)

## 启动容器

**1. 拉取镜像并启动容器**
```bash
docker run -d \
  --name nexus \
  -p 8081:8081 \
  -p 5000:5000 \
  -v nexus-data:/nexus-data \
  sonatype/nexus3

docker run -d \
  --name nexus \
  -p 8081:8081 \
  -p 5000:5000 \
  -e JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64 \
  -e JAVA_OPTS="-Xms4g -Xmx4g" \
  -v nexus-data:/nexus-data \
  sonatype/nexus3
  
# 创建数据卷
docker volume create nexus-localtime
docker volume create nexus-data

# 问题命令，不要选择此命令创建
docker run -itd \
        --name=nexus \
        -p 8081:8081 \
        -p 5000:5000 \
        --privileged=true \
        --restart=always \
        --ulimit nofile=655350 \
        --ulimit memlock=-1 \
        --memory=16G \
        --memory-swap=-1 \
        --cpuset-cpus='1-7' \
        -e INSTALL4J_ADD_VM_PARAMS="-Xms4g -Xmx4g -XX:MaxDirectMemorySize=8g" \
        -v nexus-localtime:/etc/localtime \
        -v nexus-data:/nexus-data \
        sonatype/nexus3:latest
        
# 最终解决命令
docker run -itd \
        --name=nexus \
        -p 8081:8081 \
        -p 5000:5000 \
        --privileged=true \
        --restart=always \
        --ulimit nofile=655350 \
        --ulimit memlock=-1 \
        --memory=16G \
        --memory-swap=-1 \
        --cpuset-cpus='1-7' \
        -e INSTALL4J_ADD_VM_PARAMS="-Xms4g -Xmx4g -XX:MaxDirectMemorySize=8g -Djava.util.prefs.userRoot=/nexus-data/javaprefs" \
        -v nexus-localtime:/etc/localtime \
        -v nexus-data:/nexus-data \
        sonatype/nexus3:latest
```
- `-v nexus-data:/nexus-data`：持久化存储数据（生产环境建议替换为本地目录，如 `-v /opt/nexus-data:/nexus-data`）
- `8081`：Nexus 管理界面端口
- `5000`：Docker 仓库的默认端口（可自定义）

#### 遇到的问题

> 使用问题命令后访问页面，容器出现如下报错

```
-------------------------------------------------

Started Sonatype Nexus COMMUNITY 3.78.1-02

-------------------------------------------------
2025-05-31 15:03:38,480+0000 WARN  [Timer-0] *SYSTEM java.util.prefs - Could not lock User prefs.  Unix error code 2.
2025-05-31 15:03:38,481+0000 WARN  [Timer-0] *SYSTEM java.util.prefs - Couldn't flush user prefs: java.util.prefs.BackingStoreException: Couldn't get file lock.
2025-05-31 15:03:54,729+0000 INFO  [qtp1749978269-32] *UNKNOWN org.apache.shiro.session.mgt.AbstractValidatingSessionManager - Enabling session validation scheduler...
2025-05-31 15:03:54,751+0000 INFO  [qtp1749978269-32] *UNKNOWN org.sonatype.nexus.internal.security.anonymous.AnonymousManagerImpl - Using default configuration: AnonymousConfigurationData{enabled=true, userId='anonymous', realmName='NexusAuthorizingRealm'}
2025-05-31 15:04:08,480+0000 WARN  [Timer-0] *SYSTEM java.util.prefs - Could not lock User prefs.  Unix error code 2.
2025-05-31 15:04:08,480+0000 WARN  [Timer-0] *SYSTEM java.util.prefs - Couldn't flush user prefs: java.util.prefs.BackingStoreException: Couldn't get file lock.
2025-05-31 15:04:18,680+0000 WARN  [pool-3-thread-1] anonymous com.sonatype.nexus.plugins.outreach.internal.outreach.SonatypeOutreach - Could not download page bundle
org.apache.http.conn.ConnectTimeoutException: Connect to sonatype-download.global.ssl.fastly.net:443 [sonatype-download.global.ssl.fastly.net/199.96.63.177] failed: Connect timed out
        at org.apache.http.impl.conn.DefaultHttpClientConnectionOperator.connect(DefaultHttpClientConnectionOperator.java:151)
        at org.apache.http.impl.conn.PoolingHttpClientConnectionManager.connect(PoolingHttpClientConnectionManager.java:376)
        at org.apache.http.impl.execchain.MainClientExec.establishRoute(MainClientExec.java:393)
        at org.apache.http.impl.execchain.MainClientExec.execute(MainClientExec.java:236)
        at org.apache.http.impl.execchain.ProtocolExec.execute(ProtocolExec.java:186)
        at org.apache.http.impl.execchain.RetryExec.execute(RetryExec.java:89)
        at org.apache.http.impl.execchain.RedirectExec.execute(RedirectExec.java:110)
        at org.apache.http.impl.client.InternalHttpClient.doExecute(InternalHttpClient.java:185)
        at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:83)
        at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:108)
        at com.sonatype.nexus.plugins.outreach.internal.outreach.OutreachConnector.get(OutreachConnector.java:136)
        at com.sonatype.nexus.plugins.outreach.internal.outreach.SonatypeOutreach.remote(SonatypeOutreach.java:280)
        at com.sonatype.nexus.plugins.outreach.internal.outreach.SonatypeOutreach.getOutreachBundle(SonatypeOutreach.java:164)
        at com.sonatype.nexus.plugins.outreach.internal.outreach.SonatypeOutreach.getPageBundle(SonatypeOutreach.java:153)
        at com.sonatype.nexus.plugins.outreach.internal.ui.OutreachComponent.readStatus(OutreachComponent.java:74)
        at com.sonatype.nexus.plugins.outreach.internal.ui.OutreachComponent$$EnhancerByGuice$$5c476288.GUICE$TRAMPOLINE(<generated>)
        at com.google.inject.internal.InterceptorStackCallback$InterceptedMethodInvocation.proceed(InterceptorStackCallback.java:74)
        at com.palominolabs.metrics.guice.ExceptionMeteredInterceptor.invoke(ExceptionMeteredInterceptor.java:23)
        at com.google.inject.internal.InterceptorStackCallback$InterceptedMethodInvocation.proceed(InterceptorStackCallback.java:75)
        at com.palominolabs.metrics.guice.TimedInterceptor.invoke(TimedInterceptor.java:26)
        at com.google.inject.internal.InterceptorStackCallback$InterceptedMethodInvocation.proceed(InterceptorStackCallback.java:75)
        at com.google.inject.internal.InterceptorStackCallback.invoke(InterceptorStackCallback.java:55)
        at com.sonatype.nexus.plugins.outreach.internal.ui.OutreachComponent$$EnhancerByGuice$$5c476288.readStatus(<generated>)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77)
        at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.base/java.lang.reflect.Method.invoke(Method.java:569)
        at com.softwarementors.extjs.djn.router.dispatcher.DispatcherBase.invokeJavaMethod(DispatcherBase.java:142)
        at com.softwarementors.extjs.djn.router.dispatcher.DispatcherBase.invokeMethod(DispatcherBase.java:133)
        at org.sonatype.nexus.extdirect.internal.ExtDirectDispatcher.invokeMethod(ExtDirectDispatcher.java:82)
        at com.softwarementors.extjs.djn.router.dispatcher.DispatcherBase.dispatch(DispatcherBase.java:63)
        at com.softwarementors.extjs.djn.router.processor.standard.StandardRequestProcessorBase.dispatchStandardMethod(StandardRequestProcessorBase.java:73)
        at com.softwarementors.extjs.djn.router.processor.standard.json.JsonRequestProcessor.processIndividualRequest(JsonRequestProcessor.java:502)
        at com.softwarementors.extjs.djn.router.processor.standard.json.DefaultJsonRequestProcessorThread.processRequest(DefaultJsonRequestProcessorThread.java:72)
        at com.softwarementors.extjs.djn.servlet.ssm.SsmJsonRequestProcessorThread.processRequest(SsmJsonRequestProcessorThread.java:43)
        at org.sonatype.nexus.extdirect.internal.ExtDirectJsonRequestProcessorThread.access$001(ExtDirectJsonRequestProcessorThread.java:35)
        at org.sonatype.nexus.extdirect.internal.ExtDirectJsonRequestProcessorThread$1.call(ExtDirectJsonRequestProcessorThread.java:62)
        at org.sonatype.nexus.extdirect.internal.ExtDirectJsonRequestProcessorThread$1.call(ExtDirectJsonRequestProcessorThread.java:53)
        at com.google.inject.servlet.ServletScopes.lambda$wrap$0(ServletScopes.java:431)
        at org.sonatype.nexus.extdirect.internal.ExtDirectJsonRequestProcessorThread.processRequest(ExtDirectJsonRequestProcessorThread.java:76)
        at com.softwarementors.extjs.djn.router.processor.standard.json.DefaultJsonRequestProcessorThread.call(DefaultJsonRequestProcessorThread.java:56)
        at com.softwarementors.extjs.djn.router.processor.standard.json.DefaultJsonRequestProcessorThread.call(DefaultJsonRequestProcessorThread.java:30)
        at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
        at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1136)
        at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635)
        at java.base/java.lang.Thread.run(Thread.java:840)
Caused by: java.net.SocketTimeoutException: Connect timed out
        at java.base/sun.nio.ch.NioSocketImpl.timedFinishConnect(NioSocketImpl.java:551)
        at java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:602)
        at java.base/java.net.SocksSocketImpl.connect(SocksSocketImpl.java:327)
        at java.base/java.net.Socket.connect(Socket.java:633)
        at org.apache.http.conn.ssl.SSLConnectionSocketFactory.connectSocket(SSLConnectionSocketFactory.java:368)
        at org.sonatype.nexus.internal.httpclient.NexusSSLConnectionSocketFactory.connectSocket(NexusSSLConnectionSocketFactory.java:89)
        at org.apache.http.impl.conn.DefaultHttpClientConnectionOperator.connect(DefaultHttpClientConnectionOperator.java:142)
        ... 45 common frames omitted
```

这里有两个问题

1. java.util.prefs 文件锁异常
2. SonatypeOutreach 网络连接超时

**文件锁异常**

```sh
# 查看nexus-data存储卷的地址
docker inspect nexus-data | grep Mountpoint

# 执行以下内容，修改nexus-data存储卷的权限
# =====================
# 获取宿主机上的实际路径
NEXUS_DATA_PATH=$(docker inspect nexus-data | jq -r '.[0].Mountpoint')

# 修改目录权限
chown -R 200:200 $NEXUS_DATA_PATH
chmod -R 755 $NEXUS_DATA_PATH
# =====================

# 这里我选择删除容器后重新运行
docker rm -f nexus

# -u 200:200 指定nexus用户身份
# 重点是加上 -Djava.util.prefs.userRoot=/nexus-data/javaprefs 参数
docker run -itd \
    --name=nexus \
    -p 8081:8081 \
    -p 5000:5000 \
    --privileged=true \
    --restart=always \
    --ulimit nofile=655350 \
    --ulimit memlock=-1 \
    --memory=16G \
    --memory-swap=-1 \
    --cpuset-cpus='1-7' \
    -e INSTALL4J_ADD_VM_PARAMS="-Xms4g -Xmx4g -XX:MaxDirectMemorySize=8g -Djava.util.prefs.userRoot=/nexus-data/javaprefs" \
    -v nexus-localtime:/etc/localtime \
    -v nexus-data:/nexus-data \
    -u 200:200 \
    sonatype/nexus3:latest
```



**超时**

```sh
# 进入容器内部，查看网络是否通畅
docker exec -it nexus /bin/bash

curl -I  www.baidu.com
# 打印结果如下
# HTTP/1.1 200 OK
# Accept-Ranges: bytes
# Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
# Connection: keep-alive
# Content-Length: 277
# Content-Type: text/html
# Date: Sat, 31 May 2025 15:09:15 GMT
# Etag: "575e1f59-115"
# Last-Modified: Mon, 13 Jun 2016 02:50:01 GMT
# Pragma: no-cache
# Server: bfe/1.0.8.18
```

这里网络并没有问题，随后根据[此博客](https://blog.csdn.net/u014331288/article/details/107891984)的操作解决了该问题

涉及到登录的地方可以先看后面的内容，查看`admin`的登陆密码

![Pasted image 20250601013923.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081425969.png)

这里为了验证效果，先清空容器日志重启一下容器

```sh
# 找到日志文件路径
LOG_PATH=$(docker inspect --format='{{.LogPath}}' nexus)

# 清空文件（需sudo）
sudo truncate -s 0 $LOG_PATH

# 查看文件内容行数
wc -l $LOG_PATH

# 重启容器
docker restart nexus
```

## 查看初始密码

```bash
docker exec -it nexus cat /nexus-data/admin.password
```
复制输出的密码，首次登录时使用。

---
## 配置Maven仓库

### 新建仓库（可选）
![Pasted image 20250531234731.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081425795.png)
![Pasted image 20250531234736.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081426471.png)

### 配置Maven
![Pasted image 20250531234832.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081427863.png)
找到maven的配置文件
新增如下内容
![Pasted image 20250531235542.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081427444.png)

```xml
<server><!-- 发布版本-->
  <id>nexus</id> <!-- 随意填写，但项目中repository的id得与这里一致-->
  <username>admin</username> <!-- 账户-->
  <password>admin123</password>  <!-- 密码-->
</server>
<server>
  <id>nexus-releases</id> 
  <username>admin</username>
  <password>admin123</password>
</server>
<server>
  <id>nexus-snapshots</id>
  <username>admin</username>
  <password>admin123</password>
</server>
```

新增如下内容
![Pasted image 20250531235722.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081427288.png)

```xml
<mirror>
  <id>nexus</id>
  <!--表示访问哪些工厂时需要使用镜像-->
  <mirrorOf>*</mirrorOf>
  <url>http://116.148.120.214:8081/repository/maven-public/</url>
</mirror>
```
url在此处cv
![Pasted image 20250531235704.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081427487.png)
这里最好还是加上一个阿里云的镜像

```xml
 <!-- 阿里云镜像-->
<mirror>
  <id>alimaven</id>
  <name>aliyun maven</name>
  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
  <mirrorOf>central</mirrorOf>        
</mirror>
<mirror>  
 <id>alimaven</id>  
 <mirrorOf>central</mirrorOf>  
 <name>aliyun maven</name>  
 <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>  
</mirror> 
```

### 上传某个项目的Jar包

在项目中如下配置即可
![Pasted image 20250601000117.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081427406.png)

```xml
<!-- 本地nexus私有库-->
<distributionManagement>
    <snapshotRepository>
        <id>nexus-snapshots</id>
        <name>nexus-snapshots</name>
        <url>http://116.148.120.214:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
    <repository>
        <id>nexus-releases</id>
        <name>nexus-releases</name>
        <url>http://116.148.120.214:8081/repository/maven-releases/</url>
    </repository>
</distributionManagement>
```

注意这里的id(nexus-snapshots、nexus-releases)要和之前maven/conf/setting.xml里配置的id一样

idea点击maven的`deploy`，如果遇到401错误，请查看[此博客](https://blog.csdn.net/qq_43302195/article/details/128684987)

![Pasted image 20250601002008.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081427701.png)

然后就可以在nexus上看到当前项目的jar包了
![Pasted image 20250601002245.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081427251.png)

## 上传Jar包

### 单个上传
> 除了上传项目的jar包外，如果希望连其他常用的jar包也进行上传，根据如下步骤可以实现

![Pasted image 20250601003059.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081427911.png)
![Pasted image 20250601003146.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081427080.png)

### 批量上传

> 这个脚本有一个问题，批量设置了groupId为自定义的值，而我其实更希望和maven仓库的groupId保持一致

```sh
#!/bin/bash

# TODO 修改成你的Nexus地址和仓库名
NEXUS_URL="http://localhost:8081"
# TODO 填写你的用户名和密码
REPO_NAME="maven-releases"
USERNAME="admin"
PASSWORD="your_password_or_api_key"

# TODO 修改成需上传的目录
UPLOAD_DIR="/path/to/maven-packages"

for file in "$UPLOAD_DIR"/*.jar; do
    filename=$(basename "$file")
    artifactId=$(echo "$filename" | sed -E 's/-([0-9]+\.)*[0-9]+\.jar$//')
    version=$(echo "$filename" | sed -E 's/.*-(([0-9]+\.)*[0-9]+)\.jar$/\1/')
    # TODO 填写你的groupId
    groupId="com.example"

    echo "Uploading $filename -> $groupId:$artifactId:$version"

    curl -u "$USERNAME:$PASSWORD" -X POST "$NEXUS_URL/service/rest/v1/components?repository=$REPO_NAME" \
      -H "Content-Type: multipart/form-data" \
      -F "maven2.groupId=$groupId" \
      -F "maven2.artifactId=$artifactId" \
      -F "maven2.version=$version" \
      -F "maven2.asset1.extension=jar" \
      -F "maven2.asset1.file=@$file"

    echo -e "\n"
done
```

使用脚本批量上传本地jar包，还要保证GroupId、ArtifactId、Version都和官方一样实现起来有些麻烦，不过可以使用proxy仓库的方式，可以完美解决这个问题
以下提供两个脚本可以进行尝试
> 先不上传，只是打印信息


```sh
#!/bin/bash

NEXUS_URL="http://116.148.120.214:8081"
REPO_NAME="maven-releases"
USERNAME="admin"
PASSWORD="your_password_or_api_key"
UPLOAD_DIR="/path/to/jars"

for jar in "$UPLOAD_DIR"/*.jar; do
    filename=$(basename "$jar")
    tmp_dir=$(mktemp -d)
    unzip "$jar" -d "$tmp_dir" >/dev/null 2>&1

    # 优先从 pom.xml 提取坐标
    pom_path=$(find "$tmp_dir" -name "pom.xml" | head -n1)
    if [ -f "$pom_path" ]; then
        groupId=$(grep "<groupId>" "$pom_path" | head -n1 | sed -e 's/.*<groupId>\(.*\)<\/groupId>.*/\1/')
        artifactId=$(grep "<artifactId>" "$pom_path" | head -n1 | sed -e 's/.*<artifactId>\(.*\)<\/artifactId>.*/\1/')
        version=$(grep "<version>" "$pom_path" | head -n1 | sed -e 's/.*<version>\(.*\)<\/version>.*/\1/')
    else
        # 尝试从类结构推测
        class_file=$(find "$tmp_dir" -name "*.class" | head -n1)
        if [ -f "$class_file" ]; then
            class_name=$(echo "$class_file" | sed "s|$tmp_dir/||; s|.class$||; s|/|.|g")
            groupId=$(echo "$class_name" | awk -F'.' '{for(i=1;i<NF-1;i++) printf "%s.", $i; print $(NF-1)}')
            artifactId="unknown"
            version="unknown"
        else
            echo "无法提取元数据: $filename"
            continue
        fi
    fi

    # 输出元数据信息（不执行上传）
    echo "--------------------------------------------------"
    echo "文件: $filename"
    echo "  GroupId     : $groupId"
    echo "  ArtifactId  : $artifactId"
    echo "  Version     : $version"

    rm -rf "$tmp_dir"
done
```

> 上传jar包

```sh
#!/bin/bash

NEXUS_URL="http://localhost:8081"
REPO_NAME="maven-releases"
USERNAME="admin"
PASSWORD="your_password_or_api_key"
UPLOAD_DIR="/path/to/jars"

for jar in "$UPLOAD_DIR"/*.jar; do
    filename=$(basename "$jar")
    tmp_dir=$(mktemp -d)
    unzip "$jar" -d "$tmp_dir" >/dev/null 2>&1

    # 优先从 pom.xml 提取坐标
    pom_path=$(find "$tmp_dir" -name "pom.xml" | head -n1)
    if [ -f "$pom_path" ]; then
        groupId=$(grep "<groupId>" "$pom_path" | head -n1 | sed -e 's/.*<groupId>\(.*\)<\/groupId>.*/\1/')
        artifactId=$(grep "<artifactId>" "$pom_path" | head -n1 | sed -e 's/.*<artifactId>\(.*\)<\/artifactId>.*/\1/')
        version=$(grep "<version>" "$pom_path" | head -n1 | sed -e 's/.*<version>\(.*\)<\/version>.*/\1/')
    else
        # 尝试从类结构推测
        class_file=$(find "$tmp_dir" -name "*.class" | head -n1)
        if [ -f "$class_file" ]; then
            class_name=$(echo "$class_file" | sed "s|$tmp_dir/||; s|.class$||; s|/|.|g")
            groupId=$(echo "$class_name" | awk -F'.' '{for(i=1;i<NF-1;i++) printf "%s.", $i; print $(NF-1)}')
            artifactId="unknown"
            version="unknown"
        else
            echo "无法提取元数据: $filename"
            continue
        fi
    fi

    echo "Uploading $filename -> $groupId:$artifactId:$version"

    curl -u "$USERNAME:$PASSWORD" -X POST "$NEXUS_URL/service/rest/v1/components?repository=$REPO_NAME" \
      -H "Content-Type: multipart/form-data" \
      -F "maven2.groupId=$groupId" \
      -F "maven2.artifactId=$artifactId" \
      -F "maven2.version=$version" \
      -F "maven2.asset1.extension=jar" \
      -F "maven2.asset1.file=@$jar"

    rm -rf "$tmp_dir"
done
```

以下将从其他maven仓库将jar包`下载`到本地nexus服务
### 新建Proxy仓库
> 其实可以完全不上传本地 JAR 包 ，而是直接通过 Nexus 的 代理仓库（Proxy Repository） 功能，将 Maven 官方仓库（如 Maven Central）或阿里云镜像（如阿里云 Maven 镜像）代理到 Nexus 服务器中。这样，当项目请求依赖时，Nexus 会自动从远程仓库下载并缓存，无需手动上传 JAR 包

url输入`https://maven.aliyun.com/repository/public/`
![Pasted image 20250601005445.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081427359.png)
需要在public上配置刚刚新建的仓库
![Pasted image 20250601005530.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081427684.png)
这里要注意下仓库顺序。maven查找依赖时会依次遍历仓库组中的仓库，我这里直接移到最上面
![Pasted image 20250601012726.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081427141.png)
然后注释掉其他的仓库配置，在idea里clean之后install即可下载jar包到nexus
![Pasted image 20250601012806.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081427774.png)
![Pasted image 20250601012832.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081427908.png)
同样的方法也可以配置maven官方仓库
