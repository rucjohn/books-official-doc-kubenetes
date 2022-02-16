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

| **阶段**    | **描述**                                                                           |
| --------- | -------------------------------------------------------------------------------- |
| Pending   | Pod 已被 Kubernetes 集群接受，但有一个或多个容器尚未创建并准备好运行。此阶段包括 Pod 等待调度所花费的时间以及通过网络下载镜像所花费的时间。 |
| Running   | Pod 已绑定到一个节点，并且所有容器均已创建。至少有一个容器正在运行，或者正在启动或重启的过程中。                               |
| Succeeded | Pod 中所有容器都已成功终止，且不会再重启。                                                          |
| Failed    | Pod 中所有容器都已终止，并且至少有一个容器因故障而终止。也就是说，容器要么以非 0 状态退出，要么被系统终止。                        |
| Unknown   | 因为某些原因无法取得 Pod 状态。这种情况通常是因为与 Pod 所在节点通信出错而导致。                                    |

如果节点宕机或与集群中其他节点失联，Kubernetes 会实施一种策略，将失去的节点上运行的所有 Pod 的 `phase` 设置为 `Failed`。

## 容器状态

Kubernetes 会跟踪 Pod 中每个容器的状态，就像跟踪 Pod 阶段一样。可以使用容器生命周期回调来在容器生命周期中的特定时间点触发事件。

一旦调度器将 Pod 分派给某个节点，`kubelet` 就通过容器运行时开始为 Pod 创建容器。容器的状态有三种：

* `Waiting`（等待）
* `Running`（运行中）
* `Terminated`（已终止）

要检查 Pod 中容器的状态，可以使用 `kubectl describe pod <pod name>`命令。其输出中包含了 Pod 每个容器的状态。

每种状态的含义如下：

### <mark style="color:orange;">Waiting</mark>（等待） <a href="#waiting" id="waiting"></a>

处于 `Waiting` 状态的容器仍在运行它要完成启动所需要的操作，例如：从某个容器镜像仓库拉取镜像，或者应用 Secret 数据等。当使用 `kubelet` 来查询包含 `Waiting` 状态的容器的 Pod 时，也会看到一个 `Reason` 字段，其中会给出容器处于该状态的原因。

### <mark style="color:orange;">Running</mark>（运行中） <a href="#running" id="running"></a>

`Running` 状态表里容器正在执行状态并且没有故障发生。如果配置了 `postStart` 回调，那么该回调也已完成且执行成功。如果使用 `kubelet` 来查询包含 `Running` 状态的容器的 Pod 时，会看到关于容器进入该状态的信息。

### <mark style="color:orange;">Terminated</mark>（已终止） <a href="#terminated" id="terminated"></a>

处于 `Terminated` 状态的容器已经开始执行，或者执行完成，或者由于某种原因失败。如果使用 `kubelet` 来查询包含 `Terminated` 状态的容器的 Pod 时，会看到容器进入此状态的原因、退出代码，以及容器执行期间的起止时间。

如果容器配置了 `preStop` 回调，则该回调会在容器进入 `Terminated` 状态之前执行。

## 容器重启策略

Pod 的 `spec` 字段中有一个 `restartPolicy` 字段，其值包含 Always、OnFailure 和 Never。默认是 Always。

`restartPolicy` 适用于 Pod 中的所有容器。`restartPolicy` 仅针对同一节点上的 `kubelet` 的容器重启动作。当 Pod 中的容器退出时，`kubelet` 会按指数回退方式计算重启的延迟（10s、20s、40s、...），其最长延迟时间为 5 分钟。一旦某容器执行了 10 分钟并且没有出现问题，`kubelet` 对该容器的重启回退计时将重置。

## Pod 状况

Pod 有一个 PodStatus 对象，其中包含一个 PodConditions 数组。存在以下几种情况：

* `PodScheduled`：Pod 已经被调度到某节点
* `ContainersReady`：Pod 中所有容器都已就绪
* `Initialized`：所有的初始容器都已成功启动
* `Ready`：Pod 可以为请求提供服务，并且应该被添加到对应 Service 的负载均衡池中。

| **字段名称**           | **描述**                                       |
| ------------------ | -------------------------------------------- |
| type               | Pod 状况的名称                                    |
| status             | 表明该状态是否适用，可能的取值有 "True", "False" 或 "Unknown" |
| lastProbeTime      | 上次探测 Pod 状况的时间戳                              |
| lastTransitionTime | Pod 止次从一种状态转换为另一种状态时的时间戳                     |
| reason             | 描述上次状态变化的原因的状态，驼峰编码的英文描述                     |
| message            | 描述上次状态转换的详细信息                                |

### Pod 就绪态

**FEATURE STATE:** <mark style="color:orange;">Kubernetes v1.14 \[stable]</mark>

应用可以向 PodStatus 中注入额外的反馈或者信号：_Pod Readiness_。要使用这一特性，可以设置 Pod 规范中的 `readinessGate` 列表，为 kubelet 提供一组额外的状态供其评估 Pod 就绪态时使用。

就绪态门控基于 Pod 的 `status.conditions` 字段 的当前值来做决定。如果 Kubernetes 无法在 `status.conditions` 字段中找到某个状态，则该状态的状态值默认为 `False`。

例如：

```yaml
kind: Pod
...
spec:
  redinessGates:
    - conditionType: "www.example.com/feature-1"
status:
  conditions:
    - type: Ready
      status: "False"
      lastProbeTime: null
      lastTransitionTime: 2018-01-01T00:00:00Z
    - type: "www.example.com/feature-1"
      status: "False"
      lastProbeTime: null
      lastTransitionTime: 2018-01-01T00:00:00Z
  containerStatuses:
    - containerID: docker:/abcd...
      ready: true
...
```

所添加的 Pod 状况名称必须满足 Kubernetes 标签键名格式。

### Pod 就绪态的状态

命令 `kubectl ptch` 不支持修改对象的状态。如果需要设置 Pod 的 `status.conditions`，应用或者 Operators 需要使用 `patch` 操作。可以使用 Kubernetes 客户端库之一来编写代码，针对 Pod 就绪态设置定制的 Pod 状况。

对于使用定制状态的 Pod 而言，只有当下面的陈述都适用时，该 Pod 才会被评估为就绪：

* Pod 中所有容器都已就绪
* `readinessGates` 中的所有状态都为 `True`

当 Pod 的容器都已就绪，但至少一个定制状态没有取值或者取值为 `False`，`kubelet` 将 Pod 的状况设置为 `ContainersReady`。

## Pod 的终止 <a href="#termination-of-pods" id="termination-of-pods"></a>

### 失效 Pod 的垃圾回收 <a href="#garbage-collection-of-failed-pods" id="garbage-collection-of-failed-pods"></a>
