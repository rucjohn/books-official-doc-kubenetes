
# 暂停、恢复 Deployment <a href="#pausing-and-resuming-a-deployment" id="pausing-and-resuming-a-deployment"></a>

当需要更新或计划更新 Deployment 时，可以在触发更新前暂停 Deployment 上线，然后在 Deployment 配置已更改完毕时，再恢复 Deployment 上线，这样可以使得能够在暂停和恢复之间应用多个修复，而不会触发不必要的上线操作。


*   例如，对于一个刚刚创建的 Deployment： 获取 Deployment 信息：

    ```bash
    kubectl get deploy
    ```

    输出类似于：

    ```
    NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx     3         3         3            3           1m
    ```

    获取上线状态：

    ```bash
    kubectl get rs
    ```

    输出类似于：

    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   3         3         3         1m
    ```
*   使用如下指令暂停运行：

    ```bash
    kubectl rollout pause deployment/nginx-deployment
    ```

    输出类似于：

    ```
    deployment.apps/nginx-deployment paused
    ```
*   接下来更新 Deployment 镜像：

    ```bash
    kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
    ```

    输出类似于：

    ```
    deployment.apps/nginx-deployment image updated
    ```
*   注意没有新的上线被触发：

    ```bash
    kubectl rollout history deployment/nginx-deployment
    ```

    输出类似于：

    ```
    deployments "nginx"
    REVISION  CHANGE-CAUSE
    1   <none>
    ```
*   获取上线状态确保 Deployment 更新已经成功：

    ```bash
    kubectl get rs
    ```

    输出类似于：

    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   3         3         3         2m
    ```
*   可以根据需要执行很多更新操作，例如，可以要使用的资源：

    ```shell
    kubectl set resources deployment/nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi
    ```

    输出类似于：

    ```
    deployment.apps/nginx-deployment resource requirements updated
    ```

    暂停 Deployment 之前的初始状态将继续发挥作用，但新的更新在 Deployment 被 暂停期间不会产生任何效果。
*   最终，恢复 Deployment 执行并观察新的 ReplicaSet 的创建过程，其中包含了所应用的所有更新：

    ```bash
    kubectl rollout resume deployment.v1.apps/nginx-deployment
    ```

    输出：

    ```
    deployment.apps/nginx-deployment resumed
    ```
*   观察上线的状态，直到完成。

    ```shell
    kubectl get rs -w
    ```

    输出类似于：

    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   2         2         2         2m
    nginx-3926361531   2         2         0         6s
    nginx-3926361531   2         2         1         18s
    nginx-2142116321   1         2         2         2m
    nginx-2142116321   1         2         2         2m
    nginx-3926361531   3         2         1         18s
    nginx-3926361531   3         2         1         18s
    nginx-2142116321   1         1         1         2m
    nginx-3926361531   3         3         1         18s
    nginx-3926361531   3         3         2         19s
    nginx-2142116321   0         1         1         2m
    nginx-2142116321   0         1         1         2m
    nginx-2142116321   0         0         0         2m
    nginx-3926361531   3         3         3         20s
    ```
*   获取最近上线的状态：

    ```
    kubectl get rs
    ```

    输出类似于：

    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   0         0         0         2m
    nginx-3926361531   3         3         3         28s
    ```

注意：

不可以回滚处于暂停状态的 Deployment，除非先恢复其执行状态。
