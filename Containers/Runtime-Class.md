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

RuntimeClass 资源当前只有两个重要的字段：
- RuntimeClass名（`metadata.name`）
- handler（`handler`）


对象定义如下：
```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: myclass                 # 用来引用的 RuntimeClass 名
handler: myconfiguration        # 对应的 CRI 配置名
```

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

建立将 RuntimeClass 写操作（create、update、patch 和 delete）限定于集群管理员使用。通常这是默认配置。
{% endhint %}

## 使用说明

一旦完成集群中 RuntimeClass 的配置，使用起来非常方便。在 Pod spec 中指定 `runtimeClassName` 即可。例如：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  runtimeClassName: myclass
  # ...
```

这一设置会告诉 kubelet 使用所指的 RuntimeClass 来运行该 pod。如果所指的 RuntimeClass 不存在或者 CRI 无法运行相应的 handler，那么 pod 将会进入 `Failed` 终止阶段。可以查看事件获取执行过程中的错误信息。

如果未指定 `runtimeClassName`，则将使用默认的 RuntimeClassHandler，相当于禁用 RuntimeClass 功能特性。

### CRI 配置

关于如何安装 CRI 运行时，请查阅 CRI 安装。

#### dockershim

为 dockershim 设置 RuntimeClass 时，必须将运行时处理程序设置为 docker。Dockershim 不支持自定义的可配置的运行时处理程序。

#### [containerd](https://containerd.io/)

通过 containerd 的 `/etc/containerd/config.toml` 配置文件来配置运行时 handler。handler 需配置在 runtimes 块中：
```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.${HANDLER_NAME}]
```

更详细信息，请查阅 containerd 配置文档：
[https://github.com/containerd/cri/blob/master/docs/config.md](https://github.com/containerd/cri/blob/master/docs/config.md)

#### [cri-o](https://cri-o.io/)

通过 cri-o 的 `/etc/crio/crio.conf` 配置文件来配置运行时 handler。handler 需要配置在 crio.runtime 表下面：
```
[crio.runtime.runtimes.${HANDLER_NAME}]
  runtime_path = "${PATH_TO_BINARY}"
```

更详细信息，请查阅 [CRI-O 配置文档](https://github.com/cri-o/cri-o/blob/main/docs/crio.conf.5.md)。

## 调度

**FEATURE STATE:** Kubernetes v1.16 [beta]

通过为 RuntimeClass 指定 `scheduling` 字段，可以通过设置约束，确保运行该 RuntimeClass 的 Pod 被调度到支持该 RuntimeClass 的节点上。如果未设置 `scheduling`，则假定所有节点均支持此 RuntimeClass。

为了确保 pod 会被调度到支持指定运行的 node 上，每个 node 需要设置一个能用的 lable 用于被 `runtimeclass.scheduling.nodeSelector` 挑选。在 admission 阶段，RuntimeClass 的 nodeSelector 将会与 pod 的 nodeSelector 合并，取二者的交集。如果有冲突，pod 将会被拒绝。

如果 node 需要阻止 某些需要特定 RuntimeClass 的 pod ，可以在 `tolerations` 中指定。与 `nodeSelector` 一样，tolerations 也在 admission 阶段与 pod 的 tolerations 合并，取二者的并集。

### Pod 开销

**FEATURE STATE:** Kubernetes v1.18 [beta]

可以指定与运行 Pod 相关的开销资源。声明开销即允许集群（包含调度器）在决策 Pod 和资源时将其考虑在内。若要使用 Pod 开销特性，必须确保 PodOverhead 特性门控 处于启动状态（默认为启动状态）。

Pod 开销通过 RuntimeClass 的 `overhead` 字段定义。通过使用这些字段 ，可以指定使用该 RuntimeClass 运行 Pod 时的开销并确保 Kubernetes 将这些开销计算在内。








