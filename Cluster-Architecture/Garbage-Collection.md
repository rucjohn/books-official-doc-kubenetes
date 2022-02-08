# 垃圾回收

垃圾回收是 Kubernetes 用来清理集群资源的各种机制的统称。以下几种情况允许清理资源：

* [已失效的 Pods](../Workloads/Pods/Pod-Lifecycle.md#garbage-collection-of-failed-pods)
* [已完成的 Jobs](../Workloads/Automatic-Clean-up-for-Finished-jobs.md)
* 没有属主引用的对象
* 未使用的容器和镜像
* [使用 Delete 的 StorageClass 回收策略动态配置的 PersistentVolume](../Storage/Persistent-Volumes.md#delete)
* 陈旧或过期的证书签名请求（CSRs）
* 以下场景删除的节点：
  * 当集群在云上使用云控制器时
  * 当集群在本地使用类似云控制器插件时
* [节点 Lease 对象](Nodes.md#hearbeats)

## 属主和附属
