# Kubernetes API

Kubernetes 控制平面的核心是 API 服务器。API 服务 器负责提供 HTTP API，允许用户、集群中的部分组件和集群外的组件相互通信。

Kubernetes API 允许查询和操作 Kubernetes API 对象（例如，Pod、Namespace、ConfigMap 和 Event）的状态。

大部分操作都可以通过 `kubectl` 命令行接口或类似 `kubeadm` 这类命令行工具来执行，这些工具实现也是调用 API。不过，也可使用 REST 调用直接访问 API。

## API 组和版本

为了简化删除字段或者重构资源表示等工作，Kubernetes 支持多个 API 版本，每一个版本都在不同 API 路径下，例如 `/api/v1` 或 `/apis/rbac.authroization.k8s.io/v1alpha1`。

版本化是在 API 级别而不是在资源或字段级别进行的，目的是为了确保 API 为系统资源和行为提供清晰、一致的视图，并能够控制对已废止的和实验性 API 的访问。

为了便于演化和扩展其 API，Kubernetes 实现了可被启用或禁用的 API 组。

API 资源通过其 API 组、资源类型、命名空间（对于命名空间作用域的资源而言）和名称来相互区分。API 服务器可能通过多个 API 版本来向外提供相同的下层数据，并透明地完成不同 API 版本之间的转换。

例如，假设同一资源有 `v1` 和 `v1beta1` 版本。如果最初使用其 API 的 `v1beata` 创建对象，则后续可以使用 `v1beata` 或者 `v1` 版本读取、更改或者删除该对象。

## API 扩展

有两种途径来扩展 Kubernetes API：\
1\. 可以使用自定义资源来以声明式方式定义 API 服务器如何提供所选择的资源 API。\
2\. 也可以选择实现自己的聚合层来扩展 Kubernetes API。
