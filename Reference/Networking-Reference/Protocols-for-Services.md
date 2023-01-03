# Service 所用的协议

## 支持的协议

### SCTP

特性状态：Kubernetes v1.20 [stable]

当使用支持 SCTP 流量的网络插件时，可以为大多数 Service 使用 SCTP。对于 `type: LoadBalancer` Service，以 SCTP 的支持情况取决于提供此项设施的云厂商（大部分不支持）。

运行 Windows 的节点不支持 SCTP。

