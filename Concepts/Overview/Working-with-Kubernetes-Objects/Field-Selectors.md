# 字段选择器

字段选择器（Field Selectors）允许根据一个或多个资源字段的值筛选 Kubernetes 资源。下面是一些使用字段选择器查询的例子：

* `metadata.name=mys-service`
* `metadata.namespace!=default`
* `status.phase=Pending`

下面这个 `kubectl` 命令将筛选出 <mark style="color:blue;">`status.phase`</mark> 字段值为 Running 的所有 Pod：

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

输出

<pre><code><strong>Error from server (BadRequest): Unable to find "ingresses" that match label selector "", field selector "foo.bar=baz": "foo.bar" is not a known field selector: only "metadata.name", "metadata.namespace"
</strong></code></pre>

## 支持的操作符

可以在字段选择器中使用 `=`、`==` 和 `!=` 操作符。

例如，下面例子将筛选所有不属于 `default` 命名空间的 Kubernetes 服务：

```bash
kubectl get services --all-namespaces --field-selector metadata.namespace!=default
```

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>`=` 和 `==` 的意义是相同的<mark style="color:blue;">**。**</mark>
{% endhint %}

## 链式选择器

同 Lables 和其他选择器一样，字段选择器可以通过使用逗号分隔的列表组成一个选择链。

例如，下面例子中将筛选 `status.phase` 字段不等到 `Running`，同时 `spec.restartPolicy` 字段等于 `Alway` 的所有 Pod：

```bash
kubectl get pods --field-selector=status.phase!=Running,spec.restartPolicy=Always
```

## 多种资源类型

能够跨多种资源类型来使用字段选择器。

例如，下面例子是将筛选出所有不在 `default` 命名空间中的 StatefulSet 和 Service：

```
kubectl get statefulsets,services --all-namespaces --field-selector metadata.namespace!=default
```
