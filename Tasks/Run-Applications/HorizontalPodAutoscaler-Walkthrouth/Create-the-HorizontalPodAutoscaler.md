# 创建 HPA


现在服务器正在运行，使用 `kubectl autoscale` 命令创建 HPA。

接下来将创建 HPA，该 HPA 将维护所创建的 php-apache Deployment 控制的 Pod 的 1 - 10 个副本。


粗略地说，HPA 控制器将增加和减少副本的数量（通过更新 Deployment）以保持所有 Pod 的平均 CPU 利用率为 50%。Deployment 将更新 ReplicaSet —— 这是所有 Deployment 在 Kubernetes 中工作方式的一部分 —— 然后 ReplicaSet 根据其 `.spec` 的变更来添加或删除 Pod。

由于每个 Pod 通过 `kubectl run` 请求 200 milli-cores，这意味着平均 CPU 使用率为 100 milli-cores。有关算法的更多详细信息，请参阅算法详细信息。

创建 HorizontalPodAutoscaler：
```bash
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```
输出
```
horizontalpodautoscaler.autoscaling/php-apache autoscaled
```

通过运行以下命令检查新创建的 HorizontalPodAutoscaler 的当前状态
```bash
# 可以使用 “hpa” 或 “horizontalpodautoscaler”
kubectl get hpa
```
输出
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache/scale   0% / 50%  1         10        1          18s
```

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

当前的 CPU 利用率是 0%，这是由于尚未发送任何请求到服务（`TARGET` 列显示了相应 Deployment 所控制的所有 Pod 的平均 CPU 利用率）。
{% endhint %}
