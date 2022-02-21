# Deployment 创建

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

<mark style="color:orange;">spec.selector.matchLabels</mark> 字段是 `{key,value}` 键值对映射。<mark style="color:orange;">matchLabels</mark> 中的每个 `{key,value}` 映射等同于 <mark style="color:orange;">matchExpressions</mark> 中的一个元素， 即其 `key` 字段是 “key”，`operator` 为 “In”，`values` 数组仅包含 “value”。&#x20;

在 <mark style="color:orange;">matchLabels</mark> 和 <mark style="color:orange;">matchExpressions</mark> 中给出的所有条件都必须满足才能匹配。
{% endhint %}

*   `.spec.template` 字段包含以下子字段：

    * `.spec.template.metadata.labels` 字段为 Pod 打上 `app: nginx` 标签
    * `.spec.template.spec` 字段，即 Pod 模板规范，表示 Pod 运行一个 `nginx` 容器，该容器镜像为 `nginx:1.20.1`。
    * `.spec.template.spec.containers[0].name` 字段表示创建一个名为 `nginx` 的容器。



在开始之前，请确保 Kubernetes 集群已成功启动。 然后按照以下步骤创建上述 Deployment ：

1.  通过运行以下命令创建 Deployment ：

    ```bash
    kubectl apply -f nginx-deployment.yaml
    ```


2.  运行 `kubectl get deployments` 检查 Deployment 是否已创建。如果仍在创建 Deployment，则输出以下内容：

    ```
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   0/3     0            0           1s
    ```

    显示的字段含义如下：

    * `NAME` ：显示集群中 Deployment 的名称。
    * `READY` ：显示可用的副本数。格式为：“就绪副本个数 / 期望副本个数”。
    * `UP-TO-DATE` ：显示为了达到期望状态，当前已经更新的副本数。
    * `AVAILABLE` ：显示可供用户使用的副本数。
    * `AGE` ：显示运行的时间。

3.  通过以下命令查看 Deployment rollout 状态，
    ```bash
    kubectl rollout status deployment/nginx-deployment。
    ```
    输出以下结果：

    ```
    Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
    deployment "nginx-deployment" successfully rolled out
    ```
    
4.  几秒钟后再次运行 `kubectl get deployments`。输出结果：

    ```
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3/3     3            3           18s
    ```

    显示 Deployment 已创建全部三个副本，所有副本都是最新的（最新的 Pod 模板） 并且可用。
    
5.  想要查看 Deployment 创建的 ReplicaSet（`rs`），运行 `kubectl get rs`。 输出：

    ```
    NAME                          DESIRED   CURRENT   READY   AGE
    nginx-deployment-75675f5897   3         3         3       18s
    ```

    显示的字段含义如下：

    * `NAME` ：显示命名空间中 ReplicaSet 的名称；
    * `DESIRED` ：显示期望的副本个数，即在创建 Deployment 时所定义的值。 此为期望状态；
    * `CURRENT` ：显示当前运行状态中的副本数；
    * `READY` ：显示当前运行状态中可为用户提供服务的副本数；
    * `AGE` ：显示应用已经运行的时间。

    注意， ReplicaSet 的名称始终被格式化为`[Deployment名称]-[随机字符串]`。 其中的随机字符串是使用 `pod-template-hash` 作为种子随机生成的。
    
6.  要查看每个 Pod 自动生成的标签，
    ```bash
    kubectl get pods --show-labels
    ```
    返回以下结果：
    ```
    NAME                                READY     STATUS    RESTARTS   AGE       LABELS
    nginx-deployment-75675f5897-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
    nginx-deployment-75675f5897-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
    nginx-deployment-75675f5897-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
    ```

    创建的 ReplicaSet 确保总是有三个 `nginx` Pod。

{% hint style="info" %}
<mark style="color:orange;">**注意：**</mark>

必须在 Deployment 中指定适当的选择器和 Pod 模板标签（在本例中为 `app: nginx`）。

标签或者选择器不要与其他控制器（包括其他 Deployment 和 StatefulSet）重叠。Kubernetes 是不会提示重叠的，但是如果确实有多个控制器匹配具有重叠的选择器时，可能会发生冲突，执行了难以预料的操作。
{% endhint %}

## Pod-template-hash 标签

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>不要更改此标签。
{% endhint %}

`pod-template-hash` 标签由 Deployment 控制器添加到 Deployment 创建或采用的每个 ReplicaSet 中。

此标签确保 Deployment 对应的 ReplicaSets 不重复。 它是通过对 ReplicaSet 的 `PodTemplate` 进行哈希处理，所生成的哈希值被添加到 ReplicaSet 选择器、Pod 模板标签，以及 ReplicaSet 可能匹配的所有 Pod 中。
