# 标签和选择器

标签（**Lables**）是附加到 Kubernetes 对象（比如 Pods）上的键值对。标签旨在用于指定对用户有意义和相关的对象的标识属性，但不直接暗示核心的语义。标签可有用于组织和选择对象的子集。标签可以在创建时附加到对象上，之后可以随时添加和修改。每个对象都可以定义一组 {KEY: VALUE} 标签，每个 KEY 对于给定的对象必须是唯一的。

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

标签能够支持高效的里查询和是监听操作，非常适合在用户界面和命令行中使用。应使用注解（**Annotations**）来记录非识别信息。

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

## 推荐使用的标签

除了 kubectl 和 dashboard 之外，可以使用其他工具来可视化管理 Kubernetes 对象。一组能用的标签可以让多个工具之间相互操作，用所有工具都能理解的能用方式描述对象。

除了支持工具外，推荐的标签还以一种可以查询 的方式描述应用程序。

元数据是围绕应用程序的概念组织的。 Kubernetes 不是平台即服务 (PaaS)，没有或强制执行正式的应用程序的概念。 相反，应用程序是非正式的，并使用元数据进行描述。 应用程序包含的内容的定义是松散的。

> $$\color{blue}说明：$$\
> 这些是推荐的标签。它们使管理应用程序变得更容易但不是任何核心工具所必需的。

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

```
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
