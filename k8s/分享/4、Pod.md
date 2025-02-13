# **Kubernetes Pod 详解**

在 Kubernetes (K8s) 中，**Pod 是最小的可部署单元**，它封装了一个或多个容器，以及存储、网络等资源。Pod 允许多个容器共享存储和网络，实现高效的分布式应用部署。

------

## **📌 1. Pod 的核心概念**

### **🔹 1.1 什么是 Pod？**

- Pod 是 K8s 的基本调度单位，包含一个或多个容器
- **多个容器共享相同的存储卷、网络和生命周期**
- Pod 运行在 **Node 节点上**，通过 K8s 调度管理

### **🔹 1.2 Pod 的特点**

- **可以运行单个或多个容器**
- **共享网络（IP、端口）**
- **共享存储（Volumes）**
- **生命周期受 K8s 控制**

------

## **📌 2. Pod 的结构**

Pod 主要包含以下部分：

| **组件**               | **作用**                              |
| ---------------------- | ------------------------------------- |
| **Pod Spec**           | 定义 Pod 的配置，如容器、存储、网络等 |
| **Containers**         | Pod 内部运行的一个或多个容器          |
| **Volume**             | 存储挂载，用于数据共享                |
| **Networking**         | Pod 共享同一 IP 地址                  |
| **Labels & Selectors** | 用于标识和调度 Pod                    |

### **📌 2.1 Pod YAML 结构示例**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
  - name: my-container
    image: nginx:latest
    ports:
    - containerPort: 80
  restartPolicy: Always
```

**解析：**

- `metadata`：定义 Pod 名称和标签
- `spec`：指定 Pod 运行的 **容器**、端口、重启策略等
- `containers`：定义容器的 `name` 和 `image`
- `restartPolicy`：设置 Pod 失败后的重启策略

------

## **📌 3. Pod 的类型**

### **🔹 3.1 普通 Pod（Standalone Pod）**

- 直接创建的 Pod，不受 K8s 额外管理
- 适用于临时任务，但不建议在生产环境使用

```sh
kubectl apply -f pod.yaml
kubectl get pods
```

### **🔹 3.2 通过 Controller 管理的 Pod**

| **Pod 类型**      | **描述**                                 |
| ----------------- | ---------------------------------------- |
| **Deployment**    | 适用于无状态应用，支持滚动更新           |
| **StatefulSet**   | 适用于有状态应用（数据库等）             |
| **DaemonSet**     | 在每个 Node 上运行一个 Pod（如日志采集） |
| **Job / CronJob** | 运行一次性任务或定时任务                 |

#### **🔸 Deployment 示例**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

- **replicas: 3** 表示创建 3 个相同的 Pod
- Deployment 负责 **自动重启、扩缩容和滚动更新**

------

## **📌 4. Pod 的网络**

### **🔹 4.1 Pod 的网络特点**

- **每个 Pod 都有一个独立的 IP**
- **同一个 Pod 内的容器共享 IP**
- **Pod 之间通过 Cluster 内部网络通信**
- **需要 Service 进行外部访问**

### **🔹 4.2 Pod 访问方式**

| **方式**          | **说明**                            |
| ----------------- | ----------------------------------- |
| **同一个 Pod 内** | 直接 `localhost:端口号` 访问        |
| **不同 Pod 之间** | 使用 `Pod IP` 访问                  |
| **跨 Node 访问**  | 通过 `ClusterIP Service` 访问       |
| **外部访问**      | 通过 `NodePort / LoadBalancer` 访问 |

#### **🔸 Pod 之间通信**

```sh
# 获取 Pod 的 IP
kubectl get pod -o wide
# 进入 Pod 访问另一个 Pod
kubectl exec -it my-pod -- curl <目标-Pod-IP>:80
```

#### **🔸 通过 Service 访问**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

- 通过 `my-service` 访问 `app=my-app` 的 Pod

------

## **📌 5. Pod 的存储**

### **🔹 5.1 Volume 介绍**

K8s 提供 **持久化存储 Volume**，多个容器可以共享数据。

| **存储类型**              | **说明**                             |
| ------------------------- | ------------------------------------ |
| **emptyDir**              | Pod 运行时的临时存储，Pod 删除后丢失 |
| **hostPath**              | 直接使用宿主机文件                   |
| **PersistentVolume (PV)** | 绑定外部存储                         |
| **ConfigMap / Secret**    | 用于存储配置数据                     |

### **🔹 5.2 Volume 示例**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: my-volume
  volumes:
  - name: my-volume
    emptyDir: {}
```

- `emptyDir` 用于临时存储，Pod 终止时数据丢失

------

## **📌 6. Pod 的生命周期**

Pod 生命周期包括以下状态：

| **状态**      | **说明**                         |
| ------------- | -------------------------------- |
| **Pending**   | Pod 创建中，未调度到 Node        |
| **Running**   | Pod 运行中，至少有一个容器在运行 |
| **Succeeded** | Pod 任务完成，容器退出           |
| **Failed**    | Pod 运行失败                     |
| **Unknown**   | Pod 状态未知                     |

### **🔹 6.1 Pod 生命周期管理**

```sh
# 查看 Pod 状态
kubectl get pods
kubectl describe pod my-pod

# 删除 Pod
kubectl delete pod my-pod
```

------

## **📌 7. Pod 的重启策略**

Pod 的 `restartPolicy` 影响其重启方式：

| **策略**      | **描述**           |
| ------------- | ------------------ |
| **Always**    | 总是重启（默认值） |
| **OnFailure** | 失败时重启         |
| **Never**     | 失败后不重启       |

示例：

```yaml
restartPolicy: Always
```

------

## **📌 8. Pod 调度**

Pod 调度决定其在哪个 Node 上运行，主要受 **资源、标签、Taints/Tolerations 影响**。

| **调度方式**                 | **描述**                        |
| ---------------------------- | ------------------------------- |
| **NodeSelector**             | 选择特定的 Node                 |
| **Affinity / Anti-Affinity** | 高级调度策略                    |
| **Taints / Tolerations**     | 允许特定 Pod 运行在特殊 Node 上 |

### **🔹 NodeSelector 示例**

```yaml
spec:
  nodeSelector:
    disktype: ssd
```

- 该 Pod 只能运行在 `disktype=ssd` 的 Node 上

------

# **🎯 总结**

- **Pod 是 K8s 的最小部署单元**
- **Pod 内部容器共享网络和存储**
- **Pod 通过 Service 进行访问**
- **Pod 受 Deployment / StatefulSet 管理**
- **Pod 有完整的生命周期和重启策略**
- **Pod 通过调度策略控制运行在哪个 Node 上**

🚀 **理解 Pod 是掌握 K8s 的关键！**