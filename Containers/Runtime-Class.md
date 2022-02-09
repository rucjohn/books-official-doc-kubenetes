# 容器运行时类

**FEATURE STATUS:** Kubernetes v1.20 [stable]

本页面描述了 RuntimeClass 资源和运行时的选择机制。

RuntimeClass 是一个用于选择容器运行时配置的特性，容器运行时配置用于运行 Pod 中的容器。

## 动机

可以在不同的 Pod 设置不同的 RuntimeClass，以提供性能与安全性之间的平衡。例如，如果部分工作负载需要更高组别的信息安全保证，可以决定在调度这些 Pod 尽量使它们在使用硬件虚拟化的容器运行时中运行。这样，将从这些不同运行时所提供的额外隔离中获益，代价是一些额外的开销。

还可以使用 RuntimeClass 运行具有相同容器运行时但具有不同设置的 Pod。

## 设置

1. 在节点上配置 CRI 的实现（阔以阔以于所选用的运行时）
2. 创建相应的 RuntimeClass 资源

### 1. 在节点上配置 CRI 实现

RuntimeClass 的配置依赖于运行时接口（CRI）的实现。根据使用的 CRI 实现，配置方法请参阅 CRI 配置 小节。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

RuntimeClass 假设集群中的节点配置是同构的（换言之，所有的节点在容器运行时方面的配置是相同的）。如果需要支持异构节点，配置方法将参阅 调度 小节。
{% endhint %}

所有这些配置都具有相应的 `handler` 名，并被 RuntimeClass 引用。handler 必须是有效的 DNS 标签名。

### 2. 创建相应的 RuntimeClass 资源

在上面的步骤 1 中，每个配置都需要有一个用于标识配置的 `handler`。针对 每个 handler 需要创建一个 RuntimeClass 对象。


