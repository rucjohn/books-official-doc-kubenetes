# 工作负载

工作负载是在 Kubernetes 上运行的应用程序。

在 Kubernetes 中，无论此负载是由单个组件还是由多个一同工作的组件构成，都可以在一组 **Pod** 中运行它。在 Kubernetes 中，`Pod` 代表的是集群上牌运行状态的一组容器的集合。

Kubernetes Pod 遵循预定义的生命周期。例如，当在集群中运行了某个 Pod，但是 Pod 所在的节点出现宕机时，所有该节点上的 Pod 的状态都会变成失败。Kubernetes 将这类失败视为最终状态：即使该节点后来恢复正常运行，也需要创建新的 `Pod` 以恢复应用。

不过，为了减轻用户的使用负担，通常不需要用户直接管理每个 `Pod`。而是使用负载资源来替用户管理一组 Pod。这些负载资源通过配置控制器来确保配置正确的、处于运行状态的 Pod 的个数是正确的、与用户所指定的状态相一致。

Kubernetes 提供若干内置的工作负载资源：

- Deployment 和 ReplicaSet（替换原来的 ReplicationController）。`Deployment` 很适合用来管理集群上无状态的应用。`Deployment` 中所有的 `Pod` 都是相互等价的，并且在需要的时候被替换。

- StatefulSet 能够运行一个或者多个以某种方式跟踪应用状态的 Pod。例如，如果负载要将数据作持久化存储，可以运行一个 `StatefulSet`，将每个 `Pod` 与某个 PersistentVolume 对应起来。
