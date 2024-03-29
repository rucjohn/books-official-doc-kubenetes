# 命名空间（Namespace）

在 Kubernetes 中，**命名空间（Namespace）**提供一种机制，将同一集群中的资源划分为相互隔离的组。同一命名空间内的资源名称必须唯一，但跨命名空间时没有这个要求。命名空间的作用域仅针对带有命名空间的对象，例如 Deployment、Service 等，这种作用域对集群访问的对象不适用，例如 StorageClass、Node、PersistentVolume等。

## 何时使用多个命名空间？

命名空间适用于存在很多跨多个团队或项目的用户的场景。

命名空间为名称提供了一个范围。资源的名称在命名空间内需要是唯一，不能跨命名空间。命名空间不能相互嵌套，每个 Kubernetes 资源只能存在于一个命名空间中。

命名空间是在多个用户之间划分集群资源的一种方法（通过 [资源配额](../../Policies/Resource-Quotas.md)）。

不必使用多个命名空间来划分仅仅轻微不同的资源，比如，同一软件不同版本；应该使用标签来区分同一命名空间的不同资源。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

对于生产集群，请尽量不要使用 `default` 命名空间，建议创建新的命名空间来使用。
{% endhint %}

## 初始命名空间

Kubernetes 启动时全创建四个初始命名空间：

<mark style="color:orange;">**default**</mark>

Kubernetes 默认命名空间，无需创建新的命名空间也可以使用新集群。

<mark style="color:orange;">**kube-node-lease**</mark>

该命名空间包含用于与各个节点关联的 Lease（租约）对象。节点租约允许 kubelet 发送心跳，由此控制平台能够检测到节点故障。

<mark style="color:orange;">**kube-public**</mark>

所有的客户端（包括未经身份认证的客户端）都可以读取该命名空间。该命名空间主要预留为集群使用。以便某些资源需要在整个集群中可见可读。该命名空间的公共属性只是一种约定而非要求。

<mark style="color:orange;">**kube-system**</mark>

该命名空间用于 Kubernetes 系统创建的对象。



## 使用命名空间

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>避免使用 <mark style="color:orange;">**kube-**</mark> 前缀创建命名空间，因为它是为 Kubernetes 系统命令空间预留的。
{% endhint %}

### 查看命名空间

使用以下命令列出集群中所有的命名空间：

```bash
kubctl get namespace
```

输出：

```
NAME                STATUS    AGE
default             Active    1d
kube-node-lease     Active    1d
kube-system         Active    1d
kube-publie         Active    1d
```

### 设置命名空间

为当前请求设置命名空间，请使用 `--namespace` 或 `-n` 参数。例如：

```bash
kubectl run nginx --image=nginx --namespace=<命名空间>
kubectl get pods --namespace=<命名空间>
```

### 设置命名空间偏好

在指定上下文中持久性保存命名空间，供所有后续 `kubectl` 命令使用。

```bash
kubectl config set-context --current --namespace=<命名空间>
# 验证
kubectl config view | grep namespace:
```

## 命名空间与 DNS

当创建一个 Service 时，Kubernetes 会创建一个相应的 DNS 条目，格式为：`<Service>.<Namespace>.svc.cluster.local`。

* 如果容器使用 `<Service>`，则将解析到容器所在命名空间中的 Service；
* 如果容器使用 `<Service>.<Namespace>.svc.cluster.local`，则允许跨命名空间（如：开发、测试和生产），将解析到 `<Namespace>` 下的 Service。如果希望跨命名空间访问，则需要使用完全限定域名（FQDN）。

因此，所有的命名空间名称都必须是合法的 [RFC 1123 DNS 标签](Object-Names-and-IDs.md#rfc-1123-label-names)。

{% hint style="danger" %}
<mark style="color:red;">**警告：**</mark>

通过创建与[公共顶级域名](https://data.iana.org/TLD/tlds-alpha-by-domain.txt)同名的命名空间，这些命名中的服务可以拥有与公共 DNS 记录重叠的、较短的 DNS 名称。所有命名空间中的负载在执行 DNS 查找时，如果查找的名称没有[尾部句点](https://datatracker.ietf.org/doc/html/rfc1034#page-8)，就会被重定向到这些服务上，因此呈现出比公共 DNS 更高的优先序列。

为了缓解这类问题，需要将创建命名空间的权限授予可信的用户。如果需要，可以额外部署第三方的安全控制机制，例如以准入 Webhook 的形式，阻止用户创建与[公共 TLD](https://data.iana.org/TLD/tlds-alpha-by-domain.txt) 同名的命名空间。
{% endhint %}

##

## 有没有对象不在命名空间中？

大多数 Kubernetes 资源（如：Pod、Service、Deployment等）都位于某个命名空间中，但是命名空间资源自身并不在命名空间中。而且底层资源，例如 **节点（Node）** 和 **持久卷（PersistentVolume）** 不属于任何命名空间。

查看 Kubernetes 资源对应的命名空间情况：

```bash
# 位于命名空间中的资源
kubectl api-resources --namespace=true
# 不在命名空间中的资源
kubectl api-resources --namespace=false
```

## 自动标签

**特性状态：**<mark style="color:orange;">Kubernetes 1.21 \[beta]</mark>

Kubernetes 控制平面会为所有命名空间设置一个不可变更的标签：`kubernetes.io/metadata.name`，只要 `NamespaceDefaultLabelName` 特性被启用，标签的值就是命名空间的名称。
