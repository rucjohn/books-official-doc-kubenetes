# 集群管理

集群管理概述面向任何创建和管理 Kubernetes 集群的讲者人群。

## 规划集群

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

并非所有发行版都是被积极维护的。请选择使用最新的且已测试过的 Kubernetes 发行版。
{% endhint %}

在选择之前，需要考虑以下因素：

* 是打算在测试环境尝试 Kubernetes，还是要构建一个高可用的多节点集群？请选择最适合需求的发行版。
* 正在使用类似 Google Kubernetes Engine 的 Kubernetes 托管集群，还是手动管理自建的集群？
* 集群是本地还是云上？Kubernetes 不支持混合集群，但可以建立多个集群。
* 如果本地 Kubernetes 集群，需要考虑哪种 [网络模型](Cluster-Networking.md)？
* Kubernetes 是在物理机还是在虚拟机上运行？
* 是想运行一个集群，还是打算参与 Kubernetes 二次开发？如果是打后者，请选择一个处于开发状态的发行版。某些发行版只提供二进制程序。
* 需要熟悉一个集群运行所需的所有 [组件](../Overview/Kubernetes-Components.md)。

## 管理集群

* 学习如何 [管理节点](../Cluster-Architecture/Nodes.md)。
* 学习如何配置和管理集群共享的 [资源配额](../Cluster-Architecture/Nodes.md)。

## 保护集群

| **章节**                    | **描述**                                  |
| ------------------------- | --------------------------------------- |
| [生成证书](Certificates.md)   | 描述了使用不同的工具链生成证书的步骤。                     |
| Kubernetes 容器环境           | 描述了 Kubernetes 节点上由 Kubelet 管理的容器的环境。   |
| 控制对 Kubernetes API 的访问    | 描述了 Kubernetes 如何为自己的 API 实现访问控制。       |
| 身份认证                      | 阐述了 Kubernetes 中的身份认证功能，包括许多认证选项。       |
| 鉴权                        | 与身份认证不同，用于控制如何处理HTTP请求。                 |
| 使用准入控制器                   | 阐述了在认证和挖树之后拦截到 Kubernetes API 服务的请求的插件。 |
| 在 Kubernetes 集群中使用 sysctl | 描述了管理员如何使用 `sysctl` 命令行工具来设置内核参数。       |
| 审计                        | 描述了如何与 Kubernetes 的审计日志交互。              |

## 保护 kubelet

* 节点与控制面之间的通信
* TLS 启动引导
* Kubelet 认证/授权

## 可选集群服务

* DNS 集成描述了如何将一个DNS 域名解析到一个 Kubernetes Service。
* 记录和监控集群活动阐述了 Kubernetes 的日志如何工作以及怎样实现。
