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


