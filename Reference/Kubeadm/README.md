# Kubeadm

Kubeadm 是一个提供了 `kubeadm init` 和 `kubeadm join` 的工具，作为创建 Kubernetes 集群的 “快捷路径” 的最佳实践。

kubeadm 通过执行必要的操作来启动和运行最小可用集群。按照设计，它只关注启动引导，而非配置机器。同样的，安装各种 “锦上添花” 的扩展，例如 Kubernetes Dashboard、监控方案、以及特定云平台的扩展，都不在讨论范围内。

相反，希望在 kubeadm 之上构建更高级别以及更加合规的工具，理想情况下，使用 kubeadm 作为所有部署工作的基准将会更加易于创建一致性集群。
