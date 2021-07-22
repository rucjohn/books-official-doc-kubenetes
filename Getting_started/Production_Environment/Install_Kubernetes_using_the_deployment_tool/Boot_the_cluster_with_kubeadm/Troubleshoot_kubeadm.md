# 对 kubeadm 进行故障排查

与任何程序一样，可能会在安装或者运行 kubeadm 时遇到错误。本文列举了一些觉的故障场景，并提供可帮助理解和解决这些问题的步骤。

如果你的问题未在下面列出，请执行以下步骤：
- 如果你认为问题是 kubeadm 的错误：
  - 转到 github.com/kubernetes/kubeadm 并搜索存在的问题。
  - 如果没有问题，请提交 Issues。
- 如你对 kubeadm 的工作方式有疑问，可以在[Slack](https://slack.k8s.io/)上的 #kubeadm 频道提问，或者在 [StackOverflow](https://stackoverflow.com/questions/tagged/kubernetes)上提问。请加入相关标签，例如 `#kubernetes` 和 `#kubeadm` ，这样其他人可以帮助。

## 在安装过程中，没有找到 ebtables 或者其他类似的可执行文件

如果在运行 `kubeadm init` 命令时，遇到以下的警告
```
[preflight] WARNING: ebtables not found in system path
[preflight] WARNING: ethtool not found in system path
```
那么或许在你的节点缺失 `ebtables`、`ethtool` 或者类似的可执行文件。可以使用以下命令安装它们：
- 对于 Ubuntu/Debian 用户，运行 `apt install ebtables ethtool` 命令。
- 对于 CentOS/Fedora 用户，运行 `yum install ebtables ethtool` 命令。

## 在安装过程中，kubeadm 一直等待控制平台就绪

如果在运行 `kubeadm init` 命令后，在打印以下行后挂起：
```
[apiclient] Created API client, waiting for the control plane to become read
```
这可能是由许多问题引起的。最常见的是：
- 网络连接问题。在继续之前，请检查计算机是否具有全部联通的网络连接。
- kubelet 的默认 cgroup 驱动程序配置不同于 Docker 使用的配置。检查系统日志文件（例如 `/var/log/message`）或检查 `journalctl -u kubelet` 的输出。如果你看到以下内容：
  ```
  error: failed to run Kubelet: failed to create kubelet:
  misconfiguration: kubelet cgroup driver: "systemd" is different from docker cgroup driver: "cgroupfs"
  ```
  有两种常见方法可解决 cgroup 驱动程序问题：
  - 按照[此处](../../Container_runtime.md)的说明，重新安装 Docker。
  - 更改 kubelet 配置以手动匹配 Docker cgroup 驱动程序，可以参考[配置 cgroup 驱动](../../../../Task/Management_cluster/Manage_with_kubeadm/Configure_cgroup_driver.md)
- 控制平面上的 Docker 容器持续遁入崩溃状态或（因其他原因）挂起。可以运行 `docker ps` 命令来检查以及 `docker logs` 命令来检视每个容器的运行日志。

## 当删除托管容器时 kubeadm 阻塞

如果 Docker 停止并且不删除 Kubernetes 所管理的所有容器，可能发生以下情况：
```
sudo kubeadm reset
```
```
[preflight] Running pre-flight checks
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Removing kubernetes-managed containers
(block)
```
一个可行的解决方案是重新启动 Docker 服务，然后重新运行 `kubeadm reset`：
```
sudo systemctl restart docker.service
sudo kubeadm reset
```
检查 docker 的日志也可能有用：
```
journalctl -ul docker
```

## Pods 处于 RunContainerError、CrashLoopBackOff 或 Error 状态

在 `kubeadm init` 命令运行后，系统中不应该有 pods 处于这类状态。
- 在 `kubeadm init` 命令执行完成后，如果有 pods 处于这些状态之一，请在 kubeadm 仓库捍一个 issue。`coredns`（或 `kube-dns`）应该处于 `Pending` 状态，直到部署了网络插件为止。
- 如果在部署完网络插件之后，有 pods 处于 `RunCaontainerError`、`CrashLoopBackOff` 或 `Error` 三种状态其中一种，并且 `coredns`（或 `kube-dns`）仍处于 `Pending` 状态，那很可能是安装的网络插件由于某种原因无法工作。或许需要授予它更多的 RBAC 特权或使用较新的版本。请在 Pod Network 提供商的问题跟踪器中提交问题，然后在此处分类问题。
- 如果安装的 Docker 版本早于 1.12.1，请在使用 `systemd` 来启动 `dockerd` 和重启 docker 时，删除 `MountFlags=slave` 选项。可以在 `/var/lib/systemd/system/docker.service` 中看到 MountFlags。MountFlags 可能会干扰 Kubernetes 挂载的卷，并使 Pods 处于 `CrashLoopBackOff` 状态。当 Kubernetes 不能找到 `/var/run/secrets/kubernetes.io/serviceaccount` 文件时会发生错误。

## coredns 停滞在 Pending 状态

**这一行为是预期之中的，因为系统就是这么设计的**。kubeadm 的网络供应商是中立的，因此 管理员应该选择安装 pod 的网络插件。必须完成 Pod 的网络配置，然后才能完全部署 CoreDNS。在网络被配置好之前，DNS 组件会一直处于 Pending 状态。

## HostPort 服务无法工作

此 `HostPort` 和 `HostIP` 功能是否可用取决于你的 Pod 网络配置。请联系 Pod 网络插件的作者，以确认 `HostPort` 和 `HostIP` 功能是否可用。

已验证 Calico、Canal 和 Flannel CNI 驱动程序支持 HostPort。

## 无法通过其服务 IP 访问 Pod

- 许多网络附加组件尚未启动 hairpin 模式，该模式允许 Pod 通过其服务 IP 进行访问。这是与 CNI 有关的问题。请与网络附加组件提供商联系，以获取他们所提供的 hairpin 模式的最新状态。
- 如果正在使用 VirtualBox（直接使用或通过 Vagrant 使用），需要确保 `hostname -i` 返回一个可路由的 IP 地址。默认情况下，第一个接口连接不能路由的仅主机网络。解决方法是修改 `/etc/hosts` ，请参考[Vagrantfile](https://github.com/errordeveloper/kubernetes-ansible-vagrant/blob/22dd39dfc06111235620e6c4404a96ae146f26fd/Vagrantfile#L11)。

## TLS 证书错误

以下错误指出证书可能不匹配：
```
# kubectl get pods
Unable to connect to ther server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
```

- 验证 `$HOME/.kube/config` 文件是否包含有效证书，并在必要时重新生成证书。在 kubeconfig 文件中的证书是 base64 编码的。使用 `base64 -d` 命令可以用来解码证书，`openssl x509 -text -noout` 命令可以用于查看证书信息。
- 使用如下方法取消设置 `KUBECONFIG` 环境变量的值：
  ```
  unset KUBECONFIG
  ```
  或者将其设置为默认的 `KUBECONFIG` 位置：
  ```
  export KUBECONFIG=/etc/kubernetes/admin.conf
  ```
- 另一个方法是覆盖 `kubeconfig` 的现有用户 "管理员"：
  ```
  mv $HOME/.kube $HOME/.kube.bak
  mkdir $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```

## 容器使用的非公共 IP

在某些情况下 `kubectl logs` 和 `kubectl run` 命令或许会返回以下错误，即使除此之外集群一切功能正常：
```
Error from server: Get https://10.19.0.41:10250/containerLogs/default/mysql-xxxx-xxxx/mysql: dial tcp 10.19.0.41:10250: getsockopt: no route to host
```

- 这或许是由于 Kubernetes 使用的 IP 无法与看似相同的子网上的其他 IP 进行通信的缘故，可能是由机器提供商的政策所导致。
- Digital Ocean 既分配一个共有 IP 给 `eth0`，也分配一个私有 IP 在内部用作其浮动 IP 功能的锚点，然而 `kubelet` 将选择后者作为节点的 `InternalIP` 而不是公共 IP。
  
  使用 `ip addr show` 命令代替 `ifconfig` 命令去检查这种情况，因为 `ifconfig` 命令不会显示有问题的别名 IP 地址。或者指定的 Digital Ocean 的 API 端口允许从 droplet 中查询 anchor IP：
  ```
  curl http://169.254.169.254/metadata/v1/interfaces/public/0/anchor_ipv4/address
  ```

  解决方法是通知 `kubelet` 使用哪个 `--node-ip`。当使用 Digital Ocean 时，可以是公网 IP（分配给 `eth0`），或者私网 IP（分配给 `eth1`）。私网 IP 是可选的。

  然后重启 `kubelet`：
  ```
  systemctl daemon-reload
  systemctl restart kubelet
  ```

## coredns pods 有 CrashLoopBackOff 或 Error 状态

如果有些节点运行的是旧版本的 Docker，同时启动了 SELinux，或许会遇到 CoreDNS pods 无法启动的情况。要解决此问题，可能尝试以下其中一个选项：
- 升级到 Docker 较新版本。
- 禁用 SELinux。
- 修改 coredns 部署配置 `allowPrivilegeEscalation` 为 `true`：
    ```
    kubectl -n kube-system get deployment coredns -o yaml | \
        sed 's/allowPrivilegeEscalation: false/allowPrivilegeEscalation: true/g' | \
        kubectl apply -f -
    ```

CoreDNS 处于 CrashLoopBackOff 时的另一个原因，是当 Kubernetes 中部署的 CoreDNS Pod 检测到环路时。[有许多解决方法](https://github.com/coredns/coredns/tree/master/plugin/loop#troubleshooting-loops-in-kubernetes-clusters) 可以避免在每次 CoreDNS 监测到循环并退出时，Kubernetes 尝试重启 CoreDNS Pod 的情况。

> **警告**：禁用 SELinux 或设置 allowPrivegeEscalation 为 true 可能会威胁到集群的安全性。

## etcd pods 持续重启

如果遇到以下错误：
```
rpc error: code = 2 desc =oci runtime error: exec failed: container_linux.go:247 starting container process caused "process_linux.go:110: decoding init error from pipe caused \"read parent: connection reset by peer\""
```

如果使用 Docker 1.13.1.84 运行 CentOS 7 就会出现这种问题。此版本的 Docker 会阻止 kubelet 在 etcd 容器中执行。

为解决此问题，请选择以下其中一个选项：

- 回滚到早期 版本的 Docker，例如 1.13.1-75
    ```
    yum downgrade docker-1.13.1-75.git8633870.el7.centos.x86_64 docker-client-1.13.1-75.git8633870.el7.centos.x86_64 docker-common-1.13.1-75.git8633870.el7.centos.x86_64
    ```
- 安装较新的推荐版本，例如 18.06
    ```
    sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    yum install docker-ce-18.06.1.ce-3.el7.x86_64
    ```

## 在节点被云控制管理器初始化之前，kube-proxy 就被调度了

在云环境场景中，可能出现在云控制管理器完成节点地址初始化之前，kube-proxy 就被调度到新节点了。这会导致 kube-proxy 无法正确获取节点的 IP 地址，并对管理负载平均器的代理功能产生连锁反应。

中 kube-proxy 中可以看到以下错误：
```
[server.go:610] Failed to retrieve node IP: host IP unknown; known address: []
[proxier.go:340] invalid nodeIP, initializing kube-proxy with 127.0.0.1 as nodeIP
```

一种已知的解决方案是修补 kube-proxy DaemonSet，以允许在控制平台节点上调度它，而不管它们的条件如何，将其与其他节点保持隔离，直到它们的初始保护条件消除：
```
kubectl -n kube-system path ds kube-proxy -p='{ "sepc": { "template": { "spec": { "tolerations": [ { "key": "CriticalAddonsOnly", "operator": "Exists" }, { "effect": "NoSchedule", "key": "node-role.kubernetes.io/master" } ] } } } }'
```

此问题的[跟踪信息](https://github.com/kubernetes/kubeadm/issues/1027)。

## NodeRegistration.Taints 字段在编组 kubeadm 配置时丢失

> 注意：这个问题仅适用于操作 kubeadm 数据类型的工具（例如 YAML 配置文件）。它将在 *kubeadm API v1beta2* 修复。

默认情况下，kubeadm 将 `node-role.kubernetes.io/master:NoSchedue` 污点应用于控制平面节点。如果希望 kubeadm 不污染控制平面节点，并将 `InitConfiguration.NodeRegistration.Taints` 设置成空切片，则应在编组时省略该手段。如果省略该手段，则 kubeadm 将应用默认污点。

至少有两种解决方法：  
1. 使用 `node-role.kubernetes.io/master:PreferNoSchedule` 污点代替空切片。除非非其他节点具有容量，否则将在主节点上调度 Pods。
2. 在 `kubeadm init` 退出后删除污点：
    ```
    kubectl taint node NODE_NAME node-role.kubernetes.io/master:NoSchedule-
    ```

## 节点上的 `/usr` 被以只读方式挂载

在类似 Fedora CoreOS 或者 Flatcar Container Linux 这类 Linux 发行版本中，目录 `/usr` 是以只读文件系统的形式挂载的。在支持 FlexVolume 时，类似 kubelet 和 kube-controller-manager 这类 Kubernetes 组件使用默认路径 `/usr/libexec/kubernetes/kubelet-plugins/volume/exec/`，而 FlexVolume 的目录**必须是可读写的**，该功能特性才能正常工作。

为了解决这个问题，可以使用 kubeadm 的配置文件来配置 FlexVolume 的目录。

在（使用 `kubeadm init` 创建的）主控制节点上，使用 `--config` 参数传入如下文件：
```
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
nodeRegistration:
  kubeletExtraArgs:
    volume-plugin-dir: "/opt/libexec/ubernetes/kubelet-plugins/volume/exec/"
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
controllerManager:
  extraArgs:
    flex-volume-pugin-dir: "/opt/libexec/kubernetes/kubelet-plugins/volume/exec/"
```

在加入到集群中的节点上，使用下面的文件：
```
apiVersion: kubeadm.k8s.io/v1beta2
kind: JoinConfiguration
nodeRegistration:
  kubeletExtraArgs:
    volume-plugin-dir: "/opt/libexec/ubernetes/kubelet-plugins/volume/exec/"
```

或者，可以更改 `/etc/fstab` 使得 `/usr` 目录能够以可写入的方式挂载，不过，请注意这样做本质是更改 Linux 发行版的某种设计原则。

## `kubeadm upgrade plan` 输出错误信息 context deadline exceeded

在使用 `kubeadm` 来升级某运行外部 etcd 的 Kubernetes 集群时可能显示这一错误。这并不是一个非常严重的一个缺陷，之所以出现此错误信息，原因是老的 kubeadm 版本会对外部 etcd 集群执行版本检查。可以继续执行 `kubeadm upgrade apply ...`。

这一问题已经在 1.19 版本中得到修复。

## `kubeadm reset` 会卸载 `/var/lib/kubelet`

如果已经挂载了 `/var/lib/kubelet` 目录，执行 `kubeadm reset` 操作的时候会将其卸载。

要解决这一问题，可以在执行了 `kubeadm reset` 操作之后重新挂载 `/var/lib/kubelet` 目录。

这是一个在 1.15 中引入的故障，已经在 1.20 版本中修复。

## 无法在 kubeadm 集群中安全地使用 metrics-server

在 kubeadm 集群中可以通过为 metrics-server 设置 `--kubelt-insecure-tls` 来以不安全的形式使用该服务。建议不要在生产环境集群中这样使用。

如果需要在 metrics-server 和 kubelet 之间使用 TLS，会有一个问题，kubeadm 为 kubelet 部署的是自签名的服务证书。这可能会导致 metrics-server 端报告下面的错误信息：
```
x509: certificate signed by unknown authority
x509: certificate is valid for IP-foo no IP-bar
```

参见[为 kubelet 启动签名的服务证书](../../../../Task/Management_cluster/Manage_with_kubeadm/Use_kubeadm_for_certificate_management.md) 以进一步了解如何在 kubeadm 集群中配置 kubelet 使用正确签名了的服务证书。


