# 标签和选择算符

标签（_**Lables**_）是附加到 Kubernetes 对象（比如 Pods）上的键值对。

* 标签旨在用于指定对用户有意义和相关的对象的标识属性，但不直接对核心系统有语义含义。
* 标签可有用于组织和选择对象的子集。
* 标签可以在创建时附加到对象上，之后可以随时添加和修改。每个对象都可以定义一组 `{KEY: VALUE}` 标签，每个 KEY 对于给定的对象必须是唯一的。

```json
{
  "metadata": {
    "labels": {
      "key1": "values1",
      "key2": "values2"
    }
  }
}
```

标签能够支持高效的查询和是监听操作，非常适合在用户界面和命令行中使用。

可以使用注解（**Annotations**）来记录非识别信息。

## 动机

标签能够让用户以松耦合的方式将自己的组织结构映射到系统对象上，而无需客户端存储这些映射。

服务部署和批处理流水线通常是多维度的（例如，多个分区部署、多个发行序列、多个层，每层中的多个微服务等）。管理这些通常需要交叉操作，这又打破了严格分层表示的封装，尤其是由基础设施而不是用户确定的刚性层次结构。

示例标签：

* `release: stable`
* `release: canary`
* `environment: dev`
* `environment: qa`
* `environment: production`
* `tier: frontend`
* `tier: backend`
* `tier: cache`
* `partition: customerA`
* `partition: customerB`
* `track: daily`
* `track: weekly`

## 语法和字符集

标签是键值对。

有效的标签键：

* 标签键有两段：可选的前缀和名称。
* 前缀段是可选的。
* 前缀段如果指定，必须是 DNS 子域：由 `.` 分隔一系列 DNS 标签，总共不超过 253 字符，最后跟斜杠 `\`。
* 名称段是必需的，必须小于等于63个字符，以字母数字字符（`[a-z0-9A-Z]`）开关和结尾。
* 名称段包含破折号（`-`）、下划线（`_`）、点（`.`）和字母或数字。

向最终用户对象添加标签的自动化系统组件（例如 `kube-scheduler`、`kube-controller-manager`、`kube-apiserver`、`kubectl` 或其他第三方自动化）必须指定前缀。

有效的标签值：

* 必须为 63 个字符或更少（可以为空）
* 除非标签值为空，必须以字母数字字符（`[a-z0-9A-Z]`）开头和结尾
* 包含破折号（`-`）、下划线（`_`）、点（`.`）和字母或数字。

## 推荐使用的标签

除了 kubectl 和 dashboard 之外，可以使用其他工具来可视化管理 Kubernetes 对象。一组能用的标签可以让多个工具之间相互操作，用所有工具都能理解的能用方式描述对象。

除了支持工具外，推荐的标签还以一种可以查询 的方式描述应用程序。

元数据是围绕应用程序的概念组织的。 Kubernetes 不是平台即服务 (PaaS)，没有或强制执行正式的应用程序的概念。 相反，应用程序是非正式的，并使用元数据进行描述。 应用程序包含的内容的定义是松散的。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

这些是推荐的标签。它们使管理应用程序变得更容易但不是任何核心工具所必需的。
{% endhint %}

共享标签和注解都使用同一个前缀：`app.kubernetes.io`。没有前缀的标签是私有的。共享前缀可以确保共享标签不会干扰用户自定义的标签。

### 共享标签

为了充分利用这些标签，应该在每个资源对象上都使用它们。

| 键（KEY）                       | 描述                  | 示例                 | 类型  |
| ---------------------------- | ------------------- | ------------------ | --- |
| app.kubernetes.io/name       | 应用程序的名称             | mysql              | 字符串 |
| app.kubernetes.io/instance   | 应用实例的名称             | mysql-abcxyz       | 字符串 |
| app.kubernetes.io/version    | 应用程序的当前版本           | 5.7.21             | 字符串 |
| app.kubernetes.io/component  | 架构中的组件              | database           | 字符串 |
| app.kubernetes.io/part-of    | 此应用程序所属的更高级别应用程序的名称 | wordpress          | 字符串 |
| app.kubernetes.io/managed-by | 管理应用程序的工具           | helm               | 字符串 |
| app.kubernetes.io/created-by | 创建该资源的控制器或用户        | controller-manager | 字符串 |

简单示范：

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxyz
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/created-by: controller-manager
```

### 应用和应用实例

应用可以在 Kubernetes 集群中安装一次或多次。在某些情况下，可以安装在同一命名空间中。例如，可以多次为不同的站点安装不同的WordProess。

应用的名称和实例的名称是分别记录的。例如，WordPress 应用的 `app.kubernetes.ip/name: wordpress`，而其实例名称 `app.kubernetes.ip/instance: wordpress-abcxyz`。这使得应用和应用的实例均可被识别，应用的每个实例都必须具有唯一的名称。

### 示例

为了说明使用这些标签的不同方式，以下示例具有不同的复杂性。

#### 一个简单的无状态服务

考虑使用 `Deployment` 和 `Service` 对象部署的简单无状态服务的情况。以下两个代码段表示如何以最简单的形式使用标签。

下面的 `Deployment` 用于监督运行应用本身的 pods 。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: tomcat
    app.kubernetes.io/instance: tomcat-abcxyz
```

下面的 `Service` 用于暴露应用。

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: tomcat
    app.kubernetes.io/instance: tomcat-abcxyz
```

#### 带有一个数据库的 WEB 应用程序

考虑一个稍微复杂的应用：一个使用 Helm 安装的 Web 应用，其中使用了数据库。

以下 `Deployment` 的开头用于应用程序：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: tomcat
    app.kubernetes.io/instance: tomcat-abcxyz
    app.kubernetes.io/version: "8.0.1"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: server
    app.kubernetes.io/part-of: wordpress
```

下面的 `Service` 用于暴露应用。

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: tomcat
    app.kubernetes.io/instance: tomcat-abcxyz
    app.kubernetes.io/version: "8.0.1"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: server
    app.kubernetes.io/part-of: wordpress
```

MySQL 作为一个 `StatefulSet` 暴露，包含它和它所属的较大应用程序的元数据：

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxyz
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
```

`Service` 用于暴露 MySQL。

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxyz
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
```

## 标签选择算符（_Label Selector_）

与 [名称和 IDs 不同](Object-Names-and-IDs.md)，Label 不支持唯一性。通常，我们希望不同对象具有相同的标签。

通过 Label Selector，客户端或用户可以识别一组对象。Label Selector 是 Kubernetes 中核心的分组原语。

API 目前支持两种类型的 Selector：

* 基于等值的
* 基于集合的。

Lable Selector 使用逗号（`,`）分隔多个 Label。在多个 Label 的情况下，必须满足所有条件，相当于逻辑与(`&&`)运算符。

空或未指定 Label Selector 的语义取决于上下文，使用 Selector 的 API 类型应记录它们的有效性和含义。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>对于某些 API 类型（例如 `ReplicaSet`）而言，两个实例的 Label Selector 不得在命名空间内重叠，否则控制器会将其视为冲突指令而无法确定应存在多少副本。
{% endhint %}

{% hint style="warning" %}
<mark style="color:orange;">**注意：**</mark>对于等值的和基于集合的条件而言，不存在逻辑或(`||`)操作符。要确保过滤的条件是按照合适的方式进行组织。
{% endhint %}

### 基于等值的（_Equality-based_）

有基于等值和基于不等值的需求，匹配对象必须满足所有指定的标签约束。可接受的运算符有三种：`=`、`==` 和 `!=` ，前两个表示相等，最后一个表示不相等。例如

```
enviroment = production
tier != frontend
```

也可以使用逗号将多个条件并列：

```
enviroment=production,tier!=frontend
```

基于等值的标签要求的其中一种使用场景是让 Pod 指定可部署的节点。例如，下面的示例中，Pod 将部署在带有 `accelerator=nvidia-tesla-p100` 标签的节点上。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
  - name: cuda-test
    image: "k8s.gcr.io/cuda-vector-add:v0.1"
    resources:
      limits:
        nvidia.com/gpu: 1
  nodeSelector:
    accelerator: nvidia-tesla-p100
```

### 基于集合的（_Set-based_）

基于集合的标签请求，允许通过一组值来过滤键。支持三种操作符：`in`、`notin` 和 `exists`（只可以用于键操作）。例如：

```
enviroment in (production, qa)
tier notin (frontend, backend)
partition
!partition
```

* 第一行：选择所有键等于 `enviroment` 并且值等于 `production` 或 `qa` 的资源。
* 第二行：选择所有键等于 `tier` 并且值不等于 `frontend` 或 `backend` 的资源，以及所有没有 `tier` 键的资源。
* 第三行：选择所有包含 `partition` 标签的资源，并且不校验它的值。
* 第四行：选择所有不包含 `partition` 标签的资源，并且不校验它的值。

同样，逗号充当 _**与**_ 运算符。例如：`partition, environment notin（qa)`。

基于集合的标签选择器是等式的一般形式，因为 `enviroment=production` 等同于 `environment in (production)`；`!=` 和 `notin` 也是类似的。

基于集合的请求可以和基于基于等值的请求混合使用。例如：`partition in (customerA,customerB),enviroment!=qa`。

## API

### LIST 和 WATCH 过滤

LIST 和 WATCH 操作可以使用查询参数指定标签选择器过滤一组对象。两种请求都是允许的。

* 基于等值的：`?labelSelector=environment%3Dproduction,tier%3Dfrontend`
* 基于集合的：`labelSelector=environment+in+%28production%2Cqa%29%2Ctier+in+%28frontend%29`

两种标签选择器样式均可以通过 REST 客户端列出和查看资源。例如，使用 `kubectl` 定位 `apiserver`：

```bash
kubectl get pods -l environment=production,tier=frontend
kubectl get pods -l 'environment in (production),tier in (frontend)'
```

如前所述，基于集合的请求更具表现力。例如

```bash
# 实现值的或操作
kubectl get pods -l 'environment in (production, qa)'
# 实现不匹配操作
kubectl get pods -l 'environment,enviroment notin (frontend)'
```

### 在 API 对象中设置引用

一些 Kubernetes 对象，例如 `Services` 和 `replicationcontrollers`，也使用了标签选择器去指定了其他资源的集合，例如 `Pods`。

#### Service 和 ReplicationController

一个 `Service` 指向的一组 Pods 是由标签选择器定义的。同样，一个 `ReplicationController` 应该管理的 Pods 的数量也是由标签选择器定义的。

两个对象的标签选择器都是在 `json` 或 `yaml` 文件中映射定义的，并且只支持基于等值的选择器：

```json
"selector": {
    "component": "redis"
} 
```

或者

```yaml
selector:
  componet: redis
```

这个选择器等价于 `component=redis` 或 `component in (redis)`。

#### 支持基于集合请求的资源

比较新的资源，例如 `Job`、`Deployment`、`ReplicaSet` 和 `DaemonSet`，也支持基于集合的请求。

```yaml
selector:
  matchlabels:
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
```

* `matchLabels` 是由 `key: value` 组成的映射。
* `matchlabels` 映射中的单个 `key: value` 等同于 `matchExpressions` 的元素，`key` 字段为 "key"，`operator` 为 "In"，`values` 数组仅包含 "value"。
* `matchExpressions` 是 Pod 选择器要求的列表。有效的运算符包含 `In`、`NotIn`、`Exists` 和 `DoesNotExist`。在 `In` 和 `NotIn` 的情况下，设置的值必须是非空的。

来自 `matchLabels` 和 `matchExpressions` 的所有要求都按逻辑与的关系组合到一起，它们都必须满足才能匹配。

#### 选择节点集合

通过标签进行选择的一个场景是确定节点集合，方便 Pod 调试。有关更多信息，请参阅 [选择节点 ](../../Scheduling-Preemption-and-Eviction/Assigning-Pods-to-Nodes.md)文档。
