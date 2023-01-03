# 对象名称和 ID

集群中每一个对象都有一个名称来标识在同类资源中的唯一性。

每个对象也有一个 UID 来标识在整个集群中的唯一性。

对于用户提供的非唯一性的属性，Kubernetes 提供了 _**标签（Lables）**_ 和 _**注解（Annotation）**_ 机制。

## 名称

客户端提供的字符串，引用资源 url 中的对象，如 `/api/v1/pods/some name`。

某一时刻，只能有一个给定类型的对象具有给定的名称。但是，删除对象后则可以创建同名的新对象。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

当对象所代表的是一个物理实体（例如，代表一台物理主机的 Node）时，如果在 Node 对象未被删除并重建的条件下，重新创建了同名的物理主机，则 Kubernetes 会将新的主机看做老的主机，可能会带来某种不一致性。
{% endhint %}

以下是三种常见的资源命名约束。

### DNS 子域名

大多数资源类型需要可以 DNS 子域名的名称。DNS 子域名定义可参考 [RFC 1123](https://tools.ietf.org/html/rfc1123)。命名必须满足如下规则：

* 不能超过 253 个字符
* 只能包含小写字母、数字，以及 "-" 和 "."
* 必须以字母或数字开头
* 必须以字母或数字结尾

### DNS 标签

某些资源类型需要其名称遵循 [RFC 1123](https://tools.ietf.org/html/rfc1123) 所定义的 DNS 标签标准。命名必须满足如下规则：

* 最多 63 个字符
* 只能包含小写字母、数字，以及 "-" 和 "."
* 必须以字母或数字开头
* 必须以字母或数字结尾

### 路径分段名称

某些资源类型要求名称能被安全地用作路径中的片段。换句话说，其名称不能是 `.`、`..`，也不能包含 `/` 或 `%` 字符。

下面是一个 Pod 配置清单示例：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-demo
spec:
  containers:
  - name: nginx
    image: nginx:1.20.1
    ports:
    - containerPort: 80
```

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>某些资源类型可能具有额外的命名约束
{% endhint %}

## UIDs

Kubernetes 系统生成的字符串，唯一标识对象。

在 Kubernetes 集群的整个生命周期中创建的每个对象都有一个不同的 uid，它旨在区分类似实体的历史事件。

Kubernetes UIDs 是全局唯一标识符（也叫 UUID）。UUID 是标准化的，见 ISO/IEC 9834-8 和 ITU-T X.667 。
