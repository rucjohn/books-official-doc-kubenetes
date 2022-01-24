# 字段选择器

字段选择器（Field Selectors）允许根据一个或多个资源字段的值筛选 Kubernetes 资源。下面是一些使用字段选择器查询的例子：

- metadata.name=mys-service
- metadata.namespace!=default
- status.phase=Pending

下面这个 `kubectl` 命令将筛选出 status.phase 字段值为 Running 的所有 Pod：
```
kubectl get pods --field-selector status.phase=Running
```

