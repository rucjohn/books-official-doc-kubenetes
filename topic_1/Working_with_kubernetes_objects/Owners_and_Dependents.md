# 属主和附属

在 Kubernetes 中，一些对象是其他对象的属主（_**Owner**_)。例如，ReplicaSet 是一组 Pod 的属主。这些具有属主的对象就是其所有者的附属（_**Dependent**_）。

属主关系不同于一些资源使用的 [Labels 和 Selector](Labels\_and\_Selectors.md) 机制。例如，有一个创建 `EndpointSlice` 对象的 Service，该 Service 使用标签来让控制平面确定哪些 `EndpointSlice` 对象用于该 Service。除开 Labels 之外，每个代表 Service 所管理的 `EndpointSlice` 对象都有一个属主引用。

属主引用避免 Kubernetes 因不同机制而干扰到不受其管理控制的对象。

