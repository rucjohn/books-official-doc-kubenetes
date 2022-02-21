# 使用 Deployment 运行一个无状态应用

本文介绍如何通过 Kubernetes Deployment 对象去运行一个应用。

## 教程目标

- 创建一个 nginx Deployment。
- 使用 kubectl 列举关于 Deployment 的信息。
- 更新 Deployment。

## 创建并了解一个 nginx Deployment

可以通过创建一个 Kubernetes Deployment 对象来运行一个应用, 且可以在一个 YAML 文件中描述 Deployment。

例如, 下面这个 YAML 文件描述了一个运行 nginx:1.20.1 Docker 镜像的 Deployment：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20.1
        ports:
        - containerPort: 80
```

1. 通过 YAML 文件创建一个 Deployment：

   ```bash
   kubectl apply -f nginx-deployment.yaml
   ```

2. 显示 Deployment 相关信息：

   ```bash
   kubectl describe deployment nginx-deployment
   ```
   
   输出：

   ```
    Name:                   nginx-deployment
    Namespace:              default
    CreationTimestamp:      Mon, 21 Feb 2022 17:01:48 +0800
    Labels:                 app=nginx
    Annotations:            deployment.kubernetes.io/revision: 1
    Selector:               app=nginx
    Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
      Labels:  app=nginx
      Containers:
       nginx:
        Image:        nginx:1.20.1
        Port:         80/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   nginx-deployment-58b9b8ff79 (1/1 replicas created)
    Events:
      Type    Reason             Age   From                   Message
      ----    ------             ----  ----                   -------
      Normal  ScalingReplicaSet  2m4s  deployment-controller  Scaled up replica set nginx-deployment-58b9b8ff79 to 1
   ```

3. 列出 Deployment 所创建的 Pod：

   ```bash
   kubectl get pods -l app=nginx
   ```
   
   输出：

   ```
    NAME                                READY   STATUS    RESTARTS   AGE
    nginx-deployment-58b9b8ff79-f7tt8   1/1     Running   0          5m59s
   ```

4. 展示某一个 Pod 信息：

   ```bash
   kubectl describe pod <pod-name>
   ```

   这里的 `<pod-name>` 是某一 Pod 的名称。


## 更新 Deployment

通过更新一个新的 YAML 文件来更新 Deployment。下面的 YAML 文件指定该 Deployment 镜像更新为 nginx 1.20.2。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20.2
        ports:
        - containerPort: 80
```

1. 应用新的 YAML：

   ```bash
   kubectl apply -f nginx-deployment-update.yaml
   ```

2. 查看该 Deployment，将创建新 Pod， 同时删除旧的 Pod：

   ```bash
   kubectl get pods -l app=nginx
   ```

## 扩容应用

通过应用新的 YAML 文件来增加 Deployment 中 Pods 的数量。

下面的 YAML 文件将 `replicas` 设置为 3，指定该 Deployment 应有 3 个 Pod：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3 # 副本从 1 个扩容到 3 个
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20.1
        ports:
        - containerPort: 80
```


1. 应用新的 YAML 文件：

   ```bash
   kubectl apply -f nginx-deployment-scale.yaml
   ```

2. 验证 Deployment 有 3 个 Pods：

   ```bash
   kubectl get pods -l app=nginx
   ```

   输出:

   ```
    NAME                                READY   STATUS    RESTARTS   AGE
    nginx-deployment-58b9b8ff79-f7tt8   1/1     Running   0          26m
    nginx-deployment-58b9b8ff79-tnf69   1/1     Running   0          18m
    nginx-deployment-58b9b8ff79-x2vmr   1/1     Running   0          18m
   ```

## 删除 Deployment

基于名称删除 Deployment：

```bash
kubectl delete deployment nginx-deployment
```

