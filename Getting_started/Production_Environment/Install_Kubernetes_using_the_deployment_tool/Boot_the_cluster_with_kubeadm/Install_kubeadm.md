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
net.bridge.bridge-nf-call-iptables - 1
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

如果同时检测到 Docker 和 containerd，则优先选择 Docker。这是必然的，因为 Docker 18.09 附带了 containerd 并且两者都是可以检测到的，即使你仅

