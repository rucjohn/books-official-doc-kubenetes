# kubeadm init

此命令初始化一个 Kubernetes 控制平面节点。

## 概要

运行此命令来搭建 Kubernetes 控制平面节点。

`kubeadm init [flags]` 命令执行以下步骤：
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
  /experimental-cert-rotation   启动 kubelet 客户端证书轮换
addon                         安装通过一致性测试的所需的插件
  /coredns                      将 CoreDNS 插件安装到 Kubernetes 集群中
  /kube-proxy                   将 kube-proxy 插件安装到 Kubernetes 集群中
```

## 选项

* `--apiserver-advertise-address string`  
apiserver 正在监听的 IP 地址。如果未设置，则使用默认网络接口。  
&emsp;

* `--apiserver-bind-port int32`  默认值：`6443`  
apiserver 绑定端口。  
&emsp;

* `--apiserver-cert-extra-sans stringSlice`  
用于 apiserver 证书的可选附加主题备用名称（SAN）。可以是 IP 地址和 DNS 名称。  
&emsp;

* `--cert-dir string`  默认值：`/etc/kubernetes/pki`  
保存和存储证书的路径。  
&emsp;

* `--certificate-key string`  
用于加密 kubeadm-certs Secret 中的控制平面证书的密钥。  
&emsp;

* `--config string`  
kubeadm 配置文件的路径。  
&emsp;

* `--control-plane-endpoint string`  
为控制平面指定一个稳定的 IP 地址或 DNS 名称。  
&emsp;

* `--cri-socket string`  
要连接的 CRI 套接字的路径。如果为空，则 kubeadm 将尝试自动检测此值；仅当安装了多个 CRI 或具有非标准 CRI 插槽时，才使用此选项。  
&emsp;

* `--dry-run`  
不应用任何更改；只输出将要执行的操作。  
&emsp;

* `--experimental-pathces string`  
包含名为 "target[suffix][+patchtype].extension" 的文件的目录路径。例如，"kube-apiserver0+merge.yaml" 或仅仅是 "etcd.json"。"patchtype" 可以是 `strategic`、`merge` 或 `json` 其中一个，并且它们与 kubectl 支持的补丁格式匹配。默认的 "patchtype" 为 `strategic`，"extension" 必须是 `json` 或 `yaml`。"suffix" 是一个可选字符串，可用于确定首先按字母顺序应用哪些补丁。  
&emsp;

* `--feature-gates string`  
一组用来描述各种功能性的键值对（key=value）。选项如： `IPv6DualStack=true|false(ALPHA - default=false)`。  
&emsp;

* `-h, --help`  
帮助命令。  
&emsp;

* `--ignore-preflight-errors stringSlice`  
忽略运行前检查时出现的错误的列表，显示为告警。例如：`IsPrivilegeUser,Swap`。取值为 `all` 时将忽略检查中的所有错误。  
&emsp;

* `--image-repository string`  默认值：`k8s.gcr.io`  
选择用于拉取控制平面镜像的容器仓库。  
&emsp;

* `--kubernetes-version string`  默认值：`stable-1`  
为控制平面选择一个特定的 Kubernetes 版本。  
&emsp;

* `--node-name string`  
指定节点的名称。  
&emsp;

* `--pod-network-cidr string`  
指明 pod 网络可以使用的 IP 地址段。如果设置了这个参数，控制平面将会为每一个节点自动分配 CIDRs。  
&emsp;

* `--servie-cidr string`  默认值：`10.96.0.0/12`  
指定 service 的 IP 地址段。  
&emsp;

* `--service-dns-domain string`  默认值：`cluster.local`  
指定 service 域名，例如：`myorg.internal`。  
&emsp;

* `--skip-certificate-key-print`  
不打印用于加密控制平面证书的密钥。  
&emsp;

* `--skip-phases stringSlice`  
需要跳过的阶段列表。  
&emsp;

* `--sikip-token-print`  
不打印 `kubeadm init` 生成的默认引导令牌。  
&emsp;

* `--token string`  
这个令牌用于建立控制平面节点与工作节点间的双向难。格式为 `[a-z0-9]{6}\.[a-z0-9]{16}`。例如：`abcdef.0123456789abcdef`。  
&emsp;

* `--token-ttl duration`  默认值：`24h0m0s`  
信息被自动删除之前的持续时间（例如 1s, 2m, 3h）。如果设置为 "0"，则令牌将永不过期。  
&emsp;

* `--upload-certs`  
将控制平面证书上传到 kubeadm-certs Secret。  
&emsp;


