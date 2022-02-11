# Pods

Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。

Pod 是一组（一个或多个）容器；这些容器共享存储、网络，以及共享运行这些容器的声明。Pod 的内容始终共同定位共同调度的，在共享的上下文中运行。Pod 对特定于应用程序的“逻辑主机”进行建模：它包含一个或多个相对紧耦合的应用程序容器。在非云环境中，在同一物理或虚拟机上执行的应用程序，类似于同一逻辑主机上执行的云应用程序。

除了应用容器，Pod 还可以包含了在 Pod 启动期间运行的 Init 容器。如果集群提供此功能，还可以注入临时容器进行调试。

## Pod 是什么？

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

Kubernetes 不仅支持 Docker，还支持其他许多容器运行时，但 Docker 是最常用见的；使用 Docker 中的一些术语有助于描述 Pod。
{% endhint %}

Pod 的共享上下文包括一组 Linux 命名空间、控制组（cgroup）和可能的其他隔离功能，即用来隔离 Docker 容器的技术。在 Pod 上下文中，每个独立的应用可能会进一步实施隔离。

就 Docker 概述的术语而言，Pod 类似于共享命名空间和文件系统卷的一组 Docker 容器。

## Pod 使用

下面的 Pod 例子中，它将创建一个运行着 `nginx:1.14.2` 镜像的容器。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

创建命令：

```bash
kubectl apply -f simple-pod.yaml
```

### 用于管理 Pod 的工作负载资源

通常不需要直接创建 Pod，想把，会使用诸如 Deployment 或 Job 这类的工作负载来创建 Pod。如果应用是有状态的，可以考虑 StatefulSet 资源。

Kubernetes 集群中的 Pod 主要有两种用法：

* **运行单个容器的 Pod**：“一个 Pod 一个容器” 模型是最常见的 Kubernetes 用例；在这种情况下，可以将 Pod 看作单个容器的包装器，并且 Kubernetes 直接管理 Pod，而不是容器。
* **运行外协同工作容器的 Pod**：Pod 可能封装由多个紧耦合且需要共享资源的共处容器组成的应用程序。这些位于同一位置的容器可能形成单个内聚的服务单元，即一个容器将文件写入共享卷，而一个 sidecar 容器则读取或更新共享卷中的文件。Pod 将这些容器和存储资源打包为一个可管理的实体。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

将多个并置、同管的容器组成到一个 Pod 是一种相对高级的使用场景。只有在一些场景中，容器之间紧密关联时才应该使用这种模式。
{% endhint %}

每个 Pod 都旨在运行给定应用程序的单个实例。如果希望横向扩展应用程序（例如，运行多个实例以提供更多的资源），则应该使用多个 Pod，每个实例使用一个 Pod。在 Kubernetes 中，这通常被称为副本（**Replication**）。通常使用一种工作负载资源及其控制器来创建和管理一组 Pod 副本。

### Pod 如何管理多个容器

Pod 被设计成支持形成内聚服务单元的多个协作过程（形式为容器）。Pod 中的容器被自动安排 到集群中的同一物理机或虚拟机上，并可以同时调度。容器之间可以共享资源和依赖、相互通信、协调何时以及如何方式终止。

例如，可能有一个容器充当共享眷顾中文件的 Web 服务器，以及一个单独的 "sidecar" 容器，用于从远程源更新这些文件，如下图：

![](../../.gitbook/assets/pod.jpg)

有一些 Pod 有初始容器和应用容器。初始容器在应用容器启动之前运行并完成。

Pod 原生地为其成员容器提供了两种共享资源：网络和存储。



## Pod 工作

很少直接在 Kubernetes 创建单独的 Pod，即使是单例 Pod。这是因为 Pod 被设计为相对短暂的、一次性的实体。当 Pod 手动创建或间接地由控制器创建时，它被调度到集群的某个节点上运行。Pod 会保持在该节点上运行，直到 Pod 结束执行、Pod 对象被删除、Pod 因资源不足而被驱逐，或者节点失效为止。

说明：

重启 Pod 中的容器不应与重启 Pod 混淆。Pod 不是进程，而容器运行的环境。在被删除之间，Pod 会一直存在。

当为 Pod 对象创建清单时，要确保所指定的 Pod 名称必须是合法的 DNS 子域名。

### Pod 和控制器

可以使用工作负载资源来创建和管理多个 Pod。工作负载控制器能够在 Pod 发生故障时管理副本、滚动更新，以及自动修复。例如，如果一个节点发生故障，控制器会注意到该节点上的 Pod 已停止工作并创建一个替换 Pod。调度程序将替换 Pod 放置到一个健康的节点上。

常见的工作负载资源：
- Deployment
- StatefulSet
- DaemonSet

### Pod 模板

工作负载资源控制器通常使用 Pod 模板（**Pod Template**）来创建 Pod 并管理它们。

Pod 模板是包含在工作负载对象中的规范，用来创建 Pod。这些负载资源包括 Deployment、Job 和 DaemonSet等。

工作负载控制器会使用 `PodTemplate` 来生成实际的 Pod。`PodTemplate` 是用来运行应用时指定的负载资源的目标状态的一部分。

下面的示例是一个简单的 Job 清单，其中的 `template` 指示启动一个容器。该 Pod 中的容器会打印一条消息之后暂停。
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    # This is the pod template
    spec:
      containers:
      - name: hello
        image: busybox
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: onFailure
    # The pod template ends here
```

修改 Pod 模板或者切换到新的 Pod 模板对已经存在的 Pod 没有直接影响。相反，新的 Pod 会被创建出来。例如，StatefulSet 控制器确保正在运行的 Pod 与 StatefulSet 对象的当前 Pod 模板匹配。如果编辑了 StatefulSet 并更改其 pod 模板，则 StatefulSet 根据更新后的模板创建新的 Pod，待 Pod 创建成功后，删除旧的 Pod，直至所有 Pod 替换完成。

每个工作负载资源都实现了自己的规则 来处理对 pod 模板的更改。如果想详细了解 StatefulSet，请参阅 StatefulSet 基础教程中的更新策略。

在节点上，kubelet 不直接监测或管理与 pod 模板相关的细节或模板的更新，这些细节 都被抽象出来。这种抽象和关注点分享简化了整个系统的语义，并且使得用户可以在不改变现有代码的前提下就能扩展集群的行为。

## Pod 更新与替换

如上所述，当工作负载资源的 pod 模板发生变更时，控制器会根据更新的模板创建新的 Pod，而不是更新或修复现有的 Pod。

Kubernetes 不会阻止直接管理 Pod。对运行中的 Pod 的某些字段 执行就地更新操作还是可能。不过，像 `patch` 和 `replace` 这类更新操作有一些限制：
- Pod 的绝大多数元数据都是不可变的。例如，不可以改变其 `namespace`、`name`、`uid`







