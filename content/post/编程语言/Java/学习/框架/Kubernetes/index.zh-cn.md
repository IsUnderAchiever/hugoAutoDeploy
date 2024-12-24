---
title: Kubernetes
description: Kubernetes
date: 2024-12-24
slug: Kubernetes
image: 202412242125601.png
categories:
    - Kubernetes
---
# Kubernetes

## 核心概念

### 什么是Kubernetes

**Kubernetes** 是一个开源的，用于管理云平台中多个主机上的容器化的应用，Kubernetes 的目标是让部署容器化的应用简单并且高效（powerful），Kubernetes 提供了应用部署，规划，更新，维护的一种机制。

**Kubernetes** 这个名字源于希腊语，意为“舵手”或“飞行员”。k8s 这个缩写是因为 k 和 s 之间有八个字符的关系。 Google 在 2014 年开源了 Kubernetes 项目。 Kubernetes 建立在 [Google 大规模运行生产工作负载十几年经验](https://research.google/pubs/pub43438)的基础上， 结合了社区中最优秀的想法和实践。

### 为什么需要 Kubernetes

#### 应用部署的三大阶段

1. 传统部署
   - 程序员/运维工程师手动操作部署应用，直接将应用部署在目标机器上，由于资源不隔离，容易出现资源争抢、依赖冲突等各方面问题。
2. 虚拟化部署
   - 利用 OpenStask / VMware 等虚拟化技术，将一台目标机器虚拟化为多个虚拟机器，按照需求将应用部署到不同的虚拟机中，对虚拟机进行动态的水平扩容等管理操作。
   - 相对传统部署自动化、资源隔离的能力提升了，带来的问题是虚拟化的逻辑过重，导致效率不高，且耗费资源较多。
3. 容器化部署
   - 可以理解为轻量级的虚拟化，完美弥补虚拟化技术过重的问题，且由于直接共享主机硬件资源，只是通过系统提供的命名空间等技术实现资源隔离，损耗更小，且效率更高。

#### k8s 的特点

1. 自我修复
2. 弹性伸缩
3. 自动部署和回滚
4. 服务发现和负载均衡
5. 机密和配置管理
6. 存储编排
7. 批处理

### 企业级容器调度平台

1. Apache Mesos
   1. 基本概念
      - Mesos 是一个分布式调度系统内核，早于 Docker 产生，Mesos 作为资源管理器，从 DC/OS (数据中心操作系统)的角度提供资源视图。主/从结构工作模式，主节点分配任务，并用从节点上的 Executor 负责执行，通过 Zookeeper 给主节点提供服务注册、服务发现功能。通过 Framework Marathon 提供容器调度的能力。
   2. 优势
      - 经过时间的检验，作为资源管理器的 Apache Mesos 在容器之前就已经出现很久了，支持运行容器化化和非容器化的工作负载。可以支持应用程序的健康检查，开放的架构。支持多个框架和多个调度器，通过不同的 Framework 可以运行 Hadoop/Spark/MPI等多种不同的任务。
      - 支持超大型规模的节点管理，模拟测试支持超过 5w+ 节点，在大规模上拥有较大优势。
2. Docker Swarm
   1. 基本概念
      - Docker Swarm 是一个由 Docker 开发的调度框架。由 Docker 自身开发的好处之一就是标准 Docker API 的使用，Swarm 由多个代理（Agent）组成，把这些代理称之为节点（Node）。这些节点就是主机，这些主机在启动 Docker Daemon 的时候就会打开相应的端口，以此支持 Docker 远程 API。这些机器会根据 Swarm 调度器分配给它们的任务，拉取和运行不同的镜像。
   2. 优势
      - 从 Docker1.12 版本开始，Swarm 随 Docker 一起默认安装发布。由于随 Docker 引擎一起发布，无需额外安装，配置简单。支持服务注册、服务发现，内置 Overlay Network 以及 Load Balancer。与 Docker CLI 非常类似的操作命令，对熟悉 Docker 的人非常容易上手学习。
      - 入门门槛、学习成本较低，使用更便捷，适用于中小型系统。
3. Google Kubernetes
   1. 基本概念
      - Kubernetes 是基于 Google 在过去十五年来大量生产环境中运行工作负载的经验。Kubernetes 的实现参考了 Google 内部的资源调度框架，但并不是 Borg 的内部容器编排系统的开源，而是借鉴 Google 从运行 Borg 获得的经验教训，形成了 Kubernetes 项目。
      - 它使用 Label 和 Pod 的概念来将容器划分为逻辑单元。Pods 是同地协作（co-located）容器的集合，这些容器被共同部署和调度，形成了一个服务，这是 Kubernetes 和其他两个框架的主要区别。相比于基于相似度的容器调度方式（就像 Swarm 和Mesos），这个方法简化了对集群的管理。
   2. 优势
      - 最流行等容器编排解决方案框架，基于 Google 庞大的生态圈及社区产生的产品。通过 Pods 这一抽象的概念，解决 Container 之间的依赖于通信问题。Pods，Services，Deployments 是独立部署的部分，可以通过 Selector 提供更多的灵活性。内置服务注册表和负载平衡。
      - 适用度更广，功能更强大，相较于 Mesos 来说节点规模较小，

### 集群架构与组件

#### 相关组件

##### 控制面板组件（Master）

![k8s1.drawio](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412242119647.png)

**其中标色的地方需要重点注意**

> ###### kube-api-server

API 服务器是 Kubernetes [控制平面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)的组件， 该组件负责公开了 Kubernetes API，负责处理接受请求的工作。 API 服务器是 Kubernetes 控制平面的前端。

Kubernetes API 服务器的主要实现是 [kube-apiserver](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-apiserver/)。 kube-apiserver 设计上考虑了水平扩缩，也就是说，它可通过部署多个实例来进行扩缩。 你可以运行 kube-apiserver 的多个实例，并在这些实例之间平衡流量。

> ###### kube-controller-manager

[kube-controller-manager](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-controller-manager/) 是[控制平面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)的组件， 负责运行[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)进程。

从逻辑上讲， 每个[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)都是一个单独的进程， 但是为了降低复杂性，它们都被编译到同一个可执行文件，并在同一个进程中运行。

这些控制器包括：

- 节点控制器（Node Controller）：负责在节点出现故障时进行通知和响应
- 任务控制器（Job Controller）：监测代表一次性任务的 Job 对象，然后创建 Pods 来运行这些任务直至完成
- 端点分片控制器（EndpointSlice controller）：填充端点分片（EndpointSlice）对象（以提供 Service 和 Pod 之间的链接）。
- 服务账号控制器（ServiceAccount controller）：为新的命名空间创建默认的服务账号（ServiceAccount）。

> ###### cloud-controller-manager

嵌入了特定于云平台的控制逻辑。 云控制器管理器（Cloud Controller Manager）允许你将你的集群连接到云提供商的 API 之上， 并将与该云平台交互的组件同与你的集群交互的组件分离开来。

cloud-controller-manager 仅运行特定于云平台的控制器。 因此如果你在自己的环境中运行 Kubernetes，或者在本地计算机中运行学习环境， 所部署的集群不需要有云控制器管理器。

与 kube-controller-manager 类似，cloud-controller-manager 将若干逻辑上独立的控制回路组合到同一个可执行文件中， 供你以同一进程的方式运行。 你可以对其执行水平扩容（运行不止一个副本）以提升性能或者增强容错能力。

> ###### kube-scheduler

scheduler 负责资源的调度，按照预定的调度策略将 Pod 调度到相应的机器上；

> ###### etcd

一致且高度可用的键值存储，用作 Kubernetes 的所有集群数据的后台数据库。

如果你的 Kubernetes 集群使用 etcd 作为其后台数据库， 请确保你针对这些数据有一份 [备份](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)计划。

你可以在官方[文档](https://etcd.io/docs/)中找到有关 etcd 的深入知识。

早期数据存放在内存，现在已经是持久化存储的了。

##### 节点组件

> ###### kubelet

kubelet 负责维护容器的生命周期，同时也负责 Volume（CVI）和网络（CNI）的管理；

> ###### kube-proxy

kube-proxy 负责为 Service 提供 cluster 内部的服务发现和负载均衡；kube-proxy 负责为 Service 提供 cluster 内部的服务发现和负载均衡；

>###### container runtime

Container runtime 负责镜像管理以及 Pod 和容器的真正运行（CRI）；
Kubernetes 支持许多容器运行环境，例如 [containerd](https://containerd.io/docs/)、 [CRI-O](https://cri-o.io/#what-is-cri-o) 以及 [Kubernetes CRI (容器运行环境接口)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md) 的其他任何实现。

##### 附加组件

> ###### kube-dns

kube-dns 负责为整个集群提供 DNS 服务

>###### Ingress Controller

Ingress Controller 为服务提供外网入口

>###### Prometheus

Prometheus 提供资源监控

>###### Dashboard

Dashboard 提供 GUI

>###### Federation

Federation 提供跨可用区的集群

>###### Fluentd-elasticsearch

Fluentd-elasticsearch 提供集群日志采集、存储与查询

#### 分层架构

1. 生态系统
   - 在接口层之上的庞大容器集群管理调度的生态系统，可以划分为两个范畴：
     - Kubernetes 外部：日志、监控、配置管理、CI、CD、Workflow、FaaS、OTS 应用、ChatOps 等
     - Kubernetes 内部：CRI、CNI、CVI、镜像仓库、Cloud Provider、集群自身的配置和管理等
2. 接口层
   - kubectl 命令行工具、客户端 SDK 以及集群联邦
3. 管理层
   - 系统度量（如基础设施、容器和网络的度量），自动化（如自动扩展、动态 Provision 等）以及策略管理（RBAC、Quota、PSP、NetworkPolicy 等）
4. 应用层
   - 部署（无状态应用、有状态应用、批处理任务、集群应用等）和路由（服务发现、DNS 解析等）
5. 核心层
   - Kubernetes 最核心的功能，对外提供 API 构建高层的应用，对内提供插件式应用执行环境

### 核心概念与专业术语

#### 服务的分类

##### 无状态

> 需要在服务运行环境下存储数据的服务，我们称之为`无状态应用`

代表应用

1. Nginx
2. Apache

优点：对客户端透明，无依赖关系，可以高效实现扩容、迁移

缺点：不能存储数据，需要额外的数据服务支撑

##### 有状态

代表应用

1. MySQL
2. Redis

优点：可以独立存储数据，实现数据管理
缺点：集群环境下需要实现主从、数据同步、备份、水平扩容复杂

#### 资源和对象

Kubernetes 中的所有内容都被抽象为“资源”，如 Pod、Service、Node 等都是资源。“对象”就是“资源”的实例，是持久化的实体。如某个具体的 Pod、某个具体的 Node。Kubernetes 使用这些实体去表示整个集群的状态。

 对象的创建、删除、修改都是通过 “Kubernetes API”，也就是 “Api Server” 组件提供的 API 接口，这些是 RESTful 风格的 Api，与 k8s 的“万物皆对象”理念相符。命令行工具 “kubectl”，实际上也是调用 kubernetes api。

 K8s 中的资源类别有很多种，kubectl 可以通过配置文件来创建这些 “对象”，配置文件更像是描述对象“属性”的文件，配置文件格式可以是 “JSON” 或 “YAML”，常用 “YAML”。

##### `元数据型`

1. Horizontal Pod Autoscaler（HPA）
   - Pod 自动扩容：可以根据 CPU 使用率或自定义指标（metrics）自动对 Pod 进行扩/缩容。
     - 控制管理器每隔30s（可以通过–horizontal-pod-autoscaler-sync-period修改）查询metrics的资源使用情况
     - 支持三种metrics类型
       - 预定义metrics（比如Pod的CPU）以利用率的方式计算
       - 自定义的Pod metrics，以原始值（raw value）的方式计算
       - 自定义的object metrics
     - 支持两种metrics查询方式：Heapster和自定义的REST API
     - 支持多metrics
2. PodTemplate
   - Pod Template 是关于 Pod 的定义，但是被包含在其他的 Kubernetes 对象中（例如 Deployment、StatefulSet、DaemonSet 等控制器）。控制器通过 Pod Template 信息来创建 Pod。
3. LimitRange
   - 可以对集群内 Request 和 Limits 的配置做一个全局的统一的限制，相当于批量设置了某一个范围内（某个命名空间）的 Pod 的资源使用限制。

##### `集群级`

1. Namespace

   - Kubernetes 支持多个虚拟集群，它们底层依赖于同一个物理集群，这些虚拟集群被称为命名空间。

     作用是用于实现多团队/环境的资源隔离。

     命名空间 namespace 是 k8s 集群级别的资源，可以给不同的用户、租户、环境或项目创建对应的命名空间。

     默认 namespace：

     - kube-system 主要用于运行系统级资源，存放 k8s 自身的组件
     - kube-public 此命名空间是自动创建的，并且可供所有用户（包括未经过身份验证的用户）读取。此命名空间主要用于集群使用，关联的一些资源在集群中是可见的并且可以公开读取。此命名空间的公共方面知识一个约定，但不是非要这么要求。
     - default 未指定名称空间的资源就是 default，即你在创建pod 时如果没有指定 namespace，则会默认使用 default

2. Node

   - 不像其他的资源（如 Pod 和 Namespace），Node 本质上不是Kubernetes 来创建的，Kubernetes 只是管理 Node 上的资源。虽然可以通过 Manifest 创建一个Node对象（如下 json 所示），但 Kubernetes 也只是去检查是否真的是有这么一个 Node，如果检查失败，也不会往上调度 Pod。

3. ClusterRole

   - ClusterRole 是一组权限的集合，但与 Role 不同的是，ClusterRole 可以在包括所有 Namespace 和集群级别的资源或非资源类型进行鉴权。

4. ClusterRoleBinding

   - ClusterRoleBinding：将 Subject 绑定到 ClusterRole，ClusterRoleBinding 将使规则在所有命名空间中生效。

##### `命名空间级`

###### 工作负载型

**Pod**
Pod（容器组）是 Kubernetes 中最小的可部署单元。一个 Pod（容器组）包含了一个应用程序容器（某些情况下是多个容器）、存储资源、一个唯一的网络 IP 地址、以及一些确定容器该如何运行的选项。Pod 容器组代表了 Kubernetes 中一个独立的应用程序运行实例，该实例可能由单个容器或者几个紧耦合在一起的容器组成。



Docker 是 Kubernetes Pod 中使用最广泛的容器引擎；Kubernetes Pod 同时也支持其他类型的容器引擎。


Kubernetes 集群中的 Pod 存在如下两种使用途径：

- 一个 Pod 中只运行一个容器。"one-container-per-pod" 是 Kubernetes 中最常见的使用方式。此时，您可以认为 Pod 容器组是该容器的 wrapper，Kubernetes 通过 Pod 管理容器，而不是直接管理容器。
- 一个 Pod 中运行多个需要互相协作的容器。您可以将多个紧密耦合、共享资源且始终在一起运行的容器编排在同一个 Pod 中

> 副本（replicas）

先引入“副本”的概念——一个 Pod 可以被复制成多份，每一份可被称之为一个“副本”，这些“副本”除了一些描述性的信息（Pod 的名字、uid 等）不一样以外，其它信息都是一样的，譬如 Pod 内部的容器、容器数量、容器里面运行的应用等的这些信息都是一样的，这些副本提供同样的功能。

Pod 的***“控制器”\***通常包含一个名为 “replicas” 的属性。“replicas”属性则指定了特定 Pod 的副本的数量，当当前集群中该 Pod 的数量与该属性指定的值不一致时，k8s 会采取一些策略去使得当前状态满足配置的要求。

> 控制器

当 Pod 被创建出来，Pod 会被调度到集群中的节点上运行，Pod 会在该节点上一直保持运行状态，直到进程终止、Pod 对象被删除、Pod 因节点资源不足而被驱逐或者节点失效为止。Pod 并不会自愈，当节点失效，或者调度 Pod 的这一操作失败了，Pod 就该被删除。如此，单单用 Pod 来部署应用，是不稳定不安全的。

Kubernetes 使用更高级的资源对象 ***“控制器”\*** 来实现对Pod的管理。控制器可以为您创建和管理多个 Pod，管理副本和上线，并在集群范围内提供自修复能力。 例如，如果一个节点失败，控制器可以在不同的节点上调度一样的替身来自动替换 Pod。

**适用无状态服务**

- `ReplicationController（RC）`

  - Replication Controller 简称 RC，RC 是 Kubernetes 系统中的核心概念之一，简单来说，RC 可以保证在任意时间运行 Pod 的副本数量，能够保证 Pod 总是可用的。如果实际 Pod 数量比指定的多那就结束掉多余的，如果实际数量比指定的少就新启动一些Pod，当 Pod 失败、被删除或者挂掉后，RC 都会去自动创建新的 Pod 来保证副本数量，所以即使只有一个 Pod，我们也应该使用 RC 来管理我们的 Pod。可以说，通过 ReplicationController，Kubernetes 实现了 Pod 的高可用性。

- ReplicaSet（RS）

  - RC （ReplicationController ）主要的作用就是用来确保容器应用的副本数始终保持在用户定义的副本数 。即如果有容器异常退出，会自动创建新的 Pod 来替代；而如果异常多出来的容器也会自动回收（已经成为过去时），在 v1.11 版本废弃。

    **Kubernetes 官方建议使用 RS（ReplicaSet ） 替代 RC （ReplicationController ） 进行部署，RS 跟 RC 没有本质的不同，只是名字不一样，并且 RS 支持集合式的 selector。**

    `Label 和 Selector`

    label （标签）是附加到 Kubernetes 对象（比如 Pods）上的键值对，用于区分对象（比如Pod、Service）。 label 旨在用于指定对用户有意义且相关的对象的标识属性，但不直接对核心系统有语义含义。 label 可以用于组织和选择对象的子集。label 可以在创建时附加到对象，随后可以随时添加和修改。可以像 namespace 一样，使用 label 来获取某类对象，但 label 可以与 selector 一起配合使用，用表达式对条件加以限制，实现更精确、更灵活的资源查找。

    label 与 selector 配合，可以实现对象的“关联”，“Pod 控制器” 与 Pod 是相关联的 —— “Pod 控制器”依赖于 Pod，可以给 Pod 设置 label，然后给“控制器”设置对应的 selector，这就实现了对象的关联。

- Deployment

  - Deployment 为 Pod 和 Replica Set 提供声明式更新。

    

    你只需要在 Deployment 中描述你想要的目标状态是什么，Deployment controller 就会帮你将 Pod 和 Replica Set 的实际状态改变到你的目标状态。你可以定义一个全新的 Deployment，也可以创建一个新的替换旧的 Deployment。

    1. 创建 Replica Set / Pod
    2. 滚动升级/回滚
    3. 平滑扩容和缩容
    4. 暂停与恢复 Deployment

**适用有状态服务**

`StatefulSet`

StatefulSet 中每个 Pod 的 DNS 格式为 statefulSetName-{0..N-1}.serviceName.namespace.svc.cluster.local

- serviceName 为 Headless Service 的名字
- 0..N-1 为 Pod 所在的序号，从 0 开始到 N-1
- statefulSetName 为 StatefulSet 的名字
- namespace 为服务所在的 namespace，Headless Servic 和 StatefulSet 必须在相同的 namespace
- .cluster.local 为 Cluster Domain

主要特点

1. 稳定的持久化存储
   - 即 Pod 重新调度后还是能访问到相同的持久化数据，基于 PVC 来实现
2. 稳定的网络标志
   - 稳定的网络标志，即 Pod 重新调度后其 PodName 和 HostName 不变，基于 Headless Service（即没有 Cluster IP 的 Service）来实现
3. 有序部署，有序扩展
   - 有序部署，有序扩展，即 Pod 是有顺序的，在部署或者扩展的时候要依据定义的顺序依次依次进行（即从 0到 N-1，在下一个Pod 运行之前所有之前的 Pod 必须都是 Running 和 Ready 状态），基于 init containers 来实现
4. 有序收缩，有序删除
   - 有序收缩，有序删除（即从 N-1 到 0）

组成

1. Headless Service

   - 用于定义网络标志（DNS domain）

     Domain Name Server：域名服务
     将域名与 ip 绑定映射关系

     服务名 => 访问路径（域名） => ip

2. volumeClaimTemplate

   - 用于创建 PersistentVolumes

注意事项

1. kubernetes v1.5 版本以上才支持
2. 所有Pod的Volume必须使用PersistentVolume或者是管理员事先创建好
3. 为了保证数据安全，删除StatefulSet时不会删除Volume
4. StatefulSet 需要一个 Headless Service 来定义 DNS domain，需要在 StatefulSet 之前创建好

**守护进程**

`DaemonSet`

DaemonSet 保证在每个 Node 上都运行一个容器副本，常用来部署一些集群的日志、监控或者其他系统管理应用。典型的应用包括：

- 日志收集，比如 fluentd，logstash 等
- 系统监控，比如 Prometheus Node Exporter，collectd，New Relic agent，Ganglia gmond 等
- 系统程序，比如 kube-proxy, kube-dns, glusterd, ceph 等

**任务/定时任务**

1. Job
2. CronJob

###### 服务发现

`Service`

“Service” 简写 “svc”。Pod 不能直接提供给外网访问，而是应该使用 service。Service 就是把 Pod 暴露出来提供服务，Service 才是真正的“服务”，它的中文名就叫“服务”。

可以说 Service 是一个应用服务的抽象，定义了 Pod 逻辑集合和访问这个 Pod 集合的策略。Service 代理 Pod 集合，对外表现为一个访问入口，访问该入口的请求将经过负载均衡，转发到后端 Pod 中的容器。

`Ingress`

Ingress 可以提供外网访问 Service 的能力。可以把某个请求地址映射、路由到特定的 service。

ingress 需要配合 ingress controller 一起使用才能发挥作用，ingress 只是相当于路由规则的集合而已，真正实现路由功能的，是 Ingress Controller，ingress controller 和其它 k8s 组件一样，也是在 Pod 中运行。

###### 存储

`Volume`

数据卷，共享 Pod 中容器使用的数据。用来放持久化的数据，比如数据库数据。

`CSI`

Container Storage Interface 是由来自 Kubernetes、Mesos、Docker 等社区成员联合制定的一个行业标准接口规范，旨在将任意存储系统暴露给容器化应用程序。

CSI 规范定义了存储提供商实现 CSI 兼容的 Volume Plugin 的最小操作集和部署建议。CSI 规范的主要焦点是声明 Volume Plugin 必须实现的接口。

###### 特殊类型配置

`ConfigMap`

用来放配置，与 Secret 是类似的，只是 ConfigMap 放的是明文的数据，Secret 是密文存放。

`Secret`

Secret 解决了密码、token、密钥等敏感数据的配置问题，而不需要把这些敏感数据暴露到镜像或者 Pod Spec 中。Secret 可以以 Volume 或者环境变量的方式使用。

**Secret 有三种类型：**

- Service Account：用来访问 Kubernetes API，由 Kubernetes 自动创建，并且会自动挂载到 Pod 的 /run/secrets/kubernetes.io/serviceaccount 目录中；
- Opaque：base64 编码格式的 Secret，用来存储密码、密钥等；
- kubernetes.io/dockerconfigjson：用来存储私有 docker registry 的认证信息。

`DownwardAPI`

downwardAPI 这个模式和其他模式不一样的地方在于它不是为了存放容器的数据也不是用来进行容器和宿主机的数据交换的，而是让 pod 里的容器能够直接获取到这个 pod 对象本身的一些信息。

downwardAPI 提供了两种方式用于将 pod 的信息注入到容器内部：



环境变量：用于单个变量，可以将 pod 信息和容器信息直接注入容器内部

volume 挂载：将 pod 信息生成为文件，直接挂载到容器内部中去

###### 其他

`Role`

Role 是一组权限的集合，例如 Role 可以包含列出 Pod 权限及列出 Deployment 权限，Role 用于给某个 Namespace 中的资源进行鉴权。

`RoleBinding`

RoleBinding ：将 Subject 绑定到 Role，RoleBinding 使规则在命名空间内生效。

#### 对象规约和状态

对象是用来完成一些任务的，是持久的，是有目的性的，因此 kubernetes 创建一个对象后，将持续地工作以确保对象存在。当然，kubernetes 并不只是维持对象的存在这么简单，kubernetes 还管理着对象的方方面面。每个 Kubernetes 对象包含两个嵌套的对象字段，它们负责管理对象的配置，他们分别是 “spec” 和 “status” 。

##### 规约（Spec）

“spec” 是 “规约”、“规格” 的意思，spec 是必需的，它描述了对象的期望状态（Desired State）—— 希望对象所具有的特征。当创建 Kubernetes 对象时，必须提供对象的规约，用来描述该对象的期望状态，以及关于对象的一些基本信息（例如名称）。

##### 状态（Status）

表示对象的实际状态，该属性由 k8s 自己维护，k8s 会通过一系列的`控制器`对`对应对象`进行管理，让对象尽可能的让实际状态与期望状态重合。

## K8S实战进阶

### 搭建 Kubernetes 集群

#### 搭建方案

1. minikube

   - [minikube](https://minikube.sigs.k8s.io/) 是一个工具， 能让你在本地运行 Kubernetes。 minikube 在你的个人计算机（包括 Windows、macOS 和 Linux PC）上运行一个一体化（all-in-one）或多节点的本地 Kubernetes 集群，以便你来尝试 Kubernetes 或者开展每天的开发工作。

     [官方安装文档](https://minikube.sigs.k8s.io/docs/start/)

2. **`kubeadm`**

   - 服务器要求

     1. 3 台服务器（虚拟机）
        - k8s-master：192.168.133.130
        - k8s-node1：192.168.133.131
        - k8s-node2：192.168.133.132
     2. 最低配置：2核、2G内存、20G硬盘
     3. 最好能联网，不能联网的话需要有提供对应镜像的私有仓库

   - 软件环境

     1. 操作系统：CentOS 7
     2. Docker：20+
     3. k8s：1.23.6

   - 安装步骤

     >环境配置这里选择`20.10.7`版本的Docker，因为kubernetes在`1.24`之后的版本不再支持`Docker`，所以这里选择安装`1.23.6`版本的`kubernetes`

     这里准备三个虚拟机，一个作为`master`节点，另外两个分别是`node1`和`node2`
     
     以`kubeadm`的方式进行安装`kubernetes`
     
     >[卸载k8s教程](https://blog.csdn.net/LONG_Yi_1994/article/details/127139637)
     
     下面开始安装`k8s`
     
     在三台虚拟机上都需要执行
     
     1. 初始操作
     
        ```bash
        # 关闭防火墙
        systemctl stop firewalld
        systemctl disable firewalld
        
        # 关闭selinux
        sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
        setenforce 0  # 临时
        
        # 关闭swap
        swapoff -a  # 临时
        sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久
        
        # 关闭完swap后，一定要重启一下虚拟机！！！
        # 根据规划设置主机名
        hostnamectl set-hostname <hostname>
        
        # 在master添加hosts
        cat >> /etc/hosts << EOF
        192.168.133.131 k8s-master
        192.168.133.132 k8s-node1
        192.168.133.133 k8s-node2
        EOF
        
        
        # 将桥接的IPv4流量传递到iptables的链
        cat > /etc/sysctl.d/k8s.conf << EOF
        net.bridge.bridge-nf-call-ip6tables = 1
        net.bridge.bridge-nf-call-iptables = 1
        EOF
        
        sysctl --system  # 生效
        
        
        # 时间同步
        yum install ntpdate -y
        ntpdate time.windows.com
        
        # 查看当前时间
        # 如果发现时间不正确，先重启一下虚拟机
        date
        ```
     
        >安装`Docker`过程跳过
     
     2. 安装基础软件（所有节点）
     
        1. 安装 Docker
     
        2. 添加阿里云 yum 源
     
           ```bash
           cat > /etc/yum.repos.d/kubernetes.repo << EOF
           [kubernetes]
           name=Kubernetes
           baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
           enabled=1
           gpgcheck=0
           repo_gpgcheck=0
           
           gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
           EOF
           ```
     
           
     
        3. 安装 kubeadm、kubelet、kubectl
     
           ```bash
           yum install -y kubelet-1.23.6 kubeadm-1.23.6 kubectl-1.23.6
           
           systemctl enable kubelet
           
           systemctl status kubelet
           
           # 配置关闭 Docker 的 cgroups，修改 /etc/docker/daemon.json，加入以下内容
           "exec-opts": ["native.cgroupdriver=systemd"]
           
           # 重启 docker
           systemctl daemon-reload
           systemctl restart docker
           ```
           
           
     
     3. 部署 Kubernetes Master
     
        ```sh
        # 在 Master 节点下执行
        
        kubeadm init \
              --apiserver-advertise-address=192.168.133.131 \
              --image-repository registry.aliyuncs.com/google_containers \
              --kubernetes-version v1.23.6 \
              --service-cidr=10.96.0.0/12 \
              --pod-network-cidr=10.244.0.0/16
        
        # 安装成功后，复制如下配置并执行
        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config
        kubectl get nodes
        ```
     
        执行之后看到如下信息
     
        ```
        Your Kubernetes control-plane has initialized successfully!
        
        To start using your cluster, you need to run the following as a regular user:
        
          mkdir -p $HOME/.kube
          sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
          sudo chown $(id -u):$(id -g) $HOME/.kube/config
        
        Alternatively, if you are the root user, you can run:
        
          export KUBECONFIG=/etc/kubernetes/admin.conf
        
        You should now deploy a pod network to the cluster.
        Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
          https://kubernetes.io/docs/concepts/cluster-administration/addons/
        
        Then you can join any number of worker nodes by running the following on each as root:
        
        kubeadm join 192.168.42.131:6443 --token g9tav3.xcm2xeniyr5oyib7 \
        	--discovery-token-ca-cert-hash sha256:0a77d2c43287e745286bc98863e34ed678f5c9c9ee44a5c179d375755648852a
        ```
     
     4. 加入 Kubernetes Node
     
        ```sh
        # 分别在 k8s-node1 和 k8s-node2 执行
        
        # 下方命令可以在 k8s master 控制台初始化成功后复制 join 命令
        
        kubeadm join 192.168.133.131:6443 --token j1wr0y.ej27ox9h3psu96jw --discovery-token-ca-cert-hash sha256:a751ecab11ba054ec4cb70d807de589ae6688324672f02fa4dc62560728d51d4
        
        
        # 如果初始化的 token 不小心清空了，可以通过如下命令获取或者重新申请
        # 如果 token 已经过期，就重新申请
        kubeadm token create
        
        # token 没有过期可以通过如下命令获取
        kubeadm token list
        
        # 获取 --discovery-token-ca-cert-hash 值，得到值后需要在前面拼接上 sha256:
        openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
        openssl dgst -sha256 -hex | sed 's/^.* //'
        
        # 删除旧的 kubelet 证书文件
        rm -f  /opt/kubernetes/ssl/kubelet*
         
        # 删除 kubelet kubeconfig 文件
        rm -f /opt/kubernetes/cfg/kubelet.kubeconfig
         
        # 重启 kubelet 服务，让 master 重新颁发客户端证书
        systemctl restart kubelet
        
        # 重新join
        kubeadm join 192.168.133.131:6443 --token f9gwk1.8xastiqq3p9khlzm --discovery-token-ca-cert-hash sha256:7c10606bed070f0eb4ab19399de70744fca7a3e1be7324c9102cb53ba0ac9be6
        ```
     
        可以看到子节点打印如下内容，代表已经加入`k8s`集群环境了
     
        ```
        This node has joined the cluster:
        * Certificate signing request was sent to apiserver and a response was received.
        * The Kubelet was informed of the new secure connection details.
        
        Run 'kubectl get nodes' on the control-plane to see this node join the cluster
        ```
     
        我注意到kubeadm init时报过两个，`[WARNING SystemVerification]: this Docker version is not on the list of validated versions: 26.1.4. Latest validated version: 20.10`
     
        原因是docker版本问题，后来安装`20.10.7`版本的docker之后，该警告消除掉了
     
        第二个警告是`[WARNING Swap]: swap is enabled; production deployments should disable swap unless testing the NodeSwap feature gate of the kubelet`
     
        执行以下命令即可`swapoff -a`
     
     5. 部署 CNI 网络插件
     
        ```sh
        # 在 master 节点上执行
        cd /opt/k8s
        
        # 下载 calico 配置文件，可能会网络超时
        curl https://docs.projectcalico.org/manifests/calico.yaml -O
        wget https://docs.projectcalico.org/manifests/calico.yaml
        
        # 修改 calico.yaml 文件中的 CALICO_IPV4POOL_CIDR 配置，修改为与初始化的 cidr 相同(10.244.0.0/16)
        
        # 修改 IP_AUTODETECTION_METHOD 下的网卡名称
        sed -i 's/"autodetect"/"autodetect"\n            - name: IP_AUTODETECTION_METHOD\n               value: "interface=ens33"/g' calico.yaml
        
        # 查看需要下载的镜像
        grep image calico.yaml
        
        # 删除镜像 docker.io/ 前缀，避免下载过慢导致失败
        sed -i 's#docker.io/##g' calico.yaml
        
        # 查看删除前缀的操作是否成功
        grep image calico.yaml
        
        # 构建应用
        kubectl apply -f calico.yaml
        
        # 删除应用
        # kubectl delete -f calico.yaml
        
        # 查看构建结果(Pending在执行中，需要等待变成Running)
        kubectl get po -n kube-system
        
        # 查看Pending状况
        # 格式:kubectl describe po <name> -n kube-system
        kubectl describe po calico-node-sc9g6 -n kube-system
        
        docker images | grep calico/cni
        ```
     
        > 如果拉取镜像超时，可以通过配置`/etc/docker/daemon.json`文件
     
        我在安装过程中遇到的问题如下
     
         搭建k8s集群时一直NotReady，我遇到的原因是镜像拉取失败calico/cni:v3.25.0、calico/node:v3.25.0，配置/etc/docker/daemon.json文件如下
     
        ```json
        {
          "registry-mirrors": [
            "https://hub.uuuadc.top",
            "https://docker.anyhub.us.kg",
            "https://dockerhub.jobcher.com",
            "https://dockerhub.icu",
            "https://docker.ckyl.me",
            "https://docker.awsl9527.cn"
          ],
          "exec-opts": ["native.cgroupdriver=systemd"],
          "log-driver": "json-file",
          "log-opts": {
            "max-size": "100m"
          },
          "storage-driver": "overlay2"
        }
        ```
     
        > 继续使用如下命令更新配置，再重新pull一遍即可
     
        ```sh
        sudo systemctl daemon-reload
        sudo systemctl restart docker
        ```
     
     6. 测试 kubernetes 集群
     
        ```bash
        # 创建部署
        kubectl create deployment nginx --image=nginx	
        
        # 暴露端口
        kubectl expose deployment nginx --port=80 --type=NodePort
        
        # 查看 pod 以及服务信息
        kubectl get pod,svc
        ```
     
     


3. 二进制安装

   - 利用 k8s 官方 github 仓库下载二进制包安装，安装过程较复杂，但相对较为稳定，推荐生产环境使用。

4. 命令行工具


#### 命令行工具kubectl

Kubernetes 提供 kubectl 是使用 Kubernetes API 与 Kubernetes 集群的[控制面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)进行通信的命令行工具。

这个工具叫做 kubectl。
[更多命令](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)

[kubectl命令](https://kubernetes.io/zh-cn/docs/reference/kubectl/generated/kubectl/)

##### 在任意节点使用 kubectl

###### 将 master 节点中 /etc/kubernetes/admin.conf 拷贝到需要运行的服务器的 /etc/kubernetes 目录中

```sh
scp /etc/kubernetes/admin.conf root@k8s-node1:/etc/kubernetes
scp /etc/kubernetes/admin.conf root@k8s-node2:/etc/kubernetes
```



###### 在对应的服务器上配置环境变量

```sh
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
source ~/.bash_profile
```

> 在子节点上执行`kubectl`命令查看是否成功

```sh
kubectl get pod,svc
```

##### 资源操作

###### 创建对象

```sh
kubectl create -f ./my-manifest.yaml           # 创建资源
kubectl create -f ./my1.yaml -f ./my2.yaml     # 使用多个文件创建资源
kubectl create -f ./dir                        # 使用目录下的所有清单文件来创建资源
kubectl create -f https://git.io/vPieo         # 使用 url 来创建资源
kubectl run nginx --image=nginx                # 启动一个 nginx 实例
kubectl explain pods,svc                       # 获取 pod 和 svc 的文档

# 从 stdin 输入中创建多个 YAML 对象
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - sleep
    - "1000000"
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep-less
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - sleep
    - "1000"
EOF

# 创建包含几个 key 的 Secret
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: $(echo "s33msi4" | base64)
  username: $(echo "jane" | base64)
EOF
```

nginx-demo.yaml

```yaml
apiVersion: v1 # api文档版本
kind: Pod # 资源对象类型
metadata: # Pod相关的元数据，用于描述Pod
  name: nginx-demo # Pod的名称
  labels: # 定义Pod的标签
    type: myFirstApp # 以下全部都是自定义标签
    version: 0.0.1
  namespace: 'default' # 命名空间
spec: # 期望Pod根据这里的描述进行创建
  containers: # 对于Pod中容器的描述
  - name: nginx # 容器名称
    image: nginx:1.7.9 # 指定容器的镜像
    imagePullPolicy: IfNotPresent # 指定镜像的拉取策略
    command: # 指定容器启动时，执行的命令
    - nginx
    - -g
    - 'daemon off;' # nginx -g 'daemon off;'
    workingDir: /usr/share/nginx/html # 指定工作目录
    ports: 
    - name: http # 端口名称
      containerPort: 80 # 容器内暴露的端口
      protocol: TCP # 描述该端口是基于什么协议进行通讯的
    env: # 环境变量
    - name: JVM_OPTS # 环境变量名称
      value: '-Xms128m -Xmx128m' # 环境变量值
  
    resources: # 配置资源
      requests: # 最少需要多少资源
        cpu: 100m # 1000m等于一个cpu核心，最少使用0.1个核心
        memory: 128Mi # 限制内存最少使用128兆
      limits: # 最多使用多少资源
        cpu: 200m # 限制cpu最多使用0.2个核心
        memory: 256Mi # 限制内存最多使用256兆
  restartPolicy: OnFailure # 配置重启策略
```

> 可使用如下命令创建pod

```sh
kubectl create -f nginx-demo.yaml
```



###### 显示和查找资源

```sh
# Get commands with basic output
$ kubectl get services                          # 列出所有 namespace 中的所有 service
$ kubectl get pods --all-namespaces             # 列出所有 namespace 中的所有 pod
$ kubectl get pods -o wide                      # 列出所有 pod 并显示详细信息
$ kubectl get deployment my-dep                 # 列出指定 deployment
$ kubectl get pods --include-uninitialized      # 列出该 namespace 中的所有 pod 包括未初始化的

# 使用详细输出来描述命令
$ kubectl describe nodes my-node
$ kubectl describe pods my-pod

$ kubectl get services --sort-by=.metadata.name # List Services Sorted by Name

# 根据重启次数排序列出 pod
$ kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'

# 获取所有具有 app=cassandra 的 pod 中的 version 标签
$ kubectl get pods --selector=app=cassandra rc -o \
  jsonpath='{.items[*].metadata.labels.version}'

# 获取所有节点的 ExternalIP
$ kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'

# 列出属于某个 PC 的 Pod 的名字
# “jq”命令用于转换复杂的 jsonpath，参考 https://stedolan.github.io/jq/
$ sel=${$(kubectl get rc my-rc --output=json | jq -j '.spec.selector | to_entries | .[] | "\(.key)=\(.value),"')%?}
$ echo $(kubectl get pods --selector=$sel --output=jsonpath={.items..metadata.name})

# 查看哪些节点已就绪
$ JSONPATH='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' \
 && kubectl get nodes -o jsonpath="$JSONPATH" | grep "Ready=True"

# 列出当前 Pod 中使用的 Secret
$ kubectl get pods -o json | jq '.items[].spec.containers[].env[]?.valueFrom.secretKeyRef.name' | grep -v null | sort | uniq
```

###### 更新资源

```sh
$ kubectl rolling-update frontend-v1 -f frontend-v2.json           # 滚动更新 pod frontend-v1
$ kubectl rolling-update frontend-v1 frontend-v2 --image=image:v2  # 更新资源名称并更新镜像
$ kubectl rolling-update frontend --image=image:v2                 # 更新 frontend pod 中的镜像
$ kubectl rolling-update frontend-v1 frontend-v2 --rollback        # 退出已存在的进行中的滚动更新
$ cat pod.json | kubectl replace -f -                              # 基于 stdin 输入的 JSON 替换 pod

# 强制替换，删除后重新创建资源。会导致服务中断。
$ kubectl replace --force -f ./pod.json

# 为 nginx RC 创建服务，启用本地 80 端口连接到容器上的 8000 端口
$ kubectl expose rc nginx --port=80 --target-port=8000

# 更新单容器 pod 的镜像版本（tag）到 v4
$ kubectl get pod mypod -o yaml | sed 's/\(image: myimage\):.*$/\1:v4/' | kubectl replace -f -

$ kubectl label pods my-pod new-label=awesome                      # 添加标签
$ kubectl annotate pods my-pod icon-url=http://goo.gl/XXBTWq       # 添加注解
$ kubectl autoscale deployment foo --min=2 --max=10                # 自动扩展 deployment “foo”
```

###### 修补资源

```sh
$ kubectl patch node k8s-node-1 -p '{"spec":{"unschedulable":true}}' # 部分更新节点

# 更新容器镜像； spec.containers[*].name 是必须的，因为这是合并的关键字
$ kubectl patch pod valid-pod -p '{"spec":{"containers":[{"name":"kubernetes-serve-hostname","image":"new image"}]}}'

# 使用具有位置数组的 json 补丁更新容器镜像
$ kubectl patch pod valid-pod --type='json' -p='[{"op": "replace", "path": "/spec/containers/0/image", "value":"new image"}]'

# 使用具有位置数组的 json 补丁禁用 deployment 的 livenessProbe
$ kubectl patch deployment valid-deployment  --type json   -p='[{"op": "remove", "path": "/spec/template/spec/containers/0/livenessProbe"}]'
```

###### 编辑资源

```sh
$ kubectl edit svc/docker-registry                      # 编辑名为 docker-registry 的 service
$ KUBE_EDITOR="nano" kubectl edit svc/docker-registry   # 使用其它编辑器
```

###### scale 资源

```sh
$ kubectl scale --replicas=3 rs/foo                                 # Scale a replicaset named 'foo' to 3
$ kubectl scale --replicas=3 -f foo.yaml                            # Scale a resource specified in "foo.yaml" to 3
$ kubectl scale --current-replicas=2 --replicas=3 deployment/mysql  # If the deployment named mysql's current size is 2, scale mysql to 3
$ kubectl scale --replicas=5 rc/foo rc/bar rc/baz                   # Scale multiple replication controllers
```

###### 删除资源

```sh
$ kubectl delete -f ./pod.json                                              # 删除 pod.json 文件中定义的类型和名称的 pod
$ kubectl delete pod,service baz foo                                        # 删除名为“baz”的 pod 和名为“foo”的 service
$ kubectl delete pods,services -l name=myLabel                              # 删除具有 name=myLabel 标签的 pod 和 serivce
$ kubectl delete pods,services -l name=myLabel --include-uninitialized      # 删除具有 name=myLabel 标签的 pod 和 service，包括尚未初始化的
$ kubectl -n my-ns delete po,svc --all                                      # 删除 my-ns namespace 下的所有 pod 和 serivce，包括尚未初始化的
```

##### Pod 与集群

###### 与运行的 Pod 交互

```sh
$ kubectl logs my-pod                                 # dump 输出 pod 的日志（stdout）
$ kubectl logs my-pod -c my-container                 # dump 输出 pod 中容器的日志（stdout，pod 中有多个容器的情况下使用）
$ kubectl logs -f my-pod                              # 流式输出 pod 的日志（stdout）
$ kubectl logs -f my-pod -c my-container              # 流式输出 pod 中容器的日志（stdout，pod 中有多个容器的情况下使用）
$ kubectl run -i --tty busybox --image=busybox -- sh  # 交互式 shell 的方式运行 pod
$ kubectl attach my-pod -i                            # 连接到运行中的容器
$ kubectl port-forward my-pod 5000:6000               # 转发 pod 中的 6000 端口到本地的 5000 端口
$ kubectl exec my-pod -- ls /                         # 在已存在的容器中执行命令（只有一个容器的情况下）
$ kubectl exec my-pod -c my-container -- ls /         # 在已存在的容器中执行命令（pod 中有多个容器的情况下）
$ kubectl top pod POD_NAME --containers               # 显示指定 pod 和容器的指标度量
```

###### 与节点和集群交互

```sh
$ kubectl cordon my-node                                                # 标记 my-node 不可调度
$ kubectl drain my-node                                                 # 清空 my-node 以待维护
$ kubectl uncordon my-node                                              # 标记 my-node 可调度
$ kubectl top node my-node                                              # 显示 my-node 的指标度量
$ kubectl cluster-info                                                  # 显示 master 和服务的地址
$ kubectl cluster-info dump                                             # 将当前集群状态输出到 stdout                                    
$ kubectl cluster-info dump --output-directory=/path/to/cluster-state   # 将当前集群状态输出到 /path/to/cluster-state

# 如果该键和影响的污点（taint）已存在，则使用指定的值替换
$ kubectl taint nodes foo dedicated=special-user:NoSchedule
```

#### 资源类型与别名

##### pods

> po

Pod 的设计理念是支持多个容器在一个 Pod 中共享网络地址和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。Pod 对多容器的支持是 K8 最基础的设计理念。比如你运行一个操作系统发行版的软件仓库，一个 Nginx 容器用来发布软件，另一个容器专门用来从源仓库做同步，这两个容器的镜像不太可能是一个团队开发的，但是他们一块儿工作才能提供一个微服务；这种情况下，不同的团队各自开发构建自己的容器镜像，在部署的时候组合成一个微服务对外提供服务。

Kubernetes（k8s）中的Pod与Docker Compose有相似之处

##### Replication Controller

>rc（副本控制器）

RC 是 Kubernetes 集群中最早的保证 Pod 高可用的 API 对象。通过监控运行中的 Pod 来保证集群中运行指定数目的 Pod 副本。指定的数目可以是多个也可以是 1 个；少于指定数目，RC 就会启动运行新的 Pod 副本；多于指定数目，RC 就会杀死多余的 Pod 副本。即使在指定数目为 1 的情况下，通过 RC 运行 Pod 也比直接运行 Pod 更明智，因为 RC 也可以发挥它高可用的能力，保证永远有 1 个 Pod 在运行。RC 是 Kubernetes 较早期的技术概念，只适用于长期伺服型的业务类型，比如控制小机器人提供高可用的 Web 服务。

Replica Set

>rs（副本集）

RS 是新一代 RC，提供同样的高可用能力，区别主要在于 `RS` 后来居上，能支持更多种类的匹配模式。副本集对象一般不单独使用，而是作为 Deployment 的理想状态参数使用。

##### deployments

> deploy

部署表示用户对 Kubernetes 集群的一次更新操作。部署是一个比 RS 应用模式更广的 API 对象，可以是创建一个新的服务，更新一个新的服务，也可以是滚动升级一个服务。滚动升级一个服务，实际是创建一个新的 RS，然后逐渐将新 RS 中副本数增加到理想状态，将旧 RS 中的副本数减小到 0 的复合操作；这样一个复合操作用一个 RS 是不太好描述的，所以用一个更通用的 Deployment 来描述。以 Kubernetes 的发展方向，未来对所有长期伺服型的的业务的管理，都会通过 Deployment 来管理。

##### statefulset

> sts（有状态服务集）

RC 和 RS 主要是控制提供无状态服务的，其所控制的 Pod 的名字是随机设置的，一个 Pod 出故障了就被丢弃掉，在另一个地方重启一个新的 Pod，名字变了。名字和启动在哪儿都不重要，重要的只是 Pod 总数；而 StatefulSet 是用来控制有状态服务，StatefulSet 中的每个 Pod 的名字都是事先确定的，不能更改。

对于 RC 和 RS 中的 Pod，一般不挂载存储或者挂载共享存储，保存的是所有 Pod 共享的状态；对于 StatefulSet 中的 Pod，每个 Pod 挂载自己独立的存储，如果一个 Pod 出现故障，从其他节点启动一个同样名字的 Pod，要挂载上原来 Pod 的存储继续以它的状态提供服务。

适合于 StatefulSet 的业务包括数据库服务 MySQL 和 PostgreSQL，集群化管理服务 ZooKeeper、etcd 等有状态服务。StatefulSet 的另一种典型应用场景是作为一种比普通容器更稳定可靠的模拟虚拟机的机制。传统的虚拟机正是一种有状态的宠物，运维人员需要不断地维护它，容器刚开始流行时，我们用容器来模拟虚拟机使用，所有状态都保存在容器里，而这已被证明是非常不安全、不可靠的。使用 StatefulSet，Pod 仍然可以通过漂移到不同节点提供高可用，而存储也可以通过外挂的存储来提供高可靠性，StatefulSet 做的只是将确定的 Pod 与确定的存储关联起来保证状态的连续性。

##### volume

Kubernetes 集群中的存储卷跟 Docker 的存储卷有些类似，只不过 Docker 的存储卷作用范围为一个容器，而 Kubernetes 的存储卷的生命周期和作用范围是一个 Pod。每个 Pod 中声明的存储卷由 Pod 中的所有容器共享。Kubernetes 支持非常多的存储卷类型，特别的，支持多种公有云平台的存储，包括 AWS，Google 和 Azure 云；支持多种分布式存储包括 GlusterFS 和 Ceph；也支持较容易使用的主机本地目录 emptyDir, hostPath 和 NFS。Kubernetes 还支持使用 Persistent Volume Claim 即 PVC 这种逻辑存储，使用这种存储，使得存储的使用者可以忽略后台的实际存储技术（例如 AWS，Google 或 GlusterFS 和 Ceph），而将有关存储实际技术的配置交给存储管理员通过 Persistent Volume 来配置。

##### services

> svc

##### namespace

> ns

命名空间为 Kubernetes 集群提供虚拟的隔离作用，Kubernetes 集群初始有两个命名空间，分别是默认命名空间 default 和系统命名空间 kube-system，除此以外，管理员可以可以创建新的命名空间满足需要。

##### nodes

> no

#### 格式化输出

##### 输出 json 格式

```
-o json
```



##### 仅打印资源名称

```
-o name
```



##### 以纯文本格式输出所有信息

```
-o wide
```



##### 输出 yaml 格式

```
-o yaml
```

#### API 概述

官网文档：https://kubernetes.io/zh-cn/docs/reference/using-api/

REST API 是 Kubernetes 系统的重要部分，组件之间的所有操作和通信均由 API Server 处理的 REST AP I调用，大多数情况下， API 定义和实现都符合标准的 HTTP REST 格式，可以通过 [kubectl](http://docs.kubernetes.org.cn/61.html) 命令管理工具或其他命令行工具来执行。

##### 类型

###### Alpha

- 包含 alpha 名称的版本（例如v1alpha1）。
- 该软件可能包含错误。启用一个功能可能会导致 bug。默认情况下，功能可能会被禁用。
- 随时可能会丢弃对该功能的支持，恕不另行通知。
- API 可能在以后的软件版本中以不兼容的方式更改，恕不另行通知。
- 该软件建议仅在短期测试集群中使用，因为错误的风险增加和缺乏长期支持。

###### Beta

- 包含 **beta** 名称的版本（例如 **v2beta3**）。
- 该软件经过很好的测试。启用功能被认为是安全的。默认情况下功能是开启的。
- 细节可能会改变，但功能在后续版本不会被删除
- 对象的模式或语义在随后的 beta 版本或 Stable 版本中可能以不兼容的方式发生变化。如果这种情况发生时，官方会提供迁移操作指南。这可能需要删除、编辑和重新创建API对象。
- 该版本在后续可能会更改一些不兼容地方，所以建议用于非关键业务，如果你有多个可以独立升级的集群，你也可以放宽此限制。
- **大家使用过的 Beta 版本后，可以多给社区反馈，如果此版本在后续更新后将不会有太大变化。**

###### Stable

- 该版本名称命名方式：**vX** 这里 **X** 是一个整数。
- Stable 版本的功能特性，将出现在后续发布的软件版本中。

##### 访问控制

###### 认证



###### 授权

##### 废弃 api 说明

[相关文档](https://kubernetes.io/zh-cn/docs/reference/using-api/deprecation-guide/)

### 深入Pod

#### Pod 配置文件

```yaml
apiVersion: v1 # api 文档版本
kind: Pod  # 资源对象类型，也可以配置为像Deployment、StatefulSet这一类的对象
metadata: # Pod 相关的元数据，用于描述 Pod 的数据
  name: nginx-demo # Pod 的名称
  labels: # 定义 Pod 的标签
    type: app # 自定义 label 标签，名字为 type，值为 app
    test: 1.0.0 # 自定义 label 标签，描述 Pod 版本号
  namespace: 'default' # 命名空间的配置
spec: # 期望 Pod 按照这里面的描述进行创建
  containers: # 对于 Pod 中的容器描述
  - name: nginx # 容器的名称
    image: nginx:1.7.9 # 指定容器的镜像
    imagePullPolicy: IfNotPresent # 镜像拉取策略，指定如果本地有就用本地的，如果没有就拉取远程的
    command: # 指定容器启动时执行的命令
    - nginx
    - -g
    - 'daemon off;' # nginx -g 'daemon off;'
    workingDir: /usr/share/nginx/html # 定义容器启动后的工作目录
    ports:
    - name: http # 端口名称
      containerPort: 80 # 描述容器内要暴露什么端口
      protocol: TCP # 描述该端口是基于哪种协议通信的
    - env: # 环境变量
      name: JVM_OPTS # 环境变量名称
      value: '-Xms128m -Xmx128m' # 环境变量的值
    reousrces:
      requests: # 最少需要多少资源
        cpu: 100m # 限制 cpu 最少使用 0.1 个核心
        memory: 128Mi # 限制内存最少使用 128兆
      limits: # 最多可以用多少资源
        cpu: 200m # 限制 cpu 最多使用 0.2 个核心
        memory: 256Mi # 限制 最多使用 256兆
  restartPolicy: OnFailure # 重启策略，只有失败的情况才会重启
```

> 执行以下命令进行创建

```sh
# -f 指定创建模板
kubectl create -f nginx-demo.yaml

# 查看pods状态，发现已经在运行了
kubectl get po

# 查看更加详细的信息，如端口号
kubectl get po -o wide

NAME         READY   STATUS    RESTARTS   AGE     IP               NODE        NOMINATED NODE   READINESS GATES
nginx-demo   1/1     Running   0          2m26s   10.244.169.131   k8s-node2   <none>           <none>

# 可以看到这个时候的ip(10.244.169.131)和节点(node2)
route -n

Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.42.2    0.0.0.0         UG    100    0        0 ens33
10.244.36.64    192.168.42.132  255.255.255.192 UG    0      0        0 tunl0
10.244.169.128  192.168.42.133  255.255.255.192 UG    0      0        0 tunl0
10.244.235.192  0.0.0.0         255.255.255.192 U     0      0        0 *
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.42.0    0.0.0.0         255.255.255.0   U     100    0        0 ens33
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0

# 可以看到确实被路由到了node2节点上，这个功能是由于我们之前安装了`CNI 网络插件`实现的
# ip(10.244.169.131)通过node2节点进行转发，此时再进入node2节点下查看路由信息，就可以找到当前的网关了
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.42.2    0.0.0.0         UG    100    0        0 ens33
10.244.36.64    192.168.42.132  255.255.255.192 UG    0      0        0 tunl0
10.244.169.128  0.0.0.0         255.255.255.192 U     0      0        0 *
10.244.169.130  0.0.0.0         255.255.255.255 UH    0      0        0 cali11b82779865
10.244.169.131  0.0.0.0         255.255.255.255 UH    0      0        0 cali7543b7f115c
10.244.235.192  192.168.42.131  255.255.255.192 UG    0      0        0 tunl0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.42.0    0.0.0.0         255.255.255.0   U     100    0        0 ens33
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
```

#### 探针

> 在实际运行中可能会由于各种原因导致服务挂掉，比如内存溢出
>
> 但是pod怎么监听到容器挂掉了呢？是通过`探针`来监控容器实现
>
> 需要关注下配置的`重启策略(restartPolicy)`，当容器挂掉之后，kubelet会根据配置的重启策略进行重启

容器内应用的监测机制，根据不同的探针来判断容器应用当前的状态

```sh
# 删除nginx-demo pod
kubectl delete po nginx-demo

# 修改yaml，新增探针配置
```

##### StartupProbe

```yaml
apiVersion: v1 # api文档版本
kind: Pod # 资源对象类型
metadata: # Pod相关的元数据，用于描述Pod
  name: nginx-demo # Pod的名称
  labels: # 定义Pod的标签
    type: myFirstApp # 以下全部都是自定义标签
    version: 0.0.1
  namespace: 'default' # 命名空间
spec: # 期望Pod根据这里的描述进行创建
  containers: # 对于Pod中容器的描述
  - name: nginx # 容器名称
    image: nginx:1.7.9 # 指定容器的镜像
    imagePullPolicy: IfNotPresent # 指定镜像的拉取策略
    startupProbe: # 应用启动探针配置
      httpGet: # 探测方式，基于http请求探测
        path: /api/path # http请求路径
        port: 80 # http请求端口
      failureThreshold: 3 # 失败重试多少次后，真正失败
      periodSeconds: 10 # 间隔时间
      successThreshold: 1 # 多少次成功后，真正成功
      timeoutSeconds: 5 # 请求的超时时间
    command: # 指定容器启动时，执行的命令
    - nginx
    - -g
    - 'daemon off;' # nginx -g 'daemon off;'
    workingDir: /usr/share/nginx/html # 指定工作目录
    ports: 
    - name: http # 端口名称
      containerPort: 80 # 容器内暴露的端口
      protocol: TCP # 描述该端口是基于什么协议进行通讯的
    env: # 环境变量
    - name: JVM_OPTS # 环境变量名称
      value: '-Xms128m -Xmx128m' # 环境变量值
  
    resources: # 配置资源
      requests: # 最少需要多少资源
        cpu: 100m # 1000m等于一个cpu核心，最少使用0.1个核心
        memory: 128Mi # 限制内存最少使用128兆
      limits: # 最多使用多少资源
        cpu: 200m # 限制cpu最多使用0.2个核心
        memory: 256Mi # 限制内存最多使用256兆
  restartPolicy: OnFailure # 配置重启策略
```

```sh
# 创建po
kubectl create -f nginx-demo.yaml

# 查看po状态
kubectl get po

# 查看详细信息
kubectl describe po nginx-demo
# 发现有如下日志
# Warning  Unhealthy  6s (x7 over 66s)   kubelet            Startup probe failed: HTTP probe failed with statuscode: 404

# 因为nginx并没有/api/path请求路径，所以报404
# 现在更改为/index.html后重新构建po
# 发现构建成功
#  Normal  Started    7s    kubelet            Started container nginx

# 配置文件修改成如下，不再使用http，使用tcpSocket的协议
# 只要成功建立tcp连接，我们就认为pod构建成功

# 编辑pods
kubectl edit po nginx-demo

# 删除pods
kubectl delete po nginx-demo
```

```yaml
apiVersion: v1 # api文档版本
kind: Pod # 资源对象类型
metadata: # Pod相关的元数据，用于描述Pod
  name: nginx-demo # Pod的名称
  labels: # 定义Pod的标签
    type: myFirstApp # 以下全部都是自定义标签
    version: 0.0.1
  namespace: 'default' # 命名空间
spec: # 期望Pod根据这里的描述进行创建
  containers: # 对于Pod中容器的描述
  - name: nginx # 容器名称
    image: nginx:1.7.9 # 指定容器的镜像
    imagePullPolicy: IfNotPresent # 指定镜像的拉取策略
    startupProbe: # 应用启动探针配置
      # httpGet: # 探测方式，基于http请求探测
      #   path: /index.html # http请求路径
      #   port: 80 # http请求端口
      tcpSocket:
        port: 80
      failureThreshold: 3 # 失败重试多少次后，真正失败
      periodSeconds: 10 # 间隔时间
      successThreshold: 1 # 多少次成功后，真正成功
      timeoutSeconds: 5 # 请求的超时时间
    command: # 指定容器启动时，执行的命令
    - nginx
    - -g
    - 'daemon off;' # nginx -g 'daemon off;'
    workingDir: /usr/share/nginx/html # 指定工作目录
    ports: 
    - name: http # 端口名称
      containerPort: 80 # 容器内暴露的端口
      protocol: TCP # 描述该端口是基于什么协议进行通讯的
    env: # 环境变量
    - name: JVM_OPTS # 环境变量名称
      value: '-Xms128m -Xmx128m' # 环境变量值
  
    resources: # 配置资源
      requests: # 最少需要多少资源
        cpu: 100m # 1000m等于一个cpu核心，最少使用0.1个核心
        memory: 128Mi # 限制内存最少使用128兆
      limits: # 最多使用多少资源
        cpu: 200m # 限制cpu最多使用0.2个核心
        memory: 256Mi # 限制内存最多使用256兆
  restartPolicy: OnFailure # 配置重启策略
```

![image-20240624202529600](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412242119133.png)

> 修改成exec的形式

```yaml
apiVersion: v1 # api文档版本
kind: Pod # 资源对象类型
metadata: # Pod相关的元数据，用于描述Pod
  name: nginx-demo # Pod的名称
  labels: # 定义Pod的标签
    type: myFirstApp # 以下全部都是自定义标签
    version: 0.0.1
  namespace: 'default' # 命名空间
spec: # 期望Pod根据这里的描述进行创建
  containers: # 对于Pod中容器的描述
  - name: nginx # 容器名称
    image: nginx:1.7.9 # 指定容器的镜像
    imagePullPolicy: IfNotPresent # 指定镜像的拉取策略
    startupProbe: # 应用启动探针配置
      # httpGet: # 探测方式，基于http请求探测
      #   path: /index.html # http请求路径
      #   port: 80 # http请求端口
      # tcpSocket:
      #   port: 80
      exec:
        command:
        - sh
        - c
        - "sleep 5;echo 'successful' > /inited;"
      failureThreshold: 3 # 失败重试多少次后，真正失败
      periodSeconds: 10 # 间隔时间
      successThreshold: 1 # 多少次成功后，真正成功
      timeoutSeconds: 5 # 请求的超时时间
    command: # 指定容器启动时，执行的命令
    - nginx
    - -g
    - 'daemon off;' # nginx -g 'daemon off;'
    workingDir: /usr/share/nginx/html # 指定工作目录
    ports: 
    - name: http # 端口名称
      containerPort: 80 # 容器内暴露的端口
      protocol: TCP # 描述该端口是基于什么协议进行通讯的
    env: # 环境变量
    - name: JVM_OPTS # 环境变量名称
      value: '-Xms128m -Xmx128m' # 环境变量值
  
    resources: # 配置资源
      requests: # 最少需要多少资源
        cpu: 100m # 1000m等于一个cpu核心，最少使用0.1个核心
        memory: 128Mi # 限制内存最少使用128兆
      limits: # 最多使用多少资源
        cpu: 200m # 限制cpu最多使用0.2个核心
        memory: 256Mi # 限制内存最多使用256兆
  restartPolicy: OnFailure # 配置重启策略
```

```sh
# 查看发现重试过4次，说明失败了
kubectl get po

# 寻找失败原因
# 发现是命令有问题，实际执行的命令sh c sleep 5;echo 'successful' > /inited;
# 期待执行的命令sh -c sleep 5;echo 'successful' > /inited;
# 修改代码 【- c】 -> 【- -c】

# 提示Startup probe failed: command "sh -c sleep 5;echo 'successful' > /inited;" timed out
# 因为睡眠5s，而配置的超时时间也是5s，我们继续讲睡眠时间改短点

# -it表示交互模式
# -c指定容器为nginx
# --后面的命令是执行的shell指令
kubectl exec -it nginx-demo -c nginx -- cat /inited
```

> 复制一个`nginx-liveness-demo.yaml`文件

##### LivenessProbe

```yaml
apiVersion: v1 # api文档版本
kind: Pod # 资源对象类型
metadata: # Pod相关的元数据，用于描述Pod
  name: nginx-liveness-demo # Pod的名称
  labels: # 定义Pod的标签
    type: myFirstApp # 以下全部都是自定义标签
    version: 0.0.1
  namespace: 'default' # 命名空间
spec: # 期望Pod根据这里的描述进行创建
  containers: # 对于Pod中容器的描述
  - name: nginx # 容器名称
    image: nginx:1.7.9 # 指定容器的镜像
    imagePullPolicy: IfNotPresent # 指定镜像的拉取策略
    startupProbe: # 应用启动探针配置
      # httpGet: # 探测方式，基于http请求探测
      #   path: /index.html # http请求路径
      #   port: 80 # http请求端口
      # tcpSocket:
      #   port: 80
      exec:
        command:
        - sh
        - -c
        - "sleep 2;echo 'successful' > /inited;"
      failureThreshold: 3 # 失败重试多少次后，真正失败
      periodSeconds: 10 # 间隔时间
      successThreshold: 1 # 多少次成功后，真正成功
      timeoutSeconds: 5 # 请求的超时时间

    livenessProbe: # 应用存活探针配置
      httpGet: # 探测方式，基于http请求探测
        path: /index1.html # http请求路径
        port: 80 # http请求端口
      # tcpSocket:
      #   port: 80
      # exec:
      #   command:
      #   - sh
      #   - -c
      #   - "sleep 2;echo 'successful' > /inited;"
      failureThreshold: 3 # 失败重试多少次后，真正失败
      periodSeconds: 10 # 间隔时间
      successThreshold: 1 # 多少次成功后，真正成功
      timeoutSeconds: 5 # 请求的超时时间
    command: # 指定容器启动时，执行的命令
    - nginx
    - -g
    - 'daemon off;' # nginx -g 'daemon off;'
    workingDir: /usr/share/nginx/html # 指定工作目录
    ports: 
    - name: http # 端口名称
      containerPort: 80 # 容器内暴露的端口
      protocol: TCP # 描述该端口是基于什么协议进行通讯的
    env: # 环境变量
    - name: JVM_OPTS # 环境变量名称
      value: '-Xms128m -Xmx128m' # 环境变量值
  
    resources: # 配置资源
      requests: # 最少需要多少资源
        cpu: 100m # 1000m等于一个cpu核心，最少使用0.1个核心
        memory: 128Mi # 限制内存最少使用128兆
      limits: # 最多使用多少资源
        cpu: 200m # 限制cpu最多使用0.2个核心
        memory: 256Mi # 限制内存最多使用256兆
  restartPolicy: OnFailure # 配置重启策略
```

```sh
# 请求/index1.html失败，在重试5次后，依然无法启动
#  Warning  Unhealthy  17s (x9 over 117s)   kubelet            Liveness probe failed: HTTP probe failed with statuscode: 404
#  Normal   Killing    17s (x3 over 97s)    kubelet            Container nginx failed liveness probe, will be restarted
echo 'success' > index1.html

# 拷贝到文件内部
kubectl cp index1.html nginx-liveness-demo:/usr/share/nginx/html
```

> 当三种探针同时存在时，必定先运行`startup`，之后才是`LivenessProbe`和`ReadinessProbe`

##### ReadinessProbe

```yaml
apiVersion: v1 # api文档版本
kind: Pod # 资源对象类型
metadata: # Pod相关的元数据，用于描述Pod
  name: nginx-liveness-demo # Pod的名称
  labels: # 定义Pod的标签
    type: myFirstApp # 以下全部都是自定义标签
    version: 0.0.1
  namespace: 'default' # 命名空间
spec: # 期望Pod根据这里的描述进行创建
  containers: # 对于Pod中容器的描述
  - name: nginx # 容器名称
    image: nginx:1.7.9 # 指定容器的镜像
    imagePullPolicy: IfNotPresent # 指定镜像的拉取策略
    startupProbe: # 应用启动探针配置
      # httpGet: # 探测方式，基于http请求探测
      #   path: /index.html # http请求路径
      #   port: 80 # http请求端口
      # tcpSocket:
      #   port: 80
      exec:
        command:
        - sh
        - -c
        - "sleep 2;echo 'successful' > /inited;"
      failureThreshold: 3 # 失败重试多少次后，真正失败
      periodSeconds: 10 # 间隔时间
      successThreshold: 1 # 多少次成功后，真正成功
      timeoutSeconds: 5 # 请求的超时时间

    livenessProbe: # 应用存活探针配置
      httpGet: # 探测方式，基于http请求探测
        path: /index.html # http请求路径
        port: 80 # http请求端口
      # tcpSocket:
      #   port: 80
      # exec:
      #   command:
      #   - sh
      #   - -c
      #   - "sleep 2;echo 'successful' > /inited;"
      failureThreshold: 3 # 失败重试多少次后，真正失败
      periodSeconds: 10 # 间隔时间
      successThreshold: 1 # 多少次成功后，真正成功
      timeoutSeconds: 5 # 请求的超时时间

    readinessProbe: # 应用存活探针配置
      httpGet: # 探测方式，基于http请求探测
        path: /index1.html # http请求路径
        port: 80 # http请求端口
      # tcpSocket:
      #   port: 80
      # exec:
      #   command:
      #   - sh
      #   - -c
      #   - "sleep 2;echo 'successful' > /inited;"
      failureThreshold: 3 # 失败重试多少次后，真正失败
      periodSeconds: 10 # 间隔时间
      successThreshold: 1 # 多少次成功后，真正成功
      timeoutSeconds: 5 # 请求的超时时间

    command: # 指定容器启动时，执行的命令
    - nginx
    - -g
    - 'daemon off;' # nginx -g 'daemon off;'
    workingDir: /usr/share/nginx/html # 指定工作目录
    ports: 
    - name: http # 端口名称
      containerPort: 80 # 容器内暴露的端口
      protocol: TCP # 描述该端口是基于什么协议进行通讯的
    env: # 环境变量
    - name: JVM_OPTS # 环境变量名称
      value: '-Xms128m -Xmx128m' # 环境变量值
  
    resources: # 配置资源
      requests: # 最少需要多少资源
        cpu: 100m # 1000m等于一个cpu核心，最少使用0.1个核心
        memory: 128Mi # 限制内存最少使用128兆
      limits: # 最多使用多少资源
        cpu: 200m # 限制cpu最多使用0.2个核心
        memory: 256Mi # 限制内存最多使用256兆
  restartPolicy: OnFailure # 配置重启策略
```

```sh
# 404
#  Warning  Unhealthy  1s    kubelet            Readiness probe failed: HTTP probe failed with statuscode: 404
kubectl cp index1.html nginx-liveness-demo:/usr/share/nginx/html

# node状态转为ready
```

![image-20240624205645684](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412242119552.png)

##### 类型

1. StartupProbe

   k8s 1.16 版本新增的探针，用于判断应用程序是否已经启动了。

   当配置了 startupProbe 后，会先禁用其他探针，直到 startupProbe 成功后，其他探针才会继续。

   作用：由于有时候不能准确预估应用一定是多长时间启动成功，因此配置另外两种方式不方便配置初始化时长来检测，而配置了 statupProbe 后，只有在应用启动成功了，才会执行另外两种探针，可以更加方便的结合使用另外两种探针使用。

   ```yaml
   startupProbe:
    httpGet:
     path: /api/startup
     port: 80
   ```

   

2. LivenessProbe

   用于探测容器中的应用是否运行，如果探测失败，kubelet 会根据配置的重启策略进行重启，若没有配置，默认就认为容器启动成功，不会执行重启策略。

   ```yaml
   livenessProbe:
    failureThreshold: 5
    httpGet:
     path: /health
     port: 8080
     scheme: HTTP
    initialDelaySeconds: 60
    periodSeconds: 10
    successThreshold: 1
    timeoutSeconds: 5
   ```

   

3. ReadinessProbe

   用于探测容器内的程序是否健康，它的返回值如果返回 success，那么就认为该容器已经完全启动，并且该容器是可以接收外部流量的。

   ```yaml
   readinessProbe:
    failureThreshold: 3 # 错误次数
    httpGet:
     path: /ready
     port: 8181
     scheme: HTTP
    periodSeconds: 10 # 间隔时间
    successThreshold: 1
    timeoutSeconds: 1
   ```

   

##### 探测方式

1. ExecAction

   在容器内部执行一个命令，如果返回值为 0，则任务容器时健康的。

   ```yaml
   livenessProbe:
    exec:
     command:
      \- cat
      \- /health
   ```

   

2. TCPSocketAction

   通过 tcp 连接监测容器内端口是否开放，如果开放则证明该容器健康

   ```yaml
   livenessProbe:
    tcpSocket:
     port: 80
   ```

   

3. HTTPGetAction

   生产环境用的较多的方式，发送 HTTP 请求到容器内的应用程序，如果接口返回的状态码在 200~400 之间，则认为容器健康。

   ```yaml
   livenessProbe:
    failureThreshold: 5
    httpGet:
     path: /health
     port: 8080
     scheme: HTTP
     httpHeaders:
      \- name: xxx
       value: xxx
   ```

   



##### 参数配置

```yaml
initialDelaySeconds: 60 # 初始化时间
timeoutSeconds: 2 # 超时时间
periodSeconds: 5 # 监测间隔时间
successThreshold: 1 # 检查 1 次成功就表示成功
failureThreshold: 2 # 监测失败 2 次就表示失败
```

#### 生命周期

![image-20241215130746124](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412222125032.png)

>修改nginx-demo.yaml

```yaml
apiVersion: v1 # api文档版本
kind: Pod # 资源对象类型
metadata: # Pod相关的元数据，用于描述Pod
  name: nginx-demo # Pod的名称
  labels: # 定义Pod的标签
    type: myFirstApp # 以下全部都是自定义标签
    version: 0.0.1
  namespace: 'default' # 命名空间
spec: # 期望Pod根据这里的描述进行创建
  containers: # 对于Pod中容器的描述
  - name: nginx # 容器名称
    image: nginx:1.7.9 # 指定容器的镜像
    imagePullPolicy: IfNotPresent # 指定镜像的拉取策略
    lifecycle: # 生命周期的配置
      postStart: # 生命周期启动阶段做的事情，但是不一定在容器command之前运行
        exec:
          command:
          - sh
          - -c
          - "echo 'this is postStart' > /usr/share/nginx/html/lifecycle.html"
      preStop:
        exec:
          command:
          - sh
          - -c
          - "sleep 50;echo 'this is preStop' >> /usr/share/nginx/html/lifecycle.html"
    command: # 指定容器启动时，执行的命令
    - nginx
    - -g
    - 'daemon off;' # nginx -g 'daemon off;'
    workingDir: /usr/share/nginx/html # 指定工作目录
    ports: 
    - name: http # 端口名称
      containerPort: 80 # 容器内暴露的端口
      protocol: TCP # 描述该端口是基于什么协议进行通讯的
    env: # 环境变量
    - name: JVM_OPTS # 环境变量名称
      value: '-Xms128m -Xmx128m' # 环境变量值
  
    resources: # 配置资源
      requests: # 最少需要多少资源
        cpu: 100m # 1000m等于一个cpu核心，最少使用0.1个核心
        memory: 128Mi # 限制内存最少使用128兆
      limits: # 最多使用多少资源
        cpu: 200m # 限制cpu最多使用0.2个核心
        memory: 256Mi # 限制内存最多使用256兆
  restartPolicy: OnFailure # 配置重启策略
```



```sh
# 注意preStop处的命令，有一个sleep 50
# 获取详细信息(ip地址)
kubectl get po -o wide

curl 10.244.36.70:80/lifecycle.html
# this is postStart

# 再开一个窗口，持续监听po状态 kubectl get po -w
# -w 代表持续监听pod状态
# 删除pod并查看删除的时间
# time代表打印执行命令消耗的时间
time kubectl delete po pod_name

# 此时发现并没有在50s时删除(因为我们`preStop`里定义的是sleep时间是50s)
# 是因为存在一个删除默认时间配置，默认是`30s`(terminationGracePeriodSeconds)

# 打印结果如下
NAME         READY   STATUS    RESTARTS   AGE
nginx-demo   1/1     Running   0          14s
nginx-demo   1/1     Terminating   0          15s
nginx-demo   1/1     Terminating   0          45s
nginx-demo   0/1     Terminating   0          45s
nginx-demo   0/1     Terminating   0          45s
nginx-demo   0/1     Terminating   0          45s

# 从第一个Terminating到最后，消耗了30s，并不是50s
# 因为k8s 默认给 pod 的停止宽限时间为 30s，配置参数为terminationGracePeriodSeconds: 30
```

![image-20241215171021871](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412222125697.png)

```yaml
apiVersion: v1 # api文档版本
kind: Pod # 资源对象类型
metadata: # Pod相关的元数据，用于描述Pod
  name: nginx-po # Pod的名称
  labels: # 定义Pod的标签
    type: myFirstApp # 以下全部都是自定义标签
    version: 0.0.1
  namespace: 'default' # 命名空间
spec: # 期望Pod根据这里的描述进行创建
  terminationGracePeriodSeconds: 50 # 当pod被删除时，给这个pod宽限多长时间
  containers: # 对于Pod中容器的描述
  - name: nginx # 容器名称
    image: nginx:1.7.9 # 指定容器的镜像
    imagePullPolicy: IfNotPresent # 指定镜像的拉取策略
    lifecycle: # 生命周期的配置
      postStart: # 生命周期启动阶段做的事，不一定在command之前执行
        exec:
          command:
            - sh
            - -c
            - "echo 'hello' > /usr/share/nginx/html/prestop.html"
      preStop:
        exec:
          command:
            - sh
            - -c
            - "sleep 50;echo 'sleep..' >> /usr/share/nginx/html/prestop.html"
    command: # 指定容器启动时，执行的命令
    - nginx
    - -g
    - 'daemon off;' # nginx -g 'daemon off;'
    workingDir: /usr/share/nginx/html # 指定工作目录
    ports:
    - name: http # 端口名称
      containerPort: 80 # 容器内暴露的端口
      protocol: TCP # 描述该端口是基于什么协议进行通讯的
    env: # 环境变量
    - name: JVM_OPTS # 环境变量名称
      value: '-Xms128m -Xmx128m' # 环境变量值

    resources: # 配置资源
      requests: # 最少需要多少资源
        cpu: 100m # 1000m等于一个cpu核心，最少使用0.1个核心
        memory: 128Mi # 限制内存最少使用128兆
      limits: # 最多使用多少资源
        cpu: 200m # 限制cpu最多使用0.2个核心
        memory: 256Mi # 限制内存最多使用256兆
  restartPolicy: OnFailure # 配置重启策略
```

发现这次就是`50s`了

##### Pod 退出流程

###### 删除操作

1. Endpoint 删除 pod 的 ip 地址

2. Pod 变成 Terminating 状态

   变为删除中的状态后，会给 pod 一个宽限期，让 pod 去执行一些清理或销毁操作。

   配置参数：

   ```yaml
   # 作用与 pod 中的所有容器
   terminationGracePeriodSeconds: 30
   containers:
    \- xxx
   ```

   

3. 执行 preStop 的指令

##### PreStop 的应用

如果应用销毁操作耗时需要比较长，可以在 preStop 按照如下方式进行配置

```yaml
preStop:
 exec:
  command:
   \- sh
   \- -c
   \- 'sleep 20; kill pgrep java'
```

但是需要注意，由于 k8s 默认给 pod 的停止宽限时间为 30s，如果我们停止操作会超过 30s 时，不要光设置 sleep 50，还要将 terminationGracePeriodSeconds: 30 也更新成更长的时间，否则 k8s 最多只会在这个时间的基础上再宽限几秒，不会真正等待 50s

1. 注册中心下线
2. 数据清理
3. 数据销毁

### 资源调度

#### Label 和 Selector

##### 标签（Label）

`deploy`和`rs`之间的绑定关系，如果采用mysql表中`一对多`或`多对多`就会形成`强绑定`关系，k8s并没有采用这种方式

###### 配置文件

在各类资源的 metadata.labels 中进行配置

###### kubectl

1. 临时创建 label

   ```sh
   kubectl label po <资源名称> app=hello
   
   # 指定命名空间
   kubectl label po <资源名称> app=hello -n kube-public
   ```

   

2. 修改已经存在的标签

   ```sh
   kubectl label po <资源名称> app=hello2 --overwrite
   ```

   

3. 查看 label

   ```sh
   # selector 按照 label 单值查找节点
   kubectl get po -A -l app=hello
   
   # 查看所有节点的 labels
   kubectl get po --show-labels
   ```

4. 修改配置文件的形式修改lable

   ```sh
   kubectl edit po nginx-demo
   ```

如果希望给`deploy`打上标签有两种方式

1. 修改对应的yaml配置文件(spec.meta.data.labels)
2. 使用命令打标签
   1. 创建临时label(kubectl label po pod_name app=hello)
   2. 修改已经存在的label(kubectl label po pod_name app=hello1 --overwrite)
   3. 查看label(kubectl get po [pod_name] --show-labels)

```sh
[root@k8s-master pods]# kubectl create -f nginx-po.yaml 
pod/nginx-po created
[root@k8s-master pods]# kubectl get po
NAME       READY   STATUS    RESTARTS   AGE
nginx-po   1/1     Running   0          2m5s
[root@k8s-master pods]# kubectl get po --show-labels
NAME       READY   STATUS    RESTARTS   AGE     LABELS
nginx-po   1/1     Running   0          2m32s   type=myFirstApp,version=0.0.1
[root@k8s-master pods]# kubectl get po nginx-po --show-labels
NAME       READY   STATUS    RESTARTS   AGE     LABELS
nginx-po   1/1     Running   0          2m40s   type=myFirstApp,version=0.0.1
[root@k8s-master pods]# 
```

![image-20241215192454549](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412222125573.png)

能查看到标签是因为在原来的yaml里我们已经配置了一部分label

> 除此之外，如果希望在已经存在的po上`新建/修改`label可以使用如下命令

```sh
[root@k8s-master pods]# kubectl get po nginx-po --show-labels
NAME       READY   STATUS    RESTARTS   AGE   LABELS
nginx-po   1/1     Running   0          17m   type=app1,version=0.0.1
[root@k8s-master pods]# kubectl edit po nginx-po
pod/nginx-po edited
[root@k8s-master pods]# kubectl get po nginx-po --show-labels
NAME       READY   STATUS    RESTARTS   AGE   LABELS
nginx-po   1/1     Running   0          25m   type=app2,version=0.0.1
```

##### 选择器（Selector）

>在各对象的配置 spec.selector 或其他可以写 selector 的属性中编写

###### 配置文件

在各对象的配置 spec.selector 或其他可以写 selector 的属性中编写

###### kubectl

```sh
# -l表示根据label进行匹配,匹配单个值，查找 =app2 的 pod
[root@k8s-master pods]# kubectl get po -l type=app2
NAME       READY   STATUS    RESTARTS   AGE
nginx-po   1/1     Running   0          27m
# -A显示命名空间
[root@k8s-master pods]# kubectl get po -A -l type=app2
NAMESPACE   NAME       READY   STATUS    RESTARTS   AGE
default     nginx-po   1/1     Running   0          29m
# 同时使用--show-labels
[root@k8s-master pods]# kubectl get po -A -l type=app2 --show-labels
NAMESPACE   NAME       READY   STATUS    RESTARTS   AGE   LABELS
default     nginx-po   1/1     Running   0          85m   type=app2,version=0.0.1
# in
[root@k8s-master pods]# kubectl get po -l 'version in (0.0.1, 0,0.2, 0.0.3)'
NAME       READY   STATUS    RESTARTS   AGE
nginx-po   1/1     Running   0          88m
# 同时匹配多个label
[root@k8s-master pods]# kubectl get po -l version!=0.0.2,type=app2
NAME       READY   STATUS    RESTARTS   AGE
nginx-po   1/1     Running   0          89m

# 查找 version!=1 and app=nginx 的 pod 信息
kubectl get po -l version!=1,app=nginx

# 不等值 + 语句
kubectl get po -A -l version!=1,'app in (busybox, nginx)'
```

#### Deployment

##### 功能

###### 创建

```sh
kubectl delete po nginx-po
cd ../
mkdir deployments
cd deployments
# 创建deploy
[root@k8s-master deployments]# kubectl create deploy nginx-deploy --image=nginx:latest
deployment.apps/nginx-deploy created
# 或执行 kubectl create -f xxx.yaml --record
# --record 会在 annotation 中记录当前命令创建或升级了资源，后续可以查看做过哪些变动操作。

# 查看状态
[root@k8s-master deployments]# kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   1/1     1            1           72s
# 如果没有ready，查看下pod，看看对应pod是否启动成功
[root@k8s-master deployments]# kubectl get po
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-65bdccd7c7-rj8xw   1/1     Running   0          70s
# 查看rs
[root@k8s-master deployments]# kubectl get replicaset
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-65bdccd7c7   1         1         1       4m56s
# 也可以使用缩写
[root@k8s-master deployments]# kubectl get rs
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-65bdccd7c7   1         1         1       5m19s
# 查看 pod 以及展示标签，可以看到是关联的那个 rs
[root@k8s-master deployments]# kubectl get pods --show-labels
```

仔细观察你可能会发现如下特点，他们的名字逐步增加了id

```sh
# deploy
nginx-deploy
# replicset
nginx-deploy-65bdccd7c7
# pod
nginx-deploy-65bdccd7c7-rj8xw
```

> 如果希望自己通过配置文件来创建deploy，而不是直接通过镜像来创建

```sh
[root@k8s-master deployments]# kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   1/1     1            1           8m6s
# 以yaml格式输出`nginx-deploy`的配置
[root@k8s-master deployments]# kubectl get deploy nginx-deploy -o yaml
```

```yaml
piVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy
  name: nginx-deploy
  namespace: default
spec:
  replicas: 1 # 期望的副本数
  revisionHistoryLimit: 10 # 进行滚动更新后，保留的历史版本数
  selector:
    matchLabels:
      app: nginx-deploy # 按照标签匹配rs、po
  strategy:
    rollingUpdate: # 滚动更新策略
      maxSurge: 25% # 进行滚动更新时，更新个数最多可以超过期望副本数的个数/比例
      maxUnavailable: 25% # 更新时，最大不可用更新比例，表示所有副本中，有多少个/比例更新不成功
    type: RollingUpdate
  template: # pod模板
    metadata:
      labels:
        app: nginx-deploy
    spec: # 元信息
      containers:
      - image: nginx:latest
        imagePullPolicy: IfNotPresent
        name: nginx # 容器名称
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
```

`deploy`、`rs`、`pod`通过`app=nginx-deploy`的**`label`**进行关联

> 修改已经创建好的deploy

```sh
kubectl edit deploy nginx-deploy
```

###### 滚动更新

```sh
# 只有修改了 deployment 配置文件中的 template 中的属性后，才会触发更新操作

# 修改 nginx 版本号
kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1

# 或者通过 kubectl edit deployment/nginx-deployment 进行修改

# 查看滚动更新的过程
kubectl rollout status deploy <deployment_name>

# 查看部署描述，最后展示发生的事件列表也可以看到滚动更新过程
kubectl describe deploy <deployment_name>

# 通过 kubectl get deployments 获取部署信息，UP-TO-DATE 表示已经有多少副本达到了配置中要求的数目

# 通过 kubectl get rs 可以看到增加了一个新的 rs

# 通过 kubectl get pods 可以看到所有 pod 关联的 rs 变成了新的
```

> 还可以通过修改yaml的`replicas`来增加副本数

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2024-12-15T13:37:52Z"
  generation: 4
  labels:
    app: nginx-deploy
    test: "123"
  name: nginx-deploy
  namespace: default
  resourceVersion: "110810"
  uid: 55fcd95d-2217-4c6e-91c4-e112e88edb2e
spec:
  progressDeadlineSeconds: 600
  replicas: 3 # 副本数
...
```

```sh
[root@k8s-master k8s]# kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   3/3     3            3           23h
```

> 在修改template的时候就会触发滚动更新
>
> 比如这里nginx的版本是`latest`，修改image的版本就会触发滚动更新

```yaml
...
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-deploy
    spec:
      containers:
      - image: nginx:latest
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```

查看滚动更新的过程
`kubectl rollout status deploy <deployment_name>`

查看部署描述，最后展示发生的事件列表也可以看到滚动更新过程
`kubectl describe deploy <deployment_name>`

通过 kubectl get deployments 获取部署信息，UP-TO-DATE 表示已经有多少副本达到了配置中要求的数目

通过 kubectl get rs 可以看到增加了一个新的 rs

通过 kubectl get pods 可以看到所有 pod 关联的 rs 变成了新的

> 假设当前有 5 个 nginx:1.7.9 版本，你想将版本更新为 1.9.1，当更新成功第三个以后，你马上又将期望更新的版本改为 1.9.2，那么此时会立马删除之前的三个，并且立马开启更新 1.9.2 的任务(k8s判断之前的更新没有意义，所以会从初始版本开始更新，忽略掉上一次没有意义的更新)
>
> 如下，我将image版本从`latest`更新为`1.7.9`

```sh
[root@k8s-master k8s]# kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   3/3     3            3           23h
[root@k8s-master k8s]# kubectl describe deploy nginx-deploy
Name:                   nginx-deploy
Namespace:              default
CreationTimestamp:      Sun, 15 Dec 2024 21:37:52 +0800
Labels:                 app=nginx-deploy
                        test=123
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=nginx-deploy
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx-deploy
  Containers:
   nginx:
    Image:        nginx:1.7.9
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deploy-65976b546f (3/3 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  23h    deployment-controller  Scaled up replica set nginx-deploy-65bdccd7c7 to 1
  Normal  ScalingReplicaSet  20m    deployment-controller  Scaled up replica set nginx-deploy-65bdccd7c7 to 3
  Normal  ScalingReplicaSet  18m    deployment-controller  Scaled down replica set nginx-deploy-65bdccd7c7 to 1
  Normal  ScalingReplicaSet  12m    deployment-controller  Scaled up replica set nginx-deploy-65bdccd7c7 to 3
  Normal  ScalingReplicaSet  2m16s  deployment-controller  Scaled up replica set nginx-deploy-65976b546f to 1
  Normal  ScalingReplicaSet  115s   deployment-controller  Scaled down replica set nginx-deploy-65bdccd7c7 to 2
  Normal  ScalingReplicaSet  115s   deployment-controller  Scaled up replica set nginx-deploy-65976b546f to 2
  Normal  ScalingReplicaSet  94s    deployment-controller  Scaled down replica set nginx-deploy-65bdccd7c7 to 1
  Normal  ScalingReplicaSet  94s    deployment-controller  Scaled up replica set nginx-deploy-65976b546f to 3
  Normal  ScalingReplicaSet  73s    deployment-controller  Scaled down replica set nginx-deploy-65bdccd7c7 to 0
```

###### 多个滚动更新并行

假设当前有 5 个 nginx:1.7.9 版本，你想将版本更新为 1.9.1，当更新成功第三个以后，你马上又将期望更新的版本改为 1.9.2，那么此时会立马删除之前的三个，并且立马开启更新 1.9.2 的任务

###### 回滚

有时候你可能想回退一个Deployment，例如，当Deployment不稳定时，比如一直crash looping。

默认情况下，kubernetes会在系统中保存前两次的Deployment的rollout历史记录，以便你可以随时会退（你可以修改revision history limit来更改保存的revision数）。

```sh
# 案例：
# 更新 deployment 时参数不小心写错，如 nginx:1.9.1 写成了 nginx:1.91
kubectl set image deployment/nginx-deploy nginx=nginx:1.91

# 监控滚动升级状态，由于镜像名称错误，下载镜像失败，因此更新过程会卡住
kubectl rollout status deployments nginx-deploy

# 结束监听后，获取 rs 信息，我们可以看到新增的 rs 副本数是 2 个
kubectl get rs

# 通过 kubectl get pods 获取 pods 信息，我们可以看到关联到新的 rs 的 pod，状态处于 ImagePullBackOff 状态

# 为了修复这个问题，我们需要找到需要回退的 revision 进行回退
# 通过 以下命令 可以获取 revison 的列表
kubectl rollout history deployment/nginx-deploy
# deployment.apps/nginx-deploy 
# REVISION  CHANGE-CAUSE
# 1         <none>
# 2         <none>
# 这里是none的原因是因为更新的时候没有写明原因，比如
# kubectl set image deployment/nginx-deploy nginx=nginx:1.7.9 --record
# 回退到上一个版本，即1，先查看一下该版本的信息

# 通过 以下命令 可以查看指定版本的详细信息
kubectl rollout history deployment/nginx-deploy --revision=1

# 确认要回退的版本后，可以通过 以下命令 可以回退到上一个版本
kubectl rollout undo deployment/nginx-deploy

# 也可以回退到指定的 revision
kubectl rollout undo deployment/nginx-deploy --to-revision=1

# 再次通过 以下命令 可以看到，我们的版本已经回退到对应的 revison 上了
kubectl get deployment
kubectl describe deployment

# 我们可以回退版本是因为在yaml里设置了`revisionHistoryLimit`
# 可以通过设置 .spec.revisonHistoryLimit 来指定 deployment 保留多少 revison，如果设置为 0，则不允许 deployment 回退了。
```

###### 扩容缩容

通过 kube scale 命令可以进行自动扩容/缩容，以及通过 kube edit 编辑 replcas 也可以实现扩容/缩容

扩容与缩容只是直接创建副本数，没有更新 pod template 因此不会创建新的 rs

```sh
kubectl scale --replicas=6 deployment nginx-deploy

# 查看帮助文档
kubectl scale --help
```



###### 暂停与恢复

由于每次对 pod template 中的信息发生修改后，都会触发更新 deployment 操作，那么此时如果频繁修改信息，就会产生多次更新，而实际上只需要执行最后一次更新即可，当出现此类情况时我们就可以暂停 deployment 的 rollout

```sh
# 通过 kubectl rollout pause deployment <name> 就可以实现暂停，直到你下次恢复后才会继续进行滚动更新
kubectl rollout pause deployment nginx-deploy

# 尝试对容器进行修改，然后查看是否发生更新操作了
kubectl set image deploy <name> nginx=nginx:1.17.9
kubectl get po 
```

通过以上操作可以看到实际并没有发生修改，此时我们再次进行修改一些属性，如限制 nginx 容器的最大cpu为 0.2 核，最大内存为 128M，最小内存为 64M，最小 cpu 为 0.1 核

```sh
kubectl set resources deploy <deploy_name> -c <container_name> --limits=cpu=200m,memory=128Mi --requests=cpu100m,memory=64Mi

# 通过格式化输出 kubectl get deploy <name> -oyaml，可以看到配置确实发生了修改，再通过 kubectl get po 可以看到 pod 没有被更新

# 那么此时我们再恢复 rollout，通过命令 kubectl rollout resume deploy <name>

kubectl rollout resume deploy nginx-deploy
# 恢复后，我们再次查看 rs 和 po 信息，我们可以看到就开始进行滚动更新操作了
kubectl get rs
kubectl get po

# 暂停
kubectl rollout pause deploy <name>

# 恢复
kubectl rollout resume deploy <name>
```



##### 配置文件

```yaml
apiVersion: apps/v1 # deployment api 版本
kind: Deployment # 资源类型为 deployment
metadata: # 元信息
  labels: # 标签
    app: nginx-deploy # 具体的 key: value 配置形式
  name: nginx-deploy # deployment 的名字
  namespace: default # 所在的命名空间
spec:
  replicas: 1 # 期望副本数
  revisionHistoryLimit: 10 # 进行滚动更新后，保留的历史版本数
  selector: # 选择器，用于找到匹配的 RS
    matchLabels: # 按照标签匹配
      app: nginx-deploy # 匹配的标签key/value
  strategy: # 更新策略
    rollingUpdate: # 滚动更新配置
      maxSurge: 25% # 进行滚动更新时，更新的个数最多可以超过期望副本数的个数/比例
      maxUnavailable: 25% # 进行滚动更新时，最大不可用比例更新比例，表示在所有副本数中，最多可以有多少个不更新成功
    type: RollingUpdate # 更新类型，采用滚动更新
  template: # pod 模板
    metadata: # pod 的元信息
      labels: # pod 的标签
        app: nginx-deploy
    spec: # pod 期望信息
      containers: # pod 的容器
      - image: nginx:1.7.9 # 镜像
        imagePullPolicy: IfNotPresent # 拉取策略
        name: nginx # 容器名称
      restartPolicy: Always # 重启策略
      terminationGracePeriodSeconds: 30 # 删除操作最多宽限多长时间
```

#### StatefulSet

>以上是关于无状态应用的内容，增加删除简单执行命令即可
>但是有状态应用却不一样，可能会依赖本地的文件、网络

##### 功能

###### 创建

```sh
kubectl create -f web.yaml

# 查看 service 和 statefulset => sts
kubectl get service nginx
kubectl get statefulset web

# 查看service(简写)
kubectl get svc
# 查看statefulset(简写)
kubectl get sts

# 查看 PVC 信息
kubectl get pvc

# 查看创建的 pod，这些 pod 是有序的
kubectl get pods -l app=nginx

# 查看这些 pod 的 dns
# 运行一个 pod，基础镜像为 busybox 工具包，利用里面的 nslookup 可以看到 dns 信息
kubectl run -i --tty --image busybox dns-test --restart=Never --rm /bin/sh
nslookup web-0.nginx

# 创建sts
kubectl create -f web.yaml 
# service/nginx created
# statefulset.apps/web created

# 再次查看情况
# 这里没有绑定ip，所以ip显示none
kubectl get svc
# NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
# kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   69d
# nginx        ClusterIP   None         <none>        80/TCP    47s

kubectl get sts
# NAME   READY   AGE
# web    0/2     51s
# 查看持久卷(数据卷)
kubectl get pvc
# NAME        STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
# www-web-0   Pending                                                     2m4s
# 查看详细信息，发现有问题
kubectl describe pvc www-web-0
# Name:          www-web-0
# Namespace:     default
# StorageClass:  
# Status:        Pending
# Volume:        
# Labels:        app=nginx
# Annotations:   volume.alpha.kubernetes.io/storage-class: anything
# Finalizers:    [kubernetes.io/pvc-protection]
# Capacity:      
# Access Modes:  
# VolumeMode:    Filesystem
# Used By:       web-0
# Events:
#   Type    Reason         Age                  From                         Message
#   ----    ------         ----                 ----                         -------
#   Normal  FailedBinding  8s (x15 over 3m34s)  persistentvolume-controller  no persistent volumes available for this claim and no storage class is set

# 由于没有可用的持久卷而且没有设置存储类，所以sts依旧没有ready
kubectl get sts
# NAME   READY   AGE
# web    0/2     4m18s
```

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
      annotations:
        volume.alpha.kubernetes.io/storage-class: anything
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

> 由于存储卷的问题，现在修改一下yaml，暂时不添加存储卷

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
          name: web
```

```sh
# 直接替换掉sts的yaml
kubectl replace -f web.yaml sts web
# service/nginx replaced
# The StatefulSet "web" is invalid: spec: Forbidden: updates to statefulset spec for fields other than 'replicas', 'template', 'updateStrategy', 'persistentVolumeClaimRetentionPolicy' and 'minReadySeconds' are forbidden

# 这个提示的原因是有些参数是不能替换的，所以还是删除重建吧
kubectl delete sts web
# statefulset.apps "web" deleted

kubectl delete svc nginx
# service "nginx" deleted

kubectl delete pvc www-web-0
# persistentvolumeclaim "www-web-0" deleted

kubectl create -f web.yaml 
# service/nginx created
# statefulset.apps/web created

# 查看信息
kubectl get sts
# NAME   READY   AGE
# web    2/2     39s

kubectl get svc
# NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
# kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   69d
# nginx        ClusterIP   None         <none>        80/TCP    42s

kubectl get pvc
# No resources found in default namespace.

# 查看po
kubectl get po
# NAME    READY   STATUS    RESTARTS   AGE
# web-0   1/1     Running   0          65s
# web-1   1/1     Running   0          58s

# 新建一个po，去ping这两个po，观察是否可以ping通
# busybox是linux里的一个工具镜像
kubectl run -it --image busybox:1.28.4 dns-test /bin/sh

kubectl get po
# NAME       READY   STATUS    RESTARTS   AGE
# dns-test   1/1     Running   0          77s
# web-0      1/1     Running   1          13h
# web-1      1/1     Running   1          13h

# 我这里执行完命令之后没有自动进入po内部，查看发现已经ready了
# 执行以下命令进入po内部
kubectl exec -it dns-test -- /bin/sh
#statefulset中每个pod的dns格式为statefulsetName-{0,1...n}.serviceName.namespace.svc.cluster.local
/ # nslookup web-0.nginx
# Server:    10.96.0.10
# Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

# Name:      web-0.nginx
# Address 1: 10.244.36.74 web-0.nginx.default.svc.cluster.local
```

###### 扩容缩容

```sh
# 扩容
kubectl scale statefulset web --replicas=5
# 或者如下
kubectl patch statefulset web -p '{"spec":{"replicas":5}}'

# 缩容
kubectl patch statefulset web -p '{"spec":{"replicas":3}}'
```

###### 镜像更新

```sh
# 镜像更新（目前不支持直接使用`edit`命令以及参数`-o yaml`来修改yaml里的image，需要 patch 来间接实现）

kubectl patch sts web --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"nginx:1.9.1"}]'

# 使用如下
# web是sts的名字，类型是json数据，操作是replace，将/spec/.../image替换成nginx:1.9.1
kubectl patch sts web --type='json' -p='[{"op":"replace","path":"/spec/template/spec/containers/0/image","value":"nginx:1.9.1"}]'

# 使用
[root@k8s-master ~]# kubectl patch sts web --type='json' -p='[{"op":"replace","path":"/spec/template/spec/containers/0/image","value":"nginx:1.9.1"}]'
statefulset.apps/web patched
[root@k8s-master ~]# kubectl get sts 
NAME   READY   AGE
web    1/2     24h
[root@k8s-master ~]# kubectl get po
NAME       READY   STATUS              RESTARTS   AGE
dns-test   1/1     Running             0          10h
web-0      0/1     ContainerCreating   0          40s
web-1      1/1     Running             0          55s
[root@k8s-master ~]# kubectl describe po web-0
...
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  44s   default-scheduler  Successfully assigned default/web-0 to k8s-node1
  Normal  Pulling    43s   kubelet            Pulling image "nginx:1.9.1"
# 等待pull完成
[root@k8s-master ~]# kubectl get po
NAME       READY   STATUS    RESTARTS   AGE
dns-test   1/1     Running   0          10h
web-0      1/1     Running   0          2m7s
web-1      1/1     Running   0          2m22s
[root@k8s-master ~]# kubectl get sts
NAME   READY   AGE
web    2/2     24h
# 查看历史版本
# 可以明显看到镜像从latest更新到1.9.1了
[root@k8s-master k8s]# kubectl rollout history sts
statefulset.apps/web 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

[root@k8s-master k8s]# kubectl rollout history sts --revision=2
statefulset.apps/web with revision #2
Pod Template:
  Labels:	app=nginx
  Containers:
   nginx:
    Image:	nginx:1.9.1
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>

[root@k8s-master k8s]# kubectl rollout history sts --revision=1
statefulset.apps/web with revision #1
Pod Template:
  Labels:	app=nginx
  Containers:
   nginx:
    Image:	nginx:latest
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
# 查看sts内容，先删除web-1，再创建web-1，web-0也是如此
[root@k8s-master k8s]# kubectl describe sts web
...
  Type    Reason            Age                  From                    Message
  ----    ------            ----                 ----                    -------
  Normal  SuccessfulCreate  9h                   statefulset-controller  create Pod web-2 in StatefulSet web successful
  Normal  SuccessfulCreate  9h                   statefulset-controller  create Pod web-3 in StatefulSet web successful
  Normal  SuccessfulCreate  9h                   statefulset-controller  create Pod web-4 in StatefulSet web successful
  Normal  SuccessfulDelete  9h                   statefulset-controller  delete Pod web-4 in StatefulSet web successful
  Normal  SuccessfulDelete  9h                   statefulset-controller  delete Pod web-3 in StatefulSet web successful
  Normal  SuccessfulDelete  9h                   statefulset-controller  delete Pod web-2 in StatefulSet web successful
  Normal  SuccessfulDelete  6m38s                statefulset-controller  delete Pod web-1 in StatefulSet web successful
  Normal  SuccessfulCreate  6m37s (x2 over 24h)  statefulset-controller  create Pod web-1 in StatefulSet web successful
  Normal  SuccessfulDelete  6m23s                statefulset-controller  delete Pod web-0 in StatefulSet web successful
  Normal  SuccessfulCreate  6m22s (x2 over 24h)  statefulset-controller  create Pod web-0 in StatefulSet web successful
```

> RollingUpdate

StatefulSet 也可以采用滚动更新策略，同样是修改 pod template 属性后会触发更新，但是由于 pod 是有序的，在 StatefulSet 中更新时是基于 pod 的顺序倒序更新的

> 灰度发布

利用滚动更新中的 partition 属性，可以实现简易的灰度发布的效果

例如我们有 5 个 pod，如果当前 partition 设置为 3，那么此时滚动更新时，只会更新那些 序号 >= 3 的 pod

利用该机制，我们可以通过控制 partition 的值，来决定只更新其中一部分 pod，确认没有问题后再主键增大更新的 pod 数量，最终实现全部 pod 更新

```sh
# 实现灰度发布(金丝雀发布)

# 扩容
kubectl scale sts web --replicas 6

# 更新如下内容
kubectl edit sts web
...
  updateStrategy:
    rollingUpdate:
      partition: 3 # 改为3
    type: RollingUpdate

[root@k8s-master k8s]# kubectl get po
NAME       READY   STATUS    RESTARTS   AGE
dns-test   1/1     Running   0          47h
web-0      1/1     Running   0          37h
web-1      1/1     Running   0          37h
web-2      1/1     Running   0          12h
web-3      1/1     Running   0          12h
web-4      1/1     Running   0          12h
web-5      1/1     Running   0          12h

# 如果sts发生更新，优先更新web-3、web-4、web-5(优先更新>=3的po)
# 此时更新sts的image版本，查看po的内容
# image从1.9.1更新为latest
[root@k8s-master k8s]# kubectl describe po web-3
Name:         web-3
Namespace:    default
Priority:     0
Node:         k8s-node1/192.168.42.132
Start Time:   Thu, 19 Dec 2024 21:29:18 +0800
Labels:       app=nginx
              controller-revision-hash=web-7974cc466
              statefulset.kubernetes.io/pod-name=web-3
Annotations:  cni.projectcalico.org/containerID: 0c73e705476bd47bae66ff1b3e3763705fb433000aaf7411ad30cf4dd358bfb9
              cni.projectcalico.org/podIP: 10.244.36.70/32
              cni.projectcalico.org/podIPs: 10.244.36.70/32
Status:       Running
IP:           10.244.36.70
IPs:
  IP:           10.244.36.70
Controlled By:  StatefulSet/web
Containers:
  nginx:
    Container ID:   docker://873c777e9938672b070e104e0d9453d4c8caa16761c327cdaadb45a3aa078fa9
    Image:          nginx:latest # 可以看到web-3已经更新为latest版本了
...
[root@k8s-master k8s]# kubectl describe po web-2
Name:         web-2
Namespace:    default
Priority:     0
Node:         k8s-node2/192.168.42.133
Start Time:   Thu, 19 Dec 2024 08:30:21 +0800
Labels:       app=nginx
              controller-revision-hash=web-567b56dd
              statefulset.kubernetes.io/pod-name=web-2
Annotations:  cni.projectcalico.org/containerID: 737d7c5bb252479092322bfa6edceb42b6538b29e6615402f68a6767f891de85
              cni.projectcalico.org/podIP: 10.244.169.131/32
              cni.projectcalico.org/podIPs: 10.244.169.131/32
Status:       Running
IP:           10.244.169.131
IPs:
  IP:           10.244.169.131
Controlled By:  StatefulSet/web
Containers:
  nginx:
    Container ID:   docker://d9efaad9065e835a888499244edecd5e83b66513905ea08d3908c893782f1049
    Image:          nginx:1.9.1 # 但是web-2还没有更新
# 最终逐步减小`partition`的值直至为0，这样就实现了灰度发布(金丝雀发布)
```

> OnDelete

只有在 pod 被删除时会进行更新操作

```sh
# 将如下代码进行更新
  updateStrategy:
    rollingUpdate:
      partition: 0
    type: RollingUpdate
...
# 更新为如下代码
  updateStrategy:
    type: OnDelete
# 表示更新策略设置为当删除时才更新
# 此时再修改image的版本从latest更新为1.9.1
# 此时po是不会对版本进行更新的，只有在删除对应po时才会进行更新，删除po之后就会重建po，并更新image
# 这样就能实现指定更新的po，比如我只希望更新web-3，而其他的po不希望进行更新
# 此时删除web-3这个po即可
```

###### 删除

```sh
# 在无状态应用中，删除deploy之后，对应的po也会自动删除
# 而在有状态应用中，删除sts之后，默认也会进行级联删除

# 删除 StatefulSet 和 Headless Service
# 默认级联删除：删除 statefulset 时会同时删除 pods
kubectl delete statefulset web

# 非级联删除：删除 statefulset 时不会删除 pods，删除 sts 后，pods 就没人管了，此时再删除 pod 不会重建的
kubectl deelte sts web --cascade=false

# 删除 service
kubectl delete service nginx
```

###### 删除 pvc

```sh
# StatefulSet删除后PVC还会保留着，数据不再使用的话也需要删除
kubectl delete pvc www-web-0 www-web-1
```

###### 配置文件

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
      annotations:
        volume.alpha.kubernetes.io/storage-class: anything
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

#### DaemonSet

> 为每一个匹配到的Node都部署一个守护进程
>
> 如果有以下的业务场景，以下是微服务的调用链路

![image-20241222212431460](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412222124413.png)

一般会存在第四个节点，部署普罗米修斯(监控)、elasticsearch(存放日志)

加入这种模式出现问题，我们一个个节点调查日志未免太过麻烦，最好有一个东西可以给每个节点的log信息统一写入es就好了

`DaemonSet`正好符合需求，通过`nodeSelector`匹配到对应的`pod`，通过`Fluentd`收集日志并发送到es

![image-20241222212953278](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412222129683.png)

DaemonSet 会忽略 Node 的 unschedulable 状态，有两种方式来指定 Pod 只运行在指定的 Node 节点上：

- nodeSelector：只调度到匹配指定 label 的 Node 上
- nodeAffinity：功能更丰富的 Node 选择器，比如支持集合操作
- podAffinity：调度到满足条件的 Pod 所在的 Node 上

暂时先讲一下`nodeSelector`，其他两种后文再讲

##### 配置文件

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  template:
    metadata:
      labels:
        app: logging
        id: fluentd
      name: fluentd
    spec:
      containers:
      - name: fluentd-es
        image: agilestacks/fluentd-elasticsearch:v1.3.0
        env:
         - name: FLUENTD_ARGS
           value: -qq
        volumeMounts:
         - name: containers
           mountPath: /var/lib/docker/containers
         - name: varlog
           mountPath: /varlog
      volumes:
         - hostPath:
             path: /var/lib/docker/containers
           name: containers
         - hostPath:
             path: /var/log
           name: varlog
```



##### 指定 Node 节点

DaemonSet 会忽略 Node 的 unschedulable 状态，有两种方式来指定 Pod 只运行在指定的 Node 节点上：

- nodeSelector：只调度到匹配指定 label 的 Node 上
- nodeAffinity：功能更丰富的 Node 选择器，比如支持集合操作
- podAffinity：调度到满足条件的 Pod 所在的 Node 上

> nodeSelector

先为 Node 打上标签

```sh
kubectl label nodes k8s-node1 svc_type=microsvc
```

然后再 daemonset 配置中设置 nodeSelector

```yaml
spec:
 template:
  spec:
   nodeSelector:
    svc_type: microsvc
```

> nodeAffinity

nodeAffinity 目前支持两种：`requiredDuringSchedulingIgnoredDuringExecution` 和 `preferredDuringSchedulingIgnoredDuringExecution`，分别代表必须满足条件和优选条件。

比如下面的例子代表调度到包含标签 `wolfcode.cn/framework-name` 并且值为 `spring` 或 `springboot` 的 Node 上，并且优选还带有标签 `another-node-label-key=another-node-label-value` 的Node。



```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: wolfcode.cn/framework-name
            operator: In
            values:
            - spring
            - springboot
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: pauseyyf/pause
```

>podAffinity

podAffinity 基于 Pod 的标签来选择 Node，仅调度到满足条件Pod 所在的 Node 上，支持 podAffinity 和 podAntiAffinity。这个功能比较绕，以下面的例子为例：

- 如果一个 “Node 所在空间中包含至少一个带有 auth=oauth2 标签且运行中的 Pod”，那么可以调度到该 Node
- 不调度到 “包含至少一个带有 auth=jwt 标签且运行中 Pod”的 Node 上

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: auth
            operator: In
            values:
            - oauth2
        topologyKey: failure-domain.beta.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: auth
              operator: In
              values:
              - jwt
          topologyKey: kubernetes.io/hostname
  containers:
  - name: with-pod-affinity
    image: pauseyyf/pause
```

##### 滚动更新

不建议使用 RollingUpdate，建议使用 OnDelete 模式，这样避免频繁更新 ds

#### HPA 自动扩/缩容

通过观察 pod 的 cpu、内存使用率或自定义 metrics 指标进行自动的扩容或缩容 pod 的数量。

通常用于 Deployment，不适用于无法扩/缩容的对象，如 DaemonSet

控制管理器每隔30s（可以通过–horizontal-pod-autoscaler-sync-period修改）查询metrics的资源使用情况

##### 开启指标服务

```sh
# 下载 metrics-server 组件配置文件
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml -O metrics-server-components.yaml

# 修改镜像地址为国内的地址
sed -i 's/k8s.gcr.io\/metrics-server/registry.cn-hangzhou.aliyuncs.com\/google_containers/g' metrics-server-components.yaml

# 修改容器的 tls 配置，不验证 tls，在 containers 的 args 参数中增加 --kubelet-insecure-tls 参数

# 安装组件
kubectl apply -f metrics-server-components.yaml

# 查看 pod 状态
kubectl get pods --all-namespaces | grep metrics
```

##### cpu、内存指标监控

实现 cpu 或内存的监控，首先有个前提条件是该对象必须配置了 resources.requests.cpu 或 resources.requests.memory 才可以，可以配置当 cpu/memory 达到上述配置的百分比后进行扩容或缩容

创建一个 HPA：

1. 先准备一个好一个有做资源限制的 deployment

2. 执行命令 

   ```sh
   kubectl autoscale deploy nginx-deploy --cpu-percent=20 --min=2 --max=5
   ```

   

3. 通过 `kubectl get hpa` 可以获取 HPA 信息


测试：找到对应服务的 service，编写循环测试脚本提升内存与 cpu 负载

```sh
while true; do wget -q -O- http://<ip:port> > /dev/null ; done
```

可以通过多台机器执行上述命令，增加负载，当超过负载后可以查看 pods 的扩容情况 `kubectl get pods`

查看 pods 资源使用情况
`kubectl top pods`

扩容测试完成后，再关闭循环执行的指令，让 cpu 占用率降下来，然后过 5 分钟后查看自动缩容情况

##### 自定义 metrics

- 控制管理器开启–horizontal-pod-autoscaler-use-rest-clients
- 控制管理器的–apiserver指向[API Server Aggregator](https://github.com/kubernetes/kube-aggregator)
- 在API Server Aggregator中注册自定义的metrics API

