##  Deployment 回滚 <a href="#rolling-back-a-deployment" id="rolling-back-a-deployment"></a>

有时，可能想要回滚 Deployment；例如，当前 Deployment 不稳定时（比如，容器反复启动失败）。 默认情况下，Deployment 的所有上线记录都保留在系统中，以便可以随时回滚。

说明：

当 Deployment 触发 rollout 动作时，系统会给 Deployment 创建一个新的修订版本（一个新的 ReplicaSet）。意思是仅当 Deployment 的 Pod 模板（`.spec.template`）发生更改时，才会创建新修订版本。

例如，

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
