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

Kubernetes 中的许多对象通过属主引用相互链接。属主引用告诉控制平面某个对象所依赖的其他对象。Kubernetes 使用属主引用为控制平面和其他 API 客户端提供在删除对象之前清理相关资源的机会。大多数情况下，Kubernetes 会自动管理属主引用。

属主不同于标签和选择器机制。例如，考虑一个创建 EndpointSlice 对象的服务。服务使用标签来允许控制平面确定哪些 EndpointSlice 对象用于该服务。除了标签之外，代表服务管理的每个 EndpointSlice 都有一个属主引用。属主引用帮助 Kubernetes 中的不同机制避免干扰到不受它们控制的对象。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

根据设计，Kubernetes 不允许跨命名空间指定属主。命名空间范围的附属可以指定集群范围或命名空间范围的属主。命名空间范围的属主必须与附属处于相同的命名空间。如果命名空间范围的属主附属不在同一命名空间下，那么该属主引用就会被认定为缺失，并且当附属的所有属主引用都被确定不在存在之后，该附属就会被删除。
{% endhint %}

{% hint style="warning" %}
<mark style="color:orange;">**警告：**</mark>

集群范围的附属只能指定集群范围的属主。从 v1.20 版本后，如果一个集群范围的附属指定了一个命名空间范围类型的属主，那么该附属就会被认为是拥有一个不可解析的属主引用，并且它不能够被垃圾回收。
{% endhint %}

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

从 v1.20 版本后，如果垃圾回收器检测到无效的跨命名空间的属主引用，或者一个集群范围的附属指定了一个命名空间范围类型的属主，那么它就会报告一个警告事件。该事件的原因是 `OwnerRefInvalidNamespace`，`involvedObject` 属性中包含无效的附属。可以运行 `kubectl get events -A --field-selector=reson=OwnerRefinvalidNamespace` 来获取该类型的事件。
{% endhint %}

## 级联删除

Kubernetes 检查 并删除不再具有属主引用的对象，例如，删除 ReplicaSet 时同时删除 Pod。当删除一个对象时，可以控制 Kubernetes 是否自动删除该对象的依赖项，这个过程称为级联删除。级联删除有两种类型：

* 前台级联删除
* 后台级联删除

还可以控制垃圾回收何时以怎样的方式使用 Kubernetes finalizers 删除具有属主引用的资源。

### 前台级联删除

在前台级联删除中，要删除的属主对象首先进入删除中状态。在这种状态下，属主对象会发生以下情况：

* Kubernetes apiserver 将对象的 `metadata.deletionTimestamp` 字段设置为对象被标记为删除的时间。
* Kubernetes apiserver 设置对象 `metadata.finalizers=foregroundDeletion`。
* 该对象通过 Kubernetes API 保持可见，直到删除过程完成。

属主对象进入删除中状态后，控制器删除依赖项。删除所有依赖对象后，控制器删除属主对象。此时，该对象在 Kubernetes API 中不可见。

在前台级联删除期间，能够阻止属主删除的唯一依赖项是具有 `ownerReference.blockOwnerDeletion=true` 字段的依赖项。

### 后台级联删除

在后台级联删除中，Kubernetes apiserver 会立即删除属主对象，控制器在后台清理依赖对象。默认情况下，Kubernetes 使用后台级联删除，除非手动使用前台 删除或选择孤立依赖对象。

### 孤立依赖

当 Kubernetes 删除属主对象时，留下的依赖对象称为孤儿对象。默认情况下，Kubernetes 会删除依赖对象。

## 未使用的容器和镜像

kubelet **每五分钟**对未使用的镜像执行一次垃圾回收，**每分钟**对未使用的容器执行一次垃圾回收。应该避免使用外部垃圾回收工具，因为这些工具会破坏 kubelet 的垃圾回收行动。

要为未使用的容器和镜像配置垃圾回收选项，请使用 [配置文件](../Adminster-a-Cluster/Set-Kubelet-paramenters-via-a-config-file.md) 调整 kubelet，并使用 kubeletConfiguration 资源类型更改与垃圾回收相关的参数。

### 容器镜像生命周期

Kubernetes 通过它的镜像管理器来管理所有镜像的生命周期，镜像管理器是 kubelet 的一部分，与 cadvisor 合作。kubelet 在做出垃圾回收决策时会考虑以下磁盘使用限制：

* `HighThresholdPercent`
* `LowThresholdPercent`

高于 `HighThresholdPercent` 配置的值的磁盘使用时会触发垃圾回收，它会根据上次使用的时间顺序删除镜像，从最早的第一个开始。kubelet 会删除镜像，直到磁盘使用率达到 `LowThresholdPercent` 的值。

### 容器垃圾回收

kubelet 根据以下变量垃圾回收未使用的容器：

* `MinAge`：kubelet 可以垃圾回收容器的最小年龄。`0` 为禁用。
* `MaxPerPodContainer`：每个 Pod 可拥有的最大僵死容器数。`0` 为禁用。
* `MaxContainers`：集群可拥有的最大僵死容器数。`0` 为禁用。

除了上述变量外，kubelet 还会垃圾回收未识别和已删除的容器，通常从最老的容器开始。

`MaxPerPodContainer` 的值超出允许的 `MaxContainers` 全局僵死容器数的情况下，这两个参数可能会相互冲突。在这种情况下，kubelet 会调整 `MaxPerPodContainer` 的值来解决冲突。最坏的情况是设置 `MaxPerPodContainer=1`，并驱逐最老的容器。此外，已删除的 Pod 拥有的容器在比 `MinAge` 老时会被删除。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>kubelet 仅垃圾回收它所管理的容器。
{% endhint %}

## 配置垃圾回收

可以通过配置特定管理资源的控制器选项来调整资源的垃圾回收：

* [配置 Kubernetes 对象的级联删除](../Adminster-a-Cluster/Use-Cascading-Deletion-in-a-Cluster.md)
* [配置已完成 Job 的清理](../Workloads/Automatic-Clean-up-for-Finished-jobs.md)
