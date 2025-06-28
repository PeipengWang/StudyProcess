## 界面操作

安装完成后访问界面：

![image-20250628115931685](https://raw.githubusercontent.com/PeipengWang/picture/master/flink/image-20250628115931685.png)

点击菜单Submit New Job进行提交任务

![image-20250628120050921](https://raw.githubusercontent.com/PeipengWang/picture/master/flink/image-20250628120050921.png)

![image-20250628120126344](https://raw.githubusercontent.com/PeipengWang/picture/master/flink/image-20250628120126344.png)

注意填写运行的主函数和参数，点击Submit开始运行任务：

![image-20250628120250827](https://raw.githubusercontent.com/PeipengWang/picture/master/flink/image-20250628120250827.png)

查看任务执行情况

![image-20250628120933422](https://raw.githubusercontent.com/PeipengWang/picture/master/flink/image-20250628120933422.png)

## 任务代码

### 依赖

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>flinkDemo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <scala.binary.version>2.12</scala.binary.version>
        <flink.version>1.19.0</flink.version>
    </properties>

    <dependencies>
        <!-- Flink Java API -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <!-- Flink streaming core -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients</artifactId>
            <version>${flink.version}</version>
        </dependency>
    </dependencies>
</project>
```

### 代码

```


import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

public class WordCountJob {
    public static void main(String[] args) throws Exception {

        // 1. 创建流处理执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // 2. 接入数据源：从 socket 读取文本
        env.socketTextStream("152.136.246.11", 9999)
            .flatMap(new LineSplitter())
            .keyBy(value -> value.f0)
            .sum(1)
            .print(); // 打印输出结果

        // 3. 启动作业
        env.execute("Flink 1.19.2 WordCount Demo");
    }

    // 词行 → 多个 (word, 1)
    public static class LineSplitter implements FlatMapFunction<String, Tuple2<String, Integer>> {
        @Override
        public void flatMap(String line, Collector<Tuple2<String, Integer>> out) {
            for (String word : line.split(" ")) {
                out.collect(new Tuple2<>(word.trim(), 1));
            }
        }
    }
}

```

## 本地提交jar包方法

## 安装配置参数

