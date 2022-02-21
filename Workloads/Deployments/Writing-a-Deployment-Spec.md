# Deployment 编写规范

同其他 Kubernetes 配置一样， Deployment 也需要 `apiVersion`，`kind` 和 `metadata` 字段。 有关配置文件的其他信息，请参考 [部署 Deployment ](../../Tasks/Run-Applications/Run-a-Stateless-Application-Using-a-Deployment.md)、配置容器和 [使用 kubectl 管理资源 ](../../Overview/Working-with-kubernetes-objects/)等相关文档。Deployment 对象的名称必须是合法的 DNS 子域名。

`.spec` 中只有两个必需的字段：`.spec.template` 和 `.spec.selector`。

## Pod 模板 <a href="#pod-template" id="pod-template"></a>

`.spec.template` 是一个 [Pod 模板](../Pods/#pod-mo-ban)。 它和 Pod 的语法规则是完全相同的。 只是在这里，它是嵌套的，因此不需要 `apiVersion` 或 `kind`。

除了 Pod 的必填字段外，Deployment 中的 Pod 模板必须指定适当的标签（_Labels_）和重启策略（_restartPolicy_）。 对于标签，请确保不要与其他控制器重叠。

仅允许 `.spec.template.spec.restartPolicy` 等于 `Always` ，如果未指定，则为默认值。

## 副本 <a href="#replicas" id="replicas"></a>

`.spec.replicas` 字段定义 Deployment 创建 Pod 副本数量。它是可选字段，默认值为 1。

## 选择器 <a href="#selector" id="selector"></a>

`.spec.selector` 字段定义 Deployment 匹配的 Pod。它是必需的。

`.spec.selector` 字段必须匹配 `.spec.template.metadata.labels`，否则请求会被 API 拒绝。

在 `apps/v1` API 版本中，`.spec.selector` 和 `.metadata.labels` 如果未配置，不会被默认设置为 `.spec.template.metadata.labels`，所以这两个字段必须进行明确设置。 同时在 `apps/v1` API 版本中，Deployment 创建后 `.spec.selector` 是不允许改变的。

{% hint style="warning" %}
<mark style="color:orange;">**注意：**</mark>

当尝试使用 <mark style="color:orange;">`kubectl edit deploy`</mark> 命令更改 Deployment <mark style="color:orange;">`.spec.selector`</mark> 字段时，编辑器会提示报错，并且无法保存更改内容。
{% endhint %}

当 Pod 的标签和选择器匹配，但其模板和 `.spec.template` 不同时，或者此类 Pod 的总数超过 `.spec.replicas` 的设置时，Deployment 则会终止它。 如果 Pod 总数未达到期望值，Deployment 会基于 `.spec.template` ，逐步创建新 Pod。

{% hint style="info" %}
<mark style="color:purple;">**说明：**</mark>

不应直接通过创建另一个 Deployment，或者创建类似 ReplicaSet 或 ReplicationController 这类控制器，来创建标签与此选择器匹配的 Pod。这样做的话，第一个 Deployment 会认为是由它创建了这些 Pod。

注意，Kubernetes 并不会阻止这么做。
{% endhint %}

如果有多个控制器的选择器发生重叠，则控制器之间会因冲突而无法正常工作。

## 策略 <a href="#strategy" id="strategy"></a>

`.spec.strategy` 字段定义新 Pod 替换旧 Pod 的策略。

`.spec.strategy.type` 字段有两种类型：

* RollingUpdate（默认值）
* Recreate

### Recreate <a href="#recreate-deployment" id="recreate-deployment"></a>

如果 `.spec.strategy.type==Recreate`，在创建新 Pods 之前，所有现有的 Pods 会被杀死。

### RollingUpdate <a href="#rolling-update-deployment" id="rolling-update-deployment"></a>

如果 `.spec.strategy.type==RollingUpdate`，Deployment 采用滚动更新的方式更新 Pod。可以指定 `maxUnavailable` 和 `maxSurge` 来控制滚动更新过程。

**maxUnavailable（最大不可用）**

`.spec.strategy.rollingUpdate.maxUnavailable` 是一个可选字段，定义更新过程中不可用的 Pod 的个数上限。该值可以是整数（例如，5），也可以是百分比（例如，10%）。如果是百分比的话，值会转换成整数（有小数时去除小数部分）。 如果 `.spec.strategy.rollingUpdate.maxSurge` 为 0，则此值不能为 0。 默认值为 25%。

例如，当此值设置为 30% 时，滚动更新开始时会立即将旧 ReplicaSet 缩容到期望 Pod 个数的 70%。 新 Pod 准备就绪后，可以继续对旧的 ReplicaSet 缩容，然后对新的 ReplicaSet 扩容，确保在更新期间可用的 Pod 总数在任何时候都至少是所需的 Pod 个数的 70%。

**maxSurge（最大峰值）**

`.spec.strategy.rollingUpdate.maxSurge` 是一个可选字段，定义可以创建的超出期望 Pod 个数的 Pod 数量。此值可以是整数（例如，5），也可以是百分比（例如，10%）。 如果是百分比的话，值会转换成整数（有小数时去除小数部分）。如果 `.spec.strategy.rollingUpdate.maxUnavailable` 为 0，则此值不能为 0。默认值为 25%。

例如，当此值为 30% 时，启动滚动更新后，会立即对新的 ReplicaSet 扩容，同时保证新旧 Pod 的总数不超过 Pod 总数的 130%。一旦旧 Pods 被杀死，新的 ReplicaSet 可以进一步扩容， 同时确保更新期间的任何时候运行中的 Pods 总数最多为 Pods 总数的 130%。

## progressDeadlineSeconds <a href="#progress-deadline-seconds" id="progress-deadline-seconds"></a>

`.spec.progressDeadlineSeconds` 是一个可选字段，定义在系统报告 Deployment 的 [failed progressing](Writing-a-Deployment-Spec.md)（表现为 resource 的状态中 `Type=Progressing`、`Status=False`、 `Reason=ProgressDeadlineExceeded`）之前可以等待的 Deployment 进行的秒数。 Deployment 控制器将持续重试 Deployment。 未来，在实现了自动回滚后，Deployment 控制器将在探测到这样的条件时立即回滚。

如果设置该参数，该值必须大于 `.spec.minReadySeconds`。

## minReadySeconds <a href="#min-ready-seconds" id="min-ready-seconds"></a>

`.spec.minReadySeconds` 是一个可选字段，定义新创建的 Pod 在没有任何容器崩溃情况下的最小就绪前的等待时间，只有超出这个时间 Pod 才被视为已就绪。默认值为 0。

## 修订历史限制

Deployment 的修订历史记录存储在它所控制的 ReplicaSets 中。

`.spec.revisionHistoryLimit` 是一个可选字段，定义可以保留的旧的 ReplicaSet 数量。资源都存储在 etcd 中，并占用 `kubectl get rs` 的输出。 每个 Deployment `revisionHistoryLimit` 的配置都存储在其 ReplicaSets 中；因此，一旦删除了旧的 ReplicaSet， 将失去回滚到 Deployment 对应版本的能力。 默认情况下，系统保留 10 个旧 ReplicaSet，但其理想值取决于新 Deployment 的频率和稳定性。

更具体地说，如果将该值设置为0，所有具有 0 个副本的 ReplicaSet 都会被删除。在这种情况下，Deployment rollout 无法撤销，因为其历史版本都被清理掉了。

## paused <a href="#paused" id="paused"></a>

`.spec.paused` 是一个可选字段，布尔值。用来指定暂停和恢复 Deployment。 paused 和 没有 paused 的 Deployment 之间的唯一区别是，所有对 paused Deployment 中的 PodTemplateSpec 的修改都不会触发新的 rollout。Deployment被创建之后默认是非 paused。
