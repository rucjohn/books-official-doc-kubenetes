# 容器环境

本页描述了容器环境里容器可用的资源。

## 容器环境

Kubernetes 的容器环境给容器提供了几个重要的资源：
- 文件系统，其中包含一个镜像和一个或多个的卷
- 容器自身的信息
- 集群中其他对象的信息

### 容器信息

容器的 *hostname* 是它所运行的 pod 的名称。它可以通过 `hostname` 命令或者调用 libc 中的 `gethostname()` 函数来获取。

Pod 的名称和命名空间可以通过下行API转换为环境变量。

Pod 定义中的用户所定义的环境变量也可在容器中使用，就像在 Docker 镜像中静态指定的任何环境变量一样。

### 集群信息

创建容器时，正在运行的所有服务都可用作该容器的环境变量。这里的服务仅限于新容器的 Pod 所在的命名空间中的服务，以及 Kubernetes 控制平面的服务。这些环境变量与 Docker 链接 的语法相同。

对于名为 *foo* 的服务，当映射到名为 *bar* 的容器时，以下变量是被定义了的：
```ini
FOO_SERVICE_HOST=<the host the service is running on>
FOO_SERVICE_PORT=<the port the service is running on>
```

服务具有专用的 IP 地址。如果启动了 DNS 插件，可以在容器通过 DNS 来访问服务。
