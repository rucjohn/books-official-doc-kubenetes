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

* [地址](Nodes.md#di-zhi)
* [状况](Nodes.md#zhuang-kuang)
* [容量与可分配](Nodes.md#rong-liang-yu-ke-fen-pei)
* [信息](Nodes.md#xin-xi)

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

### 地址

以下字段的用法取决于云服务商或物理机配置。

* `HostName`：由节点的内核设置。可以通过 kubelet 的 `--hostname-override` 参数覆盖。
* `ExternalIP`：通常是节点的可外部路由（由集群外可访问的）的 IP 地址。
* `InternalIP`：通常是节点的仅可在集群内部路由的 IP 地址。

### 状况

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

* 如果 Ready 条件的 `status` 处于 `Unknown` 或 `False` 状态的时间超过了 `pod-eviction-timeout` 值（一个传递给 kube-controller-manager 的参数），[节点控制器](Nodes.md#jie-dian-kong-zhi-qi) 会对节点上的所有 Pod 触发 驱逐 API 。默认的逐出超时时长为 **5分钟**。
* 某些情况下，当节点不可达时， apiserver 不能和节点的 kubelet 通信。在重新与 apiserver 建立连接之前，无法将删除 pod 的消息发送到 kubelet。与此同时，被计划删除的 Pod 可能会继续在游离的节点上运行。

节点控制器不会强制删除 Pod，直到确认它们已经在集群中停止运行。可以看到这些可能在无法访问的节点上运行的 Pod 处于 `Terminating` 或 `Unknown` 状态。如果节点永久离开集群，Kubernetes 无法从底层基础设施推断出，集群管理员可能需要手动删除节点对象。从 Kubernetes 中删除节点对象，将导致节点上的所有运行的 Pod 对象从 apiserver 中删除并释放它们的名称。

当节点出现问题时，Kubernetes 控制平面会自动创建与影响节点的条件相匹配的 [污点](Nodes.md)，当调度器将 Pod 指派给某节点时，会考虑节点上的污点。Pod 则可以通容忍度（Toleration），表达所能容忍的污点。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

API 发起的驱逐是一个先调用 Eviction API 创建驱逐对象，再由该对象体面地中止 Pod 的过程。
{% endhint %}

### 容量与可分配

### 信息

## 心跳

## 节点控制器
