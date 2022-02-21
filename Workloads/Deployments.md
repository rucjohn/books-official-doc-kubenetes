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




## 回滚 Deployment <a href="#rolling-back-a-deployment" id="rolling-back-a-deployment"></a>

有时，你可能想要回滚 Deployment；例如，当 Deployment 不稳定时（例如进入反复崩溃状态）。 默认情况下，Deployment 的所有上线记录都保留在系统中，以便可以随时回滚 （你可以通过修改修订历史记录限制来更改这一约束）。

Deployment 被触发上线时，系统就会创建 Deployment 的新的修订版本。 这意味着仅当 Deployment 的 Pod 模板（`.spec.template`）发生更改时，才会创建新修订版本 -- 例如，模板的标签或容器镜像发生变化。 其他更新，如 Deployment 的扩缩容操作不会创建 Deployment 修订版本。 这是为了方便同时执行手动缩放或自动缩放。 换言之，当你回滚到较早的修订版本时，只有 Deployment 的 Pod 模板部分会被回滚。

*   假设你在更新 Deployment 时犯了一个拼写错误，将镜像名称命名设置为 `nginx:1.161` 而不是 `nginx:1.16.1`：

    ```shell
    kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.161 --record=true
    ```

    输出类似于：

    ```shell
    deployment.apps/nginx-deployment image updated
    ```
*   此上线进程会出现停滞。你可以通过检查上线状态来验证：

    ```shell
    kubectl rollout status deployment/nginx-deployment
    ```

    输出类似于：

    ```
    Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
    ```
* 按 Ctrl-C 停止上述上线状态观测。有关上线停滞的详细信息，[参考这里](Deployments.md#deployment-status)。
*   你可以看到旧的副本有两个（`nginx-deployment-1564180365` 和 `nginx-deployment-2035384211`）， 新的副本有 1 个（`nginx-deployment-3066724191`）：

    ```shell
    kubectl get rs
    ```

    输出类似于：

    ```shell
    NAME                          DESIRED   CURRENT   READY   AGE
    nginx-deployment-1564180365   3         3         3       25s
    nginx-deployment-2035384211   0         0         0       36s
    nginx-deployment-3066724191   1         1         0       6s
    ```
*   查看所创建的 Pod，你会注意到新 ReplicaSet 所创建的 1 个 Pod 卡顿在镜像拉取循环中。

    ```shell
    kubectl get pods
    ```

    输出类似于：

    ```shell
    NAME                                READY     STATUS             RESTARTS   AGE
    nginx-deployment-1564180365-70iae   1/1       Running            0          25s
    nginx-deployment-1564180365-jbqqo   1/1       Running            0          25s
    nginx-deployment-1564180365-hysrc   1/1       Running            0          25s
    nginx-deployment-3066724191-08mng   0/1       ImagePullBackOff   0          6s
    ```

    Deployment 控制器自动停止有问题的上线过程，并停止对新的 ReplicaSet 扩容。 这行为取决于所指定的 rollingUpdate 参数（具体为 `maxUnavailable`）。 默认情况下，Kubernetes 将此值设置为 25%。
*   获取 Deployment 描述信息：

    ```shell
    kubectl describe deployment
    ```

    输出类似于：

    ```shell
    Name:           nginx-deployment
    Namespace:      default
    CreationTimestamp:  Tue, 15 Mar 2016 14:48:04 -0700
    Labels:         app=nginx
    Selector:       app=nginx
    Replicas:       3 desired | 1 updated | 4 total | 3 available | 1 unavailable
    StrategyType:       RollingUpdate
    MinReadySeconds:    0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
      Labels:  app=nginx
      Containers:
       nginx:
        Image:        nginx:1.91
        Port:         80/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    ReplicaSetUpdated
    OldReplicaSets:     nginx-deployment-1564180365 (3/3 replicas created)
    NewReplicaSet:      nginx-deployment-3066724191 (1/1 replicas created)
    Events:
      FirstSeen LastSeen    Count   From                    SubobjectPath   Type        Reason              Message
      --------- --------    -----   ----                    -------------   --------    ------              -------
      1m        1m          1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-2035384211 to 3
      22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 1
      22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 2
      22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 2
      21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 1
      21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 3
      13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 0
      13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 1
    ```

    要解决此问题，需要回滚到以前稳定的 Deployment 版本。

### 检查 Deployment 上线历史

按照如下步骤检查回滚历史：

1.  首先，检查 Deployment 修订历史：

    ```shell
    kubectl rollout history deployment.v1.apps/nginx-deployment
    ```

    输出类似于：

    ```shell
    deployments "nginx-deployment"
    REVISION    CHANGE-CAUSE
    1           kubectl apply --filename=https://k8s.io/examples/controllers/nginx-deployment.yaml --record=true
    2           kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1 --record=true
    3           kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.91 --record=true
    ```

    `CHANGE-CAUSE` 的内容是从 Deployment 的 `kubernetes.io/change-cause` 注解复制过来的。 复制动作发生在修订版本创建时。你可以通过以下方式设置 `CHANGE-CAUSE` 消息：

    * 使用 `kubectl annotate deployment.v1.apps/nginx-deployment kubernetes.io/change-cause="image updated to 1.9.1"` 为 Deployment 添加注解。
    * 追加 `--record` 命令行标志以保存正在更改资源的 `kubectl` 命令。
    * 手动编辑资源的清单。
2.  要查看修订历史的详细信息，运行：

    ```shell
    kubectl rollout history deployment.v1.apps/nginx-deployment --revision=2
    ```

    输出类似于：

    ```shell
    deployments "nginx-deployment" revision 2
      Labels:       app=nginx
              pod-template-hash=1159050644
      Annotations:  kubernetes.io/change-cause=kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1 --record=true
      Containers:
       nginx:
        Image:      nginx:1.16.1
        Port:       80/TCP
         QoS Tier:
            cpu:      BestEffort
            memory:   BestEffort
        Environment Variables:      <none>
      No volumes.
    ```

### 回滚到之前的修订版本 <a href="#rolling-back-to-a-previous-revision" id="rolling-back-to-a-previous-revision"></a>

按照下面给出的步骤将 Deployment 从当前版本回滚到以前的版本（即版本 2）。

1.  假定现在你已决定撤消当前上线并回滚到以前的修订版本：

    ```shell
    kubectl rollout undo deployment.v1.apps/nginx-deployment
    ```

    输出类似于：

    ```
    deployment.apps/nginx-deployment
    ```

    或者，你也可以通过使用 `--to-revision` 来回滚到特定修订版本：

    ```shell
    kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=2
    ```

    输出类似于：

    ```
    deployment.apps/nginx-deployment
    ```

    与回滚相关的指令的更详细信息，请参考 [`kubectl rollout`](../docs/reference/generated/kubectl/kubectl-commands/#rollout)。

    现在，Deployment 正在回滚到以前的稳定版本。正如你所看到的，Deployment 控制器生成了 回滚到修订版本 2 的 `DeploymentRollback` 事件。
2.  检查回滚是否成功以及 Deployment 是否正在运行，运行：

    ```shell
    kubectl get deployment nginx-deployment
    ```

    输出类似于：

    ```shell
    NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3         3         3            3           30m
    ```
3.  获取 Deployment 描述信息：

    ```shell
    kubectl describe deployment nginx-deployment
    ```

    输出类似于：

    ```
    Name:                   nginx-deployment
    Namespace:              default
    CreationTimestamp:      Sun, 02 Sep 2018 18:17:55 -0500
    Labels:                 app=nginx
    Annotations:            deployment.kubernetes.io/revision=4
                            kubernetes.io/change-cause=kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1 --record=true
    Selector:               app=nginx
    Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
      Labels:  app=nginx
      Containers:
       nginx:
        Image:        nginx:1.16.1
        Port:         80/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   nginx-deployment-c4747d96c (3/3 replicas created)
    Events:
      Type    Reason              Age   From                   Message
      ----    ------              ----  ----                   -------
      Normal  ScalingReplicaSet   12m   deployment-controller  Scaled up replica set nginx-deployment-75675f5897 to 3
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 1
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 2
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 2
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 1
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 3
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 0
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-595696685f to 1
      Normal  DeploymentRollback  15s   deployment-controller  Rolled back deployment "nginx-deployment" to revision 2
      Normal  ScalingReplicaSet   15s   deployment-controller  Scaled down replica set nginx-deployment-595696685f to 0
    ```


## 暂停、恢复 Deployment <a href="#pausing-and-resuming-a-deployment" id="pausing-and-resuming-a-deployment"></a>

你可以在触发一个或多个更新之前暂停 Deployment，然后再恢复其执行。 这样做使得你能够在暂停和恢复执行之间应用多个修补程序，而不会触发不必要的上线操作。

*   例如，对于一个刚刚创建的 Deployment： 获取 Deployment 信息：

    ```shell
    kubectl get deploy
    ```

    输出类似于：

    ```
    NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx     3         3         3            3           1m
    ```

    获取上线状态：

    ```shell
    kubectl get rs
    ```

    输出类似于：

    ```shell
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   3         3         3         1m
    ```
*   使用如下指令暂停运行：

    ```shell
    kubectl rollout pause deployment.v1.apps/nginx-deployment
    ```

    输出类似于：

    ```shell
    deployment.apps/nginx-deployment paused
    ```
*   接下来更新 Deployment 镜像：

    ```shell
    kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
    ```

    输出类似于：

    ```
    deployment.apps/nginx-deployment image updated
    ```
*   注意没有新的上线被触发：

    ```shell
    kubectl rollout history deployment.v1.apps/nginx-deployment
    ```

    输出类似于：

    ```shell
    deployments "nginx"
    REVISION  CHANGE-CAUSE
    1   <none>
    ```
*   获取上线状态确保 Deployment 更新已经成功：

    ```shell
    kubectl get rs
    ```

    输出类似于：

    ```shell
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   3         3         3         2m
    ```
*   你可以根据需要执行很多更新操作，例如，可以要使用的资源：

    ```shell
    kubectl set resources deployment.v1.apps/nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi
    ```

    输出类似于：

    ```
    deployment.apps/nginx-deployment resource requirements updated
    ```

    暂停 Deployment 之前的初始状态将继续发挥作用，但新的更新在 Deployment 被 暂停期间不会产生任何效果。
*   最终，恢复 Deployment 执行并观察新的 ReplicaSet 的创建过程，其中包含了所应用的所有更新：

    ```shell
    kubectl rollout resume deployment.v1.apps/nginx-deployment
    ```

    输出：

    ```shell
    deployment.apps/nginx-deployment resumed
    ```
*   观察上线的状态，直到完成。

    ```shell
    kubectl get rs -w
    ```

    输出类似于：

    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   2         2         2         2m
    nginx-3926361531   2         2         0         6s
    nginx-3926361531   2         2         1         18s
    nginx-2142116321   1         2         2         2m
    nginx-2142116321   1         2         2         2m
    nginx-3926361531   3         2         1         18s
    nginx-3926361531   3         2         1         18s
    nginx-2142116321   1         1         1         2m
    nginx-3926361531   3         3         1         18s
    nginx-3926361531   3         3         2         19s
    nginx-2142116321   0         1         1         2m
    nginx-2142116321   0         1         1         2m
    nginx-2142116321   0         0         0         2m
    nginx-3926361531   3         3         3         20s
    ```
*   获取最近上线的状态：

    ```shell
    kubectl get rs
    ```

    输出类似于：

    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   0         0         0         2m
    nginx-3926361531   3         3         3         28s
    ```

你不可以回滚处于暂停状态的 Deployment，除非先恢复其执行状态。

## Deployment 状态 <a href="#deployment-status" id="deployment-status"></a>

Deployment 的生命周期中会有许多状态。上线新的 ReplicaSet 期间可能处于 [Progressing（进行中）](Deployments.md#progressing-deployment)，可能是 [Complete（已完成）](Deployments.md#complete-deployment)，也可能是 [Failed（失败）](Deployments.md#failed-deployment)以至于无法继续进行。

### 进行中的 Deployment <a href="#progressing-deployment" id="progressing-deployment"></a>

执行下面的任务期间，Kubernetes 标记 Deployment 为 _进行中（Progressing）_：

* Deployment 创建新的 ReplicaSet
* Deployment 正在为其最新的 ReplicaSet 扩容
* Deployment 正在为其旧有的 ReplicaSet(s) 缩容
* 新的 Pods 已经就绪或者可用（就绪至少持续了 [MinReadySeconds](Deployments.md#min-ready-seconds) 秒）。

你可以使用 `kubectl rollout status` 监视 Deployment 的进度。

### 完成的 Deployment <a href="#complete-deployment" id="complete-deployment"></a>

当 Deployment 具有以下特征时，Kubernetes 将其标记为 _完成（Complete）_：

* 与 Deployment 关联的所有副本都已更新到指定的最新版本，这意味着之前请求的所有更新都已完成。
* 与 Deployment 关联的所有副本都可用。
* 未运行 Deployment 的旧副本。

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

### 失败的 Deployment <a href="#failed-deployment" id="failed-deployment"></a>

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

## 清理策略 <a href="#clean-up-policy" id="clean-up-policy"></a>

你可以在 Deployment 中设置 `.spec.revisionHistoryLimit` 字段以指定保留此 Deployment 的多少个旧有 ReplicaSet。其余的 ReplicaSet 将在后台被垃圾回收。 默认情况下，此值为 10。

显式将此字段设置为 0 将导致 Deployment 的所有历史记录被清空，因此 Deployment 将无法回滚。

## 金丝雀部署 <a href="#canary-deployment" id="canary-deployment"></a>

如果要使用 Deployment 向用户子集或服务器子集上线版本，则可以遵循 [资源管理](../zh/docs/concepts/cluster-administration/manage-deployment/#canary-deployments) 所描述的金丝雀模式，创建多个 Deployment，每个版本一个。

