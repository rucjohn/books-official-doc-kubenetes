# Kubernetes 组件

当部署完 Kubernetes，即拥有了一个完整的集群。

一个 Kubernetes 集群由一组被称作节点的机器组成。这些节点上运行 Kubernetes 所管理的容器化应用。集群具有至少一个工作节点。

工作节点托管作为应用负载的组件的 Pod。控制平台管理集群中的工作节点和 Pod。为集群提供故障转移和高可用性，这些控制平台一般跨多主机运行，集群跨多个节点运行。

本文档概述了将会正常运行的 Kubernetes 集群所需的各种组件。

这张图表展示了包含所有相互关联组件的 Kubernetes 集群。

![](../../static/components-of-kubernetes.svg)
