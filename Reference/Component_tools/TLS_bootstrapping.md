# TLS 引导

在一个 Kubernetes 集群中，工作节点上的组件（kubelet 和 kube-proxy）需要与 Kubernetes 主控组件通信，尤其是 kube-apiserver。为了确保通信本身是私密的、不被干扰的，并且确保集群的每个组件都在与另一个可信的组件通信，强烈建议使用节点上的客户端 TLS 证书。

引导这些组件，尤其是需要证书让其可以与 kube-apiserver 安全通信的工作节点，可能是一个具有挑战性的过程，因为它需要大量的额外工作，这通常超出了 Kubernetes 的范围。因此，这会儿初始化或扩展集群变得更具挑战性。

从 1.4 版本开始，Kubernetes 引入了一个证书请求和签名 API，以便简化这一过程。

本文档描述了节点初始化过程，如何为 kubelet 配置 TLS 客户端证书引导，以及其背后的工作原理。

## 初始化过程

当工作节点启动时，kubelet 执行以下动作：

1. &nbsp;&nbsp;查找自己的 `kubeconfig` 文件  
2. &nbsp;&nbsp;通过 `kubeconfig` 文件的 TLS 密钥和签名证书，检索 apiserver 的 URL 和凭据  
3. &nbsp;&nbsp;尝试使用这些凭据与 apiserver 通信  

如果 kube-apiserver 成功认证了 kubelet 的凭据数据，它会将 kubelet 视为一个有效节点，并开始为其分配 Pod。

注意， **签名的过程依赖于：**
- `kubeconfig` 中包含密钥和本地主机的证书
- 证书被 kube-apiserver 所信任的一个证书机构（CA）所签名

集群管理员需要完成以下动作：  

1. &nbsp;&nbsp;创建 CA 密钥和证书  
2. &nbsp;&nbsp;将 CA 证书发布到 kube-apiserver 所在的主控节点上  
3. &nbsp;&nbsp;为每个 kubelet 创建密钥和证书；强烈建议为每个 kubelet 使用唯一的、CN 值不同的密钥和证书  
4. &nbsp;&nbsp;使用 CA 密钥对 kubelet 证书签名  
5. &nbsp;&nbsp;将 kubelet 密钥和签名的证书发布到 kubelet 所在的特定节点上  

本文中描述的 TLS 引导过程旨在简化上述过程，甚至完全自动化，尤其是第 3 步之后操作，因为这些步骤是初始化或扩展集群时最常见的操作。

### 引导初始化

在引导初始化过程，会发生以下事情：  

1. &nbsp;&nbsp;kubelet 启动  
2. &nbsp;&nbsp;kubelet 发现没有对应的 `kubeconfig` 文件  
3. &nbsp;&nbsp;kubelet 搜索并发现 `bootstrap-kubeconfig` 文件  
4. &nbsp;&nbsp;kubelet 读取该引导文件，获取 apiserver 的 URL 和一个用途有限的 Token  
5. &nbsp;&nbsp;kubelet 使用 Token 与 apiserver 建立连接并进行身份认证  
6. &nbsp;&nbsp;kubelet 现在拥有受限制的凭据，以此来创建和获取证书签名请求（CSR）  
7. &nbsp;&nbsp;kubelet 为自己创建一个 CSR，并将其 signerName 设置为 `kubernetes.io/kube-apiserver-client-kubelet`  
8. &nbsp;&nbsp;CSR 通过以下两种方式获得批复：  
    - 如果已配置，kube-conroller-manager 会自动批复该 CSR。
    - 如果已配置，外部流程（可能是个人）使用 Kubernetes API 或通过 `kubectl` 来批复该 CSR。
9. &nbsp;&nbsp;kubelet 所需要的证书被创建  
10. &nbsp;&nbsp;证书被发放到 kubelet  
11. &nbsp;&nbsp;kubelet 取回该证书  
12. &nbsp;&nbsp;kubelet 创建一个合适的 `kubeconfig`，其中包含密钥和已签名的证书  
13. &nbsp;&nbsp;kubelet 开始正常工作  
14. &nbsp;&nbsp;kubelet 在证书快过期时自动请求更新证书（可选的，如果配置了参数）  
15. &nbsp;&nbsp;kubelet 要求更新的证书被批复并发放（可选的，如果配置了参数）  

本文的其余部分描述配置 TLS 引导的必要步骤及其局限性。

## 配置

要配置 TLS 启动引导及可选的自动批复，必须配置以下组件的选项：
- kube-apiserver
- kube-controller-manager
- kubelet
- `ClusterRoleBinding` 以及可能需要的 `ClusterRole`

此外，需要有 Kubernetes 证书机构（Ceritificate Authority，CA）

## 证书机构

就像在没有启动引导的情况下，会需要证书机构（CA）密钥和证书。这些数据会被用来对 kubelet 证书进行签名。如前所述，将证书机构密钥和证书发布到主控节点是管理员的责任。

就本文而言，假定这些数据已经发布到主控节点上的 `/var/lib/kubernetes/ca.pem`（证书）和 `/var/lib/kubernetes/ca-key.pem`（密钥）文件中，这两个文件被称作 “Kubernetes CA 证书和密钥”，并且假定证书和密钥都是 PEM 编码的。所有 Kubernetes 组件（kubelet、kube-apiserver、kube-controller-manager）都使用这些凭据，

## kube-apiserver 配置

启动 TLS 引导对 kube-apiserver 有若干需求：
- 能够识别对客户端证书进行签名的 CA
- 能够对启动引导的 kubelet 执行身份认证，并将其置入 `system:bootstrappers` 组
- 能够对启动引导的 kubelet 执行鉴权操作，允许其创建证书签名请求（CSR）

### 识别客户证书

对于所有客户端证书的认证操作而言，这是很常见的。如果还没有设置，则要为 kube-apiserver 添加 `--client-ca-file=FILENAME` 标志来启用客户端证书认证，在标志中引用一个包含用来签名的证书的证书机构包，例如：`--client-ca-file=/var/lib/kubernetes/ca.pem`。

### 初始引导认证

为了让引导的 kubelet 能够连接到 kube-apiserver 并请求证书，它必须首先向服务器进行身份认证。可以使用任何一种能够对 kubelet 进行身份认证的 [身份认证组件](../API_access_control/Authenticating.md) 。

尽管所有身份认证策略都可以用于 kubelet 初始引导凭据来进行认证，但为了便于配置，建议使用以下两个身份认证组件：
- Bootstrap Token
- 令牌认证文件

Bootstrap Token 是一种对 kubelet 进行身份认证的方法，相对简单且容易管理，并且不需要在启动 kube-apiserver 时设置额外的标志。Bootstrap Token 从 Kubernetes 1.12 开始是处于 ***beta*** 功能特性。

无论选择哪种方法，都要求 kubelet 能够以具有以下权限的用户进行身份认证：  

1. 创建和读取 CSR
2. 在启用了自动批复时，能够在请求节点客户端证书时得到自动批复

使用 Bootstrap Token 进行身份认证的 kubelet 会被认证为 `system:bootstrappers` 组中的用户，这是一种标准方法。

随着这个功能特性的逐渐成熟，需要确保 token 绑定到某个基于角色的访问控制（RBAC）策略上，从而严格限制仅限于客户端申请提供证书的请求。当 RBAC 被配置启用时，可以将 token 限制到某个组，从而提高灵活性。例如，可以在准备节点期间禁止某特定引导组的访问。

### Bootstrap Token

Bootstrap Token 在 Kubernetes 集群中存储为 Secret 对象，被发放到各个 kubelet。可以在整个集群中使用同一个 token，也可以为每个节点发放单独的 token。

这一过程有两个方面：

1. 基于 token id、secret 和范畴信息创建 Kubernetes Secret。
2. 将 token 发放给 kubelet。

- 从 kubelet 的角度，所有 token 看起来都很像，没有特别的含义。
- 从 kube-apiserver 的角度，Bootstrap Token 是很特殊的。根据其 `type`、`namespace` 和 `name`，kube-apiserver 能够认其为特殊的 token，并授予携带该 token 的任何人以特殊的引导权限，换言之，将其视为 `system:bootstrappers` 组的成员。这就满足了 TLS 引导的基本需求。

如果希望使用 Bootstrap Token，必须在 kube-apiserver 上启用以下标志：
```
--enable-bootstrap-token-auth=true
```

### 令牌认证文件

kube-apiserver 能够将 token 视作身份认证依据。这些 token 可以是任意数据，但必须表示为基于某个安全随机数生成器而得到的 128 位混沌数据。这里的随机数生成器可以是现代 Linux 系统上的 `/dev/urandom`。生成 token 的方式有很多种。例如：
```
head -c 16 /dev/urandom | od -An -t x | tr -d ' '
```

上面的命令会生成有类似开 `02b50b05283e98dd0fd71db496ef01e8` 这样的 token。

token 文件示例：
```
02b50b05283e98dd0fd71db496ef01e8,kube-let-bootstrap,10001,"system:bootstrappers"
```

以上示例中，前面三个值可能是任何值，但是最后一段""内的组名则必须是固定的。

向 kube-apiserver 添加 `--token-auth-file=FILENAME` 标志以启动令牌文件。

### 授权 kubelet 创建 CSR

既然引导节点被认证为 `system:bootstrapping` 组的成员，则需要授权它创建证书签名请求（CSR），并在证书被签名之后将其取回。幸运的是，Kubernetes 附带了一个 `ClusterRole`，这具有（并且仅有）这些权限：`system:node-bootstrapper`。

为了实现这一点，只需要创建 `ClusterRoleBinding`，将 `system:bootstrappers` 组绑定到集群角色 `system:node-bootstrapper`。
```
apiVersion: rabc.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: create-csrs-for-bootstrapping
subjects:
- kind: Group
  name: system:bootstrappers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: system:node-bootstrapper
  apiGroup: rbac.authorization.k8s.io
```







