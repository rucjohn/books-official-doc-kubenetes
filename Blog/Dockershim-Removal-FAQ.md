# 移除 Dockershim 的常见问题



## 为什么会从 Kubernetes 中移除 dockershim？

Kubernetes 的早期 版本仅适用于特定的容器运行时：Docker Engine。后来，Kubernetes 增加了对使用其他容器运行时的支持。

创建 CRI 标准是为了实现编排器（如，Kubernetes）和许多不同的容器运行时之间的交互操作。Docker Engine 没有实现（CRI）接口，因此，Kubernetes 项目创建了特殊代码来帮助过渡，并使 dockershim 代码成为 Kubernetes 的一部分。

dockershim 代码一直是一个临时解决方案（因此得名：shim）。此外，在较新的 CRI 运行时中实现了与 dockershim 不兼容的功能，例如 cgroup v2 和用户命名空间。从 Kubernetes 中移除 dockershim 允许在这些领域进行进一步的开发。

## Docker 和容器一样吗？

Docker 普及了 Linux 容器模式，并在开发底层技术方面发挥了重要作用，但是 Linux 中的容器已经存在很长时间，容器生态系统已经发展得比 Docker 广泛得多。OCI 和 CRI 等标准帮助许多工具在我们的生态系统中发展壮大，其中一些替代了 Docker 的某些方面，而另一些则增强了现有功能。

## 现有的容器镜像是否仍然有效？

是的，从 `docker build` 生成的镜像将适用于所有 CRI 实现，现有的所有镜像仍将完全相同。

## 私有镜像呢？

当然可以，所有 CRI 运行时都支持在 Kubernetes 中使用相同的 pull secrets 配置，无论是通过 Pod spec 还是 ServiceAccount。

## 在 Kubernetes 1.23 版本中还可以使用 Docker Engine 吗？

可以使用，在 1.20 版本中唯一的改动是，如果使用 Docker Engine，在 kubelet 启动时会打印一个警告日志。在 1.23 版本及之前版本都可以看到这个警告，dockershim 已经在 Kubernetes 1.24版本中移除。

## 应该使用哪个 CRI 实现？

这是一个复杂的问题，依赖于许多因素。如果正在使用 Docker Engine，迁移到 containerd 应该相对容易，并能获得更好的性能和更少的开销。但是，我们鼓励探索 CNCF landscape 提供的所有选项，做出更适合的选择。

## 还可以使用 Docker Engine 作为容器运行时吗？

首先，如果在自己的电脑上使用 Docker 用来开发或测试容器，它没有任何变化。无论在 Kubernetes 集群中使用什么容器运行时，都可以在本地使用 Docker。容器使这种交互成为可能。

Mirantis 和 Docker 已承诺为 Docker Engine 维护一个替代适配器，并在 dockershim 从 Kubernetes 移除后维护该适配器。该适配为 `cri-dockerd`。

可以安装 `cri-dockerd`，并使用它将 kubelet 连接到 Docker Engine。

## 现在是否有在生产系统中使用其他运行时的例子？

Kubernetes 所有项目在所有版本中发布的程序都经过了验证。

此外，kind 项目使用 containerd 已经有一段时间了，并且提高了其用例的稳定性。Kind 和 containerd 每天都会被多次使用来验证对 Kubernetes 代码库的任何更改。其他相关项目也遵循同样的模式，从而展示 了其他容器运行时的稳定性和可用性。例如，OpenShift 4.x 从 2019 年6 月以来，就一直在生产环境中使用 CRI-O 运行时。

至于其他示例和参考资料，可以查看 containerd 和 CRI-O 的使用者列表，这两个容器运行时是云原生基金会（CNCF）下的项目。
- containerd
- CRI-O

## 人们总在谈论 OCI，它是什么？

OCI 是 Open Container Initiative 的缩写，它标准化了容器工具和底层实现之间的大量接口。它们维护了打包容器镜像（OCI image）和运行时（OCI runtime）的标准规范。它们还以 runc 的形式维护了一个runtime-spec 的真实实现，这也是 containerd 和 CRI-O 依赖的默认运行时，CRI 对立在这些底层规范之上，为管理容器提供端到端的标准。

## 当切换 CRI 实现时，应该注意什么？

虽然 Docker 和大多数 CRI 包括（containerd）之间的底层容器化代码是相同的，但其周边部分却是存在差异。迁移时需要考虑如下常见事项：
- 日志配置
- 运行蝗资源限制
- 调用 docker 或通过其控制套接字使用 Docker Engine 的节点配置脚本
- 需要 `docker` 命令或 Docker Engine 的 Kubernetes 工具（例如，已弃用的 `kube-imagepuller` 工具）
- 配置 `registry-mirrors` 和不安全的镜像仓库等功能
- 保障 Docker Engine 可用、且运行在 Kubernetes 之外的脚本或守护进程（例如，监控和安全代理）
- GPU 或特殊硬件，以及它们如何与运行时和 Kubernetes 集成

如果只是用了 Kubernetes 资源请求/限制或基于文件的日志收集 DaemonSet，它们将继续稳定工作，但是如果用了自定义的 docker 配置，则可能需要为新的容器运行时做一些适配工作。

另外，还有一个需求关注的点，那就是当创建镜像时，系统维护或嵌入容器方面的任务将无法工作。对于前者，可以用 `crictl` 工具作为临时替代方案。对于后者，可以用新的容器他去选项，例如 img、buildah、kaniko 或 buildkit-cli-for-kubectl，他们都不需要 Docker。

