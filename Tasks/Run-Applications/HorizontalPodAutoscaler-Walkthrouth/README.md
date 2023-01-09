# HPA 演练

[HorizontalPodAutoscaler](../Horizontal-Pod-Autoscaling/)（简称 HPA）自动更新工作负载资源（例如，Deployment 或 StatefulSet），目的是自动扩缩容工作负载以满足需求。

本文档将引导完成启用 HorizontalPodAutoscaler 以自动管理Web 应用程序扩缩容的示例。此示例工作负载是运行一些 PHP 代码的 Apache httpd。

## 准备开始

必须拥有一个 Kubernetes 集群，同时 Kubernetes 集群必须带有 kubectl 命令行工具。建议在至少有两个节点的集群上运行本教程，且这些节点不是控制平面。

Kubernetes 版本必须不低于 1.23 版本，可以使用 `kubectl version` 命令查看当前 Kubernetes 版本。

按照本演练进行操作，需要一个部署并配置了 Metrics Server 的集群。Kubernetes Metrics Server 从集群中的 kubelets 采集资源指标，并通过 Kubernetes API 公开这些指标，使用 apiserver 添加代表指标读数的新资源。

要了解如何部署 Metrics Server，请参阅 Metrics-server 文档。

## 运行 php-apache 服务器并暴露服务

为了演示 HorizontalPodAutoscaler，首先需要启动一个 Deployment，使用 `hpa-example` 镜像运行一个容器，然后使用以下清单文件将其暴露为一个 Service。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  replicas: 1
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
```

运行命令：

```bash
kubectl apply -f https://k8s.io/examples/application/php-apache.yaml
```

输出

```
deployment.apps/php-apache created
service/php-apache created
```
