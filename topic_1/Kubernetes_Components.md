# Kubernetes 组件

当部署完 Kubernetes，即拥有了一个完整的集群。

一个 Kubernetes 集群由一组被称作节点的机器组成。这些节点上运行 Kubernetes 所管理的容器化应用。集群具有至少一个工作节点。

工作节点托管作为应用负载的组件的 Pod。控制平台管理集群中的工作节点和 Pod。为集群提供故障转移和高可用性，这些控制平台一般跨多主机运行，集群跨多个节点运行。

本文档概述了将会正常运行的 Kubernetes 集群所需的各种组件。

这张图表展示了包含所有相互关联组件的 Kubernetes 集群。

![](../static/components-of-kubernetes.svg)

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
- **节点控制器（Node Controller）**：负载在节点出现故障时进行通知和响应
- **任务控制器（Job Controller）**：监测代表一次性任务的 Job 对象，然后创建 Pods 来运行这些任务直至完成
- **端点控制器（Endpoints Controller）**：填充端口（Endpoints）对象（即加入 Service 与 Pod）
- **服务账户和令牌控制器（ServiceAccount && Token Controllers）**：为新的命名空间创建默认账户和 API 访问令牌

## Node 组件

节点组件在每个节点上运行，维护运行的 Pod 并提供 Kubernetes 运行环境。
