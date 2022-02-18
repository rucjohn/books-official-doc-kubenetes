# StatefulSets

***

## title: StatefulSets content\_type: concept weight: 30

StatefulSet 是用来管理有状态应用的工作负载 API 对象。

\{{< glossary\_definition term\_id="statefulset" length="all" >\}}

## 使用 StatefulSets

StatefulSets 对于需要满足以下一个或多个需求的应用程序很有价值：

* 稳定的、唯一的网络标识符。
* 稳定的、持久的存储。
* 有序的、优雅的部署和缩放。
* 有序的、自动的滚动更新。

在上面描述中，“稳定的”意味着 Pod 调度或重调度的整个过程是有持久性的。 如果应用程序不需要任何稳定的标识符或有序的部署、删除或伸缩，则应该使用 由一组无状态的副本控制器提供的工作负载来部署应用程序，比如 [Deployment](../zh/docs/concepts/workloads/controllers/deployment/) 或者 [ReplicaSet](../zh/docs/concepts/workloads/controllers/replicaset/) 可能更适用于你的无状态应用部署需要。

## 限制 <a href="#limitations" id="limitations"></a>

* 给定 Pod 的存储必须由 [PersistentVolume 驱动](https://github.com/kubernetes/examples/tree/master/staging/persistent-volume-provisioning/README.md) 基于所请求的 `storage class` 来提供，或者由管理员预先提供。
* 删除或者收缩 StatefulSet 并_不会_删除它关联的存储卷。 这样做是为了保证数据安全，它通常比自动清除 StatefulSet 所有相关的资源更有价值。
* StatefulSet 当前需要[无头服务](../zh/docs/concepts/services-networking/service/#headless-services) 来负责 Pod 的网络标识。你需要负责创建此服务。
* 当删除 StatefulSets 时，StatefulSet 不提供任何终止 Pod 的保证。 为了实现 StatefulSet 中的 Pod 可以有序地且体面地终止，可以在删除之前将 StatefulSet 缩放为 0。
* 在默认 [Pod 管理策略](StatefulSets.md#pod-management-policies)(`OrderedReady`) 时使用 [滚动更新](StatefulSets.md#rolling-updates)，可能进入需要[人工干预](StatefulSets.md#forced-rollback) 才能修复的损坏状态。

## 组件 <a href="#components" id="components"></a>

下面的示例演示了 StatefulSet 的组件。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

上述例子中：

* 名为 `nginx` 的 Headless Service 用来控制网络域名。
* 名为 `web` 的 StatefulSet 有一个 Spec，它表明将在独立的 3 个 Pod 副本中启动 nginx 容器。
* `volumeClaimTemplates` 将通过 PersistentVolumes 驱动提供的 [PersistentVolumes](../zh/docs/concepts/storage/persistent-volumes/) 来提供稳定的存储。

StatefulSet 的命名需要遵循[DNS 子域名](../zh/docs/concepts/overview/working-with-objects/names/#dns-subdomain-names)规范。

## Pod 选择算符 <a href="#pod-selector" id="pod-selector"></a>

你必须设置 StatefulSet 的 `.spec.selector` 字段，使之匹配其在 `.spec.template.metadata.labels` 中设置的标签。在 Kubernetes 1.8 版本之前， 被忽略 `.spec.selector` 字段会获得默认设置值。 在 1.8 和以后的版本中，未指定匹配的 Pod 选择器将在创建 StatefulSet 期间导致验证错误。

## Pod 标识 <a href="#pod-identity" id="pod-identity"></a>

StatefulSet Pod 具有唯一的标识，该标识包括顺序标识、稳定的网络标识和稳定的存储。 该标识和 Pod 是绑定的，不管它被调度在哪个节点上。

### 有序索引 <a href="#ordinal-index" id="ordinal-index"></a>

对于具有 N 个副本的 StatefulSet，StatefulSet 中的每个 Pod 将被分配一个整数序号， 从 0 到 N-1，该序号在 StatefulSet 上是唯一的。

### 稳定的网络 ID <a href="#stable-network-id" id="stable-network-id"></a>

StatefulSet 中的每个 Pod 根据 StatefulSet 的名称和 Pod 的序号派生出它的主机名。 组合主机名的格式为`$(StatefulSet 名称)-$(序号)`。 上例将会创建三个名称分别为 `web-0、web-1、web-2` 的 Pod。 StatefulSet 可以使用 [无头服务](../zh/docs/concepts/services-networking/service/#headless-services) 控制它的 Pod 的网络域。管理域的这个服务的格式为： `$(服务名称).$(命名空间).svc.cluster.local`，其中 `cluster.local` 是集群域。 一旦每个 Pod 创建成功，就会得到一个匹配的 DNS 子域，格式为： `$(pod 名称).$(所属服务的 DNS 域名)`，其中所属服务由 StatefulSet 的 `serviceName` 域来设定。

取决于集群域内部 DNS 的配置，有可能无法查询一个刚刚启动的 Pod 的 DNS 命名。 当集群内其他客户端在 Pod 创建完成前发出 Pod 主机名查询时，就会发生这种情况。 负缓存 (在 DNS 中较为常见) 意味着之前失败的查询结果会被记录和重用至少若干秒钟， 即使 Pod 已经正常运行了也是如此。

如果需要在 Pod 被创建之后及时发现它们，有以下选项：

* 直接查询 Kubernetes API（比如，利用 watch 机制）而不是依赖于 DNS 查询
* 缩短 Kubernetes DNS 驱动的缓存时长（通常这意味着修改 CoreDNS 的 ConfigMap，目前缓存时长为 30 秒）

正如[限制](StatefulSets.md#limitations)中所述，你需要负责创建[无头服务](../zh/docs/concepts/services-networking/service/#headless-services) 以便为 Pod 提供网络标识。

下面给出一些选择集群域、服务名、StatefulSet 名、及其怎样影响 StatefulSet 的 Pod 上的 DNS 名称的示例：

| 集群域名          | 服务（名字空间/名字）   | StatefulSet（名字空间/名字） | StatefulSet 域名                  | Pod DNS                                      | Pod 主机名      |
| ------------- | ------------- | -------------------- | ------------------------------- | -------------------------------------------- | ------------ |
| cluster.local | default/nginx | default/web          | nginx.default.svc.cluster.local | web-{0..N-1}.nginx.default.svc.cluster.local | web-{0..N-1} |
| cluster.local | foo/nginx     | foo/web              | nginx.foo.svc.cluster.local     | web-{0..N-1}.nginx.foo.svc.cluster.local     | web-{0..N-1} |
| kube.local    | foo/nginx     | foo/web              | nginx.foo.svc.kube.local        | web-{0..N-1}.nginx.foo.svc.kube.local        | web-{0..N-1} |

\{{< note >\}} 集群域会被设置为 `cluster.local`，除非有[其他配置](../zh/docs/concepts/services-networking/dns-pod-service/)。 \{{< /note >\}}

### 稳定的存储 <a href="#stable-storage" id="stable-storage"></a>

对于 StatefulSet 中定义的每个 VolumeClaimTemplate，每个 Pod 接收到一个 PersistentVolumeClaim。在上面的 nginx 示例中，每个 Pod 将会得到基于 StorageClass `my-storage-class` 提供的 1 Gib 的 PersistentVolume。 如果没有声明 StorageClass，就会使用默认的 StorageClass。 当一个 Pod 被调度（重新调度）到节点上时，它的 `volumeMounts` 会挂载与其 PersistentVolumeClaims 相关联的 PersistentVolume。 请注意，当 Pod 或者 StatefulSet 被删除时，与 PersistentVolumeClaims 相关联的 PersistentVolume 并不会被删除。要删除它必须通过手动方式来完成。

### Pod 名称标签 <a href="#pod-name-label" id="pod-name-label"></a>

当 StatefulSet \{{< glossary\_tooltip term\_id="controller" >\}} 创建 Pod 时， 它会添加一个标签 `statefulset.kubernetes.io/pod-name`，该标签值设置为 Pod 名称。 这个标签允许你给 StatefulSet 中的特定 Pod 绑定一个 Service。

## 部署和扩缩保证 <a href="#deployment-and-scaling-guarantees" id="deployment-and-scaling-guarantees"></a>

* 对于包含 N 个 副本的 StatefulSet，当部署 Pod 时，它们是依次创建的，顺序为 `0..N-1`。
* 当删除 Pod 时，它们是逆序终止的，顺序为 `N-1..0`。
* 在将缩放操作应用到 Pod 之前，它前面的所有 Pod 必须是 Running 和 Ready 状态。
* 在 Pod 终止之前，所有的继任者必须完全关闭。

StatefulSet 不应将 `pod.Spec.TerminationGracePeriodSeconds` 设置为 0。 这种做法是不安全的，要强烈阻止。更多的解释请参考 [强制删除 StatefulSet Pod](../zh/docs/tasks/run-application/force-delete-stateful-set-pod/)。

在上面的 nginx 示例被创建后，会按照 web-0、web-1、web-2 的顺序部署三个 Pod。 在 web-0 进入 [Running 和 Ready](../zh/docs/concepts/workloads/pods/pod-lifecycle/) 状态前不会部署 web-1。在 web-1 进入 Running 和 Ready 状态前不会部署 web-2。 如果 web-1 已经处于 Running 和 Ready 状态，而 web-2 尚未部署，在此期间发生了 web-0 运行失败，那么 web-2 将不会被部署，要等到 web-0 部署完成并进入 Running 和 Ready 状态后，才会部署 web-2。

如果用户想将示例中的 StatefulSet 收缩为 `replicas=1`，首先被终止的是 web-2。 在 web-2 没有被完全停止和删除前，web-1 不会被终止。 当 web-2 已被终止和删除、web-1 尚未被终止，如果在此期间发生 web-0 运行失败， 那么就不会终止 web-1，必须等到 web-0 进入 Running 和 Ready 状态后才会终止 web-1。

### Pod 管理策略 <a href="#pod-management-policies" id="pod-management-policies"></a>

在 Kubernetes 1.7 及以后的版本中，StatefulSet 允许你放宽其排序保证， 同时通过它的 `.spec.podManagementPolicy` 域保持其唯一性和身份保证。

#### OrderedReady Pod 管理

`OrderedReady` Pod 管理是 StatefulSet 的默认设置。它实现了 [上面](StatefulSets.md#deployment-and-scaling-guarantees)描述的功能。

#### 并行 Pod 管理 <a href="#parallel-pod-management" id="parallel-pod-management"></a>

`Parallel` Pod 管理让 StatefulSet 控制器并行的启动或终止所有的 Pod， 启动或者终止其他 Pod 前，无需等待 Pod 进入 Running 和 ready 或者完全停止状态。 这个选项只会影响伸缩操作的行为，更新则不会被影响。

## 更新策略 <a href="#update-strategies" id="update-strategies"></a>

StatefulSet 的 `.spec.updateStrategy` 字段让 你可以配置和禁用掉自动滚动更新 Pod 的容器、标签、资源请求或限制、以及注解。 有两个允许的值：

`OnDelete` : 当 StatefulSet 的 `.spec.updateStrategy.type` 设置为 `OnDelete` 时， 它的控制器将不会自动更新 StatefulSet 中的 Pod。 用户必须手动删除 Pod 以便让控制器创建新的 Pod，以此来对 StatefulSet 的 `.spec.template` 的变动作出反应。

`RollingUpdate` : `RollingUpdate` 更新策略对 StatefulSet 中的 Pod 执行自动的滚动更新。这是默认的更新策略。

## 滚动更新 <a href="#rolling-updates" id="rolling-updates"></a>

当 StatefulSet 的 `.spec.updateStrategy.type` 被设置为 `RollingUpdate` 时， StatefulSet 控制器会删除和重建 StatefulSet 中的每个 Pod。 它将按照与 Pod 终止相同的顺序（从最大序号到最小序号）进行，每次更新一个 Pod。

Kubernetes 控制面会等到被更新的 Pod 进入 Running 和 Ready 状态，然后再更新其前身。 如果你设置了 `.spec.minReadySeconds`（查看[最短就绪秒数](StatefulSets.md#minimum-ready-seconds)），控制面在 Pod 就绪后会额外等待一定的时间再执行下一步。

### 分区滚动更新 <a href="#partitions" id="partitions"></a>

通过声明 `.spec.updateStrategy.rollingUpdate.partition` 的方式，`RollingUpdate` 更新策略可以实现分区。 如果声明了一个分区，当 StatefulSet 的 `.spec.template` 被更新时， 所有序号大于等于该分区序号的 Pod 都会被更新。 所有序号小于该分区序号的 Pod 都不会被更新，并且，即使他们被删除也会依据之前的版本进行重建。 如果 StatefulSet 的 `.spec.updateStrategy.rollingUpdate.partition` 大于它的 `.spec.replicas`，对它的 `.spec.template` 的更新将不会传递到它的 Pod。 在大多数情况下，你不需要使用分区，但如果你希望进行阶段更新、执行金丝雀或执行 分阶段上线，则这些分区会非常有用。

### 强制回滚 <a href="#forced-rollback" id="forced-rollback"></a>

在默认 [Pod 管理策略](StatefulSets.md#pod-management-policies)(`OrderedReady`) 下使用 [滚动更新](StatefulSets.md#rolling-updates) ，可能进入需要人工干预才能修复的损坏状态。

如果更新后 Pod 模板配置进入无法运行或就绪的状态（例如，由于错误的二进制文件 或应用程序级配置错误），StatefulSet 将停止回滚并等待。

在这种状态下，仅将 Pod 模板还原为正确的配置是不够的。由于 [已知问题](https://github.com/kubernetes/kubernetes/issues/67250)，StatefulSet 将继续等待损坏状态的 Pod 准备就绪（永远不会发生），然后再尝试将其恢复为正常工作配置。

恢复模板后，还必须删除 StatefulSet 尝试使用错误的配置来运行的 Pod。这样， StatefulSet 才会开始使用被还原的模板来重新创建 Pod。

### 最短就绪秒数 <a href="#minimum-ready-seconds" id="minimum-ready-seconds"></a>

\{{< feature-state for\_k8s\_version="v1.22" state="alpha" >\}}

`.spec.minReadySeconds` 是一个可选字段，用于指定新创建的 Pod 就绪（没有任何容器崩溃）后被认为可用的最小秒数。 默认值是 0（Pod 就绪时就被认为可用）。要了解 Pod 何时被认为已就绪，请参阅[容器探针](../zh/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)。

请注意只有当你启用 `StatefulSetMinReadySeconds` [特性门控](../zh/docs/reference/command-line-tools-reference/feature-gates/)时，该字段才会生效。

## {}

* 了解 [Pods](../zh/docs/concepts/workloads/pods/)。
* 了解如何使用 StatefulSet
  * 跟随示例[部署有状态应用](../zh/docs/tutorials/stateful-application/basic-stateful-set/)。
  * 跟随示例[使用 StatefulSet 部署 Cassandra](../zh/docs/tutorials/stateful-application/cassandra/)。
  * 跟随示例[运行多副本的有状态应用程序](../zh/docs/tasks/run-application/run-replicated-stateful-application/)。
  * 了解如何[扩缩 StatefulSet](../zh/docs/tasks/run-application/scale-stateful-set/)。
  * 了解[删除 StatefulSet](../zh/docs/tasks/run-application/delete-stateful-set/)涉及到的操作。
  * 了解如何[配置 Pod 以使用卷进行存储](../zh/docs/tasks/configure-pod-container/configure-volume-storage/)。
  * 了解如何[配置 Pod 以使用 PersistentVolume 作为存储](../zh/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)。
* `StatefulSet` 是 Kubernetes REST API 中的顶级资源。阅读 \{{< api-reference page="workload-resources/stateful-set-v1" >\}} 对象定义理解关于该资源的 API。
* 阅读[Pod 干扰预算（Disruption Budget）](../zh/docs/concepts/workloads/pods/disruptions/)，了解如何在干扰下运行高度可用的应用。
