# 容器运行时

你需要在集群内每个节上安装一个容器运行时以使 Pod 可以运行在上面。本文概述了所涉及的内容并描述了与节点设置相关的任务。

本文列出了在 Linux 上结合 Kubernetes 使用的几种能用容器运行时的详细信息：
- containerd
- CRI-O
- Docker

> 提示：对于其他操作系统，请查阅特定于你所使用平台的相关文档。

## Cgroup 驱动程序

控制组用来约束分配给进程的资源。

当某个 Linux 系统发行版使用 `systemd` 作为其初始化系统时，初始化进程会生成并使用一个 root 控制组（`cgroup`），并充当 cgroup 管理器。systemd 与 cgroup 集成紧密，并将为每个 systemd 单元分配一个 cgroup。你也可以配置容器运行时和 kubelet 使用 `cgroupfs`。连同 systemd 一起使用 `cgroupfs`，意味着将有两个不同的 cgroup 管理器。

单个 cgroup 管理器将简化分配资源的视图，并且默认情况下将对可用资源和使用中的资源具有更一致的视图。当有两个管理器花在于一个系统中时，最终 将对这些资源产生两种视图。在此领域，人们已经报告过一些安全，某些节点配置让 kubelet 和 docker 使用 `cgroupfs`，而节点上运行的其余进程则使用 `systemd`，这类节点在资源压力下会变得不稳定。

更改设置，令容器运行时和 kubelet 使用 `systemd` 作为 cgroup 驱动 ，以此使系统更为稳定。对于 Docker，`/etc/docker/daemon.json` 中修改 `native.cgroupdriver=systemd` 选项。

> 注意：更改已加入集群的节点的 cgroup 驱动 是一项敏感的操作。如果 kubelet 已经使用其中一个 cgroup 驱动的语义创建了 Pod，更改运行时以使用其他的 cgroup 驱动，当为现有的 Pods 重新创建 PodSandbos 时会产生错误。重启 kubelet 也可能无法解决此类问题。如果你有切实可行的自动化方案，使用其他已更新配置的节点来替换该节点，或者使用自动化方案来重新安装。

### cgroupfs

cgroup 提供了一个原生接口并通过 cgroupfs 提供（从这句话我们可以知道 cgroupfs 就是 cgroup 的一个接口的封装）。类似于 procfs 和 sysfs，是一种虚拟文件系统。并且 cgroupfs 是可以挂载的，默认情况下挂在 `/sys/fs/cgroup` 目录。

### systemd

systemd 也是对于 cgroup 接口的一个封装。systemd 以 `PID=1` 的形式在系统肩颈的时候运行，并提供了一套系统管理守护程序、库和实用程序，用来控制、管理 Linux 系统资源。通过 `systemd-cgls` 命令可以看到 systemd 工作的进程 `PID=1`，而 `/sys/fs/cgroup/systemd` 目录维护的是自己使用的非 subsystem 的 cgroup 层级结构。
