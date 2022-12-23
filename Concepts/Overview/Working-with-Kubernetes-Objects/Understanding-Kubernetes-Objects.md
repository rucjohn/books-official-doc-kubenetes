# 理解 Kubernetes 对象

本页说明了 Kubernetes 对象在 Kubernetes API 中是如何表示的，以及如何在 `.yaml` 格式的文件中表示。



## 理解 Kubernetes 对象

在 Kubernetes 系统中，**Kubernetes 对象**是持久化的实例。Kubernetes 使用这些实例去表示整个集群的状态。具体地说，它们可以描述如下信息：

* 哪些容器化应用程序在运行（以及运行在哪些节点上）
* 这些应用程序可用的资源
* 这些应用程序的运行策略，例如重启策略、升级策略以及容错策略

Kubernetes 对象是 “目标性记录”，即一旦创建对象，Kubernetes 系统将持续工作以确保对象存在。通过创建对象，本质上是在告知 Kubernetes 系统，所需要的集群工作负载看起来是什么样子的，也就是 Kubernetes 集群所谓的的 **期望状态（Desired State）**。

操作 Kubernetes 对象，无论是创建、修改或者删除，都需要使用 Kubernetes API。比如，当使用 `kubectl` 命令行接口时，CLI 会执行必要的 Kubernetes API 调用，也可以在程序中使用客户端库直接调用 Kubernetes API。



## 对象规范（Spec）和状态（Status）

几乎每个 Kubernetes 对象都包含两个嵌套的对象字段，用于管理对象的配置；它们是：对象规范（`spec`）和对象状态（`status`）。

对于具有 `spec` 的对象，必须在创建对象时设置其内容，描述所希望对象具有的特征：**期望状态（Desired Stat）**。

`status` 描述了对象的**当前状态（Current State）**，它是由 Kubernetes 系统和组件设置并更新的。在任何时刻，Kubernetes 控制平面都一直积极地管理对象的实际状态，使之达到期望状态。

例如，Kubernetes 中的 Deployment 对象能够表示运行在集群中的应用。当创建 Deployment 时，可能需要设置 Deployment 的 `spec`，以指定该应用需要有 3 个副本运行。Kubernetes 系统读取 `Deployment.spec`，并启动所需的 3 个实例，即更新状态以之与 `spec` 相匹配。如果有的实例失败了（状态变更），Kubernetes 系统通过执行修正操作来响应 spec 和 status 间的不一致，意思是它会启动一个新的实例来替换。



## 描述 Kubernetes 对象

创建 Kubernetes 对象时，必须提供对象的 spec，用来描述该对象的期望状态，以及关于对象的一些基本信息（例如名称）。当使用 Kubernetes API 创建对象时（或者直接创建、或者基于 `kubectl`），API 请求必须在请求体中包含 JSON 格式的信息。**大多数情况下，需要在 `.yaml` 文件中为 `kubectl` 提供这些信息**。`kubectl` 在发起 API 请求时，将这些信息转换成 JSON 格式。

以下是 `.yaml` 示例文件，展示了 Kubernetes Deployment 的必要字段和 spec：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchlabels:
      app: nginx
  replicas: 2
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

使用类似于上面的 `.yaml` 文件来创建 Deployment 的一种方式是使用 `kubectl` 命名行接口（CLI）中的 `kubectl apply` 命令，将 `.yam` 文件作为参数。例如：

```bash
kubectl apply -f https://k8s.io/examples/application/deployment.yaml
```

输出：

```
deployment.apps/nginx-deployment created
```

## 必要的字段

在创建 Kubernetes 对象对应的的 `.yaml` 文件中，至少配置如下字段：

* `apiVersion`: 创建对象所使用的 Kubernetes API 版本
* `kind`: 创建对象的类别
* `metadata`: 创建对象识别唯一性的数据，包括：
  * `name`: 对象名称，必填
  * `namespace`: 命名空间，可选

每个 Kubernetes 对象的 `spec` 格式都不同，包含了特定于该对象的嵌套对象。[Kubernetes API 参考](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.22/) 能够帮助找到任何想要的对象的 `spec` 格式，也可以通过 `kubectl explain` 命令来查找 Kubernetes 对象的所有字段。
