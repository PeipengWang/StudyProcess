
# k8s
## 基本结构
两台服务器
一个 Master
一个 Node 里面有多个pod，每个pod下面有多个容器

服务之前进行内部可以直接访问
外部访问服务需要ingress

## 服务的分类--有状态服务与无状态服务

例如有个服务Nginx来做反向代理
反向代理进行路由转发请求到后端服务
两个nginx服务路由到同一个后端服务

但是如果要进行持久化，例如redis
两个redis服务，一个请求过来进行了缓存数据持久化，此时两台redis机器数据是不一致的。

无状态应用
不会对本地环境产生任何依赖，例如不会储存数据到本地
例如：
* Web服务器（如Nginx、Apache）
* 应用程序服务器（如Tomcat、Node.js）
* 微服务应用程序的中间层

有状态应用
会对本地环境产生依赖，例如存储数据到本地
例如：
* 数据库服务（如MySQL、PostgreSQL）
* 缓存服务（如Redis）
* 分布式存储系统（如Cassandra、MongoDB）

在Kubernetes中处理状态一致性的问题：
对于需要保持状态一致性的有状态服务，Kubernetes提供了一些解决方案，例如：

* StatefulSets：用于部署有状态服务的控制器，它保证每个Pod有唯一的网络标识符和稳定的持久化存储。
* PersistentVolumes 和 PersistentVolumeClaims：用于将持久化存储卷动态地分配给Pod，确保状态数据的持久性和一致性。

Redis服务，确保两个Redis实例的数据一致性可以通过Redis的复制（Replication）功能来实现。通过在Kubernetes中正确配置Redis的主从复制，可以确保数据在两个Redis实例之间保持同步。

## 资源和对象
资源清单（Resource Manifest）通常指的是用于描述和配置Kubernetes资源的YAML文件。这些清单文件包含了对于Kubernetes中各种资源（如Pod、Service、Deployment等）的定义和配置信息。以下是一些常见的Kubernetes资源清单：
1、Pod清单：描述一个或多个容器的运行规范，包括容器的镜像、命令、参数、环境变量等信息。
2、Service清单：描述服务的规范，包括服务的类型、端口映射、选择器等信息。
3、Deployment清单：描述部署应用程序的规范，包括副本数、更新策略、容器模板等信息
4、PersistentVolume清单 和 PersistentVolumeClaim清单：描述持久化存储的规范，包括存储类型、访问模式、容量等信息。
5、Ingress清单：描述Ingress资源的规范，包括主机名、路径、后端服务等信息。

基于资源创建出来的实例就叫对象
它们可以通过YAML或JSON格式的清单文件来定义，并通过Kubernetes API进行创建、管理和操作。通过使用这些对象，你可以轻松地在Kubernetes集群中部署和管理应用程序，实现高可用、可扩展和自动化的容器化工作负载。

## 对象的规约和状态
在Kubernetes中，规约（Desired State）和状态（Actual State）是两个关键概念，它们描述了Kubernetes系统如何管理和维护应用程序和基础设施的状态。
**规约（Desired State）**：
* 规约是你期望系统达到的状态或行为。
* 在Kubernetes中，你通过定义资源清单来指定所需的规约，如Pod、Service、Deployment等。
* 例如，你可以定义一个Deployment规约，指定你希望有3个Pod副本运行，每个Pod使用特定的镜像和配置。
**状态（Actual State）**：
* 状态是系统当前的实际状态或行为。
* 在Kubernetes中，Kubernetes控制器负责监视资源的状态，并确保它们与规约匹配。
* 例如，如果根据Deployment的规约应该有3个Pod运行，但实际上只有2个Pod在运行，那么Kubernetes会自动调度新的Pod来满足规约。
* Kubernetes通过不断地比较规约和状态，并自动调节系统状态，以确保系统处于所期望的状态。这种自动化的管理方式使得Kubernetes能够实现高可用性、自愈性和自动化，使得应用程序和基础设施更加健壮和可靠。

规约描述了你想要的系统状态，而状态描述了系统当前的实际状态，Kubernetes通过不断地比较规约和状态，自动调整系统状态，以满足你的期望

## 资源的分类
元空间、集群、命名空间
**集群（Cluster-wide）**：
* 集群范围内的资源包括元空间和命名空间中的资源，它们可以在整个集群中访问和使用。
* 这些资源通常是用于配置和管理整个集群的全局对象，不受命名空间的限制。
* 例如，ClusterRole、ClusterRoleBinding、PersistentVolume、StorageClass 等。

**命名空间（Namespace-level）**：
* 命名空间中的资源是与特定命名空间关联的资源，它们仅对该命名空间中的对象可见。
* 命名空间提供了一种逻辑隔离的机制，可以将集群中的资源划分为多个逻辑分组。
* 在不同的命名空间中可以存在同名的资源，但它们不会相互干扰。
* 例如，Pod、Service、Deployment、ConfigMap、Secret 等。

一个pod可能运行在命名空间A也可以运行在命名空间B里，命名空间A里的资源不到命名空间B里的资源，一个资源只能在自己的命名空间里使用，实现了资源的隔离。

**元空间（Cluster-level）**：
* 元空间中的资源是集群范围内的资源，它们不与特定的命名空间关联。
* 这些资源通常是对整个集群的配置和管理，而不是针对特定的应用程序或工作负载。
* 例如，Node、PersistentVolume、ClusterRole、ClusterRoleBinding 等。


元空间是一个公开的资源，所有服务是都可以访问的，相当于运行在集群之外的一个资源




## 命名空间级别的资源
### pod的概念
pod的作用是为容器提供了一层pause，能够实现容器的网络、文件等资源的共享  
pod是一个容器组  
k8s里所有的操作都是部署一个个的pod，一个pod可以理解为一个应用，一个pod里可以运行多个容器或者一个容器，运行多个容器时一般情况下这几个容器时无法松耦合的，尽量不要讲可以独立运行的的容器放到一个pod里。  

属性：replication副本数目  
控制器：  
   适用于无状态服务  
         ReplicationController（RC）：已经被废除掉了，需要对pod进行绑定    
         ReplicaSet（RS）：替代RC，加入了label和selector，可以通过selector来选择对哪些Pod生效   
              * ReplicaSet 用于确保指定数量的 Pod 副本运行在集群中。   
              * 当 Pod 的副本数量少于指定的副本数量时，ReplicaSet 会自动创建新的 Pod 来满足要求。   
              * 当 Pod 的副本数量多于指定的副本数量时，ReplicaSet 会自动删除多余的 Pod。   
          Deployment：这是实际工作中用的，对RS做了二次封装增强，可以实现滚动更新、回滚、暂停和恢复等功能，以确    保应用程序的无缝更新和运行。   
                * 滚动升级/回滚：可以在不中断服务的情况下逐步更新应用程序的版本。你可以定义新版本的 Pod 模板，并指定更新策略和更新速度，Deployment 会自动完成更新过程。在滚动升级过程中可以会运行一个新的，停止一个旧的。  
   * 适用于有状态的服务  
          Statefulset：专门用来进行有状态服务应用部署的工具，为每个pod分配一个具有稳定网络标识和持久化存储，保证Pod的唯一性和稳定性。  
          statefulset需要保证的几个方面：网络，文件系统  
                 Handless Service：用于定义网络标志（DNS domain），格式为stateSetName-[0-n-1]。将服务名当成一个访问路径，直接暴露在DNS中，而不需要分配Cluster IP  
                 VolumeClaim Template： 用于创建持久化卷的模板，以便为每个 Pod 分配独立的持久化存储。  
      守护进程  
        DaemonSet控制器：确保k8s的每个节点都运行一个副本。  
        用守护进程监控应用，例如日志收集、系统监控、系统程序等  
        日志收集： Fluentd、 Elasticsearch、Logstash  
        系统监控：Prometheus Node Exporter、New Relic agent、Ganglia gmonnd等  
        系统程序：Kube-proxy、kube-dns  
      任务/定时任务  
        Job：一次性任务，运行一次就销毁  
        CronJob：周期任务，在Job基础上加了定时器  
### 服务发现  
Service和Ingress  
Service 是集群内部访问的抽象，用svc标识  
**内部服务暴露**：
* Service 用于将应用程序暴露给集群内部的其他 Pod 或外部用户，通常是通过 Cluster IP 或 NodePort 来提供服务。  
**四层负载均衡**：  
* Service 是一个四层（网络层）的负载均衡器，它通过对流量进行转发来将请求路由到后端 Pod。它基于 IP 和端口级别进行负载均衡。  
**集群内部服务发现**：  
* Service 具有服务发现的功能，可以通过 DNS 或环境变量的方式让其他 Pod 发现和访问服务。  
**直接暴露到集群内部**：  
* Service 通常用于将应用程序直接暴露到集群内部，例如前端与后端服务之间的通信。  

**Ingress**

**外部服务暴露**：  
* Ingress 用于将应用程序暴露给集群外部的用户，通常是通过公共 IP 和域名来提供服务。  
**七层负载均衡**：  
* Ingress 是一个七层（应用层）的负载均衡器，它能够基于 HTTP/HTTPS 协议的域名和路径来进行请求的路由和转发。  
**多域名和路径支持**：  
* Ingress 支持多个域名和路径，并能够根据不同的域名和路径将请求路由到不同的后端服务。  
**HTTPS/TLS 支持**：  
* Ingress 可以配置 HTTPS/TLS 加密，以确保传输的安全性。  
**灵活的路由规则**：  
* Ingress 具有更灵活的路由规则，可以根据请求的域名、路径、头部等信息进行更细粒度的路由控制。  

通过Ingress提供外部访问接口让用户访问，服务内部通过service进行访问，这样就能完成一个完整的功能。  

## 配置与存储  
### 配置：  
**Volume**：挂载卷  
Volume 在 Kubernetes 中用于挂载存储到 Pod 中的文件系统。CSI Volume 是一种特殊类型的 Volume，它允许使用 CSI 插件动态地创建和管理持久卷。  
**ConfigMap **：  
用于存储非敏感的配置数据，如键值对、属性文件、JSON 或 YAML 等格式的配置信息。ConfigMap 提供了一种机制，可以将配置数据注入到 Pod 的容器环境变量、命令行参数或挂载的卷中，以供应用程序使用。  
如果需要修改一些应用的配置文件，之前需要逐个修改，但是现在可以直接修改这个ConfigMap就可以直接修改应用的配置文件。  
**Secret**：  
跟ConfigMap一模一样，加了一个功能是加密，存储敏感数据。  
严格意义上用的是base64编码，并不是加密。意义不是很大。  
**DownwardAPI**：  
Downward API 是 Kubernetes 中的一种功能，用于向 Pod 中注入一些有关 Pod 自身的信息，例如 Pod 名称、命名空间、标签、注释等。通过 Downward API，这些信息可以以环境变量的形式注入到容器中，或者通过卷的方式挂载到容器的文件系统中，以供应用程序使用。  
有两种方式：  
1、环境变量注入  
通过 Downward API，Pod 中的信息可以以环境变量的形式注入到容器中，供应用程序使用。  
2、卷挂载  
Downward API 也支持将 Pod 中的信息挂载为卷，以文件的形式存储在容器的文件系统中，供应用程序读取。  


### 存储
1、PersistentVolume (PV) 和 PersistentVolumeClaim (PVC)：  
PV 和 PVC 仍然是 Kubernetes 中用于表示持久化存储资源和声明请求这些资源的对象。CSI 插件可以与 PV 和 PVC 结合使用，以实现对第三方存储提供商的集成。  
2、StorageClass：  
StorageClass 仍然是 Kubernetes 中用于动态创建 PV 的资源对象。CSI 插件可以定义自己的 StorageClass，以描述持久卷的属性和存储后端的配置信息。  
3、CSI Volume Snapshot：  
CSI Volume Snapshot 是 Kubernetes 中用于创建和管理持久卷快照的机制。它允许 CSI 插件实现快照功能，以便用户在数据备份和恢复时更加灵活和高效。  

## 其他
Role与RoleBinding  
Role 是一种用于定义对资源的操作权限的对象。Role 允许你在命名空间级别指定对资源的操作权限，并通过 RoleBinding 将 Role 绑定到特定的用户、组或服务账号上，从而控制他们对资源的访问权限。  
以下是关于 Kubernetes 中 Role 的一些关键信息：  

1. 权限定义  
2. 命名空间级别  
3. 资源类型  
4. 角色绑定  
5. 范围  

RoleBinding 是一种用于将 Role 或 ClusterRole 绑定到特定用户、组或服务账号的对象。通过 RoleBinding，你可以赋予指定的用户或实体在特定命名空间或整个集群范围内的权限，从而控制他们对资源的访问权限。  

以下是关于 Kubernetes 中 RoleBinding 的一些关键信息：  

1. 绑定 Role 或 ClusterRole
2.  范围
3. 角色和角色绑定
4. 创建方式

