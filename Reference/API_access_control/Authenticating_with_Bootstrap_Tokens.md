# 使用启动引导令牌（Bootstrap Tokens）认证

** 特性：Kubernetes v1.18 [stable] **

启动引导令牌（Bootstrap Tokens）是一种简单的持有者令牌（Bearer Token），这种令牌是在新建集群或者在现有集群中添加新节点时使用的。
- 它被设计成能够支持 `kubeadm`，但是也可以被用在其他的案例中以便用户在使用的 `kubeadm` 的情况下启动集群。
- 它也被设计成可以通过 RBAC 策略，结合 [Kubelet TLS 启动引导](../Component_tools/TLS_bootstrapping.md) 系统进行工作。

Bootstrap Tokens 被定义成一个特定类型的 Secret（`bootstrap.kubernetes.io/token`），并存在于 `kube-system` 命名空间中。这些 Secret 会被 apiserver 的启动引导论证组件（Bootstrap Authenticator）读取。controller manager 中的 TokenCleaner 控制器能够删除过期的令牌。这些令牌还用于通过 BootstarpSigner 控制器在节点发现的过程中使用的特定的 ConfigMap 创建签名。

## 令牌格式

Bootstrap Tokens 使用 `abcdef.0123456789abcdef` 的形式。规范地说，它们必须符合正则表达式：`[a-z0-9]{6}\.[a-z0-9]{16}`。
- 令牌的第一部分是 "Token ID"，它是公开的，用于引用令牌并确保不会泄露认证使用的秘密信息。
- 令牌的第二部分是 "Token Secret"，它被共享给受信的第三方。

## 启用身份认证

Bootstrap Tokens 认证组件可以通过 apiserver 的如下标志启用：
```
--enable-bootstrap-token-auth
```

Bootstrap Tokens 被启用后，可以作为持有者令牌的凭据，用于 apiserver 请求的身份认证。
```
Authorization: Bearer abcdef.0123456789abcdef
```

令牌认证为用户名 `system:bootstrap:<token id>`，并且是组 `system:bootstrappers` 的成员。额外的组信息可以通过令牌的 Secret 来设置。

过期的令牌可以通过启用 `tokencleaner` 控制器来删除。

