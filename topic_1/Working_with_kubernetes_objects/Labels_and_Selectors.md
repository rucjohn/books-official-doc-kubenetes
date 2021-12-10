# 标签和选择器

标签（**Lables**）是附加到 Kubernetes 对象（比如 Pods）上的键值对。标签旨在用于指定对用户有意义和相关的对象的标识属性，但不直接暗示核心的语义。标签可有用于组织和选择对象的子集。标签可以在创建时附加到对象上，之后可以随时添加和修改。每个对象都可以定义一组 {KEY: VALUE} 标签，每个 KEY 对于给定的对象必须是唯一的。
```
{
  "metadata": {
    "labels": {
      "key1": "values1",
      "key2": "values2"
    }
  }
}
```
标签能够支持高效的里查询和是监听操作，非常适合在用户界面和命令行中使用。应使用注解（**Annotations**）来记录非识别信息。

## 动机

标签能够让用户以松耦合的方式将自己的组织结构映射到系统对象上，而无需客户端存储这些映射。

服务部署和批处理流水线通常是多维度的（例如，多个分区部署、多个发行序列、多个层，每层中的多个微服务等）。管理这些通常需要交叉操作，这又打破了严格分层表示的封装，尤其是由基础设施而不是用户确定的刚性层次结构。

示例标签：
- `release: stable`
- `release: canary`
- `environment: dev`
- `environment: qa`
- `environment: production`
- `tier: frontend`
- `tier: backend`
- `tier: cache`
- `partition: customerA`
- `partition: customerB`
- `track: daily`
- `track: weekly`

## 推荐使用的标签

除了 kubectl 和 dashboard 之外，可以使用其他工具来可视化管理 Kubernetes 对象。一组能用的标签可以让多个工具之间相互操作，用所有工具都能理解的能用方式描述对象。

