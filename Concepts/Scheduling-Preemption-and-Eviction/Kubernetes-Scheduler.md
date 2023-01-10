# Kubernetes 调度器

在 Kubernetes 中，**调度**是指将 Pod 放到合适的 Node 上，然后对应 Node 上的 kubelet 才能够运行这些 Pod。

## 调度概览 <a href="#scheduling" id="scheduling"></a>

调度器通过 Kubernetes 的监测（Watch）机制来发现集群中新创建且尚未被调度到 Node 上的 Pod。 调度器会将发现的每一个未调度的 Pod 调度到一个合适的 Node 上来运行。 调度器会依据下文的调度原则来做出调度选择。

如果想要理解 Pod 为什么会被调度到特定的 Node 上，或者想要尝试实现一个自定义的调度器，这篇文章将帮助了解调度。

## kube-scheduler

kube-scheduler 是 Kubernetes 集群的默认调度器，并且是集群控制平面的一部分。 如果真有这方面的需求，kube-scheduler 在设计上是允许自定义一个调度组件并替换原有的 kube-scheduler。

对每一个新创建的 Pod 或者是未被调度的 Pod，kube-scheduler 会选择一个最优的 Node 去运行这个 Pod。然而，Pod 内的每一个容器对资源都有不同的需求，而且 Pod 本身也有不同的资源需求。因此，Pod 在被调度到 Node 上之前， 根据这些特定的资源调度需求，需要对集群中的 Node 进行一次过滤。

在一个集群中，满足一个 Pod 调度请求的所有 Node 称之为 _可调度节点_。 如果没有任何一个 Node 能满足 Pod 的资源请求，那么这个 Pod 将一直停留在 未调度状态直到调度器能够找到合适的 Node。

调度器先在集群中找到一个 Pod 的所有可调度节点，然后根据一系列函数对这些可调度节点打分， 选出其中得分最高的 Node 来运行 Pod。之后，调度器将这个调度决定通知给 kube-apiserver，这个过程叫做 _绑定_。

在做调度决定时需要考虑的因素包括：单独和整体的资源请求、硬件/软件/策略限制、 亲和以及反亲和要求、数据局域性、负载间的干扰等等。

## kube-scheduler 调度流程 <a href="#kube-scheduler-implementation" id="kube-scheduler-implementation"></a>

kube-scheduler 给一个 pod 做调度选择包含两个步骤：

1. 过滤
2. 打分

过滤阶段会将所有满足 Pod 调度需求的 Node 选出来。 例如，`PodFitsResources` 过滤函数会检查候选 Node 的可用资源能否满足 Pod 的资源请求。 在过滤之后，得出一个 Node 列表，里面包含了所有可调度节点；通常情况下， 这个 Node 列表包含不止一个 Node。如果这个列表是空的，代表这个 Pod 不可调度。

在打分阶段，调度器会为 Pod 从所有可调度节点中选取一个最合适的 Node。 根据当前启用的打分规则，调度器会给每一个可调度节点进行打分。

最后，kube-scheduler 会将 Pod 调度到得分最高的 Node 上。 如果存在多个得分最高的 Node，kube-scheduler 会从中随机选取一个。

支持以下两种方式配置调度器的过滤和打分行为：

1. [调度策略](../zh/docs/reference/scheduling/policies/) 允许你配置过滤的 _**断言(Predicates)**_ 和打分的 _**优先级(Priorities)**_ 。
2. [调度配置](../zh/docs/reference/scheduling/config/#profiles) 允许你配置实现不同调度阶段的插件， 包括：`QueueSort`, `Filter`, `Score`, `Bind`, `Reserve`, `Permit` 等等。 你也可以配置 kube-scheduler 运行不同的配置文件。
