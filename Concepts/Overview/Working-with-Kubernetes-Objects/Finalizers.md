# Finalizers

Finalizers 是带命名空间的键，告诉 Kubernetes 等到特定的条件被满足后，再完全删除被标记为删除的资源。Finalizer 提醒控制器清理已删除对象拥有的资源。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>控制器通过 apiserver 监控集群的公共状态，并致力于将当前状态转变来期望状态。
{% endhint %}

当告诉 Kubernetes 需要删除一个指定了 Finalizer 的对象时：

1. Kubernetes API 通过填充 `.metadata.deletionTimestamp` 来标记要删除的对象，并返回 `202` 状态（HTTP 表示“已接受”）使其进入只读状态。
2. 此时控制平面或其他组件会采取 Finalizer 所定义的动作，而目标对象仍然处于终止中（`Terminating`）的状态。
3. 这些行动完成后，控制器会删除目标对象的 Finalizer。
4. 当 `metadata.finalizers` 为空时，Kubernetes 认为删除已完成并删除对象。

可以使用 Finalizer 控制资源的垃圾收集。例如，可以定义一个 Finalizer，在删除目标资源前清理相关资源或基础设施。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

垃圾收集：Kubernetes 用于清理集群资源的各种机制的统称
{% endhint %}

可以通过使用 Finalizer 提醒控制器在删除目标资源前执行特定的清理任务，来控制资源的垃圾收集。

Finalizers 通常不指定要执行的代码。相反，它们通常是特定资源上的键的列表，类似于 Annotations。Kubernetes 自动指定了一些 Finalizers，也可以自定义。

## Finalizers 如何工作

（1）使用清单文件创建资源时，可以在 `metadata.finalizers` 字段指定 Finalizers。当试图删除该资源时，管理该资源的控制器会注意到 `finalizers` 字段中的值，并进行以下操作：

* 修改对象，将开始执行删除的时间添加到 `metadata.deletionTimestamp` 字段。
* 将该对象标记为只读，直到其 `metadata.finalizers` 字段为空。

（2）控制器试图满足资源的 Finalizers 的条件。

* 每当一个 Finalizer 条件被满足时，控制器就会从资源的 `finalizers` 字段中删除该键。
* 当该 `finalizers` 字段为空时，垃圾收集继续进行。

也可以使用 Finalizers 来防止删除非托管资源。一个常见的 Finalizer 例子是 `kubernetes.ip/pv-protection`，它用来防止意外删除 `PersistentVolume` 对象。

* 当一个 `PersistentVolume` 对象被 Pod 使用时，Kubernetes 会添加 `pv-protection` Finalizer。
* 当试图删除 `PersistentVolume`时，它将进入 `Terminating` 状态，但是控制器判断该 Finalizer 存在而无法删除该资源。
* 当 Pod 停止使用 `PersistentVolume` 时，Kubernetes 清除 `pv-protection` Finalizer，控制器就可以删除该卷。

## 属主引用、标签和 Finalizers

与标签类似，属主引用描述了 Kubernetes 中对象之间的关系，但它们作用不同。

当一个控制器管理类似于 Pod 的对象时，它使用标签来跟踪相关对象组的变化。例如，当 Job 创建一个或多个 Pod 时，Job 控制器会给这些 Pod 应用上标签，并跟踪集群中的具有相同标签的 Pod 变化。Job 控制器还为这些 Pod 添加了属主引用，指向创建 Pod 的 Job。如果在这些 Pod 运行的时候删除了 Job，Kubernetes 会使用属主引用（而不是标签）来确定集群中哪些 Pod 需要清理。

当 Kubernetes 识别到要删除的资源上的属主引用时，它也会处理 Finalizers。

在某些情况下， Finalizers 会阻止依赖对象的删除，这可能导致目标属主对象，保持在只读状态下的时间比预期的长，且没有被完全删除。在这些情况下，应该检查目标属主和附属对象上的 Finalizers 和属主引用，来排查原因。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

当对象卡在删除状态下时，尽量避免为允许继续删除操作而手动移除 Finalizers。 Finalizers 通常是因特殊原因被添加到资源上的，所以强行删除它们会导致集群出现问题。
{% endhint %}
