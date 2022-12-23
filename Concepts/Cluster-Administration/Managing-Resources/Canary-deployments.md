# 金丝雀部署

另一个需要多标签的场景，是用来区分同一组件的不同版本或不同配置的多个部署。常见的做法是部署一个金丝雀发布来部署新应用版本（在 Pod 模板中通过镜像标签指定），操持新旧版本应用同时在线。这样，新版本在完全发布之前也可以接收实时的生产流量。

假如，使用 `track` 标签来区分不同的版本。主要稳定的发行版的标签是 `track: stable`：

```yaml
     name: frontend
     replicas: 3
     ...
     labels:
        app: guestbook
        tier: frontend
        track: stable
     ...
     image: gb-frontend:v3
```

然后，创建一个新版本，让标签设置为 `track: canary`，以便两组 Pod 不会重复：

```yaml
     name: frontend-canary
     replicas: 1
     ...
     labels:
        app: guestbook
        tier: frontend
        track: canary
     ...
     image: gb-frontend:v4
```


Service 通过筛选标签的公共子集（即忽略 `track` 标签）来覆盖两组副本，以便流量可以转发到两个应用上：

```yaml
    selector:
      app: guestbook
      tier: frontend
```

可以调整 stable 和 canary 版本的副本数量，以确定每个版本将接收实时生产流量的比例（本例为 3：1）。一旦新版本验证通过，可以将新版本应用的 `track` 标签的值从 canary 替换为 stable，此时会覆盖老版本 Pod，即删除老版本应用。

想要了解更多的具体示例，请查看 [Ghost 部署教程](https://github.com/kelseyhightower/talks/tree/master/kubecon-eu-2016/demo#deploy-a-canary)。

