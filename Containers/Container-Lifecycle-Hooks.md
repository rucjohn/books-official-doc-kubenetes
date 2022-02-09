# 容器生命周期回调

这个页面描述了 kubelet 管理的容器如何使用容器生命周期回调框架，藉由其管理生命周期中的事件触发，运行指定代码。

## 概述

类似于许多具有生命周期回调组件的编程语言框架，例如 Angular，Kubernetes 也为容器提供了生命周期回调。回调使容器能够了解其管理生命周期中的事件，并在执行相应的生命周期回调时运行在处理程序中实现的代码。

## 容器回调（Hook）

有两个回调暴露给容器：

`PostStart` 

这个回调在容器被创建之后立即被执行。但是，不能保证回调会在容器入口（ENTRYPOINT）之前执行。没有参数传递给处理程序。

`PreStop`

在容器因 API 请求或者管理事件（诸如 LivenessProbe、StartupProbe、资源抢占、资源竞争等）而被终止之前，此回调会被调用。如果容器已经处于已终止或者已完成状态，则对 preStop 回调的调用将失败。在用来停止容器的 TEAM 信号被发出之前，回调必须执行结束。Pod 的终止宽限周期在 `PreStop` 回调被执行之前即开始计数，所以无论回调函数的执行结果如何，容器最终 都会在 Pod 的终止期限内被终止。没有参数会被传统处理程序。

### 回调处理程序的实现

容器可以通过实现和注册该回调的处理程序来访问该回调。针对容器，有两种类型的回调处理程序可供实现：
- Exec：在容器的 cgroups 和命名空间中执行特定的命令（例如 `pre-stop.sh`）。命令所消耗的资源讲稿容器的资源消耗。
- HTTP：对容器上的特定端口执行 HTTP 请求。

### 回调处理程序执行

当调用容器生命周期管理回调时，Kubernetes 管理系统根据回调动作执行其处理程序，`httpGet` 和 `tcpSocket` 在 kubelet 进程执行，而 `exec` 则由容器内执行。

回调处理程序调用在包含容器的 Pod 上下文中是同步的。这意味着对于 `PostStart` 回调，容器 ENTRYPOINT 和回调异步触发。但是，如果回调运行或挂起的时间太长，则容器无法达到 `Running` 状态。

`PreStop` 回调并不会与停止容器的信号处理程序异步执行；回调必须在可以发送信号之前完成执行。如果 `PreStop` 回调在执行期间停滞不前，Pod 的阶段会变成 `Terminating` 并且一直处于该状态，直接到其 `terminationGracePeriodSeconds` 耗尽为止，这时 Pod 会被杀死。这一期限是针对  `PreStop` 回调的执行时间及容器正常停止时间的总和而言的。

例如，如果 `terminationGracePeriodSeconds` 是 60，回调函数花了 55 秒完成执行，而容器在收到信号之后花了 10 秒种来正常结束，那么容器会在其能够正常结束之前即被杀死，因为 `terminationGracePeriodSeconds` 的值小于后面两件事情所花费的总时间。

如果 `PostStart` 或 `PreStop` 回调失败，它会杀死容器。

用户应该使他们的回调处理程序尽可能的轻量级。但也需要考虑长时间运行的命令也有用的情况，比如在停止容器前保存状态。


### 回调递送保证

回调的递送应该是至少一次，这意味着对于任何给定的事件，例如 `PostStart` 或 `PreStop`，回调可以被调用多次。如何正确处理被多次调用的情况，是回调实现所要考虑的问题。

通常情况下，只会进行单次递送。例如，如果 HTTP 回调接收器宕机，无法接收流量，则不会尝试重新发送。然而，偶尔也会发生重新递送的可能。例如，如果 kubelet 在发送回调的过程中重新启动，回调可能会在 kubelet 恢复后重新发送。

### 调试回调处理程序

回调处理程序的日志不会在 Pod 事件中公开。如果处理程序由于某些原因失败，它将发送一个事件。
- 对于 `PostStart`，这是 `FailedPostStartHook` 事件
- 对于 `PreStop`，这是 `FailedPreStopHook` 事件

可以通过运行 `kubectl describe pod <POD NAME>` 命令来查看事件。下面是运行这个命令的一些事件输出示例：
```
Events:
  FirstSeen    LastSeen    Count    From                            SubobjectPath        Type        Reason        Message
  ---------    --------    -----    ----                            -------------        --------    ------        -------
  1m        1m        1    {default-scheduler }                                Normal        Scheduled    Successfully assigned test-1730497541-cq1d2 to gke-test-cluster-default-pool-a07e5d30-siqd
  1m        1m        1    {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}    spec.containers{main}    Normal        Pulling        pulling image "test:1.0"
  1m        1m        1    {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}    spec.containers{main}    Normal        Created        Created container with docker id 5c6a256a2567; Security:[seccomp=unconfined]
  1m        1m        1    {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}    spec.containers{main}    Normal        Pulled        Successfully pulled image "test:1.0"
  1m        1m        1    {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}    spec.containers{main}    Normal        Started        Started container with docker id 5c6a256a2567
  38s        38s        1    {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}    spec.containers{main}    Normal        Killing        Killing container with docker id 5c6a256a2567: PostStart handler: Error executing in Docker Container: 1
  37s        37s        1    {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}    spec.containers{main}    Normal        Killing        Killing container with docker id 8df9fdfd7054: PostStart handler: Error executing in Docker Container: 1
  38s        37s        2    {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}                Warning        FailedSync    Error syncing pod, skipping: failed to "StartContainer" for "main" with RunContainerError: "PostStart handler: Error executing in Docker Container: 1"
  1m         22s         2     {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}    spec.containers{main}    Warning        FailedPostStartHook
```

