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
  - 更改 kubelet 配置以手动匹配 Docker cgroup 驱动程序，可以参考[配置 cgroup 驱动](../../../Task/Management_cluster/Manage_with_kubeadm/Configure_cgroup_driver.md)

