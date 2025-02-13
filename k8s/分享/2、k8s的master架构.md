# **📌 1. API Server（kube-apiserver）**

💡 **Kubernetes 的唯一入口，所有请求都必须经过 API Server 处理。**

## **1.1 作用**

- 负责 **处理 kubectl、Dashboard、REST API 请求**
- 将请求数据存入 **etcd**
- 其他 Master 组件（Scheduler、Controller Manager）都通过 API Server 交互

## **1.2 主要功能**

| **功能**                          | **作用**                         |
| --------------------------------- | -------------------------------- |
| **认证（Authentication）**        | 确保只有合法用户能访问           |
| **授权（Authorization）**         | RBAC 控制用户权限                |
| **访问控制（Admission Control）** | 校验资源请求是否符合策略         |
| **数据存储**                      | 负责和 `etcd` 交互，存储集群状态 |

## **1.3 关键特点**

✅ **高可用支持**：可以部署多个 API Server 实例，并通过 **负载均衡（HAProxy、Keepalived）** 实现高可用。
✅ **扩展性强**：支持自定义资源定义（CRD）和 Webhook 进行扩展。

------

# **📌 2. Scheduler（kube-scheduler）**

💡 **决定 Pod 运行在哪个 Node 上！**

## **2.1 作用**

- 监听 **未被调度的 Pod**（Pending 状态）
- 根据 **调度算法** 选择最佳 Node
- 将调度结果提交给 API Server

## **2.2 调度流程**

1️⃣ **获取所有可用 Node**
2️⃣ **过滤不符合要求的 Node**（如资源不足、亲和性限制）
3️⃣ **打分，选择最优 Node**
4️⃣ **提交调度决策**（API Server 更新 Pod 绑定关系）

## **2.3 影响调度的因素**

| **策略**                                  | **作用**                           |
| ----------------------------------------- | ---------------------------------- |
| **资源需求**                              | CPU、内存等                        |
| **污点（Taints）和容忍度（Tolerations）** | 控制 Pod 能否运行在某些 Node 上    |
| **节点亲和性（Node Affinity）**           | 指定 Pod 运行在特定 Node           |
| **Pod 亲和性/反亲和性**                   | 控制 Pod 是否和其他 Pod 运行在一起 |

## **2.4 自定义调度**

- **调度策略** 可以使用 `Scheduler Configuration` 调整
- **自定义调度器** 通过 `kube-scheduler` 扩展

# **📌 3. Controller Manager（kube-controller-manager）**

💡 **负责 K8s 资源的自动控制，确保集群状态符合预期。**

## **3.1 作用**

- 监听 API Server 资源变化
- 自动修复集群状态，确保资源符合期望
- 负责 **副本管理、节点健康检查、服务发现** 等

## **3.2 主要控制器**

| **控制器**                     | **作用**                                |
| ------------------------------ | --------------------------------------- |
| **Replication Controller**     | 确保 Pod 副本数符合 `replicas`          |
| **Node Controller**            | 监测 Node 是否存活，宕机自动驱逐 Pod    |
| **Endpoint Controller**        | 维护 Service 的 Endpoint（关联 Pod IP） |
| **Namespace Controller**       | 管理 Namespace 生命周期                 |
| **Service Account Controller** | 负责创建 ServiceAccount                 |

## **3.3 运行方式**

**多个 Controller 在一个进程内运行**（kube-controller-manager），通过 **goroutine 并发执行**。

# **📌 4. etcd（分布式存储）**

💡 **K8s 的数据库，存储所有配置信息和状态。**

## **4.1 作用**

- 存储 **所有 K8s 资源对象**（如 Pod、Service、Deployment）
- 作为 **唯一的存储后端**，保证集群数据一致性

## **4.2 etcd 的特点**

✅ **强一致性**：使用 Raft 协议保证数据一致性
✅ **高可用**：多副本同步，防止单点故障
✅ **支持快照**：可恢复历史状态

## **4.3 etcd 高可用架构**

💡 **推荐部署 3~5 个 etcd 节点**

```
+--------------------------+
| etcd 1 (Leader)         |
+--------------------------+
| etcd 2 (Follower)       |
+--------------------------+
| etcd 3 (Follower)       |
+--------------------------+
```

- **Leader 负责处理写入**
- **Follower 负责读取**
- **Raft 算法** 确保数据一致性

## **4.4 重要命令**

```
sh复制编辑# 查看 etcd 中存储的 K8s 数据
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 get / --prefix --keys-only
```

# **📌 5. Master 高可用架构**

Master 组件可以运行在 **多个服务器上**，通过 **负载均衡** 提高可用性。

### **🔥 5.1 高可用架构**

```
------------------------------+
|     HAProxy/Keepalived      |  <--- 负载均衡
+------------------------------+
      |        |        |
+-----+   +-----+   +-----+
| Master1 | Master2 | Master3 |  <--- 运行多个 API Server
+-----+   +-----+   +-----+
      |        |        |
+------------------------------+
|      etcd Cluster（3~5个）  |  <--- 高可用 etcd
+------------------------------+
```

### **🔥 5.2 高可用策略**

✅ **API Server** 运行多个实例，通过 **负载均衡** 分流
✅ **etcd** 采用 **奇数个节点（3~5）** 组成集群，防止数据丢失
✅ **Scheduler 和 Controller Manager** 运行多个实例（Active-Passive 模式）
