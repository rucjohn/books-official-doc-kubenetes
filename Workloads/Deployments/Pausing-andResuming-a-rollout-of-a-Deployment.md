
## 暂停、恢复 Deployment <a href="#pausing-and-resuming-a-deployment" id="pausing-and-resuming-a-deployment"></a>

你可以在触发一个或多个更新之前暂停 Deployment，然后再恢复其执行。 这样做使得你能够在暂停和恢复执行之间应用多个修补程序，而不会触发不必要的上线操作。

*   例如，对于一个刚刚创建的 Deployment： 获取 Deployment 信息：

    ```shell
    kubectl get deploy
    ```

    输出类似于：

    ```
    NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx     3         3         3            3           1m
    ```

    获取上线状态：

    ```shell
    kubectl get rs
    ```

    输出类似于：

    ```shell
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   3         3         3         1m
    ```
*   使用如下指令暂停运行：

    ```shell
    kubectl rollout pause deployment.v1.apps/nginx-deployment
    ```

    输出类似于：

    ```shell
    deployment.apps/nginx-deployment paused
    ```
*   接下来更新 Deployment 镜像：

    ```shell
    kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
    ```

    输出类似于：

    ```
    deployment.apps/nginx-deployment image updated
    ```
*   注意没有新的上线被触发：

    ```shell
    kubectl rollout history deployment.v1.apps/nginx-deployment
    ```

    输出类似于：

    ```shell
    deployments "nginx"
    REVISION  CHANGE-CAUSE
    1   <none>
    ```
*   获取上线状态确保 Deployment 更新已经成功：

    ```shell
    kubectl get rs
    ```

    输出类似于：

    ```shell
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   3         3         3         2m
    ```
*   你可以根据需要执行很多更新操作，例如，可以要使用的资源：

    ```shell
    kubectl set resources deployment.v1.apps/nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi
    ```

    输出类似于：

    ```
    deployment.apps/nginx-deployment resource requirements updated
    ```

    暂停 Deployment 之前的初始状态将继续发挥作用，但新的更新在 Deployment 被 暂停期间不会产生任何效果。
*   最终，恢复 Deployment 执行并观察新的 ReplicaSet 的创建过程，其中包含了所应用的所有更新：

    ```shell
    kubectl rollout resume deployment.v1.apps/nginx-deployment
    ```

    输出：

    ```shell
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

    ```shell
    kubectl get rs
    ```

    输出类似于：

    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   0         0         0         2m
    nginx-3926361531   3         3         3         28s
    ```

你不可以回滚处于暂停状态的 Deployment，除非先恢复其执行状态。
