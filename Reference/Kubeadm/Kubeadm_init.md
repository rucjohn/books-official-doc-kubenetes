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

## 从父命令继承的选项

* `rootfs string`  
**[实验]** “真实” 主机根文件系统的路径。  
&emsp;

## `kubeadm init` 命令的工作流程

`kubeadm init` 命令通过执行下列步骤来启动一个 Kubernetes 控制平面节点。  

1. 在做出变更前运行一系列的预检选项来验证系统状态。一些检查选项仅仅触发警告，其它的则会被为错误并且退出 kubeadm，除非问题得到解决或者用户指定了 `--ignore-preflight-error=<错误列表>` 参数。  
&emsp;

2. 生成一个自签名的 CA 证书来为集群中每一个组件建立身份标识。用户可以通过将其放入 `--cert-dir` 配置的证书目录中（默认为 `/etc/kubernetes/pki`）来提供他们自己的 CA 证书以及或者密钥。apiserver 证书将为任何 `--apisrver-cert-extra-sans` 参数值提供附加的 SAN 条目，必要时将其小写。  
&emsp;

3. 将 kubeconfig 文件写入 `/etc/kubernetes` 目录，以便 kubelet、controller manager 和 scheduler 可以连接到 apiserver，它们每一个都有自己的身份标识，同时生成一个名为 `admin.conf` 的独立的 kubeconfig 文件，用于管理操作。  
&emsp;

4. 为 apiserver、controller manager 和 scheduelr 生成静态 Pod 清单文件。假如没有提供一个外部的 etcd 服务的话，也会为 etcd 生成一份额外的静态 Pod 清单文件。文件被写入到 `/etc/kubernetes/manifests` 目录，kubelet 会监视这个目录以便在系统启动的时候创建 Pod。一旦控制平面的 Pod 都运行起来，`kubeadm init` 的工作就继续往下执行。  
&emsp;

5. 对控制平面节点设置标签和污点标记，以便不会在它上面运行其它的工作负载。  
&emsp;

6. 生成令牌，将来其他节点可使用该令牌向控制平面注册自己。如 [kubeadm token](Kubeadm_token.md) 文档所述，用户可以选择通过 `--token` 提供令牌。  
&emsp;

7. 为了使得节点能够遵照 [启动引导令牌](../API_access_control/Authenticating_with_Bootstrap_Tokens.md) 和 [TLS启动引导](../Component_tools/TLS_bootstrapping.md) 这两份文档中描述的机制加入到集群中，kubeadm 会执行所有的必要配置：
    - 创建一个 ConfigMap 提供添加集群节点所需的信息，并为该 ConfigMap 设置相关的 RABC 访问规则。
    - 允许启动引导令牌访问 CSR 签名 API。
    - 配置自动签名新的 CSR 请求。
&emsp;

8. 通过 apiserver 安装一个 DNS 服务器（CoreDNS）和 kube-proxy 附加组件。在 Kubernetes 1.11 版本以后，CoreDNS 是默认的 DNS 服务器。请注意，尽管已部署 DNS 服务器，但直到安装 CNI 时才调度它。
&emsp;

> 警告：从 v1.18 版本开始，在 kubeadm 中使用 kube-dns 的支持已被废弃，并已在 v1.21 版本中删除。

## 在 kubeadm 中使用 init phases

kubeadm 允许使用 `kubeadm init phase` 命令分阶段创建控制平面节点。

要查看阶段和子阶段的有序列表，可以调用 `kubeadm init --help`。该列表将位于帮助屏幕的顶部，每个阶段旁边都有描述。

> 注意，通过调用 `kubeadm init`，所有阶段和子阶段都将按照此顺序执行。

```
sudo kubeadm init phase control-plane controller-manager --help
```

也可以使用 `--help` 查看稳定父阶段的子阶段列表：
```
sudo kubeadm init phase control-plane --help
```

`kubeadm init` 还公开了一个名为 `--skip-phases` 的参数，该参数可用于跳过某些阶段。参数接受阶段名称列表，这个名称列表可以从上面的有序列表中获取。
```
sudo kubeadm init phase control-plane all --config=configfile.yaml
sudo kubeadm init phase etcd local --config=configfile.ymal
# 现在可以修改控制平面和 etcd 清单文件
kubeadm init --skip-phase=control-plane,etcd --config=configfile.yaml
```

示例中将执行的操作是基于 `configfile.yaml` 中的配置在 `/etc/kubernetes/manifests` 中写入控制平面和 etcd 的清单文件。允许修改文件，然后使用 `--skip-phases` 跳过这些阶段。通过调用最后一个命令，将使用自定义清单文件创建一个控制平面节点。



