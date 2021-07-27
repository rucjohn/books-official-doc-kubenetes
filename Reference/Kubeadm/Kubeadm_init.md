# kubeadm init

此命令初始化一个 Kubernetes 控制平面节点。

## 概要

运行此命令来搭建 Kubernetes 控制平面节点。

"init" 命令执行以下步骤：
```
preflight                     运行前检查
certs                         证书生成
  /ca                           生成为其他 Kubernetes 组件提供身份的自签名 Kubernetes CA
  /apiserver                    生成 apiserver 证书
  /apiserver-kubelet-client     生成 apiserver 连接 kubelet 的客户端证书
  /front-proxy-ca               生成为前端代理提供身份的自签名 CA
  /front-proxy-client           生成前端代理的客户端证书
  /etcd-ca                      生成为 etcd 提供身份的自签名 CA
  /etcd-server                  生成 etcd 服务端证书
  /etcd-peer                    生成 etcd 集群间通信的证书
  /etc/-healthcheck-client      生成进行 etcd 健康检查的 liveness 探针证书
  /apiserver-etcd-client        生成 apiserver 访问 etcd 的客户端证书
  /sa                           生成用于签署 SA(service account) 令牌及其公钥的私钥
kubeconfig                    生成用于建立和管理控制平台所需的所有 kubeconfig 文件
  /admin                        生成一个供管理员和 kubeadm 自身使用的配置文件
  /kubelet                      生成一个供 kubelet 使用的仅用于集群引导目的的配置文件
  /controller-manager           生成一个供 controller manager 使用的配置文件
  /scheduler                    生成一个供 scheduler 使用的配置文件
kubelet-start                 写入 kubelet 设置并启动/重启 kubelet
control-plane                 生成建立控制平面所需的所有静态 Pod 清单文件
  /apiserver                    生成 kube-apiserver 静态 Pod 清单文件
  /controller-manager           生成 kube-controller-manager 静态 Pod 清单文件
  /scheduler                    生成 kube-scheduler 静态 Pod 清单文件
etcd                          生成用于本地 etcd 的静态 Pod 清单文件
  /local                        生成本地、单节点的 etcd 实例的静态 Pod 清单文件
upload-config                 将 kubeadm 和 kubelet 配置上传到 configmap
  /kubeadm                      将 kubeadm ClusterConfiguration 上传到 configmap
  /kubelet                      将 kubelet 组件配置上传到 configmap
upload-cents                  上传证书到 kubeadm-certs
mark-control-plane            将节点标记为控制平面
bootstrap-token               生成用于将节点加入集群的引导令牌
kubelet-finalize              在 TLS 引导后更新与 kubelet 相关的设置
  /experimental-cert-rotation   
```
