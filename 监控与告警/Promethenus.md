# Promethenus

## 简介

Prometheus是一个开源的监控和告警工具，最初由SoundCloud开发并于2012年发布。它是一个独立的应用程序，可以采集、存储和查询系统和应用程序的各种指标数据，并提供了一个强大的查询语言PromQL，以方便用户对数据进行分析和查询。Prometheus支持多种数据源，包括各种应用程序、操作系统、网络设备、数据库等，并提供了许多客户端库和API，以方便用户进行数据采集和处理。

除了数据采集和查询外，Prometheus还支持警报规则的定义和执行，可以通过多种方式进行警报通知，例如电子邮件、Slack等。它还提供了多种图形化和仪表盘工具，以便用户实时地监控和可视化指标数据。

由于其开源和易于扩展的特性，Prometheus已成为云原生应用开发和运维领域中的主流监控工具之一，并得到了广泛的应用和社区支持。

## 安装

### 安装prometheus

下载官网https://prometheus.io/download/

```
export VERSION=2.4.3
curl -LO  https://github.com/prometheus/prometheus/releases/download/v$VERSION/prometheus-$VERSION.darwin-amd64.tar.gz
```

解压配置
tar -xzf prometheus-${VERSION}.darwin-amd64.tar.gz
cd prometheus-${VERSION}.darwin-amd64
在promethes.yml 中配置
Promtheus作为一个时间序列数据库，其采集的数据会以文件的形式存储在本地中，默认的存储路径为data/，因此我们需要先手动创建该目录：
mkdir -p data
用户也可以通过参数--storage.tsdb.path="data/"修改本地数据存储的路径。
启动prometheus服务，其会默认加载当前路径下的prometheus.yaml文件：
./prometheus
启动完成后，可以通过http://localhost:9090访问Prometheus的UI界面

## 安装AlertManager

下载：https://github.com/prometheus/alertmanager/releases
解压：tar xvf alertmanager-$VERSION.darwin-amd64.tar.gz
启动：./alertmanager

### 使用Node Exporter采集主机数据

node_export下载
 https://github.com/prometheus/node_exporter/releases
 curl -OL https://github.com/prometheus/node_exporter/releases/download/v0.15.2/node_exporter-0.15.2.darwin-amd64.tar.gz
tar -xzf node_exporter-0.15.2.darwin-amd64.tar.gz
cd node_exporter-0.15.2.darwin-amd64
./node_exporter

访问http://localhost:9100/可以看到页面

### 任务和实例

通过在prometheus.yml配置文件中，添加如下配置。我们让Prometheus可以从node exporter暴露的服务中获取监控指标数据
scrape_configs:

  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']

### 几个角色

 Prometheus Server：Prometheus Server是核心组件，负责采集、存储和查询指标数据。它定期从各种数据源中获取指标数据，然后将其存储在本地的时间序列数据库中，以供后续查询和分析。Prometheus Server还支持告警规则的定义和执行，并提供了一个Web界面以及API和客户端库等多种方式进行数据查询和操作。

Exporters：Exporters是Prometheus中的另一个重要组件，用于将各种应用程序、服务、系统和网络设备等数据源的指标数据暴露给Prometheus Server。Exporters将这些指标数据转换为Prometheus Server可识别的格式，并通过HTTP接口将其暴露出去。Prometheus Server定期从这些HTTP接口中获取指标数据，并存储在本地的时间序列数据库中。

Pushgateway：Pushgateway是Prometheus中的一种特殊类型的Exporters，用于接收来自短时工作任务或批处理作业等临时任务的指标数据。这些临时任务通常不适合直接作为Exporters暴露指标数据，因此Pushgateway提供了一种简单的方式来推送临时任务的指标数据，而不必直接将其暴露给Prometheus Server。

Alertmanager：Alertmanager是Prometheus中的告警管理器，用于处理来自Prometheus Server的告警通知并采取相应的行动。Alertmanager支持多种告警通知方式，例如电子邮件、Slack、PagerDuty等，并提供了灵活的告警路由和模板等功能，以便用户自定义告警通知和处理流程。

Client Libraries：Prometheus提供了多种语言的客户端库，用于帮助开发人员在其应用程序中暴露指标数据，并将其推送到Prometheus Server或Pushgateway中。这些客户端库可以与各种编程语言和应用程序框架集成，并提供了许多功能，例如指标类型的定义、指标数据的采集和导出、指标数据的缓存和批量处理等。

## PromQL

ProMQL是Prometheus的查询语言，用于查询和分析Prometheus中存储的指标数据。ProMQL基于时间序列数据模型，允许用户使用一种简单而强大的语法来对时间序列数据进行过滤、聚合和计算。

ProMQL查询由一个或多个查询表达式组成，每个表达式由一个或多个指标名称、标签名称和操作符组成。例如，以下是一个简单的ProMQL查询表达式：
up{job="prometheus"} == 1
该表达式表示查询名为"up"的指标，其中标签"job"的值为"prometheus"且其值等于1的时间序列数据。换句话说，这个查询表达式会返回在标签"job"的值为"prometheus"的所有时间序列数据中，其指标值为1的时间序列数据。

ProMQL支持多种操作符，例如基本的算术和逻辑运算符、比较运算符、聚合函数和时间序列函数等。用户可以根据需要组合这些操作符来执行各种复杂的查询和计算。

ProMQL的优点包括语法简单易懂、功能强大、可扩展性好等。它是Prometheus监控系统的重要组成部分，为用户提供了丰富的查询和分析功能，使得用户可以方便地对系统和应用程序的指标数据进行监控和分析。

## 告警

### 简介

告警能力在Prometheus的架构中被划分成两个独立的部分。如下所示，通过在Prometheus中定义AlertRule（告警规则），Prometheus会周期性的对告警规则进行计算，如果满足告警触发条件就会向Alertmanager发送告警信息。
![](vx_images/219513970946906.png =984x)
在Prometheus中一条告警规则主要由以下几部分组成：
告警名称：用户需要为告警规则命名，当然对于命名而言，需要能够直接表达出该告警的主要内容
告警规则：告警规则实际上主要由PromQL进行定义，其实际意义是当表达式（PromQL）查询结果持续多长时间（During）后出发告警
在Prometheus中，还可以通过Group（告警组）对一组相关的告警进行统一定义。当然这些定义都是通过YAML文件来统一管理的。
Alertmanager作为一个独立的组件，负责接收并处理来自Prometheus Server(也可以是其它的客户端程序)的告警信息。Alertmanager可以对这些告警信息进行进一步的处理，比如当接收到大量重复告警时能够消除重复的告警信息，同时对告警信息进行分组并且路由到正确的通知方，Prometheus内置了对邮件，Slack等多种通知方式的支持，同时还支持与Webhook的集成，以支持更多定制化的场景。例如，目前Alertmanager还不支持钉钉，那用户完全可以通过Webhook与钉钉机器人进行集成，从而通过钉钉接收告警信息。同时AlertManager还提供了静默和告警抑制机制来对告警通知行为进行优化。
Alertmanager特性
Alertmanager除了提供基本的告警通知能力以外，还主要提供了如：分组、抑制以及静默等告警特性：

### 自定义告警规则

1、告警规则文件配置位置
告警规则应该在Prometheus的配置文件prometheus.yml中配置。在该文件中，用户可以定义和配置一组告警规则，以监控和报告关键的系统和应用程序指标数据。

例如，以下是一个包含告警规则配置的prometheus.yml文件示例：

```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

rule_files:
  - /path/to/alert.rules
```

在上述示例中，rule_files属性指定了包含告警规则的文件路径。用户可以将告警规则定义在单独的文件中，并在prometheus.yml文件中使用rule_files属性引用这些文件。

例如，在上面的示例中，用户可以创建一个名为alert.rules的文件，并在其中定义和配置所有告警规则。然后，可以将该文件路径添加到prometheus.yml文件的rule_files属性中。

请注意，在添加或更改告警规则后，用户需要重新加载Prometheus配置文件以使更改生效。可以使用kill -HUP <pid>命令重新加载配置文件，其中<pid>是Prometheus进程的PID。
2、告警规则配置实例
在Prometheus中，用户可以使用告警规则自定义和配置告警行为。告警规则是一组基于ProMQL查询语言的规则，用于判断指标数据是否符合告警条件，并触发相应的告警行为。

以下是一些自定义告警规则的示例：

```
groups:
- name: example
  rules:
  - alert: HighErrorRate
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5
    for: 10m
    labels:
      severity: page
    annotations:
      summary: High request latency
      description: description info
```

在告警规则文件中，我们可以将一组相关的规则设置定义在一个group下。在每一个group中我们可以定义多个告警规则(rule)。一条告警规则主要由以下几部分组成：
alert：告警规则的名称。
expr：基于PromQL表达式告警触发条件，用于计算是否有时间序列满足该条件。
for：评估等待时间，可选参数。用于表示只有当触发条件持续一段时间后才发送告警。在等待期间新产生告警的状态为pending。
labels：自定义标签，允许用户指定要附加到告警上的一组附加标签。
annotations：用于指定一组附加信息，比如用于描述告警详细信息的文字等，annotations的内容在告警产生时会一同作为参数发送到Alertmanager。
CPU使用率告警规则：

```
- alert: HighCpuUsage
  expr: 100 * (1 - avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m]))) > 80
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High CPU usage detected"
    description: "Average CPU usage is above 80% for 5 minutes"
```

该规则使用表达式计算CPU使用率，如果5分钟内平均CPU使用率超过80％，则触发名为"HighCpuUsage"的告警。告警的严重性标签设置为"warning"，并提供了一个摘要和详细说明。

内存使用量告警规则：

```
- alert: HighMemoryUsage
  expr: (node_memory_MemTotal_bytes - node_memory_MemFree_bytes - node_memory_Cached_bytes - node_memory_Buffers_bytes) / node_memory_MemTotal_bytes * 100 > 90
  for: 10m
  labels:
    severity: critical
  annotations:
    summary: "High memory usage detected"
    description: "Memory usage is above 90% for 10 minutes"
```

该规则使用表达式计算内存使用率，如果10分钟内内存使用率超过90％，则触发名为"HighMemoryUsage"的告警。告警的严重性标签设置为"critical"，并提供了一个摘要和详细说明。

用户可以根据需要编写和配置各种告警规则，并使用Alertmanager等工具来管理和处理告警通知。这些规则可以帮助用户实时监控系统和应用程序的各种指标数据，并在出现异常情况时及时通知相关人员进行处理。
重启Prometheus后访问Prometheus UI  http://127.0.0.1:9090/rules可以查看当前以加载的规则文件。

## AlertManager介绍

在Alertmanager中通过路由(Route)来定义告警的处理方式。路由是一个基于标签匹配的树状匹配结构。根据接收到告警的标签匹配相应的处理方式。这里将详细介绍路由相关的内容。
Alertmanager主要负责对Prometheus产生的告警进行统一处理，因此在Alertmanager配置中一般会包含以下几个主要部分：
全局配置（global）：用于定义一些全局的公共参数，如全局的SMTP配置，Slack配置等内容；
模板（templates）：用于定义告警通知时的模板，如HTML模板，邮件模板等；
告警路由（route）：根据标签匹配，确定当前告警应该如何处理；
接收人（receivers）：接收人是一个抽象的概念，它可以是一个邮箱也可以是微信，Slack或者Webhook等，接收人一般配合告警路由使用；
抑制规则（inhibit_rules）：合理设置抑制规则可以减少垃圾告警的产生





