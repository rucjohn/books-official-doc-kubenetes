# 注解（Annotation）

可以使用 Kubernetes **Annotations** 为对象附加任意非标识的元数据。客户端程序（例如工具和库）能够获取这些元数据信息。

## 为对象附加元数据

可以使用 labels 和 annotations 将元数据附加到 Kubernetes 对象。

* Labels 可以用来选择对象和查找满足某些条件的对象集合。
* Annotitions 不用于标识和选择对象。它的元数据，可以很小，也可以很大，可以是结构化的，也可以是非结构化的，能够包含 label 不允许的字符。

annotations 和 labels 一样，也是键值对：

```yaml
metadata:
  annotations:
    key1: value1
    key2: value2
```

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>键和值必须是字符串。换句话说，不能使用数字、布尔值、列表或其他类型的键或值。
{% endhint %}

以下是一些例子，用来说明哪些信息可以使用 annotations 来记录：

* 由声明性配置所管理的字段。将这些字段附加到 annotations，能够将它们与客户端或服务端设置的默认值、自动生成的字符以及通过自动调整大小或自动伸缩系统设置的字段区分开来。
* 构建、发布或镜像信息（如时间戳、发布ID、Git 分支、RP 数量、镜像哈希、仓库地址等）。
* 指向日志记录、监控、分析或审计仓库的指针。
* 可用于调试目的的客户端库或工具信息。例如，名称、版本和构建信息。
* 用户或者工具/系统的来源信息，例如来自其他生态系统组件的相关对象的 URL。
* 轻量级上线工具的元数据信息。例如，配置或检查点。
* 负责人员的电话或手机号码，或指定在何处可以找到该信息的目录条目，如团队网站。
* 从最终用户到实现修改行为或使用非标准功能的指令。

可以将此类信息存储在外部数据库或目录中而不使用 annotations，但这样做使得开发人员很难生成用于部署、管理、自检的客户端共享库和工具。

## 语法和字符集

Annotations 存储的形式是键值对。有效的键分为两部分：
- 前缀和名称，以 `/` 分隔。
- 名称段是必选项，并且必须在 63 个字符以内，以字母数字字符（`[a-z0-9A-Z]`）开头和结尾。
- 名称段允许使用 `-`、`_`、`.` 和字母数字。
- 前缀是可选的。如果指定，则前缀必须是 DNS 子域：一系列由 `.` 分隔的 DNS 标签，总计不超过 253 个字符，后跟 `/`。
- 如果省略前缀，则假定 annotations 对用户是私有的。
- 向用户对象添加 annotations 的自动化系统组件（例如：`kube-scheduler`、`kube-controller-manager`、`kube-apiserver`、`kubectl` 或其他第三方自动化）必须指定前缀。

例如，下面是一个 Pod 的配置文件，其 annotations 中包含 `imageregistry: https://hub.docker.com/`：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotations-demo
  annotations:
    imageregistry: "https://hub.docker.com"
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
```
