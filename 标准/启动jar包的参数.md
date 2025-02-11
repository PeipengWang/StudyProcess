在使用 `java -jar` 命令启动 JAR 包时，可以使用多种类型的参数，下面为你详细介绍：



### 1. JVM 参数

JVM 参数用于配置 Java 虚拟机的运行环境，以调整性能、日志记录等方面。这些参数通常以 `-` 开头，在 `java` 命令和 `-jar` 之间指定。

#### 标准及常见示例

- **内存分配参数**

  

  - `-Xms`：设置 JVM 初始堆内存大小。例如，`-Xms512m` 表示初始堆内存为 512MB。
  - `-Xmx`：设置 JVM 最大堆内存大小。例如，`-Xmx1024m` 表示最大堆内存为 1GB。
  - `-Xss`：设置每个线程的栈大小。例如，`-Xss256k` 表示每个线程栈大小为 256KB。

  

  示例命令：

  ```sh
  java -Xms512m -Xmx1024m -Xss256k -jar yourjarfile.jar
  ```

- **垃圾回收参数**

  - `-XX:+UseG1GC`：启用 G1 垃圾收集器。
  - `-XX:MaxGCPauseMillis=200`：设置垃圾回收的最大停顿时间为 200 毫秒。

  示例命令：

  ```sh
  java -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -jar yourjarfile.jar
  ```

- **日志记录参数**

  - `-XX:+PrintGCDetails`：打印详细的垃圾回收信息。
  - `-XX:+HeapDumpOnOutOfMemoryError`：当发生内存溢出错误时，生成堆转储文件。

  示例命令：

  ```sh
  java -XX:+PrintGCDetails -XX:+HeapDumpOnOutOfMemoryError -jar yourjarfile.jar
  ```

### 2. 系统属性参数

系统属性参数用于设置 Java 程序中的系统属性，以 `-D` 开头，后面跟着 `属性名=属性值` 的形式。这些属性可以在 Java 代码中通过 `System.getProperty()` 方法获取。

#### 标准及示例

示例命令：

```sh
java -Dserver.port=8081 -Dspring.profiles.active=prod -jar yourjarfile.jar
```

在 Java 代码中获取这些属性：

```java
public class Main {
    public static void main(String[] args) {
        String port = System.getProperty("server.port");
        String activeProfile = System.getProperty("spring.profiles.active");
        System.out.println("Server port: " + port);
        System.out.println("Active profile: " + activeProfile);
    }
}
```

### 3. 程序参数

程序参数是传递给 Java 程序 `main` 方法的参数，位于 `-jar` 之后，以空格分隔。这些参数可以在 `main` 方法的 `args` 数组中获取。

#### 标准及示例

示例令：

```sh
java -jar yourjarfile.jar arg1 arg2 arg3
```

在 Java 代码中获取这些参数：

```java
public class Main {
    public static void main(String[] args) {
        for (int i = 0; i < args.length; i++) {
            System.out.println("Argument " + (i + 1) + ": " + args[i]);
        }
    }
}
```

综上所述，`java -jar` 启动 JAR 包时主要有 JVM 参数、系统属性参数和程序参数三种类型，每种参数都有其特定的使用标准和用途。