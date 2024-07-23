# SpringBoot的日志输出到ElasticSearch

要将 Spring Boot 应用的日志输出到 Elasticsearch，你可以使用 Logstash 作为中间层，或者直接将日志发送到 Elasticsearch。

下面介绍Logstash方式输出日志

### 使用 Logstash

1. **配置 Logback 以发送日志到 Logstash**

修改 `src/main/resources/logback-spring.xml` 文件，配置 Logback 发送日志到 Logstash：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <springProperty scope="context" name="logstash-host" source="log.logstash-host"/>
    
    <property name="log.path" value="logs"/>
    <property name="log.pattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"/>
    
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
    </appender>
    
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>${logstash-host}</destination>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>
    
    <root level="info">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="LOGSTASH"/>
    </root>
</configuration>

```

在 `application.properties` 或 `application.yml` 中配置 `logstash-host` 属性：

```yml
log.logstash-host=localhost:5044

```

2.**配置 Logstash**

创建 Logstash 配置文件 `logstash.conf`：

```
input {
  tcp {
    port => 5044
    codec => json_lines
  }
}

filter {
  # 添加任何你需要的过滤器
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "spring-boot-logs-%{+YYYY.MM.dd}"
  }
  stdout { codec => rubydebug }
}

```

启动 Logstash：

```
./bin/logstash -f logstash.conf
```

3.**启动 Elasticsearch 和 Kibana**

确保 Elasticsearch 和 Kibana 正在运行。