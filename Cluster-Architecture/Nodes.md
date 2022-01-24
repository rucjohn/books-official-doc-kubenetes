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
* `--node-labels`：在集群中注册节点时要添加的标签。
* `--node-status-update-frequency`：指定 kubelet 向控制平面发送状态的频率。

