# 附录


## 以声明方式创建 Autoscaler

除了使用 `kubectl autoscale` 命令，也可以使用以下清单以声明方式创建 HorizontalPodAutoscaler：
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```
创建
```bash
kubectl create -f https://k8s.io/examples/application/hpa/php-apache.yaml
```
输出
```
horizontalpodautoscaler.autoscaling/php-apache created
```

