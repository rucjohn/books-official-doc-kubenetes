# TLS 启动引导

在一个 Kubernetes 集群中，工作节点上的组件（kubelet 和 kube-proxy）需要与 Kubernetes 主控组件通信，尤其是 kube-apiserver。为了确保通信本身是私密的、不被干扰的，并且确保集群的每个组件都在与另一个可信的组件通信，强烈建议使用节点上的客户端 TLS 证书。

引导这些组件，尤其是需要证书让其可以与 kube-apiserver 安全通信的工作节点，可能是一个具有挑战性的过程，因为它需要大量的额外工作，这通常超出了 Kubernetes 的范围。因此，这会儿初始化或扩展集群变得更具挑战性。

从 1.4 版本开始，Kubernetes 引入了一个证书请求和签名 API，以便简化这一过程。

本文档描述了节点初始化过程，如何为 kubelet 配置 TLS 客户端证书引导，以及其背后的工作原理。

## 初始化过程

当工作节点启动时，kubelet 执行以下动作：

1. &nbsp;查找自己的 `kubeconfig` 文件  
2. &nbsp;通过 `kubeconfig` 文件的 TLS 密钥和签名证书，检索 apiserver 的 URL 和凭据  
3. &nbsp;尝试使用这些凭据与 apiserver 通信  

如果 kube-apiserver 成功认证了 kubelet 的凭据数据，它会将 kubelet 视为一个有效节点，并开始为其分配 Pod。

注意， **签名的过程依赖于：**
- `kubeconfig` 中包含密钥和本地主机的证书
- 证书被 kube-apiserver 所信任的一个证书机构（CA）所签名

集群管理员需要完成以下动作：  

1. &nbsp;创建 CA 密钥和证书  
2. &nbsp;将 CA 证书发布到 kube-apiserver 所在的主控节点上  
3. &nbsp;为每个 kubelet 创建密钥和证书；强烈建议为每个 kubelet 使用唯一的、CN 值不同的密钥和证书  
4. &nbsp;使用 CA 密钥对 kubelet 证书签名  
5. &nbsp;将 kubelet 密钥和签名的证书发布到 kubelet 所在的特定节点上  

本文中描述的 TLS 启动引导过程旨在简化上述过程，甚至完全自动化，尤其是第 3 步之后操作，因为这些步骤是初始化或扩展集群时最常见的操作。

### 引导初始化

在启动引导初始化过程，会发生以下事情：  

1. kubelet 启动  
2. kubelet 发现没有对应的 `kubeconfig` 文件  
3. kubelet 搜索并发现 `bootstrap-kubeconfig` 文件  
4. kubelet 读取该引导文件，获取 apiserver 的 URL 和一个用途有限的 Token  
5. kubelet 使用 Token 与 apiserver 建立连接并进行身份认证  
6. kubelet 现在拥有受限制的凭据，以此来创建和获取证书签名请求（CSR）  
7. kubelet 为自己创建一个 CSR，并将其 signerName 设置为 `kubernetes.io/kube-apiserver-client-kubelet`  
8. CSR 通过以下两种方式获得批复：  
    - 如果已配置，kube-conroller-manager 会自动批复该 CSR。
    - 如果已配置，外部流程（可能是个人）使用 Kubernetes API 或通过 `kubectl` 来批复该 CSR。
9. kubelet 所需要的证书被创建  
10. 证书被发放到 kubelet  
11. kubelet 取回该证书  
12. kubelet 创建一个合适的 `kubeconfig`，其中包含密钥和已签名的证书  
13. kubelet 开始正常工作  
14. kubelet 在证书快过期时自动请求更新证书（可选的，如果配置了参数）  
15. kubelet 要求更新的证书被批复并发放（可选的，如果配置了参数）  

本文的其余部分描述配置 TLS 启动引导的必要步骤及其局限性。





