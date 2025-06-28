![image-20250628094431524](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/image-20250628094431524.png)

## 各个角色的职责说明

| 角色名称          | 英文名称        | 职责                                                   |
| ----------------- | --------------- | ------------------------------------------------------ |
| **客户端**        | Flink Client    | 负责编译和打包用户作业，提交给 JobManager              |
| **作业管理器**    | JobManager      | 核心控制器，负责任务调度、状态管理、容错、checkpoint   |
| **任务管理器**    | TaskManager     | 实际执行任务，每个 TM 有多个 Slot，执行 Operator       |
| **资源管理器**    | ResourceManager | 在 Yarn/K8s 模式中管理资源（与集群系统对接）           |
| **调度器/分发器** | Dispatcher      | 接收 REST API 作业提交请求，启动 JobMaster             |
| **历史服务**      | History Server  | 提供已完成作业的历史查看、归档等功能                   |
| **Web UI**        | Flink Web UI    | 提供作业运行状态、拓扑图、checkpoint、指标等可视化监控 |

###  **Flink Client**

- 提交任务（通过 `flink run` 或程序 `main()`）
- 会生成 JobGraph，发送给 JobManager
- 本身不参与实际执行，提交后可以关闭

------

### 2️⃣ **JobManager**

Flink 的“大脑”，职责包括：

- 接收任务（JobGraph）
- 生成执行图（ExecutionGraph）
- 分配 slot、调度 Task 到 TaskManager
- 管理 checkpoint 与容错机制
- 跟踪任务执行状态，失败自动重启

------

### 3️⃣ **TaskManager**

Flink 的“劳动力”，职责包括：

- 每个 TM 负责执行一个或多个 Operator
- 拥有多个 Slot（可执行并行任务）
- 汇报状态和心跳给 JobManager
- 执行数据传输（Operator 链之间的数据流）

------

### 4️⃣ **Dispatcher（调度器）**

- 接收 REST 请求
- 启动 JobManager（也叫 JobMaster）去处理作业
- 每个作业一个 JobMaster

> ✔ 在 `Session 模式` 和 `Per-Job 模式` 中都会用到 Dispatcher。

------

### 5️⃣ **ResourceManager（可选）**

- 与集群资源系统（如 YARN/K8s）对接
- 申请或释放 TaskManager 资源
- 向 JobManager 提供 Slot 信息

------

### 6️⃣ **Flink Web UI**

- 默认端口：`8081`
- 显示所有作业状态
- 可查看：
  - DAG 拓扑图
  - Operator Metrics
  - Checkpoint 状态
  - Slot 分配情况
  - 日志信息等

## Flink 架构的几个关键点总结

| 点                             | 说明                         |
| ------------------------------ | ---------------------------- |
| JobManager 是协调中心          | 管调度、恢复、checkpoint     |
| TaskManager 是执行单元         | 每个 Slot 执行一个算子链     |
| Operator 是算子单元            | 如 map、filter、window 等    |
| Slot 是最小计算资源单位        | 可以并发运行多个 Task        |
| 每个作业可以有自己的 JobMaster | 用于隔离和独立调度           |
| Flink 架构支持高可用           | JobManager 可 HA；状态可恢复 |