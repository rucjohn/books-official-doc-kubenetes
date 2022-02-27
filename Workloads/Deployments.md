# Deployments

**Deployment** 为 Pod 和 ReplicaSet 提供声明式的更新能力。

Deployment 声明式配置负责描述其期望状态，Deployment 控制器以受控速率更改实际状态，使其逐渐改变成期望状态。可以定义 Deployment 来创建新的 ReplicaSet，或者删除现有 Deployment 并将其所有资源用于新的 Deployment。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

不用管理由 Deployment 创建的 ReplicaSet。 如果存在以下未覆盖的使用场景，可以考虑在 Kubernetes github仓库中提 Issue。
{% endhint %}

## 用例

以下是 Deployments 的典型用例：

* [创建 Deployment 同时自动创建 ReplicaSet](Deployments.md#creating-a-deployment)。 ReplicaSet 在后台创建 Pod。 检查 ReplicaSet 的使用状态，查看其是否成功。
* 通过更新 Deployment 的 PodTemplateSpec 来声明 Pod 的新状态 。 新的 ReplicaSet 被创建，并且 Deployment 以受控的速率将 Pod 从旧 ReplicaSet 迁移到新 ReplicaSet。 每个新的 ReplicaSet 都会更新 Deployment 的修订版本。
* 如果 Deployment 的当前状态不稳定，则可以[回滚到较早的 Deployment 版本](Deployments.md#rolling-back-a-deployment)。 每次回滚都会更新 Deployment 的修订版本。
* [扩大 Deployment 规模以承担更多负载](Deployments.md#scaling-a-deployment)。
* [暂停 Deployment ](Deployments.md#pausing-and-resuming-a-deployment)，以应用 PodTemplateSpec 所作的多项修改， 然后恢复，它将开始新的部署。
* [使用 Deployment 状态](Deployments.md#deployment-status) 来判断部署是否完成。
* [清理较旧的、不再需要的 ReplicaSet](Deployments.md#clean-up-policy) 。






## 清理策略 <a href="#clean-up-policy" id="clean-up-policy"></a>

你可以在 Deployment 中设置 `.spec.revisionHistoryLimit` 字段以指定保留此 Deployment 的多少个旧有 ReplicaSet。其余的 ReplicaSet 将在后台被垃圾回收。 默认情况下，此值为 10。

显式将此字段设置为 0 将导致 Deployment 的所有历史记录被清空，因此 Deployment 将无法回滚。

## 金丝雀部署 <a href="#canary-deployment" id="canary-deployment"></a>

如果要使用 Deployment 向用户子集或服务器子集上线版本，则可以遵循 [资源管理](../zh/docs/concepts/cluster-administration/manage-deployment/#canary-deployments) 所描述的金丝雀模式，创建多个 Deployment，每个版本一个。

