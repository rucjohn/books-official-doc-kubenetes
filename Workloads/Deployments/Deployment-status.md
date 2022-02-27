
# Deployment 状态 <a href="#deployment-status" id="deployment-status"></a>

Deployment 在其生命周期中会有许多状态，上线时新的 ReplicaSet 期间可能处于 `Progressing`（进行中），可能是 `Complete`（已完成），也可能是 `Failed`（失败）。

## Progressing（进行中） <a href="#progressing-deployment" id="progressing-deployment"></a>

执行以下任务时，Kubernetes 会将 Deployment 标记为 _Progressing（进行中）_：

* Deployment 创建新的 ReplicaSet
* Deployment 正在为其最新的 ReplicaSet 扩容
* Deployment 正在为其旧有的 ReplicaSet 缩容
* 新的 Pods 已经就绪或者可用（就绪至少持续了 [MinReadySeconds](Deployments.md#min-ready-seconds) 秒）

当上线状态变为 `Progressing`，Deployment 控制器将具有以下属性的状况添加到 Deployment 的 `.status.conditions`：

* `type: Progressing`
* `status: "True"`
* `reson: NewReplicaSetCreated` 或 `reson: FoundNewReplicaSet` 或 `reson: ReplicaSetUpdated`

可以使用 `kubectl rollout status` 监控 Deployment 的进度。

## Complete（已完成） <a href="#complete-deployment" id="complete-deployment"></a>

当 Deployment 具有以下特征时，Kubernetes 将其标记为 _Complete（已完成）_：

* 与 Deployment 关联的所有副本都已更新到指定的最新版本，即之前请求的所有更新都已完成。
* 与 Deployment 关联的所有副本都可用。
* 没有运行 Deployment 的旧副本。

当上线状态变为 `Complete`，Deployment 控制器将具有以下属性的状况添加到 Deployment 的 `.status.conditions`：

* `type: Progressing`
* `status: "True"`
* `reson: NewReplicaSetAvailable`

你可以使用 `kubectl rollout status` 检查 Deployment 是否已完成。 如果上线成功完成，`kubectl rollout status` 返回退出代码 0。

```shell
kubectl rollout status deployment/nginx-deployment
```

输出类似于：

```shell
Waiting for rollout to finish: 2 of 3 updated replicas are available...
deployment "nginx-deployment" successfully rolled out
$ echo $?
0
```

## Failed（失败） <a href="#failed-deployment" id="failed-deployment"></a>

你的 Deployment 可能会在尝试部署其最新的 ReplicaSet 受挫，一直处于未完成状态。 造成此情况一些可能因素如下：

* 配额（Quota）不足
* 就绪探测（Readiness Probe）失败
* 镜像拉取错误
* 权限不足
* 限制范围（Limit Ranges）问题
* 应用程序运行时的配置错误

检测此状况的一种方法是在 Deployment 规约中指定截止时间参数： （\[`.spec.progressDeadlineSeconds`]（#progress-deadline-seconds））。 `.spec.progressDeadlineSeconds` 给出的是一个秒数值，Deployment 控制器在（通过 Deployment 状态） 标示 Deployment 进展停滞之前，需要等待所给的时长。

以下 `kubectl` 命令设置规约中的 `progressDeadlineSeconds`，从而告知控制器 在 10 分钟后报告 Deployment 没有进展：

```shell
kubectl patch deployment.v1.apps/nginx-deployment -p '{"spec":{"progressDeadlineSeconds":600}}'
```

输出类似于：

```
deployment.apps/nginx-deployment patched
```

超过截止时间后，Deployment 控制器将添加具有以下属性的 DeploymentCondition 到 Deployment 的 `.status.conditions` 中：

* Type=Progressing
* Status=False
* Reason=ProgressDeadlineExceeded

参考 [Kubernetes API 约定](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#typical-status-properties) 获取更多状态状况相关的信息。

除了报告 `Reason=ProgressDeadlineExceeded` 状态之外，Kubernetes 对已停止的 Deployment 不执行任何操作。更高级别的编排器可以利用这一设计并相应地采取行动。 例如，将 Deployment 回滚到其以前的版本。

如果你暂停了某个 Deployment，Kubernetes 不再根据指定的截止时间检查 Deployment 进展。 你可以在上线过程中间安全地暂停 Deployment 再恢复其执行，这样做不会导致超出最后时限的问题。

Deployment 可能会出现瞬时性的错误，可能因为设置的超时时间过短， 也可能因为其他可认为是临时性的问题。例如，假定所遇到的问题是配额不足。 如果描述 Deployment，你将会注意到以下部分：

```shell
kubectl describe deployment nginx-deployment
```

输出类似于：

```
<...>
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     True    ReplicaSetUpdated
  ReplicaFailure  True    FailedCreate
<...>
```

如果运行 `kubectl get deployment nginx-deployment -o yaml`，Deployment 状态输出 将类似于这样：

```
status:
  availableReplicas: 2
  conditions:
  - lastTransitionTime: 2016-10-04T12:25:39Z
    lastUpdateTime: 2016-10-04T12:25:39Z
    message: Replica set "nginx-deployment-4262182780" is progressing.
    reason: ReplicaSetUpdated
    status: "True"
    type: Progressing
  - lastTransitionTime: 2016-10-04T12:25:42Z
    lastUpdateTime: 2016-10-04T12:25:42Z
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: 2016-10-04T12:25:39Z
    lastUpdateTime: 2016-10-04T12:25:39Z
    message: 'Error creating: pods "nginx-deployment-4262182780-" is forbidden: exceeded quota:
      object-counts, requested: pods=1, used: pods=3, limited: pods=2'
    reason: FailedCreate
    status: "True"
    type: ReplicaFailure
  observedGeneration: 3
  replicas: 2
  unavailableReplicas: 2
```

最终，一旦超过 Deployment 进度限期，Kubernetes 将更新状态和进度状况的原因：

```
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     False   ProgressDeadlineExceeded
  ReplicaFailure  True    FailedCreate
```

可以通过缩容 Deployment 或者缩容其他运行状态的控制器，或者直接在命名空间中增加配额 来解决配额不足的问题。如果配额条件满足，Deployment 控制器完成了 Deployment 上线操作， Deployment 状态会更新为成功状况（`Status=True` and `Reason=NewReplicaSetAvailable`）。

```
Conditions:
  Type          Status  Reason
  ----          ------  ------
  Available     True    MinimumReplicasAvailable
  Progressing   True    NewReplicaSetAvailable
```

`Type=Available` 加上 `Status=True` 意味着 Deployment 具有最低可用性。 最低可用性由 Deployment 策略中的参数指定。 `Type=Progressing` 加上 `Status=True` 表示 Deployment 处于上线过程中，并且正在运行， 或者已成功完成进度，最小所需新副本处于可用。 请参阅对应状况的 Reason 了解相关细节。 在我们的案例中 `Reason=NewReplicaSetAvailable` 表示 Deployment 已完成。

你可以使用 `kubectl rollout status` 检查 Deployment 是否未能取得进展。 如果 Deployment 已超过进度限期，`kubectl rollout status` 返回非零退出代码。

```shell
kubectl rollout status deployment/nginx-deployment
```

输出类似于：

```
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
error: deployment "nginx" exceeded its progress deadline
```

`kubectl rollout` 命令的退出状态为 1（表明发生了错误）：

```shell
$ echo $?
```

```
1
```

### 对失败 Deployment 的操作 <a href="#operating-on-a-failed-deployment" id="operating-on-a-failed-deployment"></a>

可应用于已完成的 Deployment 的所有操作也适用于失败的 Deployment。 你可以对其执行扩缩容、回滚到以前的修订版本等操作，或者在需要对 Deployment 的 Pod 模板应用多项调整时，将 Deployment 暂停。
