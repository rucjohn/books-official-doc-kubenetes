# kubeadm init

此命令初始化一个 Kubernetes 控制平面节点。

## 概要

运行此命令来搭建 Kubernetes 控制平面节点。

"init" 命令执行以下步骤：
```
preflight                     运行前检查
certs                         证书生成
  /ca                             生成为其他 Kubernetes 组件提供身份的自签名 Kubernetes CA
  /apiserver                      生成 apiserver 证书
  /apiserver-kubelet-client       生成 apiserver 连接 kubelet 的客户端证书
  /front-proxy-ca                 生成为前端代理提供身份的自签名 CA
  /front-proxy-client             生成前端代理的客户端证书
  /etcd-ca                        生成为 etcd 提供身份的自签名 CA
```
