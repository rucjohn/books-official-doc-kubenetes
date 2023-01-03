# 服务定义

## Service 资源

Kubernetes Service 定义了这样一种抽象：逻辑上的一组 Pod，以及可以访问它们的策略 —— 通常称之为微服务。

Service 所针对的 Pod 集合通常是通过 Selector 来确定的。要了解定义服务端点的其他方法，请参阅 [不带 Selector 的 Service](Service-Definition.md#services-without-selectors)。

### 云原生服务发现

如果想要在应用程序中使用 Kubernetes API 进行服务发现，则可以查询 apiserver 用于匹配 EndpointSlices。只要服务中的 Pod 集合发生变更，Kubernetes 就会为服务更新 EndpointSlices。

对于非本机应用程序，Kubernetes 提供了在应用他可以餐厅后端 Pod 之间放置网络端口或负载均衡器的方法。

## 定义 Service

Service 在 Kubernetes 中是一个 REST 对象，和 Pod 类似。像所有 REST 对象一样，Service 定义可以基于 `POST` 方式，请求 apiserver 创建新的实例。Service 对象的名称是合法的 RFC 1035 标签名称。

例如，有一组 Pod，它们对外 暴露了 9376 端口，同时还被打上 `app.kubernetes.io/name=MyApp` 标签：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
  - portocol: TCP
    port: 80
    targetPort: 9376
```

上述配置创建一个名称名为 "my-service" 的 Service 对象，它会将请求代理到 TCP 端口的 9376，并且具有标签 `app.kubernetes.io/name=MyApp` 的 Pod 上。

Kubernetes 为该服务分配一个 IP 地址（有时称为 "集群 IP"），该 IP 地址由服务代理使用。（请参阅 [虚拟 IP 寻址机制](Service-Definition.md#xu-ni-ip-xun-zhi-ji-zhi)）。

Service Selector 控制器不断扫描与其选择算符匹配的 Pod，然后将所有更新发布到同样名称为 `my-service` 的 Endpoint 对象上。

{% hint style="info" %}
<mark style="color:blue;">**说明：**</mark>

需要注意的，Service 能够将一个接收 `port` 映射到任意的 `targetPort` 上。默认情况下，`targetPort` 将被设置为与 `port` 字段相同的值。
{% endhint %}

Pod 中的端口定义是有名字的，可以在 Service 的 `targetPort` 属性中引用这些名称。例如，可以通过以下方式将 Service 的 `targetPort` 绑定到 Pod 端口：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: proxy
spec:
  containers:
  - name: nginx
    image: nginx:stable
    ports:
      - containerPort: 80
        name: http-web-svc
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app.kubernetes.io/name: proxy
  ports:
  - name: name-of-service-port
    portocol: TCP
    port: 80
    targetPort: http-web-svc
```

即使 Service 中使用同一配置名称混合使用多个 Pod，各 Pod 通过不同的端口号支持相同的网络协议，此功能也可以使用。这为 Service 的部署和深化提供了很大的灵活性。例如，可以在新版本中更改 Pod 中后端软件暴露的端口号，而不会因此破坏客户端。

服务的默认协调是 TCP，还可以使用任何其他 [受支持的协议](../../../Reference/Networking-Reference/Protocols-for-Services.md)。

由于许多服务需要公开多个端口，因此 Kubernetes 在服务对象上支持多个端口定义。每个端口定义可以具有相同的 `protocol`，也可以具有不同的协议。

#### 没有选择算符的 Service <a href="#services-without-selectors" id="services-without-selectors"></a>

## 虚拟 IP 寻址机制

阅读 [虚拟 IP 和服务代理](../../../Reference/Networking-Reference/Virtual-IPs-and-Service-Proxies.md) 以了解 Kubernetes 提供的使用虚拟 IP 地址公开服务的机制。
