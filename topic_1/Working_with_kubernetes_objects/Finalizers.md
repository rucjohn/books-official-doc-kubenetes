# Finalizers

Finalizers 是带命名空间的键，告诉 Kubernetes 等到特定的条件被满足后，再完全删除被标记为删除的资源。Finalizer 提醒控制器清理已删除对象拥有的资源。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>控制器通过 apiserver 监控集群的公共状态，并致力于将当前状态转变来期望状态。
{% endhint %}

当告诉 Kubernetes 删除了一个指定 Finalizer 的对象时，Kubernetes API 会将该对象标记为删除，使其进入只读状态。此时控制平面或其他组件会采取 Finalizer 所定义的行动，而目标对象仍然处于终止中（`Terminating`）的状态。这些行动完成后，控制器会删除目标对象的 Finalizer。当 `metadata.finalizers` 为空时，Kubernetes 认为删除已完成。

可以使用 Finalizer 控制资源的垃圾收集。例如，可以定义一个 Finalizer，在删除目标资源前清理相关资源或基础设施。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

垃圾收集：Kubernetes 用于清理集群资源的各种机制的统称
{% endhint %}

可以通过使用 Finalizer 提醒控制器在删除目标资源前执行特定的清理任务，来控制资源的垃圾收集。

Finalizers 通常不指定要执行的代码。相反，它们通常是特定资源上的键的列表，类似于 Annotations。Kubernetes 自动指定了一些 Finalizers，也可以自定义。

## Finalizers 如何工作

1. 使用清单文件创建资源时，可以在 `metadata.finalizers` 字段指定 Finalizers。当试图删除该资源时，管理该资源的控制器会注意到 `finalizers` 字段中的值，并进行以下操作：
  - 修改对象，将开始执行删除的时间添加到 `metadata.deletionTimestamp` 字段。
  - 将该对象标记为只读，直到其 `metadata.finalizers` 字段为空。
2. 控制器试图满足资源的 Finalizers 的条件。
  - 每当一个 Finalizer 条件被满足时，控制器就会从资源的 `finalizers` 字段中删除该键。
  - 当该 `finalizers` 字段为空时，垃圾收集继续进行。

也可以使用 Finalizers 来防止删除非托管资源。一个常见的 Finalizer 例子是 `kubernetes.ip/pv-protection`，它用来防止意外删除 `PersistentVolume` 对象。
  - 当一个 `PersistentVolume` 对象被 Pod 使用时，Kubernetes 会添加 `pv-protection` Finalizer。
  - 当试图删除 `PersistentVolume`时，它将进入 `Terminating` 状态，但是控制器判断该 Finalizer 存在而无法删除该资源。
  - 当 Pod 停止使用 `PersistentVolume` 时，Kubernetes 清除 `pv-protection` Finalizer，控制器就可以删除该卷。




