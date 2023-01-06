# 无头服务(Headless Service)

有时不需要或不想要负载均衡，以及单独的 Service IP。遇到这种情况，可以通过指定 ClusterIP（`spec.clusterIP`）的值为 `None` 来创建 `Headless` Service。

可以使用一个无头服务与其他服务发现机制进行对接，而不必与 Kubernetes 的实现捆绑在一起。

对于 Headless Service 并不会分配 ClusterIP，kube-proxy 不会处理它们，而且平台也不会为它们进行负载均衡和路由。DNS 如何实现自动配置，依赖于 Service 是否定义了选择算符。

### 带选择算符的服务

对定义了选择算符的无头服务，Kubernetes 控制平面在 Kubernetes API 中创建 EndpointSlice 对象，并且修改 DNS 配置返回 A 或 AAAA 记录，通过这个地址直接到达 Service 的后端 Pod 上。

### 无选择算符的服务

对没有定义选择算符的无头服务，控制平面不会创建 EndpointSlice 对象。然而 DNS 系统会根据以下情况进行查找和配置：

* 对于 `type: ExternalName` 服务，查找和配置其 CNAME 记录
*   对所有其他类型的服务，针对 Service 的就绪端点的所有 IP 地址，查找和配置 DNS 的 A 或 AAAA 记录

    * 对于 IPv4 端点，DNS 系统创建 A 记录。
    * 对于 IPv6 端点，DNS 系统创建 AAAA 记录。

