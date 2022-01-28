# 节点

Kubernetes 通过将容器放在节点（_**Node**_）上运行的 Pod 中来执行工作负载。

节点可以是一个虚拟机或者物理机器，取决 于所在的集群配置。每个节点包含运行 Pods 所需的服务；这些节点由控制平面负责管理。

通常集群中会有若干个节点，而如果是在一个学习或在测试的环境中，集群中也允许只有一个节点。

节点上的组件包含 kubelet、容器运行时以及 kube-proxy。

## 管理

有两种方式向 apiserver 添加节点：

1. 节点上的 `kubelet` 向控制平面注册；
2. 手动添加一个 Node 对象。

当创建了 Node 对象或者节点上的 `kubelet` 执行了自注册操作之后，控制平面会检查新的 Node 对象是否合法。例如，如果使用下面的配置来创建 Node 对象：

```json
{
  "kind": "Node",
  "apiVersion": "v1",
  "metadata": {
    "name": "10.240.79.157",
    "labels": {
      "name": "my-first-k8s-node"
    }
  }
}
```

Kubernetes 会在内部创建一个 Node 对象作为节点的表示。Kubernetes 检查 `kubelet` 向 apiserver 注册节点时使用的 `metadata.name` 字段是否匹配。

* 如果节点是健康的（即所有必要的服务都在运行中），则该节点可以用来运行 Pod。
* 否则，直到就放假节点变为健康之前，所有的集群活动都会忽略该节点。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

Kubernetes 保留无效节点的对象，并持续检查该节点的健康状态。只有当人为或者控制器显式地删除该 Node 对象，才会停止健康检查操作。
{% endhint %}

Node 对象的名称必须是合法的 [DNS 子域名](../Overview/Working-with-kubernetes-objects/Object-Names-and-IDs.md)。

### 节点自注册

当 kubelet 标志 `--register-node=true`（默认）时，它会尝试向 apiserver 注册自己。这是首选模式，被绝大多数发行版选用。

对于自注册模式，kubelet 使用下列参数启动：

* `--kubeconfig`：用于向 apiserver 表明身份的凭据路径。
* `--cloud-provider`：与某个云驱动进行通信以读取与自身相关的元数据的方式。
* `--register-node`：自动向 apiserver 注册。
* `--register-with-taints`：使用所给的污点列表（逗号分隔的 =:）注册节点。当 `--register-node=false` 时无效。
* `--node-ip`：节点 IP 地址。
* `--node-labels`：在集群中注册节点时要添加的标签。（参见 [NodeRestriction 准入控制器](../API/API-Access-Control/Using-Admission-Controllers.md#noderestriction) 所实施的标签限制）
* `--node-status-update-frequency`：指定 kubelet 向控制平面发送状态的频率。

启用 [节点授权模式](../API/API-Access-Control/Using-Node-Authorization.md) 和 [NodeRestriction 准入控制器](../API/API-Access-Control/Using-Admission-Controllers.md#noderestriction) 时，仅授权 `kubelet` 创建或创建其自己节点的资源。

### 手动节点管理

可以使用 `kubectl` 来创建和修改 Node 对象。

如果希望手动创建节点对象时，请设置 kubelet 标志 `--register-node=false`。

可以修改 Node 对象（忽略 `--register-node` 参数）。例如，修改节点上的标签或标记其为不可调度。

可以结合使用节点上的标签和 Pod 上的选择器来控制调度。例如，可以限制某个 Pod 只能在符合要求的节点上运行。

如果标记节点为不可调度（unschedulable），将阻止新 Pod 调度到该节点之上，但不会影响任何已经在其上的 Pod。这是重启节点或者执行其他维护操作之前的一个有用的准备步骤。

要标记一个节点为不可调度，执行以下命令：

```bash
kubectl cordon $NODENAME
```

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

被 DaemonSet 控制器创建的 Pod 能够容忍节点的不可调度属性。DaemonSet 通常提供节点本地的服务，即使节点上的负载应用已经被腾空，这些服务也仍需运行在节点之上。
{% endhint %}

## 节点状态

一个节点的状态包含以下信息：

* [地址](Nodes.md#addresses)
* [状况](Nodes.md#conditions)
* [容量与可分配](Nodes.md#capacity-and-allocatable)
* [信息](Nodes.md#info)

可以使用 `kubectl` 来查看节点状态和其他细节：

```bash
kubectl describe node <节点名称>
```

例如，输出以下内容：

```
Name:               master-1
Roles:              control-plane,master,worker
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=master-1
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node-role.kubernetes.io/master=
                    node-role.kubernetes.io/worker=
                    test=ok
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: 192.168.124.101/24
                    projectcalico.org/IPv4IPIPTunnelAddr: 10.233.106.0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Mon, 11 Oct 2021 17:13:48 +0800
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  master-1
  AcquireTime:     <unset>
  RenewTime:       Wed, 26 Jan 2022 10:38:12 +0800
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Wed, 12 Jan 2022 17:37:15 +0800   Wed, 12 Jan 2022 17:37:15 +0800   CalicoIsUp                   Calico is running on this node
  MemoryPressure       False   Wed, 26 Jan 2022 10:34:06 +0800   Thu, 13 Jan 2022 12:41:32 +0800   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Wed, 26 Jan 2022 10:34:06 +0800   Fri, 10 Dec 2021 17:51:05 +0800   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Wed, 26 Jan 2022 10:34:06 +0800   Fri, 10 Dec 2021 17:51:05 +0800   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Wed, 26 Jan 2022 10:34:06 +0800   Fri, 10 Dec 2021 17:51:05 +0800   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  192.168.124.101
  Hostname:    master-1
Capacity:
  cpu:                2
  ephemeral-storage:  48002152Ki
  hugepages-2Mi:      0
  memory:             5944880Ki
  pods:               110
Allocatable:
  cpu:                1600m
  ephemeral-storage:  48002152Ki
  hugepages-2Mi:      0
  memory:             5258891260
  pods:               110
System Info:
  Machine ID:                 845be1f7030d4956bd2cda32a605526c
  System UUID:                845BE1F7-030D-4956-BD2C-DA32A605526C
  Boot ID:                    b7f04b50-e911-4a54-a3bc-8182cb46dce8
  Kernel Version:             3.10.0-957.el7.x86_64
  OS Image:                   CentOS Linux 7 (Core)
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  docker://20.10.8
  Kubelet Version:            v1.20.4
  Kube-Proxy Version:         v1.20.4
PodCIDR:                      10.233.64.0/24
PodCIDRs:                     10.233.64.0/24
Non-terminated Pods:          (8 in total)
  Namespace                   Name                                CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                   ----                                ------------  ----------  ---------------  -------------  ---
  default                     nginx111-55b8654cf7-fdkf8           0 (0%)        0 (0%)      0 (0%)           0 (0%)         26d
  kube-system                 calico-node-vp47z                   250m (15%)    0 (0%)      0 (0%)           0 (0%)         36d
  kube-system                 kube-apiserver-master-1             250m (15%)    0 (0%)      0 (0%)           0 (0%)         35d
  kube-system                 kube-controller-manager-master-1    200m (12%)    0 (0%)      0 (0%)           0 (0%)         36d
  kube-system                 kube-proxy-lpqh5                    0 (0%)        0 (0%)      0 (0%)           0 (0%)         36d
  kube-system                 kube-scheduler-master-1             100m (6%)     0 (0%)      0 (0%)           0 (0%)         106d
  kube-system                 nodelocaldns-g2jfh                  100m (6%)     0 (0%)      70Mi (1%)        170Mi (3%)     36d
  kube-system                 snapshot-controller-0               0 (0%)        0 (0%)      0 (0%)           0 (0%)         106d
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                900m (56%)  0 (0%)
  memory             70Mi (1%)   170Mi (3%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)
Events:              <none>
```

### 地址 <a href="#addresses" id="addresses"></a>

以下字段的用法取决于云服务商或物理机配置。

* `HostName`：由节点的内核设置。可以通过 kubelet 的 `--hostname-override` 参数覆盖。
* `ExternalIP`：通常是节点的可外部路由（由集群外可访问的）的 IP 地址。
* `InternalIP`：通常是节点的仅可在集群内部路由的 IP 地址。

### 状况 <a href="#conditions" id="conditions"></a>

`conditions` 字段描述了所有 Running 节点的状态。

| 节点状况                                                  | 描述                                                                                                                                            |
| ----------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Ready                                                 | <p>True：节点是健康的，并且已经准备好接收 Pod。<br>False：节点不健康且不能接收 Pod。<br>Unknown：表示节点控制器在最近 <code>node-monitor-grace-period</code> 期间（默认 40 秒）没有收到节点的信息。</p> |
| DiskPressure                                          | True 表示节点存在磁盘空间压力，即节点可用磁盘空间低；否则为 False。                                                                                                       |
| MemoryPressure                                        | True 表示节点存在内存压力，即节点可用内存低，否则为 False                                                                                                            |
| PIDPressure                                           | True 表示节点存在进程压力，即节点上进程过程多，否则为 False                                                                                                           |
| NetworkUnavailable                                    | True 表示节点网络配置不正确，否则为 False                                                                                                                    |
| <mark style="color:orange;">SchedulingDisabled</mark> | <mark style="color:orange;">节点被保护，被标记为不可能调度</mark>                                                                                            |

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

如果使用命令行工具来打印已保护（Cordoned）节点的细节，其中 Condition 字段可能包括 SchedulingDisabled。该状况不是 Kubernetes API 中定义的 Condition，被保护的节点在其规范中被标记为不可调度（Unschedulable）。
{% endhint %}

在 Kubernetes API 中，节点的状况表示节点资源中 `.status` 的一部分。例如，以下 JSON 结构描述了一个健康节点：

```json
"conditions": [
  {
    "type": "Ready",
    "status": "True",
    "reason": "KubeletReady",
    "message": "kubelet is posting ready status",
    "lastHeartbeatTime": "2022-1-26 10:34:06",
    "lastTransitionTime": "2021-12-10T17:51:05Z",
  }
]
```

* 如果 Ready 条件的 `status` 处于 `Unknown` 或 `False` 状态的时间超过了 `pod-eviction-timeout` 值（一个传递给 kube-controller-manager 的参数），[节点控制器](Nodes.md#node-controller) 会对节点上的所有 Pod 触发 驱逐 API 。默认的逐出超时时长为 **5分钟**。
* 某些情况下，当节点不可达时， apiserver 不能和节点的 kubelet 通信。在重新与 apiserver 建立连接之前，无法将删除 pod 的消息发送到 kubelet。与此同时，被计划删除的 Pod 可能会继续在游离的节点上运行。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

API 发起的驱逐是一个先调用 Eviction API 创建驱逐对象，再由该对象体面地中止 Pod 的过程。
{% endhint %}

节点控制器不会强制删除 Pod，直到确认它们已经在集群中停止运行。可以看到这些可能在无法访问的节点上运行的 Pod 处于 `Terminating` 或 `Unknown` 状态。如果节点永久离开集群，Kubernetes 无法从底层基础设施推断出，集群管理员可能需要手动删除节点对象。从 Kubernetes 中删除节点对象，将导致节点上的所有运行的 Pod 对象从 apiserver 中删除并释放它们的名称。

当节点出现问题时，Kubernetes 控制平面会自动创建与影响节点的条件相匹配的 [污点](../Scheduling-Preemption-and-Eviction/Taints-and-Toleratiions.md)，当调度器将 Pod 指派给某节点时，会考虑节点上的污点。Pod 则可以通容忍度（Toleration），表达所能容忍的污点。

### 容量与可分配 <a href="#capacity-and-allocatable" id="capacity-and-allocatable"></a>

描述节点上的可用资源：CPU、内存和可以调度到节点上的 Pod 数量上限。

* `capacity` 块中的字段表示节点拥有的资源总量。
* `allocatable` 块中的字段表示节点上可供 Pod 消耗的资源量。

可以在 [预留计算资源](../Adminster-a-Cluster/Reserve-Compute-Resources-for-System-Daemons.md) 章节了解有关容量和可分配资源的更多信息。

### 信息 <a href="#info" id="info"></a>

`System Info` 块描述了节点的一般信息，比如内核版本、Kubernetes 版本（kubelet 和 kube-proxy 版本）、容器运行时详细信息，以及节点使用的操作系统等。

`kubelet` 从节点收集这些信息并其同步到 Kubernetes API。

## 心跳 <a href="#hearbeats" id="hearbeats"></a>

由 Kubernetes 节点发送的心跳有助于集群确定每个节点的可用性，并在检测到故障时采取措施。

对于节点，心跳有两种形式：

* 更新节点的 `.status`
* `kube-node-lease` 命令空间中的 [Lease 对象](../API/Kubernetes-API/Cluster-Resources/Lease.md)。每个节点都有一个关联的 lease 对象。

与节点的 .status 更新相比，Lease 是一种轻量级资源。使用 Leases 心跳在大型集群中可以减少这些更新对性能的影响。

kubelet 负责创建和更新的 `.status`，以及更新它们对应的 `Lease`。

* 当状态发生变化时，或者在配置的时间间隔内没有更新事件时，kubelet 会更新 `.status`。`.status` 更新的默认间隔是 **5 分钟**（比不可达节点的 40 秒默认超时时间长很多）。
* `Lease` 更新有单独的更新机制，`kubelet` 会**每 10 秒**（默认更新间隔时间）创建并更新其 Lease 对象。如果 Lease 的更新操作失败，kubelet 会采用指数回退机制，从 **200 毫秒**开始并以指数递增不断重试，最长重试间隔为 **7 秒钟**。

## 节点控制器 <a href="#node-controller" id="node-controller"></a>

节点控制器是 Kubernetes 控制平面组件，管理节点的方方面面。

节点控制器在节点的生命周期中扮演多个角色：

1. 第一个是当节点注册时为它分配一个 CIDR 区段（如果启动了 CIDR 分配）。
2. 第二个是操持节点控制器内的节点列表与云服务商所提供的可用机器列表同步。如果在云环境下运行，只要某节点不健康，节点控制器就会询问云服务是否节点的虚拟机仍可用。如果不可用，节点控制器会将该节点从它的节点列表删除。
3. 第三个是监控节点的健康状况。节点控制器将负责：

* 在节点不可达的情况下，更新节点的 `.status` 的 `NodeReady` 状况。在这种情况下，节点控制器将 `NodeReady` 状况更新为 `ConditionUnknown`。
* 如果节点仍然无法访问：对于不可达节点上的所有 Pod 触发发起驱动 API。默认情况下，节点控制器在将节点标记为 `ConditionUnknown` 后等待 **5 分钟**提交第一个驱逐请求。

节点控制器间隔 `--node-monitor-period` 参数中的秒数值，检查每个节点的状态。

### 逐出速率限制

大部分情况下，节点控制器把逐出速率限制在每秒 0.1 个（参数：`--node-eviction-rate`（默认 0.1）。这表示每 10 秒钟内最多从一个节点驱逐 Pod。

当一个可用区域（Availablitity Zone）中的节点变为不健康时，节点的驱逐行为就发生改变。节点控制器会同时检查 可用区域中不健康（NodeReady 状况为 `ConditionUnknown` 或 `ConditionFalse`）的节点的百分比：

* 如果不健康节点的比例超过 `--unhealthy-zone-threshold` （默认 0.55），驱逐速率将会降低。
* 如果集群较小（即小于等于 `--large-cluster-size-threshold` 个节点 - 默认 50），驱逐操作将会停止。
* 否则驱逐速率降为每秒 `--secondary-node-eviction-rate` 个（默认 0.01）。

在单个可用区域实现这些策略的原因，是当一个可用区域可能从控制平台脱离时，其他可用区域可能仍然保持连接。如果集群没有跨越云服务端的多个可用区域，那整个集群就只有一个可用区域。

跨多个可用区域部分的节点的一个关键原因是，当某个可用区域梦遗体出现故障时，工作负载可以转移到健康的可用区域。因此 ，如果一个可用区域中的所有节点都不健康时，节点控制器会以正常的速度 `--node-eeviction-rate` 进行驱逐操作。在所有的可用区域都不健康（即集群中没有健康节点）的极端情况下，节点控制器将假设控制平面与节点间的连接出了某些问题，它将停止所有驱逐动作（如果故障后部分节点重新连接，节点控制器就会从剩下不健康或者不可达节点中驱逐 Pod）。

节点控制器还负责驱逐运行在拥有 `NoExecute` 污点的节点上的 Pod，除非这些 Pod 能够容忍此污点。

节点控制器还负责根据节点故障（例如节点不可访问或没有就绪）为其添加污点。这意味着调度器不会将 Pod 调度到不健康的节点上。

### 资源容量跟踪

Node 对象会跟踪节点上资源的容量（例如：可用内存和 CPU 数量）。通过自注册机制生成的 Node 对象会在注册期间报告自身容量。如果手动添加了 Node，就需要在添加节点时手动设置节点容量。

Kubernetes 调度器保证节点上有足够的资源供其上的所有 Pod 使用。它会检查节点上所有容器的请求的总和不会超过节点的容量。总的请求包括由 kubelet 启动的所有容器，但不包括由容器运行时直接启动的容器，也不包括不受 kubelet 控制的其他进程。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

如果要为非 Pod 进程显式保留资源。请参考 [为系统守护进程预留资源](../Adminster-a-Cluster/Reserve-Compute-Resources-for-System-Daemons.md)。
{% endhint %}

## 节点拓扑

**FEATURE STATE:** _<mark style="color:orange;">Kubernetes v1.16 \[alpha]</mark>_

如果启用了 `TopologyManager` 特性门控，`kubelet` 可以在作出资源分配决策时使用拓扑提示。

参考 [控制节点上拓扑管理策略 ](../Adminster-a-Cluster/Control-Topology-Management-Policies-on-a-node.md)了解详细信息。

## 优雅关闭节点

**FEATURE STATE:** _<mark style="color:orange;">Kubernetes v1.21 \[beta]</mark>_

kubelet 会尝试检测节点系统关闭事件并终止在节点上运行的 Pods。

在节点终止期间，kubelet 保证 Pod 遵从常规的 [Pod 终止流程](../Workloads/Pods/Pod-Lifecycle.md#termination-of-pods)。

优雅关闭节点特性依赖于 systemd，因为它要利用 [systemd 抵制器锁](https://www.freedesktop.org/wiki/Software/systemd/inhibit/) 在给定的期限内延迟节点关闭。

优雅关闭节点特性受 `GracefulNodeShutdown` 特性门控 控制，在 1.21 版本中是默认启用的。

注意：默认情况下，下面描述的两个配置选项，`ShutdownGracePeriod` 和 `ShutdownGracePeriodCriticalPods` 都被设置为 `0`，因此不会激活优雅关闭节点功能。要激活此功能特性，这两个 kubelet 配置选项要适当配置，并设置为非零值。

在优雅关闭节点过程中，kubelet 分两个阶段来终止 Pods：

1. 终止在节点上运行的常规 Pod
2. 终止在节点上运行的[关键 Pod](../Adminster-a-Cluster/Guaranteed-Scheduling-For-Critical-Add-On-Pods.md#marking-pod-as-critical)

优雅关闭节点特性对应 两个 `KubeletConfiguration` 选项：

* `ShutdownGracePeriod`：指定节点应延迟关闭的总持续时间。此时间是 Pod 优雅终止的时间总和，不区分常规 Pod 还是 关键 Pod。
* `ShutdownGracePeriodCriticalPods`：在节点关闭期间指定用于终止关键 Pod 的持续时间。该值应小于 `ShutdownGracePeriod`.

例如，如果设置了 `ShutdownGracePeriod=30s` 和 `ShutdownGracePeriodCriticalPods=10s`，则 kubelet 将延迟 30 秒关闭节点。在关闭期间，将保留 20 （30 - 10）秒用于优雅终止常规 Pod，保留最后 10 秒用于终止关键 Pod。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

当 Pod 在正常节点关闭期间被驱逐时，它们会被标记为 `failed`。运行 `kubelet get pods` 将被驱逐的 pod 的状态显示为 `Shutdown`。并且，`kubelet describe pod` 表示 pod 因节点关闭而被驱逐。

```
Status:         Failed
Reason:         Shutdown
Message:        Node is shutting, evicting pods
```

`failed` 的 pod 对象将被保留，直到被明确 删除或 由[ GC 清理](../Workloads/Pods/Pod-Lifecycle.md#garbage-collection-of-failed-pods)。与突然的节点终止相比这是一种行为变化。
{% endhint %}

## 交换内存管理

FEATURE STATE: _<mark style="color:orange;">Kubernetes v1.22 \[alpha]</mark>_

在 Kubernetes 1.22 之前，节点不支持使用交换内存，并且默认情况下，如果在节点上检测到交换内存配置，kubelet 将无法启动。在 v1.22 之后，可以在每个节点的基础上启用交换内存支持。

要在节点上启动交换内存，必须启用 kubelet 的 `NodeSwap` 特性门控，同时使用 `--fail-swap-on` 命令行参数或将 `failSwapOn` 配置设置为 `false`。

用户还可以选择配置 `memorySwap.swapBehavior` 指定节点使用交换内存的方式。例如：

```yaml
memorySwap:
  swapBehavior: LimitedSwap
```

`swapBehavior` 的配置选项有：

* `LimitedSwap`：Kubernetes 工作负载的交换内存会受限制。不受 Kubernetes 管理的节点上的工作负载仍然可以交换。
* `UnlimitedSwap`：Kubernetes 工作负载可以使用尽可能多的交换内存请求，一直到系统限制。

如果启用了特性门控，但未指定 `memorySwap` 配置，默认情况下 kubelet 将使用 `LimitedSwap` 设置。

`LimitedSwap` 设置的行为还取决 于节点运行的是 v1 还是 v2 的控制组（也就是 `cgroup`）：

* **cgroupsv1**：Kubernetes 工作负载可以使用交换内存，达到 Pod 的内存限制（如果设置）。
* **cgroupsv2**：Kubernetes 工作负载不能使用交换内存。
