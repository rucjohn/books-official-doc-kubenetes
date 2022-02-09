# 镜像

容器镜像（**Image**）所承载的是封装了应用程序及其所有软件依赖的二进制数据。容器镜像是可执行的软件包，可以单独运行，并对其运行时环境做出非常明确的假设。

通常情况下，构建应用的容器镜像并将其推送到镜像仓库（**Registry**），然后在 Pod 中引用它。

## 镜像名称

容器镜像通常会被赋予 `pause`、`example/mycontainer` 或 `kube-apiserver` 这类的名称。

* 镜像名称可以包含所在仓库的主机名。例如：`fictional.registry.example/imagename`。
* 镜像名称可以包含所在仓库的端口号，例如：`fictional.registry.example:10443/imagename`。

如果不指定仓库的主机名，Kubernetes 默认使用 Docker 公共仓库。

在镜像名称之后，可以添加一个 **Tag**，使用标签能辩识同一镜像的不同版本。

镜像 Tag 可以包含小写字母、大写字母、数字、下划线`_`、句号`.` 和连字符`-`。关于在镜像标签中休息可以使用分隔字符（`_`、`-`和`.`）还有一些额外的规则。如果不指定 Tag，Kubernetes 默认使用 `latest`。

## 更新镜像

当最初创建一个 Deployment、StatefulSet、Pod 或者其他包含 Pod 模板的对象时，如果没有显式设定的话，Pod 中所有容器默认镜像拉取策略为 `IfNotPresent`。这一策略会使得 kubelet 在镜像已经存在的情况下直接略过拉取镜像的操作。

### 镜像拉取策略

容器的 `imagePullPolicy` 和镜像的 Tag 会影响 kubelet 尝试拉取/下载指定的镜像。

* <mark style="color:orange;">IfNotPresent</mark>：只有当镜像在本地不存在时才会拉取。
* <mark style="color:orange;">Always</mark>：每当 kubelet 启动一个容器时，kubelet 会查询容器的镜像仓库，将名称解析为一个镜像摘要。如果 kubelet 有一个容器镜像，并且对应的摘要已在本地缓存，kubelet 就会使用其缓存的镜像；否则，kubelet 就会使用解析后的摘要拉取镜像，并使用该镜像来启动容器。
* <mark style="color:orange;">Never</mark>：kubelet 不会尝试获取镜像。如果镜像已经以某种方式存在本地，kubelet 会尝试启动容器；否则，会启动失败。

只要能够可靠地访问镜像仓库，`imagePullPolicy: Always` 也可以很高效。容器运行时可以注意到节点上已经存的镜像层，这样就不需要再次下载。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

在生产环境中部署容器时，应该避免使用 `:latest` Tag，因这这使得正在运行的镜像的版本难以追踪，并且难以正确地回滚。

相反，应指定一个有意义的 Tag，如 `v1.42.0`。
{% endhint %}

为了确保 Pod 总是使用相同版本的容器镜像，可以指定镜像的摘要；将 `<image-name>:<tag>` 替换为 `<image-name>@<digest>`，例如`image@sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2`。

当使用镜像 Tag 时，如果镜像仓库修改了代码所对应的镜像 Tag，可能会出现新旧代码混杂在 Pod 中运行的情况。镜像摘要唯一标识了镜像的特定版本，因此 Kubernetes 每次启动了指定镜像名称和摘要的容器时，都会运行相同的镜像版本。指定一个镜像可以固定所运行的代码，这样镜像仓库的变化就会导致版本的混乱。

有一些第三方的 [准入控制器](../API/API-Access-Control/Using-Admission-Controllers.md) 在创建 Pod（和 Pod 模板）时产生变更，这样运行的工作负载就是根据镜像摘要，而不是 Tag 来定义的。无论镜像仓库上的 Tag 发生什么变化，都能确保所有的工作负载运行相同的代码，所以指定镜像摘要很有用。

#### 默认镜像拉取策略

当向 apiserver 提交一个新的 Pod 时，集群会在满足特定条件时设置 `imagePullPolicy` 字段：

* 如果省略了 `imagePullPolicy` 字段，并且容器镜像的 Tag 是 `:latest`，自动设置 `imagePullPolicy: Always`。
* 如果省略了 `imagePullPolicy` 字段，没有指定容器镜像的 Tag，自动设置 `imagePullPolicy: Always`。
* 如果省略了 `imagePullPolicy` 字段，并且为容器镜像指定了非 `:latest` Tag，自动设置 `imagePullPolicy: IfNotPresent`。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

容器的 `imagePullPolicy` 的值总是在对象初次创建时设置的，如果后来镜像的 Tag 发生变化，则不会更新。

例如，如果使用一个非 `:latest` 的镜像 Tag 创建一个 Deployment，并在随后更新该 Deployment 的镜像 Tag 为 `:latest`，则 `imagePullPolicy` 字段不会变成 `Always`。必须手动更改已经创建的资源的拉取策略。
{% endhint %}

#### 必要的镜像拉取

如果想总是强制执行拉取，可以使用下述其中一种方式：

* 设置 `imagePullPolicy: Always`。
* 省略 `imagePullPolicy` 字段，并使用 `:latest` Tag；当提交 Pod 时，Kubernetes 会将策略设置为 `Always`。
* 省略 `imagePullPolicy` 字段和镜像的 Tag；当提交 Pod 时，Kubernetes 会将策略设置为 `Always`。
* 启用 准入控制器 [AlwaysPullImages](../API/API-Access-Control/Using-Admission-Controllers.md#alwayspullimages)。

### ImagePullBackOff

当 kubelet 使用容器运行时创建 Pod 时，容器可能因为 `ImagePullBackOff` 导致状态为 [Waiting](../Workloads/Pods/Pod-Lifecycle.md#waiting)。

`ImagePullBackOff` 状态意味着容器无法启动，因为 Kubernetes 无法拉取镜像，原因包括如下：

* 无效的镜像名称
* 从私有仓库拉取而没有 `imagePullSecret`

`BackOff` 部分表示 Kubernetes 将继续尝试拉取镜像，并增加回退延迟。

Kubernetes 会增加每次尝试之间的延迟，直到达到编译限制，即 **300 秒（5 分钟）**。



## 具有镜像索引的多架构镜像

除了提供二进制的镜像之外，容器仓库也可以提供[容器镜像索引](https://github.com/opencontainers/image-spec/blob/main/image-index.md)。镜像索引 可以根据特定体系结构版本的容器指向镜像的多个[镜像清单](https://github.com/opencontainers/image-spec/blob/main/manifest.md)。这背后的理念是让可以为镜像命名（例如：`pause`、`example/mycontainer`、`kube-apiserver`）的同时，允许不同的系统基于它们所使用的机器体系结构取回正确的二进制镜像。

Kubernetes 自身通常在命名容器镜像时添加后 `-$(ARCH)`。为了向前兼容，请在生成较老的镜像时也提供后缀。这里的理念是为某镜像生成针对所有平台都适用的清单时，生成例如`pause-amd64`这类镜像，以便较老的配置文件或者将镜像后缀编码到其中的 YAML 文件也能兼容。

## 使用私有仓库

从私胡仓库读取镜像时可能需要密钥。凭证可以用以下方式提供：
- 配置节点向私有仓库进行身份验证
    - 所有 Pod 均可读取任何可配置的私有仓库
    - 需要集群管理员配置节点
- 预拉镜像
    - 所有 Pod 都可以使用节点上缓存的所有镜像
    - 需要所有节点的 root 访问权限才能进行设置
- 在 Pod 中设置 ImagePullSecrets
    - 只有提供自己密钥的 Pod 才能访问私有仓库
- 特定厂商发行版的扩展或者本地扩展
    - 如果在使用定制的节点配置，或者云平台提供提供商可以实现让节点向容器我不愿离开认证的机制

### 配置 Node 对私有仓库认证

如果在节点上运行的是 Docker，可以配置 Docker 容器运行时向私胡容器仓库认证身份。

此方法适用于能够对节点进行配置的场合。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

Kubernetes 默认仅支持 Docker 配置中的 auths 和 HttpHeaders 部分，不支持 Docker 凭据辅助程序（credHelpers 或 credsStore）。
{% endhint %}

Docker 将私有仓库的密钥保存在 `$HOME/.dockercfg` 或 `$HOME/.docker/config.json` 文件中。如果将相同的文件放在下面所列的搜索路径中，`kubelet` 会在拉取镜像时将其用作凭据数据来源：
- `{--root-dir:/var/lib/kubelet}/config.json`
- `{--root-dir:/var/lib/kubelet}/.dockercfg`
- `{kubelet 当前工具目录}/config.json`
- `{kubelet 当前工具目录}/.dockercfg`
- `${HOME}/.docker/config.json`
- `${HOME}/.cockercfg`
- `/.docker/config.json`
- `/.dockercfg`

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

可能不得不为 kubelet 进程显式地设置 `HOME=root` 环境变量。
{% endhint %}

推荐采用如下步骤来配置节点以便访问私有仓库。
1. 针对 要使用的每组凭据，运行 `docker login [服务器]` 命令。这会更新本地环境中的 `$HOME/.docker/config.json` 文件。
2. 在编辑器中打开查看 `$HOME/.docker/config.json` 文件，确保其中仅包含想要使用的凭据信息。
3. 获取节点列表，例如：
    - 节点名称：`nodes=$(kubectl get nodes -o jsonpath='{range.items[*].metadata}{.name} {end}')`
    - 节点IP：`nodes=$(kubectl get nodes -o jsonpath='{range .items[*].status.addresses[?(@.type=="InternalIP")]}{.address} {end}')`
4. 将本地的 `.docker/config.json` 拷贝到所有节点，放入如上所列的目录之一。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

对于产品环境的集群，可以使用配置管理工具来将这些设置应用到所期望的节点上。
{% endhint %}

创建使用私有镜像的 Pod 来验证。例如：
```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: private-image-test-1
  labels:
    app: private-image-test
spec:
  containers:
    - name: uses-private-image
      image: $PRIVATE_IMAGE_NAME
      imagePullPolicy: Always
      command: [ "echo", "SUCCESS" ]
EOF
```

通过以下命令，检查镜像是否下载成功：
```bash
kubectl describe pods -l "app=private-image-test"
```

必须确保集群中所有节点的 `.docker/config.json` 文件内容相同。否则，可能出现 Pod 在有的节点正常，有的节点无法启动的问题。例如，如果使用节点自动扩缩容，那么每个实例模板都需要包含 `.docker/config.json`，或者挂载一个包含该文件的驱动器。

在 `.docker/config.json` 中配置了私有仓库密钥后，所有 Pod 都能读取私有仓库的镜像。

### config.json 说明

对于 `config.json` 的解释在原始 Docker 实现和 Kubernetes 的解释之间有所不同。
- 在 Docker 中，`auths` 键只能指定根 URL
- 在 Kubernetes 中，`auths` 键允许 glob URLs 以及前缀匹配的路径。这意味着，像下列的 `config.json` 是有效的：
```json
{
    "auths": {
        "*my-registry.io/images": {
            "auth": "…"
        }
    }
}
```

例如以下语法匹配根据 URL（`*my-registry.io`）：
```
pattern:
    { term }

term:
    '*'         匹配任何无分隔符字符序列
    '?'         匹配任意单个非分隔符
    '[' [ '^' ] 字符范围
                  字符集（必须非空）
    c           匹配字符 c （c 不为 '*','?','\\','['）
    '\\' c      匹配字符 c

字符范围: 
    c           匹配字符 c （c 不为 '\\','?','-',']'）
    '\\' c      匹配字符 c
    lo '-' hi   匹配字符范围在 lo 到 hi 之间字符
```

现在镜像拉取操作会将每种有效模板的凭据都传统给 CRI 容器运行时。例如下面的容器镜像名称会匹配成功：
- `my-registry.io/images`
- `my-registry.io/images/my-image`
- `my-registry.io/images/another-image`
- `sub.my-registry.io/images/my-image`
- `a.sub.my-registry.io/images/my-image`

kubelet 为每个找到的凭证的镜像按顺序拉取。这意味着在 `config.json` 中可能有多项：
```json
{
    "auths": {
        "my-registry.io/images": {
            "auth": "…"
        },
        "my-registry.io/images/subpath": {
            "auth": "…"
        }
    }
}
```

如果一个容器指定了要拉取的镜像 `my-registry.io/images/subpath/my-image`，并且其中一个失败，kubelet 将尝试从另一个身份验证源下载镜像。

### 提前拉取镜像

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

该方法适用能够控制节点配置的场合。如果是云供应商负责管理节点并自动配置节点，这一方案无法可靠地工作。
{% endhint %}

默认情况下，`kubelet` 会尝试从指定的仓库拉取每个镜像。但是，如果容器属性 `imagePullPolicy` 设置为 `IfNotPresent` 或者 `Never`，则会优先使用（对应 `IfNotPresent`）或者一定使用（对应 `Never`）本地镜像。

如果希望使用提前拉取镜像的方法代替仓库认证，就必须保证集群中所有节点提前拉取的镜像是相同的。

这一方案可以用来提前载入看电视来的镜像以提高速度，或者作为向私胡仓库执行身份认证的一种替代方案。

所有的 Pod 都可以在使用节点上提前拉取的镜像。

### 在 Pod 上指定 ImagePullSecrets

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

运行使用私有仓库中镜像的容器时，推荐使用这种方法。
{% endhint %}

Kubernetes 支持在 Pod 中设置容器镜像仓库的密钥。

#### 使用 Docker Config 创建 Secret

运行以下命令：
```bash
 kubectl create secret docker-registry <名称> \
     --docker-server=<DOCKER_REGISTRY_SERVER> \
     --docker-username=<DOCKER_USER> \
     --docker-password=<DOCKER_PASSWORD> \
     --docker-email=<DOCKER_EMAIL>
```

如果已经有 Docker 凭据文件，则可以将凭据文件导入到 Kubernetes Secret ，而不是执行上面的命令。具体操作可以参阅 [从私有仓库拉取镜像任务]()。


当在使用多个私有容器仓库时，这种方法就特别有用。原因是 `kubectl create secret docker-registry` 创建是仅适用于某个私有仓库的 Secret。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

Pod 只能引用位于自身所在命名空间的 Secret，因此 需要针对 每个命名空间重复执行上述过程。
{% endhint %}


#### 在 Pod 中引用 ImagePullSecrets

现在，在创建 Pod 时，可以在 Pod 定义中增加 `imagePullSecrets` 部分来引用 Secret。

例如：
```bash
cat <<EOF > pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo
  namespace: awesomeapps
spec:
  containers:
    - name: foo
      image: janedoe/awesomeapp:v1
  imagePullSecrets:
    - name: myregistrykey
EOF

cat <<EOF >> ./kustomization.yaml
resources:
- pod.yaml
EOF
```

需要对使用私有仓库的每个 Pod 执行以上操作。不过设置该字段 的过程也可以通过服务账号资源设置 `imagePullSecrets` 来自动完成。有关详细指令可参见 [将 ImagePullSecrets 添加到服务账户任务]

也可以将此方法与节点组别的 `.docker/config.json` 配置结合使用。来自不同来源的凭据会被合并。

## 案例

配置私有仓库有多种方案，以下是一些常用场景和建议的解决方案。
1. 集群运行非专有镜像（例如，开源镜像）。镜像不需要隐藏。
    - 使用 Docker Hub 上的公共镜像
        - 无需配置
        - 某些云厂商会自动为公共镜像提供高速缓存，以便提升可用性并缩短拉取镜像所需时间
2. 集群运行一些专有镜像，这些镜像需要对外隐藏，对所有集群用户可见。
    - 使用托管的私有 Docker 仓库
        - 可以托管在Docker Hub 或者其他地址
        - 按照上面的描述，在每个节点上让他发错了配置 `.docker/config.json` 文件
    - 或者，在防火墙内运行一个组织内部的私有仓库，并开放读取权限
        - 不需要配置 Kubernetes
    - 使用控制镜像访问的托管容器镜像仓库服务
        - 与手动配置节点相比 ，这种方案能更好地处理集群自动扩缩容
    - 或者，在不方便更改节点配置的集群中，使用 `imagePullSecrets`
3. 集群使用专有镜像，且有些镜像需要更严格的访问控制
    - 确保 AlwaysPullImages 准入控制器被启动。否则，所有 Pod 都可以使用所有镜像。
    - 确保将第三数据存储在 Secret 资源中，而不是将其打包在镜像里
4. 集群是多租户的，并且每个租户需要自己的私有仓库
    - 确保 AlwaysPullImages 准入控制器被启动。否则，所有 Pod 都可以使用所有镜像。
    - 为私有仓库启用鉴权
    - 为每个租户生成访问仓库的凭据，放置在 Secret 中，并将 Secret 发布到各租户的命名空间下。
    - 租户将 Secret 添加到每个命名空间中的 imagePullSecrets

如果需要访问多个仓库，可以为每个仓库创建一个 Secret。kubelet 将所有 `imagePullSecrets` 合并为一个虚拟的 `.docker/config.json` 文件。



