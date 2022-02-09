# Pod 生命周期

本页描述 Pod 的生命周期。Pod 遵循一个预定义的生命周期，起始于 `Pending` 阶段，如果至少其中有一个主要容器正确启动，则进入 `Running`，之后取决于 Pod 中是否有容器以失败状态结束而进入 `Succeeded` 或 `Failed` 阶段。

在 Pod 运行期间，`kubelet` 能够重启容器以处理一些失效场景。在 Pod 内部，Kubernetes 跟踪不同容器的状态，并确定使 Pod 重新变得健康所需要采取的动作。

在 Kubernetes API 中，Pod 包含规范部分和实际状态部分。Pod 对象的状态包含了一组 Pod 状况（**Conditions**）。如果应用需要的话，也可以向其中注入自定义的就绪信息。

Pod 在其生命周期中只会被调度一次。一旦 Pod 被调度到某个节点，Pod 会一直在该节点运行，直接 Pod 停止或者被终止。

## Pod 生命期

## Pod 阶段 <a href="#phase" id="waiting"></a>

## 容器状态

### Waiting（等待） <a href="#waiting" id="waiting"></a>

### Running（运行中） <a href="#running" id="running"></a>

### Terminated（已终止） <a href="#terminated" id="terminated"></a>

## Pod 的终止 <a href="#termination-of-pods" id="termination-of-pods"></a>

### 失效 Pod 的垃圾回收 <a href="#garbage-collection-of-failed-pods" id="garbage-collection-of-failed-pods"></a>
