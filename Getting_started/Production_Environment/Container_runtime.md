# 容器运行时

你需要在集群内每个节上安装一个容器运行时以使 Pod 可以运行在上面。本文概述了所涉及的内容并描述了与节点设置相关的任务。

本文列出了在 Linux 上结合 Kubernetes 使用的几种能用容器运行时的详细信息：

* containerd
* CRI-O
* Docker

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

## 容器运行时

### containd

本节包含使用 containerd 作为 CRI 运行时的必要步骤。

使用以下命令在系统上安装 containerd：

* _安装和配置的先决条件_

```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# 设置必要的 sysctl 参数，这些参数在重新启动后仍然存在
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# 应用 sysctl 参数而无需重新启动
sudo sysctl --system
```

*   _安装 containerd_ 1. 从官方 Docker 仓库安装 `containerd.io` 软件包。

    > 可以在 [安装 Docker 引擎](https://docs.docker.com/engine/install/#server) 中找到有关各自 Linux 发行版设置 Docker 存储库和安装 `containerd.io` 软件包的说明。

    1.  配置 containerd

        ```
        sudo mkdir -p /etc/containerd
        containerd config default | sudo tee /etc/containerd/config.toml
        ```
    2.  重新启动 containerd

        ```
        sudo systemctl restart containerd
        ```
    3.  使用 `systemd` cgroup 驱动程序 结合 `runc` 使用 `systemd` cgroup 驱动，在 `/etc/containerd/config.toml` 中设置

        ```
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
        ...
        [plugins."io.containerd.grpv.v1.cri".containerd.runtimes.runc.options]
         SystemdCgroup = true
        ```

        如果您应用此更改，请确保两次重新启动 containerd

        ```
        sudo systemctl restart containerd
        ```

### CRI-O

本节包含安装 CRI-O 作为容器运行时的必要步骤。

> 说明：CRI-O 的主要以及次要版本必须与 Kubernetes 的主要和次要版本相匹配。更多信息请查阅 [CRI-O 兼容性列表](https://github.com/cri-o/cri-o#compatibility-matrix-cri-o--kubernetes)。

使用以下命令在系统中安装 CRI-O。

* _安装并配置前置环境_

```
# 创建.conf 文件以在启动时加载模块
cat <<EOF | sudo tee /etc/modules-load.d/crio.conf
oerlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# 配置 sysctl 参数，这些配置在重启之后仍然起作用
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

* 设置系统环境变量并下载软件包

在下列操作系统上安装 CRI-O ，使用下表中合适的值设置环境变量 `OS`

| 操作系统            | 环境变量 `$OS`        |
| --------------- | ----------------- |
| CentOS 8        | CentOS\_8         |
| CentOS 8 Stream | CentOS\_8\_Stream |
| CentOS 7        | CentOS\_7         |

然后，将 `$VERSION` 设置为与你的 Kubernetes 相匹配的 CRI-O 版本。例如，如果你要安装 CRI-O 1.20，请设置 `VERSION=1.20`。

你也可以安装一个特定的发行版本。例如，要安装 1.20.0 版本，设置 `VERSJON=1.20:1.20.0`

然后执行

```
sudo curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable.repo https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/devel:kubic:libcontainers:stable.repo
sudo curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo
sudo yum install cri-o
```

* 启动 CRI-O

```
sudo systemctl daemon-reload
sudo systemctl enable crio --now
```

参阅 \[CRI-O 安装指南] 了解进一步的详细信息。

* cgroup 驱动&#x20;

默认情况下，CRI-O 使用 `systemd` cgroup 驱动程序。要切换到 `cgroupfs` 驱动程序，编辑 `/etc/crio/crio.conf` 或者放置一个插件在 `/etc/crio/crio.conf.d/02-cgroup-manager.conf` 中的配置，例如：

```
[crio.runtime]
conmon_group = "pod"
cgroup_manager = "cgroupfs"
```

> 另请注意，更改后的 `conmon_cgroup`，将 CRI-O 与 `cgroupfs` 一起使用时，必须将其设置为 `pod`。通常有必要操持 kubelet 的 cgroup 驱动程序配置（通常我用过 kubeadm 完成）和 CRI-O 一致。

### Docker

1. 在每个邛上，根据 [安装 Docker 引擎](https://docs.docker.com/engine/install/#server) 为你的 Linux 发行版安装 Docker。你可以此文件中找到最新的经过验证的 Docker 版本 [依赖关系](https://github.com/kubernetes/kubernetes/blob/master/build/dependencies.yaml)。
2. 配置 Docker 守护程序，尤其是使用 `systemd` 来管理容器的 cgroup。

```
sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
    "exec-opts": [
        "native.cgroupdriver=systemd"
    ],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m",
        "max-file": "5"
    },
    "storage-driver": "overlay2"
}
EOF
```

> 说明：对于 Linux 内核版本 4.0 或更高版本，或使用 3.10.0-51 及 更高版本的 RHEL 或 CentOS 的系统，`overlay2` 是首先的存储驱动程序。

1. 重新启动 Docker 并在开机时启动

```
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```
