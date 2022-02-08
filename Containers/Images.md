# 镜像

容器镜像（Image）所承载的是封装了应用程序及其所有软件依赖的二进制数据。容器镜像是可执行的软件包，可以单独运行，并对其运行时环境做出非常明确的假设。

通常情况下，构建应用的容器镜像并将其推送到镜像仓库（Registry），然后在 Pod 中引用它。

## 镜像名称

容器镜像通常会被赋予 `pause`、`example/mycontainer` 或 `kube-apiserver` 这类的名称。
- 镜像名称可以包含所在仓库的主机名。例如：`fictional.registry.example/imagename`。
- 镜像名称可以包含所在仓库的端口号，例如：`fictional.registry.example:10443/imagename`。

如果不指定仓库的主机名，Kubernetes 默认使用 Docker 公共仓库。

在镜像名称之后，可以添加一个 **Tag**，使用标签能辩识同一镜像的不同版本。

镜像 Tag 可以包含小写字母、大写字母、数字、下划线`_`、句号`.` 和连字符`-`。关于在镜像标签中休息可以使用分隔字符（`_`、`-`和`.`）还有一些额外的规则。如果不指定 Tag，Kubernetes 默认使用 `latest`。

## 更新镜像

当最初创建一个 Deployment、StatefulSet、Pod 或者其他包含 Pod 模板的对象时，如果没有显式设定的话，Pod 中所有容器默认镜像拉取策略为 `IfNotPresent`。这一策略会使得 kubelet 在镜像已经存在的情况下直接略过拉取镜像的操作。

### 镜像拉取策略

容器的 `imagePullPolicy` 和镜像的 Tag 会影响 kubelet 尝试拉取/下载指定的镜像。

- IfNotPresent：只有当镜像在本地不存在时才会拉取。
- Always：每当 kubelet 启动一个容器时，kubelet 会查询容器的镜像仓库，将名称解析为一个镜像摘要。如果 kubelet 有一个容器镜像，并且对应的摘要已在本地缓存，kubelet 就会使用其缓存的镜像；否则，kubelet 就会使用解析后的摘要拉取镜像，并使用该镜像来启动容器。
- Never：kubelet 不会尝试获取镜像。如果镜像已经以某种方式存在本地，kubelet 会尝试启动容器；否则，会启动失败。

只要能够可靠地访问镜像仓库，`imagePullPolicy: Always` 也可以很高效。容器运行时可以注意到节点上已经存的镜像层，这样就不需要再次下载。

说明：

在生产环境中部署容器时，应该避免使用 `:latest` Tag，因这这使得正在运行的镜像的版本难以追踪，并且难以正确地回滚。

相反，应指定一个有意义的 Tag，如 `v1.42.0`。

为了确保 Pod 总是使用相同版本的容器镜像，可以指定镜像的摘要；将 `<image-name>:<tag>` 替换为 `<image-name>@<digest>，例如：`image@sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2`。

当使用镜像 Tag 时，如果镜像仓库修改了代码所对应的镜像 Tag，可能会出现新旧代码混杂在 Pod 中运行的情况。镜像摘要唯一标识了镜像的特定版本，因此 Kubernetes 每次启动了指定镜像名称和摘要的容器时，都会运行相同的镜像版本。指定一个镜像可以固定所运行的代码，这样镜像仓库的变化就会导致版本的混乱。

有一些第三方的 [准入控制器]() 在创建 Pod（和 Pod 模板）时产生变更，这样运行的工作负载就是根据镜像摘要，而不是 Tag 来定义的。无论镜像仓库上的 Tag 发生什么变化，都能确保所有的工作负载运行相同的代码，所以指定镜像摘要很有用。

#### 默认镜像拉取策略

当向 apiserver 提交一个新的 Pod 时，集群会在满足特定条件时设置 `imagePullPolicy` 字段：
- 如果省略了 `imagePullPolicy` 字段，并且容器镜像的 Tag 是 `:latest`，自动设置 `imagePullPolicy: Always`。
- 如果省略了 `imagePullPolicy` 字段，没有指定容器镜像的 Tag，自动设置 `imagePullPolicy: Always`。
- 如果省略了 `imagePullPolicy` 字段，并且为容器镜像指定了非 `:latest` Tag，自动设置 `imagePullPolicy: IfNotPresent`。

说明：

容器的 `imagePullPolicy` 的值总是在对象初次创建时设置的，如果后来镜像的 Tag 发生变化，则不会更新。

例如，如果使用一个非 `:latest` 的镜像 Tag 创建一个 Deployment，并在随后更新该 Deployment 的镜像 Tag 为 `:latest`，则 `imagePullPolicy` 字段不会变成 `Always`。必须手动更改已经创建的资源的拉取策略。

#### 必要的镜像拉取

如果想总是强制执行拉取，可以使用下述其中一种方式：
- 设置 `imagePullPolicy: Always`。
- 省略 `imagePullPolicy` 字段，并使用 `:latest` Tag；当提交 Pod 时，Kubernetes 会将策略设置为 `Always`。
- 省略 `imagePullPolicy` 字段和镜像的 Tag；当提交 Pod 时，Kubernetes 会将策略设置为 `Always`。
- 启用 准入控制器 AlwaysPullImages。

### ImagePullBackOff

当 kubelet 使用容器运行时创建 Pod 时，容器可能因为 `ImagePullBackOff` 导致状态为 Waiting。

`ImagePullBackOff` 状态意味着容器无法启动，因为 Kubernetes 无法拉取镜像，原因包括如下：
- 无效的镜像名称
- 从私有仓库拉取而没有 `imagePullSecret`

`BackOff` 部分表示 Kubernetes 将继续尝试拉取镜像，并增加回退延迟。

Kubernetes 会增加每次尝试之间的延迟，直到达到编译限制，即 **300 秒（5 分钟）**。




