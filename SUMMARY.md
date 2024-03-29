# Table of contents

* [前言](README.md)
* [词汇表](Glossary.md)
* [概念](Concepts/README.md)
  * [概述](Concepts/Overview/README.md)
    * [组件](Concepts/Overview/Kubernetes-Components.md)
    * [对象](Concepts/Overview/Working-with-Kubernetes-Objects/README.md)
      * [理解 Kubernetes 对象](Concepts/Overview/Working-with-Kubernetes-Objects/Understanding-Kubernetes-Objects.md)
      * [管理 Kubernetes 对象](Concepts/Overview/Working-with-Kubernetes-Objects/Kubernetes-Object-Management.md)
      * [对象名称和 ID](Concepts/Overview/Working-with-Kubernetes-Objects/Object-Names-and-IDs.md)
      * [标签（Label）和选择算符（Selector）](Concepts/Overview/Working-with-Kubernetes-Objects/Labels-and-Selectors.md)
      * [命名空间（Namespace）](Concepts/Overview/Working-with-Kubernetes-Objects/Namespaces.md)
      * [注解（Annotation）](Concepts/Overview/Working-with-Kubernetes-Objects/Annotations.md)
      * [字段选择器](Concepts/Overview/Working-with-Kubernetes-Objects/Field-Selectors.md)
      * [Finalizers](Concepts/Overview/Working-with-Kubernetes-Objects/Finalizers.md)
      * [属主与附属](Concepts/Overview/Working-with-Kubernetes-Objects/Owners-and-Dependents.md)
  * [架构](Concepts/Cluster-Architecture/README.md)
    * [节点](Concepts/Cluster-Architecture/Nodes.md)
  * [工作负载](Concepts/Workloads/README.md)
    * [工作负载资源](Concepts/Workloads/Workload-Resources/README.md)
      * [Job](Concepts/Workloads/Workload-Resources/Jobs/README.md)
  * [服务、负载均衡和网络](Concepts/Services-Load-Balancing-and-Networking/README.md)
    * [服务](Concepts/Services-Load-Balancing-and-Networking/Service/README.md)
      * [服务定义](Concepts/Services-Load-Balancing-and-Networking/Service/Service-Definition.md)
      * [无头服务(Headless Service)](Concepts/Services-Load-Balancing-and-Networking/Service/Headless-Services.md)
      * [发布服务](Concepts/Services-Load-Balancing-and-Networking/Service/Publishing-Services.md)
  * [配置](Concepts/Configuration/README.md)
    * [配置最佳实践](Concepts/Configuration/Configuration-Best-Practices.md)
  * [策略](Concepts/Policies/README.md)
    * [资源配额](Concepts/Policies/Resource-Quotas.md)
  * [调度、抢占和驱逐](Concepts/Scheduling-Preemption-and-Eviction/README.md)
    * [Kubernetes 调度器](Concepts/Scheduling-Preemption-and-Eviction/Kubernetes-Scheduler.md)
  * [集群管理](Concepts/Cluster-Administration/README.md)
    * [证书](Concepts/Cluster-Administration/Certificates.md)
    * [管理资源](Concepts/Cluster-Administration/Managing-Resources/README.md)
      * [组织资源配置](Concepts/Cluster-Administration/Managing-Resources/Organizing-resource-configurations.md)
      * [kubectl 中的批量操作](Concepts/Cluster-Administration/Managing-Resources/Bulk-operations-in-kubectl.md)
      * [有效地使用标签](Concepts/Cluster-Administration/Managing-Resources/Using-labels-effectively.md)
      * [金丝雀部署](Concepts/Cluster-Administration/Managing-Resources/Canary-deployments.md)
      * [更新标签](Concepts/Cluster-Administration/Managing-Resources/Updating-labels.md)
    * [集群网络系统](Concepts/Cluster-Administration/Cluster-Networking.md)
* [任务](Tasks/README.md)
  * [管理集群](Tasks/Administer-a-Cluster/README.md)
    * [手动生成证书](Tasks/Administer-a-Cluster/Generate-Certificates-Manually.md)
  * [运行应用](Tasks/Run-Applications/README.md)
    * [Pod 水平自动扩缩容](Tasks/Run-Applications/Horizontal-Pod-Autoscaling/README.md)
      * [HPA 是如何工作的？](Tasks/Run-Applications/Horizontal-Pod-Autoscaling/How-does-a-HorizontalPodAutoscaler-work.md)
    * [HPA 演练](Tasks/Run-Applications/HorizontalPodAutoscaler-Walkthrouth/README.md)
      * [创建 HPA](Tasks/Run-Applications/HorizontalPodAutoscaler-Walkthrouth/Create-the-HorizontalPodAutoscaler.md)
      * [附录](Tasks/Run-Applications/HorizontalPodAutoscaler-Walkthrouth/Appendix.md)
  * [TLS](Tasks/TLS/README.md)
    * [管理集群中的 TLS 认证](Tasks/TLS/Manage-TLS-Certificates-in-a-Cluster.md)
* [参考](Reference/README.md)
  * [API 访问控制](Reference/API-Access-Control/README.md)
    * [使用 RBAC 鉴权](Reference/API-Access-Control/Using-RBAC-Authorization/README.md)
      * [API 对象](Reference/API-Access-Control/Using-RBAC-Authorization/API-objects.md)
      * [默认 Roles 和 Role Bindings](Reference/API-Access-Control/Using-RBAC-Authorization/Default-roles-and-role-bindings.md)
      * [初始化与预防权限提升](Reference/API-Access-Control/Using-RBAC-Authorization/Privilege-escalation-prevention-and-bootstrapping.md)
      * [命令行工具](Reference/API-Access-Control/Using-RBAC-Authorization/Command-line-utilities.md)
      * [ServiceAccount 权限](Reference/API-Access-Control/Using-RBAC-Authorization/ServiceAccount-permissions.md)
      * [EndpointSlices 和 Endpoints 写权限](Reference/API-Access-Control/Using-RBAC-Authorization/Write-access-for-EndpointSlices-and-Endpoints.md)
      * [从 ABAC 升级](Reference/API-Access-Control/Using-RBAC-Authorization/Upgrading-from-ABAC.md)
  * [组件工具](Reference/Component-tools/README.md)
    * [kube-scheduler](Reference/Component-tools/kube-scheduler.md)
  * [调度](Reference/Scheduling/README.md)
    * [调度器配置](Reference/Scheduling/Scheduler-Configuration.md)
    * [调度策略](Reference/Scheduling/Scheduling-Policies.md)
  * [网络参数](Reference/Networking-Reference/README.md)
    * [Service 所用的协议](Reference/Networking-Reference/Protocols-for-Services.md)
    * [虚拟 IP 和服务代理](Reference/Networking-Reference/Virtual-IPs-and-Service-Proxies.md)
  * [命令行工具 (kubectl)](Reference/Command-line-tool-kubectl/README.md)
* [博客](Blog/README.md)
  * [移除 Dockershim 的常见问题](Blog/Dockershim-Removal-FAQ.md)
