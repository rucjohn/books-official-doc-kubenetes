# 控制平面节点通信

本文列举了控制平面（确切说是 apiserver ）与 Kubernetes 集群之间的通信路。目的是允许用户自定义安装并加固网络配置，集群能够在不可信的网络上运行（或是在云服务商提供的公开的 IP）。

## 节点到控制平面

Kubernetes 采用的是星型（Hub-and-Spoke）API 模式。所有从集群（或所运行的 Pods）发出的 API 都汇聚到 apiserver。其他控制平面组成均未设计成可对外暴露服务。apiserver 使用安全的 HTTPS 端口（默认是 _**6443**_）监听远程连接请求，并启用客户端 [身份认证](../API/API-Access-Control/Authenticating.md) 机制，尤其是在允许匿名请求或服务账户令牌的情况下。

