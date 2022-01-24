# 字段选择器

字段选择器（Field Selectors）允许根据一个或多个资源字段的值筛选 Kubernetes 资源。下面是一些使用字段选择器查询的例子：

* `metadata.name=mys-service`
* `metadata.namespace!=default`
* `status.phase=Pending`

下面这个 `kubectl` 命令将筛选出 status.phase 字段值为 Running 的所有 Pod：

```bash
kubectl get pods --field-selector status.phase=Running
```

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

字段选择器本质上是资源过滤器。默认情况下，字段选择器/过滤器是未被应用的，这意味着指定类型的所有有资源都会被筛选出来，这使得以下两个 `kubectl` 查询是等价的：

```bash
kubectl get pods
kubectl get pods --field-selector ""
```
{% endhint %}

## 支持的字段

不同的 Kubernetes 资源类型支持不同的字段选择器。所有资源类型都支持 `metadata.name` 和 `metadata.namespace` 字段。使用不被支持的字段选择器会报错。例如：

```bash
kubectl get ingress --field-selector foo.bar=baz
```

```
Error from server (BadRequest): Unable to find "ingresses" that match label selector "", field selector "foo.bar=baz": "foo.bar" is not a known field selector: only "metadata.name", "metadata.namespace"
```
