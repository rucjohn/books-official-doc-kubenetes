# 更新标签

有时，现有的 Pod 和其它资源需要在创建新资源之前重新标记，这就需要使用 `kubectl label` 命令。

例如，如果想要将所有 nginx pod 标记为前端层，运行：

```bash
kubectl label pods -l app=nginx tier=fe
```

输出

```
pod/my-nginx-2035384211-j5fhi labeled
pod/my-nginx-2035384211-u2c7e labeled
pod/my-nginx-2035384211-u3t6x labeled
```

首选用 `app=nginx` 过滤所有的 Pod，然后用 `tier=fe` 标记它们。

想要 查看刚才标记的 Pod，运行：

```bash
kubectl get pods -l app=nginx -L tier
```

输出

```
NAME                        READY     STATUS    RESTARTS   AGE       TIER
my-nginx-2035384211-j5fhi   1/1       Running   0          23m       fe
my-nginx-2035384211-u2c7e   1/1       Running   0          23m       fe
my-nginx-2035384211-u3t6x   1/1       Running   0          23m       fe
```

这将输出所有标签为 `app=nginx` 的 Pod，并增加一个额外描述 Pod 的 tier 标签列。

想要了解更多信息，请参考 [标签](../../Overview/Working-with-Kubernetes-Objects/Labels-and-Selectors.md) 和 [kubectl label](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#label) 命令文档。
