# CRI

CRI，全称 Container Runtime Interface，是一个插件接口，它使 kubelet 能够使用各种容器运行时（Container Runtime），则无需重要编译集群组件。

需要在集群的每个节点都配置一个可工作的容器运行时，以便 kubelet 可以启动 Pod 以及容器。

Container Runtime Interface （CRI）是 kubelet 和容器运行时之间通信的主要协议。

Kubernetes CRI 定义了用于集群组件 kubelet 和容器运行时之间通信的主要 gRPC 协议。

## API

FEATURE STATE: Kubernetes v1.23 [stable]

当通过 gRPC 连接到容器运行时时，kubelet 充当客户端。运行时和镜像服务节点必须在容器运行时中可用，可以在 kubelet 中单独配置 `--image-service-endpoint` 和 `--container-runtime-endpoint`。

对于 Kubernetes v1.23，kubelet 优先支持 CRI v1。如果容器运行时不支持 v1 的 CRI，则 kubelet 会尝试协商较低的受支持版本。

v1.23 版本中 kubelet 也可以协商 CRI v1alpha2，但这个版本已被弃用。如果 kubelet 无法协商支持的 CRI 版本，kubelet 则放弃且不会注册为节点。

## 升级

升级 Kubernetes 时，kubelet 会尝试在组件重启时自动选择最新的 CRI 版本。如果失败，则将如上所述内容回退。如果由于容器运行时已升级而需要 gRPC 重拨，则容器运行时还必须支持最初选择的版本，否则重拨预计会失败，因为这需要重新启动 kubelet。
