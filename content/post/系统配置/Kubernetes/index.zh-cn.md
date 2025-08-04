---
title: K8S环境配置
description: K8S环境配置
date: 2025-06-07
slug: K8S环境配置
image: 202412242125601.png
categories:
    - 系统配置
---

# K8S环境配置

>接下来在ubuntu上安装`20.10.7`版本的Docker和`1.23.6`版本的`kubernetes`
>可参考[安装Docker文档](https://blog.csdn.net/qq_37843943/article/details/107245223)

以下是基于国内镜像源安装指定版本（例如 `20.10.7`）的详细步骤：

---
## Docker安装
### **1. 卸载旧版本的 Docker**
如果系统中已经安装了旧版本的 Docker，请先卸载：
```bash
sudo apt remove docker docker-engine docker.io containerd runc
```

---

### **2. 更新包索引并安装必要的依赖**
确保系统的包索引是最新的，并安装一些必要的工具：
```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
```

---

### **3. 使用国内镜像源**

#### **方法 1：使用阿里云镜像源**
阿里云提供了 Docker 的国内镜像源，以下是具体步骤：

1. 添加阿里云的 GPG 密钥：
   ```bash
   curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

2. 添加阿里云的 Docker APT 源：
   ```bash
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

#### **方法 2：使用清华大学开源软件镜像站**
清华大学也提供了 Docker 的国内镜像源，以下是具体步骤：

1. 添加清华大学的 GPG 密钥：
   ```bash
   curl -fsSL https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

2. 添加清华大学的 Docker APT 源：
   ```bash
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

---

### **4. 更新包索引**
更新 APT 包索引以包含国内镜像源中的内容：
```bash
sudo apt update
```

---

### **5. 查看可用的 Docker 版本**
在安装之前，可以查看国内镜像源中有哪些版本可供安装：
```bash
apt-cache madison docker-ce
```
输出示例：
```
docker-ce | 5:20.10.7~3-0~ubuntu-focal | https://mirrors.aliyun.com/docker-ce/linux/ubuntu focal/stable amd64 Packages
docker-ce | 5:20.10.6~3-0~ubuntu-focal | https://mirrors.aliyun.com/docker-ce/linux/ubuntu focal/stable amd64 Packages
...
```
从输出中找到你想要安装的版本号（例如 `5:20.10.7~3-0~ubuntu-focal`）。

---

### **6. 安装指定版本的 Docker**
使用以下命令安装指定版本的 Docker（将 `<VERSION_STRING>` 替换为实际的版本号）：
```bash
sudo apt install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
```

~~例如，安装 `20.10.7` 版本：~~
```bash
sudo apt install docker-ce=5:20.10.7~3-0~ubuntu-focal docker-ce-cli=5:20.10.7~3-0~ubuntu-focal containerd.io
```
这里没有找到对应版本的Docker
输入以下命令
```shell
wget https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/dists/focal/pool/stable/amd64/docker-ce_20.10.7~3-0~ubuntu-focal_amd64.deb
wget https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/dists/focal/pool/stable/amd64/docker-ce-cli_20.10.7~3-0~ubuntu-focal_amd64.deb
wget https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/dists/focal/pool/stable/amd64/containerd.io_1.4.6-1_amd64.deb

sudo chmod 755 containerd.io_1.4.6-1_amd64.deb docker-ce_20.10.7~3-0~ubuntu-focal_amd64.deb docker-ce-cli_20.10.7~3-0~ubuntu-focal_amd64.deb

sudo dpkg -i docker-ce_20.10.7~3-0~ubuntu-focal_amd64.deb docker-ce-cli_20.10.7~3-0~ubuntu-focal_amd64.deb containerd.io_1.4.6-1_amd64.deb
```

---

### **7. 验证 Docker 是否安装成功**
安装完成后，可以通过以下命令验证 Docker 是否正常工作：
```bash
sudo docker --version
```
输出示例：
```
Docker version 20.10.7, build f0df350
```

运行一个测试容器：
```bash
sudo docker run hello-world
```
如果看到类似以下输出，则说明 Docker 安装成功：
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

---

### **8. （可选）启用非 root 用户运行 Docker**
默认情况下，Docker 需要 `sudo` 权限才能运行。如果你希望普通用户也可以运行 Docker，可以将用户添加到 `docker` 组：
```bash
sudo usermod -aG docker $USER
```
然后重新登录或重启系统以使更改生效。

---

### **9. 锁定 Docker 版本（可选）**
为了避免意外升级到其他版本，可以锁定 Docker 的版本：
```bash
sudo apt-mark hold docker-ce docker-ce-cli containerd.io
```

---

## K8S安装
好的！以下是安装 **Kubernetes 1.23.6 单节点集群** 的详细步骤，同时使用国内镜像源（如阿里云、清华大学开源软件镜像站）来加速下载和安装过程。

---

### **1. 环境准备**

#### **1.1 确保 Docker 已正确安装**
运行以下命令验证 Docker 是否正常工作：
```bash
sudo docker --version
```
如果输出类似于 `Docker version xx.xx.x`，说明 Docker 已正确安装。

#### **1.2 关闭 Swap 分区**
Kubernetes 不支持开启 Swap 分区。运行以下命令关闭 Swap：
```bash
sudo swapoff -a
```

为了确保系统重启后仍然生效，编辑 `/etc/fstab` 文件，注释掉与 Swap 相关的行：
```bash
sudo nano /etc/fstab
```
找到类似以下内容并注释掉（在行首添加 `#`）：
```
#/swapfile none swap sw 0 0
```

#### **1.3 启用内核模块**
Kubernetes 需要启用一些内核模块，运行以下命令加载必要的模块：
```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

将这些模块设置为开机自动加载：
```bash
echo "overlay" | sudo tee /etc/modules-load.d/overlay.conf
echo "br_netfilter" | sudo tee /etc/modules-load.d/br_netfilter.conf
```

#### **1.4 配置网络参数**
运行以下命令配置 Kubernetes 所需的网络参数：
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

---

### **2. 安装 Kubernetes 组件**

#### **2.1 添加 Kubernetes 国内镜像源**
由于 Kubernetes 官方仓库在国内访问较慢，我们使用国内镜像源（如阿里云）。

创建 Kubernetes 的 APT 源文件：
```bash
sudo nano /etc/apt/sources.list.d/kubernetes.list
```

添加以下内容（以阿里云镜像源为例）：
```
deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
```

> **注意**：即使你使用的是 Ubuntu 24.04（Noble），仍然可以使用 `xenial` 分支，因为 Kubernetes 的仓库并未为每个 Ubuntu 版本单独维护分支。

保存并退出编辑器。

#### **2.2 添加 GPG 密钥**
运行以下命令添加 Kubernetes 的 GPG 密钥：
```bash
curl -fsSL https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/kubernetes-archive-keyring.gpg
```

#### **2.3 更新包索引**
更新 APT 包索引以包含 Kubernetes 镜像源中的内容：
```bash
sudo apt update
```

#### **2.4 安装指定版本的 Kubernetes 组件**
安装 Kubernetes 1.23.6 的核心组件（`kubelet`、`kubeadm` 和 `kubectl`）：
```bash
sudo apt install -y kubelet=1.23.6-00 kubeadm=1.23.6-00 kubectl=1.23.6-00
```

锁定版本以防止意外升级：
```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

---

### **3. 初始化单节点 Kubernetes 集群**

#### **3.1 配置容器运行时（Docker）**
Kubernetes 1.23 仍然支持 Docker 作为容器运行时，但需要通过 `cri-dockerd` 来适配 CRI 接口。

##### **步骤 3.1.1：安装 `cri-dockerd`**
下载并安装 `cri-dockerd`：
```bash
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.4/cri-dockerd_0.3.4.3-0.ubuntu-focal_amd64.deb
sudo dpkg -i cri-dockerd_0.3.4.3-0.ubuntu-focal_amd64.deb
```

如果出现依赖问题，运行以下命令修复：
```bash
sudo apt install -f
```

##### **步骤 3.1.2：配置 `cri-dockerd`**
编辑 `cri-dockerd` 的服务文件：
```bash
sudo systemctl enable cri-docker.service
sudo systemctl start cri-docker.service
```

#### **3.2 初始化 Kubernetes 集群**
运行以下命令初始化单节点 Kubernetes 集群：
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket=unix:///var/run/cri-dockerd.sock
```

> **说明**：
> - `--pod-network-cidr=10.244.0.0/16` 是为 Flannel 网络插件预留的 CIDR。
> - `--cri-socket=unix:///var/run/cri-dockerd.sock` 指定使用 `cri-dockerd` 作为容器运行时。

初始化完成后，你会看到类似以下的输出：
```
Your Kubernetes control-plane has initialized successfully!
...
```
>实际操作下来我这里卡住了
>可能是由于网络原因拉取镜像超时了，那先配置下Docker国内镜像吧
```shell
# 将当前用户添加到docker组
sudo usermod -aG docker $USER

sudo mkdir -p /etc/docker 

sudo tee /etc/docker/daemon.json <<EOF
{
    "registry-mirrors": [
        "https://docker.1ms.run",
        "https://docker.xuanyuan.me"
    ]
}
EOF

sudo systemctl daemon-reload 
sudo systemctl restart docker

# 将当前用户添加到 docker 组
sudo usermod -aG docker $USER
newgrp docker

# 重试init，还是没反应，拉取镜像
docker pull registry.aliyuncs.com/google_containers/kube-apiserver:v1.23.6
docker pull registry.aliyuncs.com/google_containers/kube-controller-manager:v1.23.6
docker pull registry.aliyuncs.com/google_containers/kube-scheduler:v1.23.6
docker pull registry.aliyuncs.com/google_containers/kube-proxy:v1.23.6
docker pull registry.aliyuncs.com/google_containers/pause:3.6
docker pull registry.aliyuncs.com/google_containers/etcd:3.5.1-0
docker pull registry.aliyuncs.com/google_containers/coredns:v1.8.6

docker tag registry.aliyuncs.com/google_containers/kube-apiserver:v1.23.6 k8s.gcr.io/kube-apiserver:v1.23.6
docker tag registry.aliyuncs.com/google_containers/kube-controller-manager:v1.23.6 k8s.gcr.io/kube-controller-manager:v1.23.6
docker tag registry.aliyuncs.com/google_containers/kube-scheduler:v1.23.6 k8s.gcr.io/kube-scheduler:v1.23.6
docker tag registry.aliyuncs.com/google_containers/kube-proxy:v1.23.6 k8s.gcr.io/kube-proxy:v1.23.6
docker tag registry.aliyuncs.com/google_containers/pause:3.6 k8s.gcr.io/pause:3.6
docker tag registry.aliyuncs.com/google_containers/etcd:3.5.1-0 k8s.gcr.io/etcd:3.5.1-0
docker tag registry.aliyuncs.com/google_containers/coredns:v1.8.6 k8s.gcr.io/coredns/coredns:v1.8.6

# 生成一个配置文件用来初始化
kubeadm config print init-defaults > kubeadm-config.yaml
```

>修改配置文件内容
```yaml
nodeRegistration:
  criSocket: /var/run/dockershim.sock # 修改为 unix:///var/run/cri-dockerd.sock
  imagePullPolicy: IfNotPresent # 修改为 Never，强制使用本地镜像
  name: node
  taints: null

# 修改后的配置
nodeRegistration:
  criSocket: unix:///var/run/cri-dockerd.sock
  imagePullPolicy: Never
  name: node
  taints: null

# 这里指定的是官网，我们需要更新为国内镜像
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: 1.23.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12

# 修改后的配置
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: 1.23.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16 # 确保添加 podSubnet

```
>重新init，初始化命令如下

```shell
sudo kubeadm init --config kubeadm-config.yaml --ignore-preflight-errors=ImagePull --v=5

# 这次很快就有反应了,但是依然没有成功，也没有报错，有两个警告如下
# [WARNING Hostname]: hostname "node" could not be reached
# [WARNING Hostname]: hostname "node": lookup node on 127.0.0.53:53: server misbehaving

# 设置主机名
sudo hostnamectl set-hostname k8s-master

# 更新/etc/hosts文件
# 这里最好慎重一点，如果和之前的hostname不一致，可能会遇到一些问题，比如浏览器无法打开，被之前的host占用等
echo "127.0.0.1   localhost" | sudo tee /etc/hosts
echo "127.0.1.1   k8s-master" | sudo tee -a /etc/hosts

hostname
# 应输出：k8s-master

hostname -f
# 应输出：k8s-master

ping -c 1 k8s-master
# 应显示来自 127.0.1.1 的响应

# 检查 kubeadm-config.yaml 中的节点名称
# nodeRegistration.name与实际主机名称保持一致
nodeRegistration:
  criSocket: unix:///var/run/cri-dockerd.sock
  imagePullPolicy: Never
  name: node
  taints: null
# 修改为
nodeRegistration:
  criSocket: unix:///var/run/cri-dockerd.sock
  imagePullPolicy: Never
  name: k8s-master
  taints: null

# 没有启动成功可能是因为制平面组件（如 kube-apiserver）未能启动
# 以下是排查步骤
sudo journalctl -u kubelet --since "5 minutes ago"

# 重新init
sudo kubeadm init --config kubeadm-config.yaml --ignore-preflight-errors=ImagePull --v=5

# 这次直接报错，我们先删除相关的文件目录
sudo kubeadm reset --force  # 确保彻底重置
sudo rm -rf /etc/kubernetes/*
sudo rm -rf /var/lib/kubelet/*
sudo rm -rf /var/lib/etcd/*

# 重新init这次直接报错
# couldn't initialize a Kubernetes cluster
# k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/init.runWaitControlPlanePhase

# 查看报错日志
sudo journalctl -u kubelet --since "10 minutes ago" | grep -i error

# kubelet.go:2461] "Error getting node" err="node \"k8s-master\" not found"

# 检查hostname和/etc/hosts都没有问题
rm -f /etc/kubernetes/manifests/*.yaml

# 重试上面的reset命令和rm操作，然后重新初始化
sudo kubeadm init --config kubeadm-config.yaml --ignore-preflight-errors=ImagePull --v=5

# 服了，又失败，使用以下命令init
```shell
sudo kubeadm init \
  --image-repository=registry.aliyuncs.com/google_containers \
  --pod-network-cidr=10.244.0.0/16 \
  --ignore-preflight-errors=Swap 
```

#### **3.3 配置普通用户访问权限**
为了让普通用户能够管理 Kubernetes 集群，运行以下命令：
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### **3.4 安装网络插件（Flannel）**
运行以下命令安装 Flannel 网络插件：
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

如果无法访问 GitHub，可以使用国内镜像地址（如阿里云）：
```bash
kubectl apply -f https://mirrors.aliyun.com/kubernetes/flannel/kube-flannel.yml
```

---

### **4. 验证 Kubernetes 集群状态**

#### **4.1 查看节点状态**
运行以下命令查看节点状态：
```bash
kubectl get nodes
```

输出应类似于：
```
NAME         STATUS   ROLES                  AGE     VERSION
tong-ubuntu  Ready    control-plane,master  5m      v1.23.6
```

#### **4.2 查看 Pod 状态**
运行以下命令查看所有 Pod 的状态：
```bash
kubectl get pods --all-namespaces
```

输出应显示所有 Pod 处于 `Running` 状态。

---

```shell
kubectl get nodes
# NAME         STATUS     ROLES                  AGE     VERSION
# k8s-master   NotReady   control-plane,master   6m42s   v1.23.6

kubectl get pods -n kube-system -o wide | grep calico
# calico-kube-controllers-6c5578449-rx8pd   0/1     ContainerCreating   0          77s     <none>           k8s-master   <none>           <none>
# calico-node-b762p                         0/1     Running             0          77s     10.0.0.2         k8s-master   <none>           <none>

```

### 5. 安装busybox

> 启动一个busybox pod试试

新建`pod.yaml`文件

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dns-test
  namespace: default
spec:
  containers:
  - name: dns-test
    image: busybox:1.28.4
    args: ["sh", "-c", "sleep 3600"]
  tolerations:
  - key: "node-role.kubernetes.io/master"
    operator: "Exists"
    effect: "NoSchedule"
```

> 运行pod

```sh
kubectl apply -f pod.yaml

# 查看pod
k get po

# 结果如下
# NAME       READY   STATUS    RESTARTS   AGE
# dns-test   1/1     Running   0          7s
```

