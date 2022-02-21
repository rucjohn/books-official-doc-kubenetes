#  Deployment 更新 <a href="#updating-a-deployment" id="updating-a-deployment"></a>

{% hint style="info" %}
说明：

仅当 Deployment Pod 模板（即 `.spec.template`）发生改变时，才会触发 Deployment rollout 动作；例如，模板的标签或容器镜像被更新。其他更新（如，对 Deployment 执行扩缩容操作）不会触发 rollout 动作。
{% endhint %}

## 更新步骤

1.  先来更新 nginx Pod 的镜像，使用 `nginx:1.20.3` 镜像替代 `nginx:1.20.1` 镜像。

    ```bash
    kubectl set image deployment/nginx-deployment nginx=nginx:1.20.3
    ```

    输出：

    ```
    deployment.apps/nginx-deployment image updated
    ```

    或者，编辑 Deployment ，将 `.spec.template.spec.containers[0].image` 从 `nginx:1.20.1` 更改至 `nginx:1.20.3`。

    ```bash
    kubectl edit deployment/nginx-deployment
    ```

    输出：

    ```
    deployment.apps/nginx-deployment edited
    ```
    
2.  要查看 rollout 状态，运行：

    ```bash
    kubectl rollout status deployment/nginx-deployment
    ```

    输出：

    ```
    Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
    ```

    或者

    ```
    deployment "nginx-deployment" successfully rolled out
    ```

## 获取更多信息

*   部署成功后，可以通过运行 `kubectl get deployments` 来查看 Deployment。输出如下：

    ```
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3/3     3            3           36s
    ```
    
*   查看 ReplicaSet 信息：

    ```bash
    kubectl get rs
    ```

    输出：

    ```
    NAME                          DESIRED   CURRENT   READY   AGE
    nginx-deployment-1564180365   3         3         3       6s
    nginx-deployment-2035384211   0         0         0       36s
    ```
    显示 Deployment 创建了新的 ReplicaSet，并将其扩容到了 3 个副本，同时将旧 ReplicaSet 缩容到 0 个副本，最后所有 Pod 都已完成更新操作。
    
    
*   查看 Pod 信息:

    ```bash
    kubectl get pods
    ```

    输出类似于：

    ```
    NAME                                READY     STATUS    RESTARTS   AGE
    nginx-deployment-1564180365-khku8   1/1       Running   0          14s
    nginx-deployment-1564180365-nacti   1/1       Running   0          14s
    nginx-deployment-1564180365-z9gth   1/1       Running   0          14s
    ```

    下次要更新这些 Pods 时，只需再次更新 Deployment Pod 模板即可。

    Deployment 可确保在更新时仅关闭一定数量的 Pod。默认情况下，它确保至少需要 75% 的 Pod 个数处于运行状态（最大不可用比例为 25%）。

    Deployment 还确保仅所创建 Pod 数量只可能比期望 Pods 数高一点点。 默认情况下，它可确保启动的 Pod 个数比期望个数最多多出 25%（最大峰值 25%）。

    例如，如果仔细查看上述 Deployment ，将看到它首先创建了一个新的 Pod，然后删除了一些旧的 Pods， 并创建了新的 Pods。它不会杀死老 Pods，直到有足够的数量新的 Pods 已经出现。 在足够数量的旧 Pods 被杀死前并没有创建新 Pods。它确保至少 2 个 Pod 可用，同时 最多总共 4 个 Pod 可用。

*   获取 Deployment 的更多信息

    ```bash
    kubectl describe deployments
    ```

    输出类似于：

    ```
    Name:                   nginx-deployment
    Namespace:              default
    CreationTimestamp:      Thu, 30 Nov 2017 10:56:25 +0000
    Labels:                 app=nginx
    Annotations:            deployment.kubernetes.io/revision=2
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
          Environment:  <none>
          Mounts:       <none>
        Volumes:        <none>
      Conditions:
        Type           Status  Reason
        ----           ------  ------
        Available      True    MinimumReplicasAvailable
        Progressing    True    NewReplicaSetAvailable
      OldReplicaSets:  <none>
      NewReplicaSet:   nginx-deployment-1564180365 (3/3 replicas created)
      Events:
        Type    Reason             Age   From                   Message
        ----    ------             ----  ----                   -------
        Normal  ScalingReplicaSet  2m    deployment-controller  Scaled up replica set nginx-deployment-2035384211 to 3
        Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 1
        Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 2
        Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 2
        Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 1
        Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 3
        Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 0
    ```

    可以看到，当第一次创建 Deployment 时，它创建了一个 ReplicaSet（nginx-deployment-2035384211） 并将其直接扩容至 3 个副本。更新 Deployment 时，它创建了一个新的 ReplicaSet （nginx-deployment-1564180365），并将其扩容为 1，然后将旧 ReplicaSet 缩容到 2， 以便至少有 2 个 Pod 可用且最多创建 4 个 Pod。 然后，它使用相同的滚动更新策略继续对新的 ReplicaSet 扩容并对旧的 ReplicaSet 缩容。 最后，你将有 3 个可用的副本在新的 ReplicaSet 中，旧 ReplicaSet 将缩容到 0。

## 回退（多 Deployment 动态更新）

Deployment 控制器每次更新 Deployment 时，都会创建一个 ReplicaSet，然后启动所需的 Pod。 如果更新了 Deployment，标签匹配 `.spec.selector` 但模板不匹配 `.spec.template` 的 Pod 把对应的 ReplicaSet 将被缩容。最终，新的 ReplicaSet 扩容为 `.spec.replicas` 个副本， 所有旧 ReplicaSets 缩放为 0 个副本。

如果对正在执行 rollout 操作的 Deployment 执行更新，Deployment 会立即创建一个新的 ReplicaSet 并开始对其扩容，之前正在被扩容的 ReplicaSet 将会被回退，添加到旧 ReplicaSets 列表中并开始缩容。

例如，
- 假设创建一个 Deployment 生成 5 个 `nginx:1.14.2` 镜像的 Pod，然后更新 Deployment 创建 5 个 `nginx:1.16.1` 的 Pod，假如此时只有 3 个`nginx:1.14.2` 副本已创建。
- 在这种情况下，Deployment 会立即开始杀死 3 个 `nginx:1.14.2` Pod， 并开始创建 `nginx:1.16.1` Pod。
- 它不会等待 `nginx:1.14.2` 的 5 个副本都创建完成后才开始执行变更动作。

## 更改标签选择器

通常不鼓励更新标签选择器。建议提前规划选择器。在任何情况下，如果需要更新标签选择器，请格外小心，请务必小心并确保已掌握所有含义。

说明：
在 apps/v1` API 版本中，Deployment 标签选择器在创建后是不允许改变的。

* 添加选择器时要求使用新标签更新 Deployment 规范中的 Pod 模板标签，否则将返回验证错误。 此更改是非重叠的，也就是说新的选择器不会选择使用旧选择算符所创建的 ReplicaSet 和 Pod， 这会导致创建新的 ReplicaSet 时所有旧 ReplicaSet 都会被孤立。
* 选择器的更新如果更改了某个算符的键名，这会导致与添加算符时相同的行为。
* 删除选择器的操作会删除从 Deployment 选择器中删除现有算符。 此操作不需要更改 Pod 模板标签。现有 ReplicaSet 不会被孤立，也不会因此创建新的 ReplicaSet， 但请注意已删除的标签仍然存在于现有的 Pod 和 ReplicaSet 中。
