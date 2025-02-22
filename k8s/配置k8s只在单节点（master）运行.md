根据你的错误信息：

```
0/1 nodes are available: 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate.
```

这是因为 Kubernetes 默认会给 master 节点添加 `node-role.kubernetes.io/master` 的污点（taint），这会阻止非控制平面 Pods 调度到 master 节点上。为了让 Pods 能够部署到 master 节点，你有两个选择：

### 1. **让 Pod 容忍 master 节点的污点（Tolerate the taint）**

你可以通过在 KEDA 的 Deployment 配置中添加 `tolerations` 来让 Pod 容忍 master 节点上的污点。可以修改 `keda-operator` 的 Deployment 配置，添加如下内容：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: keda-operator
  namespace: keda
spec:
  template:
    spec:
      tolerations:
      - key: "node-role.kubernetes.io/master"
        operator: "Exists"
        effect: "NoSchedule"
```

在 KEDA 的 Deployment 中加上这个 toleration 配置后，Pod 就可以调度到 master 节点了。

### 2. **移除 master 节点的污点（Remove the taint）**

如果你只是希望将 Pods 部署到 master 节点，并且不介意在 master 节点上运行应用程序，可以直接移除该污点。这会让所有的 Pods 都能调度到 master 节点。

运行以下命令来移除污点：

```bash
kubectl taint nodes <master-node-name> node-role.kubernetes.io/master:NoSchedule-
```

这将移除 master 节点上的污点，允许任何 Pod 被调度到 master 节点。

你可以运行以下命令来确认污点是否已经移除：

```bash
kubectl describe node <master-node-name>
```

### 总结

- 如果你只想在 master 节点上运行 KEDA，你可以选择第一种方法，向 KEDA 的 Deployment 配置中添加 `tolerations`。
- 如果你希望所有 Pod 都可以调度到 master 节点，可以选择第二种方法，移除 master 节点的污点。

这两种方式都可以让你在 master 节点上部署 Pod，选择最适合你的方案。