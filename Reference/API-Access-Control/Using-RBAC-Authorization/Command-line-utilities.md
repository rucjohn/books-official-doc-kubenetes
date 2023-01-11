# 命令行工具

### kubectl create role

创建 Role 对象，定义在某一命名空间中的权限。例如：

*   创建名称为 "pod-reader" 的 Role 对象，允许用户对 Pods 执行 `get`、`watch` 和 `list` 操作：

    ```bash
    kubectl create role pod-reader --verb=get --verb=watch --verb=list --resource=pods
    ```
*   创建名称 "pod-reader" 的 Role 对象并指定 `resourceNames`：

    ```bash
    kubectl create role pod-reader --verb=get --resource=pods --resource-name=readablepod --resource-name=anotherpod
    ```
*   创建名为 "foo" 的 Role 对象并指定 `apiGroups`：

    ```bash
    kubectl create role foo --verb=get,list,watch --resource=replicasets.apps
    ```
*   创建名为 "foo" 的 Role 对象并指定子资源权限：

    ```bash
    kubectl create role foo --verb=get,list,watch --resource=pods,pods/status
    ```
*   创建名为 "my-component-lease-holder" 的 Role 对象，使期具有对特定名称的资源执行 get/update 的权限：

    ```bash
    kubectl create role my-component-lease-holder --verb=get,list,watch,update --resource=lease --resource-name=my-component
    ```
