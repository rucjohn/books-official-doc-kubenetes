# 理解 Kubernetes 对象

本页说明了 Kubernetes 对象在 Kubernetes API 中是如何表示的，以及如何在 `.yaml` 格式的文件中表示。

在 Kubernetes 系统中，Kubernetes 对象是持久化的实例。Kubernetes 使用这些实例去表示整体集群的状态。具体地说，它们可以描述如下信息：
- 哪些容器化应用程序在运行（以及运行在哪些节点上）
- 这些应用程序可用的资源
- 这些应用程序的运行策略，例如重启策略、升级策略和容错策略

Kubernetes 对象是 “目标性记录”，即一旦创建对象，Kubernetes 系统将持续工作以确保对象存在。通过创建对象，本质上是在告知 Kubernetes 系统，所需要的集群工作负载看起来是什么样子的，也就是 Kubernetes 集群的 **期望状态（Desired State）**。

操作 Kubernetes 对象，无论是创建、修改或者删除，都需要使用 Kubernetes API。比如，当使用 `kubectl` 命令行接口时，CLI 会执行必要的 Kubernetes API 调用，也可以在程序中使用客户端库直接调用 Kubernetes API。
