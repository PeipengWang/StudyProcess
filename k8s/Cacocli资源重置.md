要删除 Calico 资源，可以按以下步骤操作：

### 1. **删除 Calico DaemonSet 和 Deployment**

首先，删除 Calico 相关的 DaemonSet 和 Deployment：

```bash
kubectl delete daemonset calico-node -n kube-system
kubectl delete deployment calico-kube-controllers -n kube-system
```

### 2. **删除 Calico 相关的 Role 和 RoleBinding**

删除相关的 Role 和 RoleBinding 资源：

```bash
kubectl delete clusterrole calico-node
kubectl delete clusterrolebinding calico-node
kubectl delete clusterrole calico-kube-controllers
kubectl delete clusterrolebinding calico-kube-controllers
```

### 3. **删除 Calico 配置**

如果你修改了 `calico.yaml` 配置并应用了它，可以通过以下命令删除相关的配置资源：

```bash
kubectl delete -f calico.yaml
```

### 4. **清理 Calico 网络配置（如果需要）**

如果你使用了 Calico CNI 网络插件，在某些情况下可能需要手动清理网络配置文件。例如，删除 `/etc/cni/net.d/` 中的 Calico 网络配置文件。

```bash
rm -rf /etc/cni/net.d/*
```

### 5. **删除 Calico 镜像**

如果你不再需要 Calico，可以删除相关的 Docker 镜像：

```bash
docker rmi calico/cni:v3.25
docker rmi calico/node:v3.25
docker rmi calico/kube-controllers:v3.25
```

### 6. **删除相关的 ConfigMap 和 Secret（如果有）**

检查并删除与 Calico 相关的 ConfigMap 和 Secret：

```bash
kubectl delete configmap calico-config -n kube-system
kubectl delete secret calico-secrets -n kube-system
```

### 7. **删除 Calico 网络资源**

如果你使用了 Calico 的 IPPool 资源，也需要删除：

```bash
kubectl delete ippool --all -n kube-system
```

### 8. **删除 Calico 所使用的 CRDs（Custom Resource Definitions）**

Calico 使用 CRD（自定义资源定义）来管理网络池等资源。如果你不再需要这些 CRDs，可以删除它们：

```bash
kubectl delete crd ippools.crd.projectcalico.org
kubectl delete crd felixconfigurations.crd.projectcalico.org
kubectl delete crd bgppeers.crd.projectcalico.org
kubectl delete crd ipamblocks.crd.projectcalico.org
kubectl delete crd ipamhandles.crd.projectcalico.org
kubectl delete crd ipconfigs.crd.projectcalico.org
kubectl delete crd networksets.crd.projectcalico.org
```

### 9. **删除节点上的 Calico 网络插件**

如果 Calico 安装成功，节点上可能会存在 Calico 配置文件和二进制文件。如果你要彻底删除，可以手动删除相关文件：

```bash
rm -rf /etc/cni/net.d/
rm -rf /opt/cni/bin/
```

### 10. **重启 kubelet**

删除 Calico 后，建议重启 kubelet 服务，以确保没有残留的配置影响后续操作：

```bash
systemctl restart kubelet
```

### 总结：

通过这些步骤，你可以删除 Kubernetes 中的 Calico 网络插件和相关资源。如果你打算重新安装 Calico，确保所有资源都已经清理干净，以避免配置冲突。





要检查 Calico 是否正常安装和运行，可以通过以下步骤进行诊断：

### 1. **检查 Calico Pod 状态**

首先检查 Calico 的 Pod 是否正常运行，尤其是 `calico-node` 和 `calico-kube-controllers` 这两个关键组件：

```bash
kubectl get pods -n kube-system | grep calico
```

你应该能够看到类似下面的输出：

```bash
calico-kube-controllers-xxxxxxx-xxxxx   1/1     Running   0          2d
calico-node-xxxxxxx-xxxxx               1/1     Running   0          2d
```

如果 Pod 处于 `Running` 状态，并且没有 `Pending` 或 `CrashLoopBackOff` 等异常状态，说明 Calico 的主要组件是健康的。

### 2. **检查 Calico 节点状态**

使用 `calicoctl`（如果已经安装）来检查 Calico 节点状态。首先，确保你已安装并配置 `calicoctl` 工具。然后，运行以下命令来查看 Calico 节点的状态：

```bash
calicoctl get nodes
```

输出应该类似：

```bash
NAME           HOSTNAME   IPV4   IPV6   CONNTRACK
k8s-node-1     node-1     10.0.0.1   -      0
k8s-node-2     node-2     10.0.0.2   -      0
```

如果节点列表正常显示并且没有错误信息，Calico 节点运行正常。

### 3. **检查 Calico 网络策略**

如果 Calico 网络插件正常运行，你应该可以看到相关的网络策略。使用以下命令来查看网络策略：

```bash
kubectl get networkpolicies --all-namespaces
```

如果没有配置策略，输出会为空；但如果有配置网络策略，你应该看到这些策略在集群中生效。

### 4. **查看 Calico 日志**

查看 Calico 节点容器的日志以确保没有错误。可以查看 `calico-node` Pod 的日志：

```bash
kubectl logs -n kube-system calico-node-<pod-name>
```

查找任何与网络或配置相关的错误信息。如果没有日志输出，可能是由于容器启动失败，你可以通过 `kubectl describe pod` 查看相关的事件和错误信息。

### 5. **检查 Calico CNI 插件是否安装**

Calico 安装后，CNI 插件应该已经配置。可以通过查看节点上 `/etc/cni/net.d/` 目录中的文件来验证是否有 Calico 的 CNI 配置文件：

```bash
ls /etc/cni/net.d/
```

你应该看到一个名为 `10-calico.conflist` 的文件，这表示 Calico CNI 插件已正确安装。

### 6. **检查网络连通性**

确保 Kubernetes 集群中的各个节点之间能够互相通信。你可以尝试在 Pod 之间 ping 通，或者检查是否有网络隔离问题。比如：

1. **在 Pod 内部进行测试**：

   在一个 Pod 中运行网络连通性测试：

   ```bash
   kubectl run -i --tty --rm busybox --image=busybox --restart=Never -- /bin/sh
   ```

   然后，在其中执行 `ping` 测试：

   ```bash
   ping <other-pod-ip>
   ```

2. **查看 Calico 网络策略是否影响连通性**：

   如果网络策略过于严格，可能会导致 Pod 之间无法通信。检查集群中的网络策略配置：

   ```bash
   kubectl get networkpolicies --all-namespaces
   ```

   你可以查看是否有策略限制了网络流量，导致容器之间无法正常通信。

### 7. **检查 CNI 插件是否正确配置**

你可以查看 Calico 节点是否正确配置了 CNI 插件，查看日志和配置文件。检查 `/etc/cni/net.d/` 目录中 `10-calico.conflist` 文件中的配置是否正确。

### 8. **验证 Calico IP Pool 配置**

检查 Calico 的 IP 池配置是否与集群初始化时的 CIDR 匹配。你可以使用以下命令查看 IP 池：

```bash
calicoctl get ippool
```

这应该显示与 Kubernetes 网络 CIDR 匹配的 IP 池配置。如果 IP 池配置不正确，Calico 节点可能无法正确分配 Pod IP 地址。

### 9. **检查节点和 Pod 的 IP 路由**

Calico 使用 IP 路由和 BGP 协议来连接不同的节点和 Pod。你可以在节点上检查路由表，查看是否有来自 Calico 的网络路由配置：

```bash
ip route show
```

确保路由表中存在正确的 Calico 路由，以确保节点之间的通信。

------

通过以上步骤，你应该能够诊断 Calico 是否正常运行。如果仍然遇到问题，建议查看 Calico 的官方文档或社区支持，以便获得更多的帮助。