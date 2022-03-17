# 词汇表

## API Group

Kubernetes API 中的一组相关路径。

通过更改 API server 的配置，可以启用或禁用每个 API Group。还可以禁用或启动指向特定资源的路径。API Group 使扩展 Kubernetes API 更加的容易。API Group 在 REST 路径和序列化对象的 apiVersion 字段中指定。

## CustomResourceDefinition

通过定制化的代码给 Kubernetes API 服务器增加资源对象，而无需编译完整的定制 API 服务。

当 Kubernetes 公开支持的 API 资源不能满足需求时，定制资源对象（Custom Resource Definitions）可以在现有环境上扩展 Kubernetes API。

## DaemonSet

