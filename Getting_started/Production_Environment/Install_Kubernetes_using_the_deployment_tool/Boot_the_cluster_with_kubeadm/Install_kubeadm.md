# 安装 kubeadm

本页面显示如何安装 `kubeadm` 工具箱。

## 准备开始

- 一台兼容的 Linux 主机。Kubernetes 项目为基于 Debian 和 RedHat 的 Linux 发行版以及一些不提供包管理器的发行版提供通过的指令。
- 每台机器 2GB 或更多的内存（如果少于这个数字将会影响应用的运行内存）。
- 2 核 CPU 或更多。
- 集群中的所有机器的网络彼此均能相互连接（公网或内网都可以）。
- 节点这中不可以有重要的主机名、MAC 地址或 product_uuid。
- 开启机器上的某些端口。
- 禁用交换分区。为了保证 kubelet 正常工作。必须禁用交换分区。

## 确保每个节点上 MAC 地址和 product_uuid 的唯一性

- 可以使用命令 `ip link` 或 `ifconfig -a` 来获取网络接口的 MAC 地址。
- 可以使用 `sudo cat /sys/class/dmi/id/product_uuid` 命令对 product_uuid 校验。

一般来讲，硬件设备会拥有唯一的地址，但是有些虚拟机的地址可能会重复。Kubernetes 使用这些值来唯一确定集群中的节点。如果这些值在每个节点上不唯一，可能会导致安装[失败](https://github.com/kubernetes/kubeadm/issues/31)。

## 检查网络适配器

如果有一个以上的网络适配器，同时 Kubernetes 组件通过默认路由不可达，我们建议预告添加 IP 路由规则，这样 Kubernetes 集群就可以通过对应的适配器完成连接。

## 允许 iptables 检查桥接流量

确保 `br_netfilter` 模块被加载。这一操作可以通过运行 `lsmod | grep br_netfilter` 来完成。若要显式加载该模块，可以执行 `sudo modprobe br_netfilter`。

为了让 Linux 节点上的 iptables 能够正确地查看桥接流量，需要确保在 `sysctl` 配置 `net.bridge.bridge-nf-call-iptables = 1`。例如：
```
cat  <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br-netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sudo sysctl --system
```

## 检查所需端口

**控制平面节点**

协议 | 方向 | 端口范围 | 作用 | 使用者
--- | --- | --- | --- | ---
TCP | 入站 | 6443 | Kubernetes API 服务器 | 所有组件
TCP | 入站 | 2379-2380 | etcd 服务器客户端 API | kube-apiserver, etcd
TCP | 入站 | 10250 | Kubelet API | kubelet 自身、控制平面组件
TCP | 入站 | 10251 | kube-scheduler | kube-scheduler 自身
TCP | 入站 | 10252 | kube-controller-manager | kube-controller-manager 自身

**工作节点**

协议 | 方向 | 端口范围 | 作用 | 使用者
--- | --- | --- | --- | ---
TCP | 入站 | 10250 | Kubelet API | kubelet 自身、控制平面组件
TCP | 入站 | 30000-32767 | NodePort 服务 | 所有组件

30000-32767 是 NodePort 服务的默认端口范围，可以进行更改。

使用 \* 标记的任意端口号都可以被覆盖，所以需要保证所定制的端口是开放的。

虽然控制平面节点已经包含了 etcd 的端口，也可以使用自定义的外部 etcd 集群，或是指定自定义端口。

使用的 Pod 网络插件也可能需要某些特定端口开启。由于各个 POd 网络插件都有所不同，请参阅各自文档中对端口的要求。

## 安装 runtime

为了在 Pod 中运行容器，Kubernetes 使用 [容器运行时](../../Container_runtime.md)。

**Linux 节点**

默认情况下，Kubernetes 使用容器运行时接口（Container Runtime Interface, CRI）来与你所选择的容器运行时交互。

如果不指定运行时，则 kubeadm 会自动尝试检测到系统上已经安装的运行时，方法是扫描一组众所周知的 Unix 套接字。下面的表格列举了一些容器运行时及其对应的套接字路径：

运行时 | 域套接字
--- | ---
Docker | /var/run/dockershim.sock
containerd | /run containerd/containerd.sock
CRI-O | /var/run/crio/crio.sock

- 如果同时检测到 Docker 和 containerd，则优先选择 Docker。这是必然的，因为 Docker 18.09 附带了 containerd 并且两者都是可以检测到的，即使你仅安装了 Docker。
- 如果检测到其他两个或多个运行时，kubeadm 输出错误信息并退出。

kubelet 通过内置的 `dockershim` CRI 实现与 Docker 集成。

## 安装 kubeadm、kubelet 和 kubectl

你需要在每台机器上安装以下的软件包：
- kubeadm：用来初始化集群的指令。
- kubelet：在集群中的每个节点上用来启动 Pod 和容器等。
- kubectl：用来与集群通信的命令行工具。

kubeadm 不能帮忙安装或者管理 `kubelet` 和 `kubectl`，所以需要确保它们与通过 kubeadm 安装的控制平台的版本相匹配。如果不这样做，则存在发生版本偏差的风险，可能会导致一些预料之外的错误和问题。然后，控制平面与 kubelet 间的相差一个次要版本不一致是支持的，但是 kubelet 的版本不可能超过 API 服务器的版本。例如，1.7.0 版本的 kubelet 可以完全兼容 1.8.0 版本的 API 服务器，反之则不可以。

- 基于 RedHat 的发行版

  ```
  cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
  [kubernetes]
  name=Kubernetes
  baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
  enabled=1
  gpgcheck=1
  repo_gpgcheck=1
  gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
  exclude=kubelet kubeadm kubectl
  EOF

  # 将 SELinux 设置为 permissive 模式（相当于将其禁用）
  sudo setenforce 0
  sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

  sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

  sudo systemctl enable --now kubelet
  ```
  
  > 1. 通过运行命令 `setenforce 0` 和 `sed ...` 将 SELinux 设置为 permissive 模式可以有效地将其禁用。这是允许容器访问主机文件系统所必需的，而这些操作为了例如 Pod 网络工作正常。必须这么做，直到 kubelet 做出对 SELinux 的支持进行升级为止。
  
  > 2. 如果你知道如何配置 SELinux 则可将其保持启动状态，但可能需要设定 kubeadm 不支持的部分配置。
  
- 基于 Debian 的发行版

  更新 apt 包索引 并安装使用 Kubernetes apt 仓库所需要的包：

  ```
  sudo apt-get update
  sudo apt-get install -y apt-transport-https ca-certificates curl
  ```
  下载 Google Cloud 公开签名密钥：
  ```
  sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
  ```
  添加 Kubernetes apt 仓库：
  ```
  echo "deb [signed-by=/usr/shar/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
  ```
  更新 apt 包索引 ，安装 kubelet、kubeadm 和 kubectl，并锁定其版本：
  ```
  sudo apt-get update
  sudo apt-get install -y kubelet kubeadm kubectl
  sudo apt-mark hold kubelet kubeadm kubectl
  ```

- 无包管理器的情况

kubelet 现在每隔几秒就会重启，因为它陷入了一个等待 kubeadm 指令的死循环。

## 配置 cgroup 驱动程序

容器运行时和 kubelet 都具有名字为 “cgroup driver” 的属性，该懒得骑 于在 Linux 机器 上管理 CGroups 而主非常重要。

> 需要确保容器运行时和 kubelet 所使用的是相同的 cgroup 驱动，否则 kubelet 进程会失败。
