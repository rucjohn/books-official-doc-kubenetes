# 有效地使用标签

到目前为止，在示例中使用的资源最多只使用了一个标签。在许多情况下，应使用多个 Labels 来区分不同集合。

例如，不同的应用可能会为 `app` 标签设置不同的值。但是，在遇到多层应用时，还需要区分每一层。前端可以携带以下标签：

```yaml
labels:
  app: guestbook
  tier: frontend
```

Redis 的主节点和从节点会有不同的 `tier` 标签，甚至还有一个额外的 `role` 标签：

```yaml
labels:
  app: guestbook
  tier: backend
  role: master
```

以及

```
labels:
  app: guestbook
  tier: backend
  role: slave
```

标签允许按照标签指定的任何维度对资源进行切片和切块：

```bash
kubectl apply -f examples/guestbook/all-in-one/guestbook-all-in-one.yaml
kubectl get pods -Lapp -Ltier -Lrole
```

输出

```
NAME                           READY     STATUS    RESTARTS   AGE       APP         TIER       ROLE
guestbook-fe-4nlpb             1/1       Running   0          1m        guestbook   frontend   <none>
guestbook-fe-ght6d             1/1       Running   0          1m        guestbook   frontend   <none>
guestbook-fe-jpy62             1/1       Running   0          1m        guestbook   frontend   <none>
guestbook-redis-master-5pg3b   1/1       Running   0          1m        guestbook   backend    master
guestbook-redis-slave-2q2yf    1/1       Running   0          1m        guestbook   backend    slave
guestbook-redis-slave-qgazl    1/1       Running   0          1m        guestbook   backend    slave
my-nginx-divi2                 1/1       Running   0          29m       nginx       <none>     <none>
my-nginx-o0ef1                 1/1       Running   0          29m       nginx       <none>     <none>
```



```bash
kubectl get pods -lapp=guestbook,role=slave
```

输出

```
NAME                          READY     STATUS    RESTARTS   AGE
guestbook-redis-slave-2q2yf   1/1       Running   0          3m
guestbook-redis-slave-qgazl   1/1       Running   0          3m
```

