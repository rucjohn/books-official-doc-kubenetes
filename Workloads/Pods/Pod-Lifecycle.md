# Pod 生命周期

本页描述 Pod 的生命周期。Pod 遵循一个预定义的生命周期，起始于 `Pending` 阶段，如果至少其中有一个主要容器正确启动，则进入 `Running`，之后取决于 Pod 中是否有容器以失败状态结束而进入 `Succeeded` 或 `Failed` 阶段。

在 Pod 运行期间，`kubelet` 能够重启容器以处理一些失效场景。在 Pod 内部，Kubernetes 跟踪不同容器的状态，并确定使 Pod 重新变得健康所需要采取的动作。

在 Kubernetes API 中，Pod 包含规范部分和实际状态部分。Pod 对象的状态包含了一组 Pod 状况（**Conditions**）。如果应用需要的话，也可以向其中注入自定义的就绪信息。

Pod 在其生命周期中只会被调度一次。一旦 Pod 被调度到某个节点，Pod 会一直在该节点运行，直接 Pod 停止或者被终止。

## Pod 寿命

与单个应用程序容器一样，Pod 被认为是相对短暂的（不是长期存在的）实体。Pod 会被创建并赋予一个唯一的 ID（UID），并被调度到节点，并在终止（根据重启策略）或删除之前运行在该节点上。

如果节点宕机，调度到该节点的 Pod 也被计划在给定超时期限结束后删除。

Pod 自身不具备自愈能力。如果 Pod 被调度到某个节点，但该节点之后失效，Pod 会被删除；类似地，Pod 无法在因节点资源耗尽或者节点维护而被驱逐期间继续存活。Kubernetes 使用一种高级抽象秋管理这些相对而言可随时丢弃的 Pod 实例，它被称为控制器。

任何给定的 Pod（由 UID 定义）从不会被 “重新调度（*rescheduled*）”到不同的节点；但是，Pod 可以被一个新的、几乎相同的 Pod 替换掉。如果需要，新 Pod 名字可以不变，但是其 UID 仍然是不同的。

如果某个对象声称，其寿命与某 Pod 神秘顾客，例如存储卷，这就意味着该对象在此 Pod（UID 相同）存在期间也一直存在。如果 Pod 因为任何原因被删除，甚至某个完全相同的替代 Pod 被创建时，这个相关的对象也会被删除并重建。

![](../../.gitbook/assets/pod.jpg)


## Pod 阶段 <a href="#phase" id="waiting"></a>

## 容器状态

### Waiting（等待） <a href="#waiting" id="waiting"></a>

### Running（运行中） <a href="#running" id="running"></a>

### Terminated（已终止） <a href="#terminated" id="terminated"></a>

## Pod 的终止 <a href="#termination-of-pods" id="termination-of-pods"></a>

### 失效 Pod 的垃圾回收 <a href="#garbage-collection-of-failed-pods" id="garbage-collection-of-failed-pods"></a>
