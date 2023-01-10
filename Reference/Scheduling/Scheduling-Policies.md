# 调度策略


在 Kubernetes v1.23 版本之前，可以使用调度策略来指定 **predicates** 和 **priorities** 进程。例如，可以通过运行 `kube-scheduler --policy-config-file <filename>` 或者 `kube-scheduler --policy-configmap <ConfigMap>` 设置调度策略。

从 Kubernetes v1.23 版本开始，不再支持这种调度策略。同样地也不支持相关的 `policy-config-file`、`policy-configmap`、`policy-configmap-namespace` 和 `use-legacy-policy-config` 标志。可以通过使用 [调度器配置](../Scheduler-Configuration.md) 来实现类似的行为。
