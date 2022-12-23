# 概述

**Kubernetes** 是一个可移植、可扩展的开源平台，用于管理容器化的工作负载和服务，有助于声明式配置和自动化。它拥有庞大且快速发展的生态系统。Kubernetes 的服务、支持和工具随处可见。

**Kubernetes** 这个名字源于希腊语，意思是 “舵手” 或者 “飞行员”。_**k8s**_ 这个缩写单词，是因为 "k" 和 "s" 之间有八个字符。Google 在 2014 年开源了 Kubernetes 项目。Kubernetes 结合了 Google 超过十几年的大规模运行生产工作负载方面的经验，以及来自社区的最佳创意和实践。

## 时光回溯

让我们回顾一下为什么 Kubernetes 如此有用。

![](../../.gitbook/assets/container\_evolution.png)

**传统部署时代：**

早期，各个组织机构在物理服务器上运行应用程序。无法为物理服务器中的应用程序定义资源边界，这导致资源分配问题。例如，如果一台物理服务器上运行多个应用程序，则可能会出现一个应用程序占用大部分资源的情况，因此导致其他应用程序的性能下降。对此的解决方案是在不同的物理服务器上运行不同的应用程序，这也导致资源无法充分利用且无法扩展，同时维护这些物理服务器的成本很高。

**虚拟化部署时代：**

引入虚拟化作为新的解决方案。它允许在单个物理服务器的 CPU 上运行多个虚拟机（VM）。虚拟化允许应用程序在 VM 之间隔离并提供一定级别的安全性，因为一个应用程序的信息不能被另一个应用程序随意访问。

虚拟化技术能够更好地利用物理服务器上的资源，因为可以轻松地添加或删除应用程序，实现更好的可伸缩性，降低硬件成本等等。

每个虚拟机都是一台完整的计算机，在虚拟化硬件之上运行所有组件，包括其自己的操作系统。

**容器部署时代：**

容器类似于虚拟机，但是它们具有更宽松的隔离特性，可以在应用程序之间共享操作系统（OS）。因此，容器被认为是轻量级的。容器与虚拟机类似，它有自己的文件系统、CPU、内存、进程空间等。由于它们与底层基础架构分离，因此可以跨云和操作系统进行移植。

容器之所以流行，是因为它们提供了额外的功能：

* 应用程序的快速创建与部署：与使用虚拟机镜像相比，容器镜像创建更容易、更高效。
* 持续开发、集成和部署：提供可靠且频繁的容器镜像构建和部署，以及快速高效回滚（由于镜像不可变性）。
* 开发与运维关注点分离：在构建/发布时创建应用程序容器镜像，而不是在部署时，从而将应用程序与基础设施解耦。
* 可观测性：不仅可以显示操作系统级别的信息和指标，还可以显示应用程序的运行状况和其他指标信号。
* 开发、测试和生产的环境一致性：在本地笔记本上运行的方式与在云上的运行方式相同。
* 跨云和操作系统的可移植性：可在 Ubuntu、RHEL、CentOS、本地、Google Kubernetes Engine 和其他任何地方运行。
* 以应用程序为中心的管理：将抽象级别从在虚拟硬件上运行操作系统，提高到使用逻辑资源在操作系统上运行应用程序。
* 松耦合、分布式、弹性、开放的微服务：应用程序被分解成更小的、独立的部分，可以动态部署和管理，而不是在一台大型单机上整体运行。
* 资源隔离：可预测的应用程序性能。
* 资源利用：高效率和高密度。

## 为什么需要 Kubernetes，它能做什么？

容器是打包和运行应用程序的好方式。在生产环境中，需要管理运行应用程序的容器，并确保不会停机。例如，如果一个容器发生故障，则需要启动另一个容器。如果这种行为由系统处理会不会更容易？

这就是 Kubernetes 解决这些问题的方法！Kubernetes 提供了一个可弹性运行的分布式系统的框架。它负责应用程序的扩展和故障转移，提供部署模式等等。例如，Kubernetes 可以轻松实现金丝雀部署。

Kubernetes 将提供以下能力：

* **服务发现和负载均衡：**Kubernetes 可以使用 DNS 名称或 IP 地址对外公开容器，进入容器的流量如果很大，Kubernetes 能够负载均衡并分配网络流量，从而使部署稳定。
* **存储编排：**Kubernetes 允许自动挂载选择的存储系统，例如本地存储、公共云提供商等。
* **自动部署和回滚：**可以使用 Kubernetes 声明已部署容器的所需状态，然后以可控的方式将实际状态变更为所期望的状态。例如，可以自动化部署创建新容器，删除现有容器并将其所有资源用于新容器。
* **自动调度计算：**Kubernetes 允许指定每个容器所需的 CPU 和内存。若容器指定了资源限制，Kubernetes 可以做出更好的决策来管理集群资源。
* **自我修复：**Kubernetes 重新启动失败的容器、替换容器、杀死不响应用户定义的运行状态检查的容器，并在未准备好服务之前不允许客户端接入。
* **密钥和配置管理：**Kubernetes 允许存储和管理敏感信息，例如密码、OAuth 令牌和 ssh 密钥。可以在不重建容器镜像的情况下部署和更新密钥和应用程序配置，也无需在堆栈配置中暴露密钥。

## Kubernetes 不是什么

Kubernetes 不是传统的、包罗万象的 PasS（平台即服务）系统。由于 Kubernetes 在容器级别而不是在硬件级别运行，因此它提供了一些 PaaS 产品常用的通用功能，例如部署、扩展、负载均衡，并允许用户集成他们的日志记录、监控和告警解决方案。但是，Kubernetes 不是单一的，这些默认解决方案都是可选的和可插拔的。Kubernetes 为构建开发者平台提供了构建块，但在重要的地方保留了用户选择和灵活性。

Kubernetes：

* 不限制支持的应用程序类型。Kubernetes 旨在支持极其多种多样的工作负载，包括无状态、有状态和数据处理工作负载。如果应用程序可以在容器中运行，那么它应该可以在 Kubernetes 上很好地运行。
* 不部署源代码，也不构建应用程序。持续集成、交付和部署（CI/CD）工作取决于组织的文件和偏好以及技术要求。
* 不提供应用程序级别的服务作为内置服务，例如中间件（消息中间件）、数据处理框架（Spark）、数据库（MySQL）、缓存、集群存储系统（Ceph）。这样的组件可以在 Kubernetes 上运行，并且或者可以由运行在 Kubernetes 上的应用程序通过可移植机制（例如，开放服务代理）来访问。
* 不要求日志记录、监视或警报解决方案。它提供了一些集成作为概念证明，并提供了收集和导出指标的机制。
* 不提供或不要求配置语言/系统（例如 jsonnet），它提供了声明性 API，该声明性 API 可以由任意形式的声明性规范所构成。
* 不提供也不采用任何全面的机器配置、维护、管理或自我修复系统。
* 此外，Kubernetes 不仅仅是一个编排系统，实际上它消除了编排的需要。编排的技术定义是执行已定义的工作流程：A --> B --> C。相比之下，Kubernetes 包含一组独立的、可组合的控制过程，这些过程连续地将当前状态驱动到所提供的所需状态。如何从 A 到 C 的方式无关紧要，也不需要集中控制，这使得系统更易于使用且功能更强大、系统更健壮、更为弹性和可扩展。