# Pod 水平自动扩缩容

在 Kubernetes 中， **HorizontalPodAutoscaler** 自动更新工作负载（例如，Deployment 或者 StatefulSet），目的是自动扩缩容工作负载以满足需求。


水平扩缩容意味着对增加的负载的响应是部署更多的 Pod。这与 “垂直（Vertical）” 扩缩容不同，对于 Kubernetes，垂直扩缩容意味着更多的资源（例如，内存或 CPU）分配给已经为工作负载运行的 Pod。

如果负载减少，并且 Pod 的数量高于配置的最小值，HorizontalPodAutoscaler 会指示工作负载资源缩减。

水平 Pod 自动扩缩容不适用于无法扩缩容的对象（例如，DaemonSet）。

HorizontalPodAutoscaler 被实现为 Kubernetes API 资源和控制器。

资源决定了控制器的行为。在 Kubernetes 控制平成内运行的水平 Pod 自动扩缩容控制器会定期调整其目标（例如，Deployment）的所需规模，以匹配观察到的指标，例如：平均 CPU 利用率、平均内存利用率或指定的任何其他自定义的指标。

使用水平 Pod 自动扩缩容演练示例。

## API 对象

HorizontalPodAutoscaler 是 Kubernetes `autoscaling` API 组中的 API 资源。当前的稳定版本可以在 `autocaling/v2` API 版本中找到，其中包括对基于内存和自定义指标执行扩缩容的支持。在使用 `autoscaling/v1` 时，\`autoscaling/v2` 中引入 的新字段作为注释保留。

创建 HorizontalPodScaler 对象时，需要确保所给的名称是一个合法的 DNS 子域名。有关 API 对象的更多信息，请查阅 [HorizontalPodAutoscaler 对象文档](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#horizontalpodautoscaler-v2-autoscaling)。

