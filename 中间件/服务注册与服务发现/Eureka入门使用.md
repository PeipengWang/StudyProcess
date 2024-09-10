Eureka 是 Netflix 提供的一个服务注册与发现的解决方案，它在微服务架构中起着重要的作用，帮助服务实例之间进行动态的服务发现与注册。Eureka 包含两个核心组件：
1. **Eureka Server**：提供服务注册和发现功能。
2. **Eureka Client**：注册到 Eureka Server，并通过它查找其他服务。

以下是一个简单的 Eureka 使用教程，包括如何搭建一个 Eureka Server 和 Eureka Client。

### 1. 搭建 Eureka Server

#### 步骤 1.1：引入依赖
首先，创建一个 Spring Boot 项目并添加 Eureka Server 的依赖。

**Maven 配置**（`pom.xml`）：

```xml
<dependencies>
    <!-- Eureka Server 依赖 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
    <!-- Spring Boot Web 依赖 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR12</version> <!-- 选择一个合适的 Spring Cloud 版本 -->
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

#### 步骤 1.2：启用 Eureka Server
在主类（`EurekaServerApplication`）中添加 `@EnableEurekaServer` 注解，表示这是一个 Eureka Server。

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

#### 步骤 1.3：配置 Eureka Server
在 `application.yml` 或 `application.properties` 中添加 Eureka 的配置。关闭自身的注册行为，因为我们仅需要它作为一个服务注册中心。

**`application.yml`**：

```yaml
server:
  port: 8761  # 设置 Eureka Server 的端口

eureka:
  client:
    register-with-eureka: false  # 该应用本身不作为客户端注册到 Eureka Server
    fetch-registry: false  # 该应用不需要从 Eureka Server 拉取注册表
  server:
    enable-self-preservation: false  # 关闭自我保护机制（可选）
```

#### 步骤 1.4：启动 Eureka Server
运行 `EurekaServerApplication`，然后在浏览器中访问 `http://localhost:8761`，你会看到 Eureka Server 的管理页面。

### 2. 搭建 Eureka Client

#### 步骤 2.1：引入依赖
接着创建一个微服务客户端，并将其注册到 Eureka Server 上。在该项目的 `pom.xml` 文件中引入 Eureka Client 相关的依赖。

**Maven 配置**（`pom.xml`）：

```xml
<dependencies>
    <!-- Eureka Client 依赖 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <!-- Spring Boot Web 依赖 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

#### 步骤 2.2：启用 Eureka Client
在微服务的主类中，添加 `@EnableEurekaClient` 注解，表示该应用将作为 Eureka 客户端，向 Eureka Server 注册。

```java
@SpringBootApplication
@EnableEurekaClient
public class EurekaClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }
}
```

#### 步骤 2.3：配置 Eureka Client
在 `application.yml` 文件中，配置 Eureka Server 地址，并设置微服务的名称。

**`application.yml`**：

```yaml
server:
  port: 8081  # 微服务的端口号

spring:
  application:
    name: my-eureka-client  # 微服务的名称

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/  # 指定 Eureka Server 地址
  instance:
    prefer-ip-address: true  # 使用 IP 地址注册
```

#### 步骤 2.4：启动 Eureka Client
运行 `EurekaClientApplication`，该微服务将注册到 Eureka Server 中。你可以再次访问 `http://localhost:8761`，在 Eureka Server 的管理页面上看到该客户端已经注册成功。

### 3. 服务调用
当多个微服务注册到 Eureka Server 后，Eureka Client 可以通过服务名称调用其他微服务，而无需知道其具体的 IP 和端口。

#### 示例：使用 `RestTemplate` 进行服务调用

在需要调用其他服务的微服务中，定义一个 `RestTemplate` 并通过服务名调用其他服务。

**步骤 3.1：定义 `RestTemplate` Bean**：

```java
@Configuration
public class RestTemplateConfig {

    @LoadBalanced  // 启用负载均衡
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

**步骤 3.2：通过服务名称调用其他服务**：

```java
@RestController
public class MyController {

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/call-service")
    public String callService() {
        // 通过服务名称调用服务，而不是使用具体的 IP 地址或端口
        String response = restTemplate.getForObject("http://my-other-service/endpoint", String.class);
        return response;
    }
}
```

### 4. 总结
1. **Eureka Server** 是服务注册中心，微服务通过 Eureka Client 向其注册。
2. **Eureka Client** 向 Eureka Server 注册并使用服务发现机制调用其他服务。
3. 使用 `RestTemplate` 和 `@LoadBalanced` 注解可以实现通过服务名调用其他微服务。

通过以上步骤，你已经成功搭建了一个基本的 Eureka Server 和 Eureka Client 的服务注册与发现系统。