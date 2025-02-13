# **🔍 K8s Node 节点架构详解**

在 Kubernetes（K8s）架构中，**Node（节点）** 负责运行 **Pod**，并提供计算资源（CPU、内存、存储等）。每个 Node 上运行多个核心组件，用于管理 Pod 的生命周期、网络通信和存储挂载。

## **📌 1. Node 主要组件**

| **组件**              | **作用**                                     |
| --------------------- | -------------------------------------------- |
| **Kubelet**           | 负责 Pod 生命周期管理，保证 Pod 按照定义运行 |
| **Kube Proxy**        | 维护集群网络规则，负责 Pod 之间的通信        |
| **Container Runtime** | 负责管理容器的运行（如 Docker、containerd）  |
| **Pod**               | Kubernetes 调度的最小单位，运行具体的应用    |

# **📌 2. Node 详细架构**

💡 **Node 组件在 Linux 服务器上运行，整体架构如下：**

```
lua复制编辑+----------------------------------------------------+
|                      Node                          |
|----------------------------------------------------|
|                     Pod (多个)                     |
|   +--------------------------------------------+   |
|   | Container 1 + Container 2 + ...           |   |
|   +--------------------------------------------+   |
|                                                    |
|----------------------------------------------------|
| Kubelet  | Kube Proxy | Container Runtime (Docker, containerd) |
|----------------------------------------------------|
|            Linux 操作系统（Ubuntu, CentOS）         |
|----------------------------------------------------|
|            物理/虚拟服务器（云计算资源）            |
+----------------------------------------------------+
```

# **📌 3. Kubelet（核心组件）**

💡 **Kubelet 是 Node 上最核心的组件，负责 Pod 的生命周期管理。**

## **3.1 Kubelet 的作用**

- 监听 **API Server**，获取调度到本节点的 Pod
- **启动、监控和管理 Pod**（通过 Container Runtime）
- 负责 **健康检查**（Liveness、Readiness 探针）
- 负责 **日志收集、度量数据上报**（如 Prometheus 监控）
- 通过 **cAdvisor** 采集资源使用情况

## **3.2 Kubelet 工作流程**

1️⃣ 监听 API Server，获取要运行的 **Pod 配置**
2️⃣ 调用 **Container Runtime（如 Docker）** 运行 Pod 内的容器
3️⃣ 持续监控容器健康状态（如 CrashLoopBackOff）
4️⃣ 如果 Pod 崩溃，**自动重启** 容器
5️⃣ 采集 Pod **日志、监控数据**，上报给 API Server

## **3.3 关键命令**

```
sh复制编辑# 查看 Kubelet 运行状态
systemctl status kubelet

# 查看当前 Node 的 Pod
kubectl get pods -o wide
```

# **📌 4. Kube Proxy（网络通信核心）**

💡 **Kube Proxy 负责 Kubernetes 集群的网络管理，确保 Pod 之间可以互相通信。**

## **4.1 作用**

- **维护 Service 访问规则**（iptables、IPVS）
- **管理 Pod 之间的网络连接**
- **支持负载均衡**，保证流量分发到多个 Pod

## **4.2 工作方式**

1️⃣ **监听 API Server**，获取 Service 和 Endpoint 信息
2️⃣ 维护 **iptables 或 IPVS 规则**，确保 Pod 间通信
3️⃣ 实现 **ClusterIP、NodePort、LoadBalancer** 类型的 Service
4️⃣ 负责 **Pod 之间的 DNS 解析**（如 `my-service.default.svc.cluster.local`）

## **4.3 关键命令**

```
sh复制编辑# 查看 kube-proxy 状态
systemctl status kube-proxy

# 查看 Service 规则（iptables）
iptables -t nat -L -n -v
```

------

# **📌 5. Container Runtime（容器运行时）**

💡 **Kubernetes 使用 Container Runtime 来运行和管理容器。**

## **5.1 支持的容器运行时**

- **Docker**（最常用）
- **containerd**（轻量级，性能更优）
- **CRI-O**（专为 K8s 设计的容器运行时）

## **5.2 工作方式**

1️⃣ Kubelet 通过 **CRI（Container Runtime Interface）** 调用容器运行时
2️⃣ 容器运行时拉取 **镜像**，创建并运行容器
3️⃣ 管理容器的 **生命周期**，提供日志、监控等

## **5.3 关键命令**

```
sh复制编辑# 查看当前运行的容器
docker ps

# 进入容器内部
docker exec -it <container_id> sh
```

------

# **📌 6. Pod（Kubernetes 运行单元）**

💡 **Pod 是 Kubernetes 的最小调度单元，包含一个或多个容器。**

## **6.1 组成**

- **一个或多个容器**
- **共享网络（同一 IP 地址）**
- **共享存储卷（Volume）**

## **6.2 Pod 通信**

- Pod 之间通过 **Pod IP** 直接通信
- Pod 访问 Service，通过 **ClusterIP** 或 **NodePort**

## **6.3 关键命令**

```
sh复制编辑# 查看所有 Pod
kubectl get pods -o wide

# 进入 Pod 内部
kubectl exec -it <pod_name> -- sh
```

------

# **📌 7. Node 高可用**

## **7.1 Node 失效检测**

💡 **K8s 通过 Node Controller 监测 Node 状态，默认 5 分钟无响应就判定为 "NotReady"。**

- **心跳检测**：Kubelet 每 10 秒发送心跳到 API Server
- Node 失效策略
  - **5 分钟内无心跳**：标记为 `NotReady`
  - **Pod 仍然运行**：不会立刻删除 Pod
  - **超过 5 分钟**：驱逐（Eviction）Pod，重新调度到其他 Node

```
sh复制编辑# 查看 Node 状态
kubectl get nodes
```

------

# **📌 8. Node 资源限制**

💡 **K8s 允许对 Node 资源进行限制，以防止资源耗尽。**

## **8.1 资源配额**

- **CPU 限制**（使用 `millicores`）
- **内存限制**（Mi、Gi）
- **存储限制**（PV、PVC）

```
yaml复制编辑apiVersion: v1
kind: LimitRange
metadata:
  name: resource-limits
spec:
  limits:
  - type: Container
    max:
      cpu: "2"
      memory: "2Gi"
    min:
      cpu: "100m"
      memory: "128Mi"
```

------

# **📌 9. Node 关键命令**

```
sh复制编辑# 查看 Node 信息
kubectl get nodes -o wide

# 查看 Node 详细信息
kubectl describe node <node_name>

# 查看 Pod 在 Node 上的分布情况
kubectl get pods -o wide

# 标记 Node 为不可调度
kubectl cordon <node_name>

# 恢复 Node 调度能力
kubectl uncordon <node_name>

# 驱逐所有 Pod（Node 维护）
kubectl drain <node_name> --ignore-daemonsets --delete-emptydir-data
```

------

# **📌 10. 总结**

| **组件**              | **作用**                        |
| --------------------- | ------------------------------- |
| **Kubelet**           | 负责 Pod 运行，监控容器状态     |
| **Kube Proxy**        | 负责网络管理，维持 Service 规则 |
| **Container Runtime** | 运行容器（Docker, containerd）  |
| **Pod**               | Kubernetes 运行的最小单元       |

🚀 **Node 作为 Kubernetes 计算资源的核心，承担了容器的运行、网络管理和存储调度，让 K8s 实现强大的容器编排能力！**