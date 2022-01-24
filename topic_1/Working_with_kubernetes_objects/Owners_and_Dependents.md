# 属主和附属

在 Kubernetes 中，一些对象是其他对象的属主（_**Owner**_)。例如，ReplicaSet 是一组 Pod 的属主。这些具有属主的对象就是其所有者的附属（_**Dependent**_）。

属主关系不同于一些资源使用的 [Labels 和 Selector](Labels\_and\_Selectors.md) 机制。例如，有一个创建 `EndpointSlice` 对象的 Service，该 Service 使用标签来让控制平面确定哪些 `EndpointSlice` 对象用于该 Service。除开 Labels 之外，每个代表 Service 所管理的 `EndpointSlice` 对象都有一个属主引用。

属主引用避免 Kubernetes 因不同机制而干扰到不受其管理控制的对象。

## 对象规范中的属主引用

附属对象有一个 `metadata.ownerReferences` 字段，用于引用其属主对象。一个有效的属主引用，由与附属对象位于同一命名空间中的对象和 UID 组成。Kubernetes 自动为一些对象的附属资源设置属主引用的值，这些对象包含 ReplicaSet、DaemonSet、Deployment、Job、CronJob、ReplicationController 等。也可以通过改变这个字段的值，来手动配置这些关系，不过通常没必要，Kubernetes 会自动管理这些附属关系。

附属对象还有一个 `ownerReferences.blockOwnerDeletion` 字段，该字段使用的是布尔值，用于控制特定的附属对象是否可以阻止垃圾收集删除其属主对象。如果控制器（例如：Deployment 控制器）设置了 `metadata.ownerReferences` 字段的值，则 Kubernetes 会自动设置 `blockOwnerDeletion=true`。也可以手动设置 `blockOwnerDeletion` 字段的值，以控制哪些附属对象允许阻止垃圾收集。
