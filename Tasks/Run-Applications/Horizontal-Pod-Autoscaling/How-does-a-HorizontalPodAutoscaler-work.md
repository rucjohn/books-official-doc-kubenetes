# HPA 是如何工作的？



<figure><img src="../../../.gitbook/assets/hpa-how-work.svg" alt=""><figcaption><p>图 1. HorizontalPodAutoscaler 控制 Deployment 及其 R'e'p'li'ca'S'eReplicaSet 的规模</p></figcaption></figure>

Kubernetes 将水平 Pod 自动扩缩容实现为一个间歇运行的控制回路（它不是一个连续的过程）。间隔由 `kube-controller-manager` 的 `--horizontal-pod-autoscaler-sync-period` 参数来控制（默认间隔为 15 秒）。

在每个时间段内，控制器都会根据每个 HorizontalPodAutoscaler 定义中指定的指标查询资源利用率。控制器找到由 `scaleTargetRef` 定义的目标资源，然后根据目标资源的 `.spec.selector` 标签选择 Pod，并从资源指标 API（针对每个 Pod 的资源指标）或自定义指标获取指标 API（适用于所有其他指标）。

- 对于按 Pod 统计的资源指标（如，CPU），控制器从资源指标 API 中获取每一个 HorizontalPodAutoscaler 指定的 Pod 的度量值，根据平均的资源使用率或原始值计算出扩缩容的比例，进而计算出目标副本数。
  - 如果设置了目标使用率，控制器获取每个 Pod 的容器资源使用情况，并计算资源使用率。
  - 如果设置了 `target` 值，将直接使用原始数据（不再计算百分比）。

- 如果 Pod 使用自定义指标，控制器机制与资源指标类似，区别在于自定义指标只使用原始值，而不是使用率。

- 如果 Pod 使用对象指标和外部指标（每个指标描述一个对象信息）。这个指标将直接根据目标设定值相比较，并生成一个上面提到的扩缩容比例。在 `autoscaling/v2` 版本 API 中，这个指标也可以根据 Pod 数量一部分后再计算。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

如果 Pod 某些容器不支持资源采集，那么控制器将不会使用该 Pod 的 CPU 使用率。下面的算法细节章节将会介绍详细的算法。
{% endhint %}


horizontalPodAutoscaler 的常见用途是将其配置为从聚合 API（`metrics.k8s.io`、`custom.metrics.k8s.io` 或 `external.metrics.k8s.io`）获取指标。`metrics.k8s.io` API 通常由名为 Metrics Server 的插件提供，需要单独启动。有关资源指标的更多信息，请参阅 Metrics Server。

对 Metrics API 的支持解释了这些不同 API 的稳定性保证和支持状态。

HorizontalPodAutoscaler 控制器访问支持扩缩容的相应工作负载资源（如，Deployment 和 StatefulSet）。这些资源每个都有一个名为 `scale` 的子资源，该接口允许动态设置副本的数量并检查 它们的每个当前状态。有关 Kubernetes API 子资源的一般信息，请参阅 Kubernetes API 概念。


