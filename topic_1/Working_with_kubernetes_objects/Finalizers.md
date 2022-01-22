# Finalizers

Finalizers 是带命名空间的键，告诉 Kubernetes 等到特定的条件被满足后，再完全删除被标记为删除的资源。Finalizer 提醒 Controller Manager 清理已删除对象拥有的资源。

当告诉 Kubernetes 删除了一个指定 Finalizer 的对象时，Kubernetes API 会将该对象标记为删除，使其进入只读状态。此时控制平面或其他组件会采取 Finalizer 所定义的行动，而目标对象仍然处于终止中（`Terminating`）的状态。这些行动完成后，控制器会删除目标对象的 Finalizer。当 `metadata.finalizers` 为空时，Kubernetes 认为删除已完成。
