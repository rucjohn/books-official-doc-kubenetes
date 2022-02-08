# 控制平面节点通信

本文列举了控制平面（确切说是 apiserver ）与 Kubernetes 集群之间的通信路。目的是允许用户自定义安装并加固网络配置，集群能够在不可信的网络上运行（或是在云服务商提供的公开的 IP）。

## 节点到控制平面

Kubernetes 采用的是星型（Hub-and-Spoke）API 模式。所有从集群（或所运行的 Pods）发出的 API 都汇聚到 apiserver。其他控制平面组成均未设计成可对外暴露服务。apiserver 使用安全的 HTTPS 端口（默认是 _**6443**_）监听远程连接请求，并启用客户端 [身份认证](../API/API-Access-Control/Authenticating.md) 机制，尤其是在允许匿名请求或服务账户令牌的情况下。

应为节点提供集群的公共根证书，以便它们能够基于有效的客户端凭据安全地连接到 apiserver。目前使用的方法是提供给 kubelet 的客户端凭据是以客户端证书的形式。相关证书配置，请参阅 kubelet [TLS 引导](../Tools/Compoent-tools/TLS-bootstrapping.md)。

如果 Pod 想要连接到 apiserver，可以使用服务账号进行安全连接。当 Pod 被实例化时，Kubernetes 自动把公共根证书和一个有效的持有者令牌注入到 Pod 里。`kubernetes` 服务（位于 `default` 命令空间）配置了一个虚拟 IP 地址，用于（通过 kube-proxy）转发请求到 apiserve 的 HTTPS 端口。

控制平面组件也可能通过安全端口与集群的 apiserver 通信。

这样，从节点和节点上运行的 Pod 到控制平面的默认的连接操作模式是默认安全的，并且可以在不受信任的或公共网络上运行。

## 控制平面到节点

从控制平面（apiserver）到节点有两种主要的通信方式：

1. 从 apiserver 到集群中每个节点上运行的 kubelet 进程。
2. 从 apiserver 通过它的代理功能连接到任何节点、Pod 或者服务。

### apiserver 到 kubelet

从 apiserver 到 kubelet 的连接用于：

* 获取 Pod 日志
* 挂接（通过 kubectl）到运行中的 Pod
* 提供 kubelet 的端口转发功能

这些连接汇聚到 kubelet 的 HTTPS 端口。默认情况下，apiserver 不检查 kubelet 的服务证书。这使得这类连接容易受到中间人攻击，在不受信息或公共网络上运行是不安全的。

为了对这类连接进行认证，使用 `--kubelet-certificate-authority` 标志给 apiserver 提供一个根证书，用于 kubelet 的服务证书。

如果无法实现这点，又要求避免在不受信任或公共网络上进行连接，可以在 apiserver 和 kubelet 之间使用 [SSH 隧道](Control-Plane-Node-Communication.md#ssh-tunnels)。

最后，应该启动 kubelet 用户认证/鉴权 来保护 kubelet API。

### apiserver 到节点、Pod 或者服务

### SSH 隧道 <a href="#ssh-tunnels" id="ssh-tunnels"></a>

### Konnectivity 服务
