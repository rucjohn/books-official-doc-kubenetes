# 属主和附属

在 Kubernetes 中，一些对象是其他对象的属主（_**Owner**_)。例如，ReplicaSet 是一组 Pod 的属主。这些具有属主的对象就是其所有者的附属（_**Dependent**_）。

属主关系不同于一些资源使用的 [Labels 和 Selector](Labels-and-Selectors.md) 机制。例如，有一个创建 `EndpointSlice` 对象的 Service，该 Service 使用标签来让控制平面确定哪些 `EndpointSlice` 对象用于该 Service。除开 Labels 之外，每个代表 Service 所管理的 `EndpointSlice` 对象都有一个属主引用。

属主引用避免 Kubernetes 因不同机制而干扰到不受其管理控制的对象。

## 对象规范中的属主引用

附属对象有一个 `metadata.ownerReferences` 字段，用于引用其属主对象。一个有效的属主引用，由与附属对象位于同一命名空间中的对象和 UID 组成。Kubernetes 自动为一些对象的附属资源设置属主引用的值，这些对象包含 ReplicaSet、DaemonSet、Deployment、Job、CronJob、ReplicationController 等。也可以通过改变这个字段的值，来手动配置这些关系，不过通常没必要，Kubernetes 会自动管理这些附属关系。

附属对象还有一个 `ownerReferences.blockOwnerDeletion` 字段，该字段使用的是布尔值，用于控制特定的附属对象是否可以阻止垃圾回收删除其属主对象。如果控制器（例如：Deployment 控制器）设置了 `metadata.ownerReferences` 字段的值，则 Kubernetes 会自动设置 `blockOwnerDeletion=true`。也可以手动设置 `blockOwnerDeletion` 字段的值，以控制哪些附属对象允许阻止垃圾回收。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

根据设计，Kubernetes 不允许跨命名空间指定属主。命名空间范围的附属可以指定集群范围或命名空间范围的属主。命名空间范围的属主必须与附属处于相同的命名空间。如果命名空间范围的属主附属不在同一命名空间下，那么该属主引用就会被认定为缺失，并且当附属的所有属主引用都被确定不在存在之后，该附属就会被删除。
{% endhint %}

{% hint style="warning" %}
<mark style="color:orange;">**警告：**</mark>

集群范围的附属只能指定集群范围的属主。从 v1.20 版本后，如果一个集群范围的附属指定了一个命名空间范围类型的属主，那么该附属就会被认为是拥有一个不可解析的属主引用，并且它不能够被垃圾回收。
{% endhint %}

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

从 v1.20 版本后，如果垃圾回收器检测到无效的跨命名空间的属主引用，或者一个集群范围的附属指定了一个命名空间范围类型的属主，那么它就会报告一个警告事件。该事件的原因是 `OwnerRefInvalidNamespace`，`involvedObject` 属性中包含无效的附属。可以运行 `kubectl get events -A --field-selector=reson=OwnerRefinvalidNamespace` 来获取该类型的事件。
{% endhint %}

## 属主关系与 Finalizer

当告诉 Kubernetes 删除一个资源，apiserver 允许控制器处理该资源的任何 [Finalizer 规则](Finalizers.md)。Finalizer 防止意外删除集群所依赖的、用于正常动作的资源。例如，如果试图删除一个仍被 Pod 使用的 `persistentVolume`，该资源不会被立即删除，因为 `PersistentVolume` 有 `kubernetes.io/pv-protection` Finalizer。相反，它将进入 `Termination` 状态，直到 Kubernetes 清除这个 Finalizer，而这种情况只会发生在 `PersistentVolume` 不再被挂载到 Pod 上时。

当你使用孤立级联删除时，Kubernetes 也会向属主资源添加 Finalizer：

* 在前台删除中，会添加 `foreground` Finalizer，这样控制器必须在删除了拥有 `ownerReferences.blockOwnerDeletion=true` 的附属资源后，才能删除属主对象。
* 如果指定了孤立删除策略，会添加 `orphan` Finalizer，这样控制器在删除属主对象后，会忽略附属资源。
