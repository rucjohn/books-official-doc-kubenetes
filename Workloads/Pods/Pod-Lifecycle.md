# Pod 生命周期

本页描述 Pod 的生命周期。Pod 遵循一个预定义的生命周期，起始于 `Pending` 阶段，如果至少其中有一个主要容器正确启动，则进入 `Running`，之后取决于 Pod 中是否有容器以失败状态结束而进入 `Succeeded` 或 `Failed` 阶段。

在 Pod 运行期间，`kubelet` 能够重启容器以处理一些失效场景。在 Pod 内部，Kubernetes 跟踪不同容器的状态，并确定使 Pod 重新变得健康所需要采取的动作。

在 Kubernetes API 中，Pod 包含规范部分和实际状态部分。Pod 对象的状态包含了一组 Pod 状况（**Conditions**）。如果应用需要的话，也可以向其中注入自定义的就绪信息。

Pod 在其生命周期中只会被调度一次。一旦 Pod 被调度到某个节点，Pod 会一直在该节点运行，直接 Pod 停止或者被终止。

## Pod 寿命

与单个应用程序容器一样，Pod 被认为是相对短暂的（不是长期存在的）实体。Pod 会被创建并赋予一个唯一的 ID（UID），并被调度到节点，并在终止（根据重启策略）或删除之前运行在该节点上。

如果节点宕机，调度到该节点的 Pod 也被计划在给定超时期限结束后删除。

Pod 自身不具备自愈能力。如果 Pod 被调度到某个节点，但该节点之后失效，Pod 会被删除；类似地，Pod 无法在因节点资源耗尽或者节点维护而被驱逐期间继续存活。Kubernetes 使用一种高级抽象秋管理这些相对而言可随时丢弃的 Pod 实例，它被称为控制器。

任何给定的 Pod（由 UID 定义）从不会被 “重新调度（_rescheduled_）”到不同的节点；但是，Pod 可以被一个新的、几乎相同的 Pod 替换掉。如果需要，新 Pod 名字可以不变，但是其 UID 仍然是不同的。

如果某个对象声称，其寿命与某 Pod 神秘顾客，例如存储卷，这就意味着该对象在此 Pod（UID 相同）存在期间也一直存在。如果 Pod 因为任何原因被删除，甚至某个完全相同的替代 Pod 被创建时，这个相关的对象也会被删除并重建。

![](../../.gitbook/assets/pod.jpg)

一个包含多个容器的 Pod 中包含一个用来拉取文件的程序和一个 Web 服务器，无使用持久卷作为容器间共享的存储。

## Pod 阶段 <a href="#waiting" id="waiting"></a>

Pod 的 `status` 字段 是一个 PodStatus 对象，其中包含一个 `phase` 字段。

Pod 的阶段（Phase）是 Pod 在其生命周期中所处位置的简单宏观概述。该阶段并不是对容器或 Pod 状态的综合汇总，也不是为了成为完整的状态机。

Pod 阶段的数量和含义是严格定义的。除了本文档中列举的内容外，不应再假定 Pod 有其他的 `phase` 值。

`phase` 字段：

| 值             | 描述                                                                               |
| ------------- | -------------------------------------------------------------------------------- |
| **Pending**   | Pod 已被 Kubernetes 集群接受，但有一个或多个容器尚未创建并准备好运行。此阶段包括 Pod 等待调度所花费的时间以及通过网络下载镜像所花费的时间。 |
| **Running**   | Pod 已绑定到一个节点，并且所有容器均已创建。至少有一个容器正在运行，或者正在启动或重启的过程中。                               |
| **Succeeded** | Pod 中所有容器都已成功终止，且不会再重启。                                                          |
| **Failed**    | Pod 中所有容器都已终止，并且至少有一个容器因故障而终止。也就是说，容器要么以非 0 状态退出，要么被系统终止。                        |
| **Unknown**   | 因为某些原因无法取得 Pod 状态。这种情况通常是因为与 Pod 所在节点通信出错而导致。                                    |

如果节点宕机或与集群中其他节点失联，Kubernetes 会实施一种策略，将失去的节点上运行的所有 Pod 的 `phase` 设置为 `Failed`。&#x20;

## 容器状态

Kubernetes 会跟踪 Pod 中每个容器的状态，就像跟踪 Pod 阶段一样。可以使用容器生命周期回调来在容器生命周期中的特定时间点触发事件。

一旦调度器将 Pod 分派给某个节点，`kubelet` 就通过容器运行时开始为 Pod 创建容器。容器的状态有三种：
- `Waiting`（等待）
- `Running`（运行中）
- `Terminated`（已终止）

要检查 Pod 中容器的状态，可以使用 `kubectl describe pod <pod name>`命令。其输出中包含了 Pod 每个容器的状态。

每种状态的含义如下：

### Waiting（等待） <a href="#waiting" id="waiting"></a>

### Running（运行中） <a href="#running" id="running"></a>

### Terminated（已终止） <a href="#terminated" id="terminated"></a>

## Pod 的终止 <a href="#termination-of-pods" id="termination-of-pods"></a>

### 失效 Pod 的垃圾回收 <a href="#garbage-collection-of-failed-pods" id="garbage-collection-of-failed-pods"></a>
