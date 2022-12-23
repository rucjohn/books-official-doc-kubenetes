# 组织资源配置

许多应用需要创建多个资源，例如 Deployment 和 Service。 可以通过将多个资源组合在同一个文件中（在 YAML 中以 `---` 分隔） 来简化对它们的管理。例如：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-svc
  labels:
    app: nginx
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

可以用创建单个资源的方式来创建多个资源：

```bash
kubectl apply -f https://k8s.io/examples/application/nginx-app.yaml
```

输出：

```
service/my-nginx-svc created
deployment.apps/my-nginx created
```

资源将按照在文件中的顺序进行创建。 因此，最好先指定 Service，这样在控制器（例如：Deployment）创建 Pod 时，能够确保调度器可以将与 Service 关联的多个 Pod 分散到不同节点。

`kubectl create` 也接受多个 `-f` 参数:

<pre class="language-bash"><code class="lang-bash">kubectl apply -f https://k8s.io/examples/application/nginx/nginx-svc.yaml \ 
<strong>              -f https://k8s.io/examples/application/nginx/nginx-deployment.yaml
</strong></code></pre>

还可以指定目录路径，而不用添加多个单独的文件：

```bash
kubectl apply -f https://k8s.io/examples/application/nginx/
```

> kubectl 将读取目录中后缀为 `.yaml`、`.yml` 或者 `.json` 的文件。

建议，将同一个微服务或同一应用层相关的资源放到同一个文件中， 将同一个应用相关的所有文件按组存放到同一个目录中。 如果应用的各层使用 DNS 相互绑定，那么可以将堆栈的所有组件一起部署。

还可以使用 URL 作为配置源，便于直接使用已经提交到 Github 上的配置文件进行部署：

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/zh-cn/examples/application/nginx/nginx-deployment.yaml
```
