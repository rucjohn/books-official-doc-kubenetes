# 词汇表

## Affinity

亲和性（_<mark style="color:orange;">**affinity**</mark>_）是一组规则，为调度程序程序提供在何处放置 Pod 提示信息。

亲和性有两种：

* 节点亲和性
* Pod 亲和性

这些规则是使用 Kubernetes 标签（Label）和 Pod 中指定的选择器（Selector）定义的，这些规则可以是强制的，也可以是弹性的，这取决于希望调度程序执行的严格程度。

## Annotation

注解（_<mark style="color:orange;">**annotation**</mark>_）是以键值对的形式给资源对象附加随机的无法标识的元数据。

注解中的元数据可大可小、结构化或非结构化，并且能够包含标签（Label）不允许使用的字符。工具和软件库等客户端可以检索这些元数据

## API Group

Kubernetes API 中的一组相关路径。

通过更改 API server 的配置，可以启用或禁用每个 API Group。还可以禁用或启动指向特定资源的路径。API Group 使扩展 Kubernetes API 更加的容易。API Group 在 REST 路径和序列化对象的 apiVersion 字段中指定。

## API Server

也称为：_<mark style="color:orange;">**kube-apiserver**</mark>_

API Server 是 Kubernetes 控制平面的组件，它公开了 Kubernetes API。API Server 是 Kubenetes 控制平面的入口。

Kubernetes API Server 的主要实现就是 kube-apiserver。kube-apiserver 设计上考虑了水平伸缩，也就是说，它可以通过部署多个实例进行伸缩，并在这些实例间平衡流量。

## Applications

各种容器化应用运行所在的层。

## Cgroup

一组具有资源隔离、审计和限制的 Linux 进程。

cgroup 是一个 Linux 内核特性，对一组进程的资源使用（CPU、内存、磁盘 I/O 和网络等）进行限制、审计和隔离。

## CustomResourceDefinition

通过定制化的代码给 Kubernetes API 服务器增加资源对象，而无需编译完整的定制 API 服务。

当 Kubernetes 公开支持的 API 资源不能满足需求时，定制资源对象（Custom Resource Definitions）可以在现有环境上扩展 Kubernetes API。

## DaemonSet

确保 Pod 副本在集群中的一组节点上运行。

用来部署系统守护进程，例如：日志搜索和监控代理，这些进程通常必须运行在每个节点上。

### Deployment[^1] <a href="#deployment" id="deployment"></a>

管理多副本应用的一种 API 对象，通常通过运行无状态的 Pod 来完成工作。

每个副本表现为一个 Pod，Pod 分布在集群中的节点上。对于确实需要本地状态的工作负载，请参考使用 StatefulSet。





[^1]: 
