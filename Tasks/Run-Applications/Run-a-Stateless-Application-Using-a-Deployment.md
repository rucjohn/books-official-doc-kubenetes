# 使用 Deployment 运行一个无状态应用

本文介绍如何通过 Kubernetes Deployment 对象去运行一个应用。

## 教程目标

- 创建一个 nginx Deployment。
- 使用 kubectl 列举关于 Deployment 的信息。
- 更新 Deployment。

## 创建并了解一个 nginx Deployment

可以通过创建一个 Kubernetes Deployment 对象来运行一个应用, 且可以在一个 YAML 文件中描述 Deployment。

例如, 下面这个 YAML 文件描述了一个运行 nginx:1.14.2 Docker 镜像的 Deployment：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

1. 通过 YAML 文件创建一个 Deployment：

   ```shell
   kubectl apply -f nginx-deployment.yaml
   ```

2. 显示 Deployment 相关信息：

   ```shell
   kubectl describe deployment nginx-deployment
   ```
   
   输出：

   ```
   Name:     nginx-deployment
   Namespace:    default
   CreationTimestamp:  Tue, 30 Aug 2016 18:11:37 -0700
   Labels:     app=nginx
   Annotations:    deployment.kubernetes.io/revision=1
   Selector:   app=nginx
   Replicas:   2 desired | 2 updated | 2 total | 2 available | 0 unavailable
   StrategyType:   RollingUpdate
   MinReadySeconds:  0
   RollingUpdateStrategy:  1 max unavailable, 1 max surge
   Pod Template:
     Labels:       app=nginx
     Containers:
      nginx:
       Image:              nginx:1.14.2
       Port:               80/TCP
       Environment:        <none>
       Mounts:             <none>
     Volumes:              <none>
   Conditions:
     Type          Status  Reason
     ----          ------  ------
     Available     True    MinimumReplicasAvailable
     Progressing   True    NewReplicaSetAvailable
   OldReplicaSets:   <none>
   NewReplicaSet:    nginx-deployment-1771418926 (2/2 replicas created)
   No events.
   ```

3. 列出 Deployment 所创建的 Pod：

   ```shell
   kubectl get pods -l app=nginx
   ```
   
   输出：

   ```
   NAME                                READY     STATUS    RESTARTS   AGE
   nginx-deployment-1771418926-7o5ns   1/1       Running   0          16h
   nginx-deployment-1771418926-r18az   1/1       Running   0          16h
   ```

4. 展示某一个 Pod 信息：

   ```shell
   kubectl describe pod <pod-name>
   ```

   这里的 `<pod-name>` 是某一 Pod 的名称。


## 更新 Deployment

通过更新一个新的 YAML 文件来更新 Deployment。下面的 YAML 文件指定该 Deployment 镜像更新为 nginx 1.16.1。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.16.1
        ports:
        - containerPort: 80
```

1. 应用新的 YAML：

   ```shell
   kubectl apply -f nginx-deployment-update.yaml
   ```

2. 查看该 Deployment，将创建新 Pod， 同时删除旧的 Pod：

   ```shell
   kubectl get pods -l app=nginx
   ```

## 扩容应用

通过应用新的 YAML 文件来增加 Deployment 中 Pods 的数量。

下面的 YAML 文件将 `replicas` 设置为 4，指定该 Deployment 应有 4 个 Pod：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 4 # 副本从 2 个更新到 4 个
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```


1. 应用新的 YAML 文件：

   ```shell
   kubectl apply -f nginx-deployment-scale.yaml
   ```

2. 验证 Deployment 有 4 个 Pods：

   ```shell
   kubectl get pods -l app=nginx
   ```

   输出:

   ```
   NAME                               READY     STATUS    RESTARTS   AGE
   nginx-deployment-148880595-4zdqq   1/1       Running   0          25s
   nginx-deployment-148880595-6zgi1   1/1       Running   0          25s
   nginx-deployment-148880595-fxcez   1/1       Running   0          2m
   nginx-deployment-148880595-rwovn   1/1       Running   0          2m
   ```

## 删除 Deployment

基于名称删除 Deployment：

```shell
kubectl delete deployment nginx-deployment
```

