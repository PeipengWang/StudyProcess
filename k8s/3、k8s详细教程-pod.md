# 深入Pod

## 1 Pod的配置文件

```
apiVersion: v1 # api 文档版本
kind: Pod  # 资源对象类型，也可以配置为像Deployment、StatefulSet这一类的对象
metadata: # Pod 相关的元数据，用于描述 Pod 的数据
  name: nginx-demo # Pod 的名称
  labels: # 定义 Pod 的标签
    type: app # 自定义 label 标签，名字为 type，值为 app
    test: 1.0.0 # 自定义 label 标签，描述 Pod 版本号
  namespace: 'default' # 命名空间的配置
spec: # 期望 Pod 按照这里面的描述进行创建
  containers: # 对于 Pod 中的容器描述
  - name: nginx # 容器的名称
    image: nginx:1.7.9 # 指定容器的镜像
    imagePullPolicy: IfNotPresent # 镜像拉取策略，指定如果本地有就用本地的，如果没有就拉取远程的
    command: # 指定容器启动时执行的命令
    - nginx
    - -g
    - 'daemon off;' # nginx -g 'daemon off;'
    workingDir: /usr/share/nginx/html # 定义容器启动后的工作目录
    ports:
    - name: http # 端口名称
      containerPort: 80 # 描述容器内要暴露什么端口
      protocol: TCP # 描述该端口是基于哪种协议通信的
    - env: # 环境变量
      name: JVM_OPTS # 环境变量名称
      value: '-Xms128m -Xmx128m' # 环境变量的值
    reousrces:
      requests: # 最少需要多少资源
        cpu: 100m # 限制 cpu 最少使用 0.1 个核心
        memory: 128Mi # 限制内存最少使用 128兆
      limits: # 最多可以用多少资源
        cpu: 200m # 限制 cpu 最多使用 0.2 个核心
        memory: 256Mi # 限制 最多使用 256兆
  restartPolicy: OnFailure # 重启策略，只有失败的情况才会重启
```

## 2 探针

### 2.1 类型

#### 2.1.1 StartupProbe

k8s 1.16 版本新增的探针，用于判断应用程序是否已经启动了。

当配置了 startupProbe 后，会先禁用其他探针，直到 startupProbe 成功后，其他探针才会继续。

作用：由于有时候不能准确预估应用一定是多长时间启动成功，因此配置另外两种方式不方便配置初始化时长来检测，而配置了 statupProbe 后，只有在应用启动成功了，才会执行另外两种探针，可以更加方便的结合使用另外两种探针使用。

```
startupProbe:
 httpGet:
  path: /api/startup
  port: 80
```

#### 2.1.2 LivenessProbe

用于探测容器中的应用是否运行，如果探测失败，kubelet 会根据配置的重启策略进行重启，若没有配置，默认就认为容器启动成功，不会执行重启策略。

```
livenessProbe:
 failureThreshold: 5
 httpGet:
  path: /health
  port: 8080
  scheme: HTTP
 initialDelaySeconds: 60
 periodSeconds: 10
```

#### 2.1.3 ReadinessProbe

用于探测容器内的程序是否健康，它的返回值如果返回 success，那么就认为该容器已经完全启动，并且该容器是可以接收外部流量的。

```
readinessProbe:
  failureThreshold: 3 # 错误次数
  httpGet:
    path: /ready
    port: 8181
    scheme: HTTP
  periodSeconds: 10 # 间隔时间
  successThreshold: 1
  timeoutSeconds: 1
```

### 2.2 探针方式

#### 2.2.1 ExecAction(容器内部执行命令)

在容器内部执行一个命令，如果返回值为 0，则任务容器时健康的。

```
livenessProbe:
 exec:
  command:
   \- cat
   \- /health
```

#### 2.2.2 TCPSocketAction

通过 tcp 连接监测容器内端口是否开放，如果开放则证明该容器健康

```
livenessProbe:
 tcpSocket:
  port: 80
```

#### 2.2.3 HTTPGetAction

生产环境用的较多的方式，发送 HTTP 请求到容器内的应用程序，如果接口返回的状态码在 200~400 之间，则认为容器健康。

```
livenessProbe:
  failureThreshold: 5
  httpGet:
    path: /health
    port: 8080
    scheme: HTTP
    httpHeaders:
      - name: xxx
        value: xxx
```

### 2.3  参数配置

```
initialDelaySeconds: 60 # 初始化时间
timeoutSeconds: 2 # 超时时间
periodSeconds: 5 # 监测间隔时间
successThreshold: 1 # 检查 1 次成功就表示成功
failureThreshold: 2 # 监测失败 2 次就表示失败
```

## 3 Pod生命周期

![image-20240421153344672](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240421153344672.png)

### 3.1 Pod的退出

Endpoint 删除pod的ip地址

Pod编程Terminating状态

执行PreStop指令

### 3.2 PreStop的应用

注册中心下线

数据清理

数据销毁

