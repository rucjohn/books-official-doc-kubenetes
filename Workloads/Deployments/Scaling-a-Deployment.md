## 扩展 Deployment <a href="#scaling-a-deployment" id="scaling-a-deployment"></a>

可以使用以下命令扩展 Deployment：

```shell
kubectl scale deployment/nginx-deployment --replicas=5
```

输出：

```
deployment.apps/nginx-deployment scaled
```

假设集群启用了[Pod 的水平自动缩放](../zh/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)， 你可以为 Deployment 设置自动缩放器，并基于现有 Pods 的 CPU 利用率选择 要运行的 Pods 个数下限和上限。

```shell
kubectl autoscale deployment.v1.apps/nginx-deployment --min=10 --max=15 --cpu-percent=80
```

输出类似于：

```
deployment.apps/nginx-deployment scaled
```

### 比例缩放 <a href="#proportional-scaling" id="proportional-scaling"></a>

RollingUpdate 的 Deployment 支持同时运行应用程序的多个版本。 当自动缩放器缩放处于上线进程（仍在进行中或暂停）中的 RollingUpdate Deployment 时， Deployment 控制器会平衡现有的活跃状态的 ReplicaSets（含 Pods 的 ReplicaSets）中的额外副本， 以降低风险。这称为 _比例缩放（Proportional Scaling）_。

例如，你正在运行一个 10 个副本的 Deployment，其 [maxSurge](Deployments.md#max-surge)=3，[maxUnavailable](Deployments.md#max-unavailable)=2。

*   确保 Deployment 的这 10 个副本都在运行。

    ```shell
    kubectl get deploy
    ```

    输出类似于：

    ```
    NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment     10        10        10           10          50s
    ```
*   更新 Deployment 使用新镜像，碰巧该镜像无法从集群内部解析。

    ```shell
    kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:sometag
    ```

    输出类似于：

    ```
    deployment.apps/nginx-deployment image updated
    ```
*   镜像更新使用 ReplicaSet `nginx-deployment-1989198191` 启动新的上线过程， 但由于上面提到的 `maxUnavailable` 要求，该进程被阻塞了。检查上线状态：

    ```shell
    kubectl get rs
    ```

    输出类似于：

    ```
    NAME                          DESIRED   CURRENT   READY     AGE
    nginx-deployment-1989198191   5         5         0         9s
    nginx-deployment-618515232    8         8         8         1m
    ```
* 然后，出现了新的 Deployment 扩缩请求。自动缩放器将 Deployment 副本增加到 15。 Deployment 控制器需要决定在何处添加 5 个新副本。如果未使用比例缩放，所有 5 个副本 都将添加到新的 ReplicaSet 中。使用比例缩放时，可以将额外的副本分布到所有 ReplicaSet。 较大比例的副本会被添加到拥有最多副本的 ReplicaSet，而较低比例的副本会进入到 副本较少的 ReplicaSet。所有剩下的副本都会添加到副本最多的 ReplicaSet。 具有零副本的 ReplicaSets 不会被扩容。

在上面的示例中，3 个副本被添加到旧 ReplicaSet 中，2 个副本被添加到新 ReplicaSet。 假定新的副本都很健康，上线过程最终应将所有副本迁移到新的 ReplicaSet 中。 要确认这一点，请运行：

```shell
kubectl get deploy
```

输出类似于：

```
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment     15        18        7            8           7m
```

上线状态确认了副本是如何被添加到每个 ReplicaSet 的。

```shell
kubectl get rs
```

输出类似于：

```shell
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   7         7         0         7m
nginx-deployment-618515232    11        11        11        7m
```
