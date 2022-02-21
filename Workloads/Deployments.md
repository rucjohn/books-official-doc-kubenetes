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

## 创建 Deployment <a href="#creating-a-deployment" id="creating-a-deployment"></a>

下面是 Deployment 示例。其中创建了一个 ReplicaSet，负责启动三个 `nginx` Pods：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  slector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20.1
        ports:
        - containerPort: 80
```

在该例中：

* `.metadata.name` 字段定义名为 `nginx-deployment` 的 Deployment。
* `.spec.replicas` 字段定义 Deployment 创建三个Pod 副本。
* `.spec.selector` 字段定义 Deployment 匹配的 Pod。 在这种情况下，选择在 Pod 模板中定义的标签（`app: nginx`）。 不过，更复杂的选择规则是也可能的，只要 Pod 模板本身满足所给规则即可。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

<mark style="color:orange;">`spec.selector.matchLabels`</mark> 字段是 `{key,value}` 键值对映射。<mark style="color:orange;">`matchLabels`</mark> 中的每个 `{key,value}` 映射等同于 <mark style="color:orange;">`matchExpressions`</mark> 中的一个元素， 即其 `key` 字段是 “key”，`operator` 为 “In”，`values` 数组仅包含 “value”。&#x20;

在 <mark style="color:orange;">`matchLabels`</mark> 和 <mark style="color:orange;">`matchExpressions`</mark> 中给出的所有条件都必须满足才能匹配。
{% endhint %}

*   `template` 字段包含以下子字段：

    * `template.metadata.labels` 字段为 Pod 打上 `app: nginx` 标签
    * `template.spec` 字段，即 Pod 模板规范，表示 Pod 运行一个 `nginx` 容器，该容器镜像为 `nginx:1.20.1`。
    * `template.spec.containers[0].name` 字段表示创建一个 名为 `nginx` 的容器。



在开始之前，请确保 Kubernetes 集群已成功启动。 然后按照以下步骤创建上述 Deployment ：

1.  通过运行以下命令创建 Deployment ：

    ```shell
    kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
    ```


2.  运行 `kubectl get deployments` 检查 Deployment 是否已创建。如果仍在创建 Deployment，则输出类似于以下内容：

    ```
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   0/3     0            0           1s
    ```

    所显示的字段含义：

    * `NAME` ：显示集群中 Deployment 的名称。
    * `READY` ：显示可用的副本数。格式为：“就绪个数/期望个数”。
    * `UP-TO-DATE` ：显示为达到期望状态，当前已经更新的副本数。
    * `AVAILABLE` ：显示可供用户使用的副本数。
    * `AGE` ：显示运行的时间。

    请注意，期望的副本数是根据 `.spec.replicas` 字段决定的，本例中设置为 3。
3.  想要查看 Deployment 部署状态，运行 `kubectl rollout status deployment/nginx-deployment`命令。

    输出类似以下结果：

    ```
    Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
    deployment "nginx-deployment" successfully rolled out
    ```
4.  几秒钟后再次运行 `kubectl get deployments`。输出结果：

    ```
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3/3     3            3           18s
    ```

    请注意，Deployment 已创建全部三个副本，并且所有副本都是最新的（它们包含最新的 Pod 模板） 并且可用。
5.  想要查看 Deployment 创建的 ReplicaSet（`rs`），运行 `kubectl get rs`。 输出：

    ```
    NAME                          DESIRED   CURRENT   READY   AGE
    nginx-deployment-75675f5897   3         3         3       18s
    ```

    所显示的字段含义：

    * `NAME` ：列出命名空间中 ReplicaSet 的名称；
    * `DESIRED` ：显示应用的期望副本个数，即在创建 Deployment 时所定义的值。 此为期望状态；
    * `CURRENT` ：显示当前运行状态中的副本数；
    * `READY` ：显示应用中有多少副本可以为用户提供服务；
    * `AGE` ：显示应用已经运行的时间。

    注意， ReplicaSet 的名称始终被格式化为`[Deployment名称]-[随机字符串]`。 其中的随机字符串是使用 `pod-template-hash` 作为种子随机生成的。
6.  要查看每个 Pod 自动生成的标签，运行 `kubectl get pods --show-labels`。返回以下输出：

    ```
    NAME                                READY     STATUS    RESTARTS   AGE       LABELS
    nginx-deployment-75675f5897-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
    nginx-deployment-75675f5897-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
    nginx-deployment-75675f5897-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
    ```

    创建的 ReplicaSet 确保总是有三个 `nginx` Pod。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

必须在 Deployment 中指定适当的选择器和 Pod 模板标签（在本例中为 `app: nginx`）。

标签或者选择器不要与其他控制器（包括其他 Deployment 和 StatefulSet）重复。Kubernetes 不会提示重复，但是如果多个控制器匹配具有重复的选择器时，可能会发生冲突，执行难以预料的操作。
{% endhint %}

### Pod-template-hash 标签

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>不要更改此标签。
{% endhint %}

`pod-template-hash` 标签由 Deployment 控制器添加到 Deployment 创建或采用的每个 ReplicaSet。

此标签确保 Deployment 对应的 ReplicaSets 不重复。 它是通过对 ReplicaSet 的 `PodTemplate` 进行哈希处理，所生成的哈希值被添加到 ReplicaSet 选择器、Pod 模板标签，以及 ReplicaSet 可能匹配的所有 Pod 中。

## 更新 Deployment <a href="#updating-a-deployment" id="updating-a-deployment"></a>

{% hint style="info" %}
说明：

仅当 Deployment Pod 模板（即 `.spec.template`）发生改变时，才会触发 Deployment rollout，例如，模板的标签或容器镜像被更新。其他更新（如，对 Deployment 执行扩缩容操作）不会 rollout 动作。
{% endhint %}

按照以下步骤更新 Deployment：

1.  先来更新 nginx Pod 的镜像，使用 `nginx:1.20.3` 镜像替代 `nginx:1.20.1` 镜像。

    ```shell
    kubectl set image deployment.v1.apps/nginx-deployment nginx= nginx=nginx:1.20.3
    ```

    或者使用下面的命令：

    ```shell
    kubectl set image deployment/nginx-deployment nginx=nginx:1.20.3
    ```

    输出：

    ```
    deployment.apps/nginx-deployment image updated
    ```

    或者，可以编辑 Deployment ，并将 `.spec.template.spec.containers[0].image` 从 `nginx:1.20.1` 更改至 `nginx:1.20.3`。

    ```shell
    kubectl edit deployment.v1.apps/nginx-deployment
    ```

    输出：

    ```
    deployment.apps/nginx-deployment edited
    ```
2.  要查看部署状态，运行：

    ```shell
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

获取关于已更新的 Deployment 的更多信息：

*   部署成功后，可以通过运行 `kubectl get deployments` 来查看 Deployment。输出于：

    ```shell
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3/3     3            3           36s
    ```
*   运行 `kubectl get rs` ，查看 Deployment 创建了新的 ReplicaSet ，并将其扩容到 3 个副本，同时将旧 ReplicaSet 缩容到 0 个副本，最后完成了所有 Pod 的更新操作：

    ```shell
    kubectl get rs
    ```

    输出类似于：

    ```
    NAME                          DESIRED   CURRENT   READY   AGE
    nginx-deployment-1564180365   3         3         3       6s
    nginx-deployment-2035384211   0         0         0       36s
    ```
*   现在运行 `get pods` 应仅显示新的 Pods:

    ```shell
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

    Deployment 可确保在更新时仅关闭一定数量的 Pod。默认情况下，它确保至少所需 Pods 75% 处于运行状态（最大不可用比例为 25%）。

    Deployment 还确保仅所创建 Pod 数量只可能比期望 Pods 数高一点点。 默认情况下，它可确保启动的 Pod 个数比期望个数最多多出 25%（最大峰值 25%）。

    例如，如果仔细查看上述 Deployment ，将看到它首先创建了一个新的 Pod，然后删除了一些旧的 Pods， 并创建了新的 Pods。它不会杀死老 Pods，直到有足够的数量新的 Pods 已经出现。 在足够数量的旧 Pods 被杀死前并没有创建新 Pods。它确保至少 2 个 Pod 可用，同时 最多总共 4 个 Pod 可用。
*   获取 Deployment 的更多信息

    ```shell
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

### 翻转（多 Deployment 动态更新）

Deployment 控制器每次注意到新的 Deployment 时，都会创建一个 ReplicaSet 以启动所需的 Pods。 如果更新了 Deployment，则控制标签匹配 `.spec.selector` 但模板不匹配 `.spec.template` 的 Pods 的现有 ReplicaSet 被缩容。最终，新的 ReplicaSet 缩放为 `.spec.replicas` 个副本， 所有旧 ReplicaSets 缩放为 0 个副本。

当 Deployment 正在上线时被更新，Deployment 会针对更新创建一个新的 ReplicaSet 并开始对其扩容，之前正在被扩容的 ReplicaSet 会被翻转，添加到旧 ReplicaSets 列表 并开始缩容。

例如，假定你在创建一个 Deployment 以生成 `nginx:1.14.2` 的 5 个副本，但接下来 更新 Deployment 以创建 5 个 `nginx:1.16.1` 的副本，而此时只有 3 个`nginx:1.14.2` 副本已创建。在这种情况下，Deployment 会立即开始杀死 3 个 `nginx:1.14.2` Pods， 并开始创建 `nginx:1.16.1` Pods。它不会等待 `nginx:1.14.2` 的 5 个副本都创建完成 后才开始执行变更动作。

### 更改标签选择算符

通常不鼓励更新标签选择算符。建议你提前规划选择算符。 在任何情况下，如果需要更新标签选择算符，请格外小心，并确保自己了解 这背后可能发生的所有事情。

在 API 版本 `apps/v1` 中，Deployment 标签选择算符在创建后是不可变的。

* 添加选择算符时要求使用新标签更新 Deployment 规约中的 Pod 模板标签，否则将返回验证错误。 此更改是非重叠的，也就是说新的选择算符不会选择使用旧选择算符所创建的 ReplicaSet 和 Pod， 这会导致创建新的 ReplicaSet 时所有旧 ReplicaSet 都会被孤立。
* 选择算符的更新如果更改了某个算符的键名，这会导致与添加算符时相同的行为。
* 删除选择算符的操作会删除从 Deployment 选择算符中删除现有算符。 此操作不需要更改 Pod 模板标签。现有 ReplicaSet 不会被孤立，也不会因此创建新的 ReplicaSet， 但请注意已删除的标签仍然存在于现有的 Pod 和 ReplicaSet 中。

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

## 编写 Deployment 规约 <a href="#writing-a-deployment-spec" id="writing-a-deployment-spec"></a>

同其他 Kubernetes 配置一样， Deployment 需要 `apiVersion`，`kind` 和 `metadata` 字段。 有关配置文件的其他信息，请参考 [部署 Deployment ](../zh/docs/tasks/run-application/run-stateless-application-deployment/)、配置容器和 [使用 kubectl 管理资源](../zh/docs/concepts/overview/working-with-objects/object-management/)等相关文档。

Deployment 对象的名称必须是合法的 [DNS 子域名](../zh/docs/concepts/overview/working-with-objects/names/#dns-subdomain-names)。 Deployment 还需要 [`.spec` 部分](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status)。

### Pod 模板 <a href="#pod-template" id="pod-template"></a>

`.spec` 中只有 `.spec.template` 和 `.spec.selector` 是必需的字段。

`.spec.template` 是一个 [Pod 模板](../zh/docs/concepts/workloads/pods/#pod-templates)。 它和 \{{< glossary\_tooltip text="Pod" term\_id="pod" >\}} 的语法规则完全相同。 只是这里它是嵌套的，因此不需要 `apiVersion` 或 `kind`。

除了 Pod 的必填字段外，Deployment 中的 Pod 模板必须指定适当的标签和适当的重新启动策略。 对于标签，请确保不要与其他控制器重叠。请参考[选择算符](Deployments.md#selector)。

只有 [`.spec.template.spec.restartPolicy`](../zh/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) 等于 `Always` 才是被允许的，这也是在没有指定时的默认设置。

### 副本 <a href="#replicas" id="replicas"></a>

`.spec.replicas` 是指定所需 Pod 的可选字段。它的默认值是1。

### 选择算符 <a href="#selector" id="selector"></a>

`.spec.selector` 是指定本 Deployment 的 Pod [标签选择算符](../zh/docs/concepts/overview/working-with-objects/labels/)的必需字段。

`.spec.selector` 必须匹配 `.spec.template.metadata.labels`，否则请求会被 API 拒绝。

在 API `apps/v1`版本中，`.spec.selector` 和 `.metadata.labels` 如果没有设置的话， 不会被默认设置为 `.spec.template.metadata.labels`，所以需要明确进行设置。 同时在 `apps/v1`版本中，Deployment 创建后 `.spec.selector` 是不可变的。

当 Pod 的标签和选择算符匹配，但其模板和 `.spec.template` 不同时，或者此类 Pod 的总数超过 `.spec.replicas` 的设置时，Deployment 会终结之。 如果 Pods 总数未达到期望值，Deployment 会基于 `.spec.template` 创建新的 Pod。

你不应直接创建、或者通过创建另一个 Deployment，或者创建类似 ReplicaSet 或 ReplicationController 这类控制器来创建标签与此选择算符匹配的 Pod。 如果这样做，第一个 Deployment 会认为它创建了这些 Pod。 Kubernetes 不会阻止你这么做。

如果有多个控制器的选择算符发生重叠，则控制器之间会因冲突而无法正常工作。

### 策略 <a href="#strategy" id="strategy"></a>

`.spec.strategy` 策略指定用于用新 Pods 替换旧 Pods 的策略。 `.spec.strategy.type` 可以是 “Recreate” 或 “RollingUpdate”。“RollingUpdate” 是默认值。

#### 重新创建 Deployment <a href="#recreate-deployment" id="recreate-deployment"></a>

如果 `.spec.strategy.type==Recreate`，在创建新 Pods 之前，所有现有的 Pods 会被杀死。

#### 滚动更新 Deployment <a href="#rolling-update-deployment" id="rolling-update-deployment"></a>

Deployment 会在 `.spec.strategy.type==RollingUpdate`时，采取 滚动更新的方式更新 Pods。你可以指定 `maxUnavailable` 和 `maxSurge` 来控制滚动更新 过程。

**最大不可用**

`.spec.strategy.rollingUpdate.maxUnavailable` 是一个可选字段，用来指定 更新过程中不可用的 Pod 的个数上限。该值可以是绝对数字（例如，5），也可以是 所需 Pods 的百分比（例如，10%）。百分比值会转换成绝对数并去除小数部分。 如果 `.spec.strategy.rollingUpdate.maxSurge` 为 0，则此值不能为 0。 默认值为 25%。

例如，当此值设置为 30% 时，滚动更新开始时会立即将旧 ReplicaSet 缩容到期望 Pod 个数的70%。 新 Pod 准备就绪后，可以继续缩容旧有的 ReplicaSet，然后对新的 ReplicaSet 扩容，确保在更新期间 可用的 Pods 总数在任何时候都至少为所需的 Pod 个数的 70%。

**最大峰值**

`.spec.strategy.rollingUpdate.maxSurge` 是一个可选字段，用来指定可以创建的超出 期望 Pod 个数的 Pod 数量。此值可以是绝对数（例如，5）或所需 Pods 的百分比（例如，10%）。 如果 `MaxUnavailable` 为 0，则此值不能为 0。百分比值会通过向上取整转换为绝对数。 此字段的默认值为 25%。

例如，当此值为 30% 时，启动滚动更新后，会立即对新的 ReplicaSet 扩容，同时保证新旧 Pod 的总数不超过所需 Pod 总数的 130%。一旦旧 Pods 被杀死，新的 ReplicaSet 可以进一步扩容， 同时确保更新期间的任何时候运行中的 Pods 总数最多为所需 Pods 总数的 130%。

### 进度期限秒数 <a href="#progress-deadline-seconds" id="progress-deadline-seconds"></a>

`.spec.progressDeadlineSeconds` 是一个可选字段，用于指定系统在报告 Deployment [进展失败](Deployments.md#failed-deployment) 之前等待 Deployment 取得进展的秒数。 这类报告会在资源状态中体现为 `Type=Progressing`、`Status=False`、 `Reason=ProgressDeadlineExceeded`。Deployment 控制器将持续重试 Deployment。 将来，一旦实现了自动回滚，Deployment 控制器将在探测到这样的条件时立即回滚 Deployment。

如果指定，则此字段值需要大于 `.spec.minReadySeconds` 取值。

### 最短就绪时间 <a href="#min-ready-seconds" id="min-ready-seconds"></a>

`.spec.minReadySeconds` 是一个可选字段，用于指定新创建的 Pod 在没有任意容器崩溃情况下的最小就绪时间， 只有超出这个时间 Pod 才被视为可用。默认值为 0（Pod 在准备就绪后立即将被视为可用）。 要了解何时 Pod 被视为就绪，可参考[容器探针](../zh/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)。

### 修订历史限制

Deployment 的修订历史记录存储在它所控制的 ReplicaSets 中。

`.spec.revisionHistoryLimit` 是一个可选字段，用来设定出于会滚目的所要保留的旧 ReplicaSet 数量。 这些旧 ReplicaSet 会消耗 etcd 中的资源，并占用 `kubectl get rs` 的输出。 每个 Deployment 修订版本的配置都存储在其 ReplicaSets 中；因此，一旦删除了旧的 ReplicaSet， 将失去回滚到 Deployment 的对应修订版本的能力。 默认情况下，系统保留 10 个旧 ReplicaSet，但其理想值取决于新 Deployment 的频率和稳定性。

更具体地说，将此字段设置为 0 意味着将清理所有具有 0 个副本的旧 ReplicaSet。 在这种情况下，无法撤消新的 Deployment 上线，因为它的修订历史被清除了。

### paused（暂停的） <a href="#paused" id="paused"></a>

`.spec.paused` 是用于暂停和恢复 Deployment 的可选布尔字段。 暂停的 Deployment 和未暂停的 Deployment 的唯一区别是，Deployment 处于暂停状态时， PodTemplateSpec 的任何修改都不会触发新的上线。 Deployment 在创建时是默认不会处于暂停状态。
