# kubectl 中的批量操作

资源创建并不是 `kubectl` 可以批量执行的唯一操作。kubectl 还可以人配置文件中提取资源名，以便执行其他操作，特别是删除之前创建的资源：

```bash
kubectl delete -f https://k8s.io/examples/application/nginx-app.yaml
```

输出

```
deployment.apps "my-nginx" deleted
service "my-nginx-svc" deleted
```

在仅有两种资源的情况下，可以使用 _**"资源类型/资源名"**_ 的语法在命令行中同时指定这两种资源：

```bash
kubectl delete deployments/my-nginx services/my-nginx-svc
```

对于资源数目较大的情况，可以使用 `-l` 或 `--selector` 指定筛选器（标签查询），很容器根据标签筛选资源：

```bash
kubectl delete deployment,services -l app=nginx
```

由于 `kubectl` 用来输出资源名称的语法与其所接受的资源名称的语法相同，所以，可以使用 `$()` 或 `xargs` 进行链式操作：

```bash
kubectl get $(kubectl create -f docs/concepts/cluster-administration/nginx/ -o name | grep service)
kubectl create -f docs/concepts/cluster-administration/nginx/ -o name | grep service | xargs -i kubectl get {}
```

输出

```
NAME           TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)      AGE
my-nginx-svc   LoadBalancer   10.0.0.208   <pending>     80/TCP       0s
```

上面的命令中，首选使用配置文件创建资源，并使用 `-o name` 的输出格式（以 "资源/名称" 的形式打印每个资源）打印所创建的资源。然后，通过 `grep` 来过滤 "service" ，最后再打印 `kubectl get` 的内容。

如果碰巧在某个路径下的多个子路径中组织资源，那么也可以递归地在所有子路径上执行操作，方法是 `--filename,-f` 后指定 `--recursive,-R` 参数。

例如，假设有一个目录路径为 `project/k8s/development`，它保存开发环境所需的所有清单，并按资源类型组织：

```
project/k8s/development
├── configmap
│   └── my-configmap.yaml
├── deployment
│   └── my-deployment.yaml
└── pvc
    └── my-pvc.yaml
```

默认情况下，对 `project/k8s/development` 执行的批量操作只执行目录的第一层，而不是所有子目录。如果试图使用以下命令指定目录创建资源时，会报错：

```bash
kubectl apply -f project/k8s/development
```

输出

```
error: you must provide one or more resources by argument or filename (.json|.yaml|.yml|stdin)
```

正确的做法是，在 `--filename,-f` 参数后指定 `--recursive,-R` 参数：

```bash
kubectl apply -f project/k8s/development --recurisve
```

输出

```
configmap/my-config created
deployment.apps/my-deployment created
persistentvolumeclaim/my-pvc created
```

`--recursive` 可以用来接受 `--filename,-f` 参数的任何操作，例如 `kubectl {create,get,delete,describe,rollout}` 等。

有多个 `-f` 参数出现的时候，`--recursive`参数也同样可以正常执行：

```bash
kubectl apply -f project/k8s/namespaces -f project/k8s/development --recursive
```

如果需要进一步学习 `kubectl` 的内容，请参阅 [命令行工具（kubectl）](../../Reference/Command-line-tool-kubectl/)。
