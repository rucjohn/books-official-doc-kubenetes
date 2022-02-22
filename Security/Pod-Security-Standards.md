# 安全性标准

Pod 安全性标准定义了三种不同的策略（_Policy_）来广泛覆盖安全应用场景。这些策略是渐进式的（_Cumulative_），安全级别从高度宽松至高度限制。本指南概述了每个策略的要求。

| Profile | 描述 |
| ------- | ---- |
| **Privileged** | 不受限制的策略，提供最大可能范围的权限许可。此策略允许已知的特权提升。 |
| **Baseline** | 限制性最弱的策略，禁止已知的策略提升。允许使用默认的（规定最少）Pod 配置。 |
| **Restricted** | 限制性非常强的策略，遵循当前的保护 Pod 的最佳实践。 |


## Profile 细节

### Privileged

**_Privileged_ 策略是有目的地开放且完全无限制的策略。**

此类策略通常针对由特权较高、受信任的用户所管理的系统级或基础架构级负载。

Privileged 策略定义中限制较少。
- 对于默认允许（Allow-by-default）实施机制（如，gatekeeper），Privileged 策略可能意味着不应用任何约束，而不是实施某策略实例。
- 对于默认拒绝（Deny-by-default）实施机制（如，Pod 安全策略），Privileged 策略应该默认允许所有控制（即，禁止所有限制）。

### Baseline

**_Baseline_ 策略的目标是便于常见的容器化应用采用，同时禁止已知的特权提升。**

此策略针对的是应用运维人员和非关键性应用的开发人员。

应该禁止以下列出的控制：

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

在下述表格中，通配符（`*`）意味着一个列表中的所有元素。
例如 `spec.containers[*].securityContext` 表示所定义的所有容器的安全性上下文对象。如果所列出的任一容器不能满足要求，整个 Pod 将无法通过校验。
{% endhint %}

| 控制（Control） | 策略（Policy） |
| -------------- | ------------- |
| HostProcess | Windows Pod 提供了运行 HostProcess 容器的能力，这使得对 Windows 节点的特权访问成为可能。Baseline 策略中对宿主的特权访问是被禁止的。HostProcess Pod 基于 Kubernetes v1.22 [alpha]。<br>**限制的字段**<br>- `spec.securityContext.windowsOptions.hostProcess` <br>- `spec.containers[*].securityContext.windowsOptions.hostProcess` <br>- `spec.initContainers[*].securityContext.windowsOptions.hostProcess` <br>- `spec.ephemeralContainers[*].securityContext.windowsOptions.hostProcess` <br>**允许的值**<br>- 未定义 / nil<br>- false |
| 宿主命名空间 | 必须禁止共享宿主命名空间。<br>**限制的字段**<br>- `spec.hostNetwork` <br>- `spec.hostPID` <br>- `spec.hostIPC` <br>**允许的值**<br>- 未定义 / nil<br>- false |
| 特权容器 | 特权 Pod 关闭了大多数安全机制，必须禁止。<br>**限制的字段**<br>- `spec.containers[*].securityContext.privileged` <br>- `spec.initContainers[*].securityContext.privileged` <br>- `spec.ephemeralContainers[*].securityContext.privileged` <br>**允许的值**<br>- 未定义 / nil<br>- false |
| Capabilities | 必须禁止添加除下列字段之外的能力。<br>**限制的字段**<br>- `spec.containers[*].securityContext.capabilities.add` <br>- `spec.initContainers[*].securityContext.capabilities.add` <br>- `spec.ephemeralContainers[*].securityContext.capabilities.add` <br>**允许的值**<br>- 未定义 / nil<br>- AUDIT_WRITE<br>- CHOWN<br>- DAC_OVERRIDE<br>- FOWNER<br>- FSETID<br>- KILL<br>- MKNOD<br>- NET_BIND_SERVICE<br>- SETFACP<br>- SETGID<br>- SETPCAP<br>- SETUID<br>- SYS_CHROOT |
| HostPath 卷 | 必须禁止 HostPath 卷。<br>**限制的字段**<br>- `spec.volumes[*].hostPath` <br>**允许的值**<br>- 未定义 / nil |
| 宿主端口 | 应该禁止使用宿主端口，或者至少限制在一个已知的列表中。<br>**限制的字段**<br>- `spec.containers[*].ports[*].hostPort` <br>- `spec.initContainers[*].ports[*].hostPort` <br>- `spec.ephemeralContainers[*].ports[*].hostPort` <br>**允许的值**<br>- 未定义 / nil<br>- 已知列表<br>- 0 |
| AppArmor | 在受支持的主机上，默认使用 runtime/default AppArmor Profile。 Baseline 策略应避免覆盖或者禁用默认策略，以及限制覆盖一些 Profile 集合的权限。<br>**限制的字段**<br>- `metadata.annotations["container.apparmor.security.beta.kubernetes.io/*"]<br>**允许的值**<br>- 未定义 / nil<br>- runtime/default<br>- localhost/* |



### Restricted

**_Restricted_ 策略旨在实施当前保护 Pod 的最佳实践，尽管这样作可能会牺牲一些兼容性。**
该类策略主要针对运维人员和安全性很重要的应用的开发人员，以及不太被信任的用户。
下面列举的控制需要被实施（禁止）：


在下述表格中，通配符（`*`）意味着一个列表中的所有元素。
例如 `spec.containers[*].securityContext` 表示 _所定义的所有容器_ 的安全性上下文对象。
如果所列出的任一容器不能满足要求，整个 Pod 将无法通过校验。



## 策略实例化   {#policy-instantiation}

将策略定义从策略实例中解耦出来有助于形成跨集群的策略理解和语言陈述，
以免绑定到特定的下层实施机制。

随着相关机制的成熟，这些机制会按策略分别定义在下面。特定策略的实施方法不在这里定义。

[**Pod 安全性准入控制器**](/zh/docs/concepts/security/pod-security-admission/)

- {{< example file="security/podsecurity-privileged.yaml" >}}Privileged namespace{{< /example >}}
- {{< example file="security/podsecurity-baseline.yaml" >}}Baseline namespace{{< /example >}}
- {{< example file="security/podsecurity-restricted.yaml" >}}Restricted namespace{{< /example >}}

[**PodSecurityPolicy**](/zh/docs/concepts/policy/pod-security-policy/)

- {{< example file="policy/privileged-psp.yaml" >}}Privileged{{< /example >}}
- {{< example file="policy/baseline-psp.yaml" >}}Baseline{{< /example >}}
- {{< example file="policy/restricted-psp.yaml" >}}Restricted{{< /example >}}


## 常见问题    {#faq}

### 为什么不存在介于 Privileged 和 Baseline 之间的策略类型

这里定义的三种策略框架有一个明晰的线性递进关系，从最安全（Restricted）到最不安全，
并且覆盖了很大范围的工作负载。特权要求超出 Baseline 策略者通常是特定于应用的需求，
所以我们没有在这个范围内提供标准框架。
这并不意味着在这样的情形下仍然只能使用 Privileged 框架，只是说处于这个范围的
策略需要因地制宜地定义。

SIG Auth 可能会在将来考虑这个范围的框架，前提是有对其他框架的需求。


### 安全策略与安全上下文的区别是什么？

[安全上下文](/zh/docs/tasks/configure-pod-container/security-context/)在运行时配置 Pod
和容器。安全上下文是在 Pod 清单中作为 Pod 和容器规约的一部分来定义的，所代表的是
传递给容器运行时的参数。


安全策略则是控制面用来对安全上下文以及安全性上下文之外的参数实施某种设置的机制。
在 2020 年 7 月，
[Pod 安全性策略](/zh/docs/concepts/policy/pod-security-policy/)已被废弃，
取而代之的是内置的 [Pod 安全性准入控制器](/zh/docs/concepts/security/pod-security-admission/)。

Kubernetes 生态系统中还在开发一些其他的替代方案，例如
- [OPA Gatekeeper](https://github.com/open-policy-agent/gatekeeper)。
- [Kubewarden](https://github.com/kubewarden)。
- [Kyverno](https://kyverno.io/policies/pod-security/)。


### 我应该为我的 Windows Pod 实施哪种框架？

Kubernetes 中的 Windows 负载与标准的基于 Linux 的负载相比有一些局限性和区别。
尤其是 Pod SecurityContext 字段
[对 Windows 不起作用](/zh/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#v1-podsecuritycontext)。
因此，目前没有对应的标准 Pod 安全性框架。


如果你为一个 Windows Pod 应用了 Restricted 策略，**可能会** 对该 Pod 的运行时产生影响。
Restricted 策略需要强制执行 Linux 特有的限制（如 seccomp Profile，并且禁止特权提升）。
如果 kubelet 和/或其容器运行时忽略了 Linux 特有的值，那么应该不影响 Windows Pod 正常工作。
然而，对于使用 Windows 容器的 Pod 来说，缺乏强制执行意味着相比于 Restricted 策略，没有任何额外的限制。

你应该只在 Privileged 策略下使用 HostProcess 标志来创建 HostProcess Pod。
在 Baseline 和 Restricted 策略下，创建 Windows HostProcess Pod 是被禁止的，
因此任何 HostProcess Pod 都应该被认为是有特权的。


### 沙箱（Sandboxed） Pod 怎么处理？

现在还没有 API 标准来控制 Pod 是否被视作沙箱化 Pod。
沙箱 Pod 可以通过其是否使用沙箱化运行时（如 gVisor 或 Kata Container）来辨别，不过
目前还没有关于什么是沙箱化运行时的标准定义。

沙箱化负载所需要的保护可能彼此各不相同。例如，当负载与下层内核直接隔离开来时，
限制特权化操作的许可就不那么重要。这使得那些需要更多许可权限的负载仍能被有效隔离。

此外，沙箱化负载的保护高度依赖于沙箱化的实现方法。
因此，现在还没有针对所有沙箱化负载的建议策略。
