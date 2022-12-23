# Kubernetes 组件

当部署完 Kubernetes，即拥有了一个完整的集群。

一个 Kubernetes 集群由一组被称作节点的机器组成。这些节点上运行 Kubernetes 所管理的容器化应用。集群具有至少一个工作节点。

工作节点托管作为应用负载的组件的 Pod。控制平台管理集群中的工作节点和 Pod。为集群提供故障转移和高可用性，这些控制平台一般跨多主机运行，集群跨多个节点运行。

本文档概述了将会正常运行的 Kubernetes 集群所需的各种组件。

这张图表展示了包含所有相互关联组件的 Kubernetes 集群。

![](../../static/components-of-kubernetes.svg)

## 控制平面组件（Control Plane Components）

控制平面的组件对集群做出全局决策（例如：调度），以及检测和响应集群事件（例如：当部署的副本数量不满足时启动新的 pod）。

控制平面组件可以在集群的任何机器上运行。但是为了简单起见，通常会在同一台机器上启动所有控制平面组件，并且不在该机器上运行运行用户容器。

### kube-apiserver

API 服务器是 Kubernetes 控制平面的一个组件，用于公开 Kubernetes API。API 服务器是 Kubernetes 控制平面的前端。

Kubernetes API 服务器的主要实现是 kube-apiserver。kube-apiserver 通过部署更多实例来实现水平扩展，可以运行多个 kube-apiserver 实例来平衡实例间的流量。

### etcd

etcd 是兼具一致性和高可用的键值数据库，可以作为保存 Kubernetes 所有集群数据的后台数据库。

如果 Kubernetes 集群使用 etcd 作为后端存储，请确保其有对应的数据备份方案。

详细内容，请参见 [etcd 官方文档](https://etcd.io/docs/)。

### kube-scheduler

控制平面组件，负责监视新创建的、未指定运行节点（node）的 Pods，选择节点让 Pod 在其上运行。

调度决策考虑的因素包括单个 Pod 和 Pod 集合的资源需求、硬件/软件/策略约束、亲和性和反亲和性规范、数据位置、工作负载间的干扰和最后时限。

### kube-controller-manager

运行控制器进程的控制平面组件。

从逻辑上讲，每个控制器都是一个单独的进程，但是为降低复杂度，它们都被编译在同一个可执行文件，并在一个进程中运行。

这些控制器包括：

* **节点控制器（Node Controller）**：负载在节点出现故障时进行通知和响应
* **任务控制器（Job Controller）**：监测代表一次性任务的 Job 对象，然后创建 Pods 来运行这些任务直至完成
* **端点控制器（Endpoints Controller）**：填充端口（Endpoints）对象（即加入 Service 与 Pod）
* **服务账户和令牌控制器（ServiceAccount && Token Controllers）**：为新的命名空间创建默认账户和 API 访问令牌

## 节点组件（Node Components）

节点组件在每个节点上运行，维护运行的 Pod 并提供 Kubernetes 运行环境。

### kubelet

一个在集群中每个节点上运行的代理。它保证容器都运行在 Pod 中。

kubelet 接收一组通过各类机制提供给它的 PodSpecs，确保这些 PodSpecs 中描述的容器处于运行状态且健康。kubelet 不会管理不是由 Kubernetes 创建的容器。

### kube-proxy

kube-proxy 是集群中每个节点上运行的网络代理，实现了 Kubernetes Service 概念的一部分。

kube-proxy 维护节点上的网络规则。这些网络规则允许从集群内部或外部的网络能与 Pod 进行网络通信。

如果操作系统提供了数据包过滤层并可用的话，kube-proxy 会通过它来实现网络规则。否则，kube-proxy 仅转发流量本身。

### 容器运行时（Container Runtime）

负责运行容器的软件。

Kubernetes 支持多个容器运行时：Docker、containerd、CRI-O，以及任何实现了 Kubernetes CRI 的容器运行时。

## 插件（Addons）

插件使用 Kubernetes 资源（DaemonSet、Deployment等）实现集群功能。因为这些插件提供集群组别的功能，所以这些资源属于 `kube-system` 命名空间。

### DNS

尽管其他插件都并非严格意义上的必需组件，但几乎所有 Kubernetes 集群都应该有集群 DNS，因为很多示例都需要 DNS 服务。

集群 DNS 是一个 DNS 服务器，和环境中的其他 DNS 服务器一起工作，它为 Kubernetes 服务提供 DNS 记录。

Kubernetes 启动的容器自动将此 DNS 服务器包含在其 DNS 搜索列表中。

### WEB 界面（仪表盘）

Dashboard 是 Kubernetes 集群通用的、基于 WEB 的用户界面。它使用户可以管理集群中运行的应用程序以及集群本身，并进行故障排查。

### 容器资源监控

容器资源监控将关于容器的一些觉的时间序列指标保存到一个集中的数据库中，并提供可用于浏览这些数据的界面。

### 集群层面日志

集群层面日志机制负责将容器的日志数据保存到一个集中的日志存储中，该存储能够提供搜索和浏览接口。
