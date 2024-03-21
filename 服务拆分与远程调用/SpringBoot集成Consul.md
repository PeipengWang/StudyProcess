# Consule
Consul 是由 HashiCorp 开发的一款软件工具，提供了一组功能，用于服务发现、配置管理和网络基础设施自动化。它旨在帮助组织管理现代分布式和微服务架构系统的复杂性。以下是Consul的一些关键方面和功能：  
服务发现：Consul 允许服务自行注册并以动态和自动化的方式发现其他服务。这在微服务架构中特别重要，因为服务需要定位并与其他服务通信。  
健康检查：Consul 可以对已注册的服务执行健康检查。如果服务变得不健康，Consul 可以自动更新其路由，以避免将流量发送到该服务，直到它再次变得健康。  
键值存储：Consul 包括分布式键值存储，可用于动态配置、特性标志和其他需要在服务之间共享的数据。  
安全通信：Consul 支持安全通信，可用于保护服务之间的通信，确保数据的机密性和完整性。  
Consul 是一种强大的工具，可帮助组织更好地管理其分布式系统和微服务架构中的各种方面。  
学习如何使用Consul需要掌握一些基本概念和实践技巧。以下是学习Consul的步骤：  

## 基本概念与原理  
### consule的角色  
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/fee0c81fa37044db81bdb5a5f7caa412.png)

Consul 有三种主要的角色：开发者 (Dev)、客户端 (Client) 和服务器 (Server)。这些角色用于组织 Consul 集群中的节点，以实现服务发现、健康检查和键值存储等功能。以下是对这三种角色的简要描述：  

开发者角色 (Dev)：  
开发者角色通常用于本地开发和测试环境，而不是生产环境。  
在开发者角色下，Consul代理以开发模式启动，不需要连接到其他Consul节点。这使得它们能够在单独的节点上运行，用于本地服务发现和开发目的。  
开发者角色不适用于构建生产用的Consul集群，因为它们不具备高可用性和冗余。  
客户端角色 (Client)： 代理，接受http或者DNS请求信息，转发给server  
客户端角色是Consul集群中的节点之一，它们负责与其他节点通信，并可以用于执行DNS或HTTP API查询。  
客户端节点向服务器节点发送查询请求，以查找和发现服务。它们通常部署在应用程序服务器上，以便应用程序可以利用Consul的服务发现功能。  
客户端节点不具备存储集群数据的功能，但可以将查询请求路由到服务器节点来获取有关服务的信息。  
服务器角色 (Server)：3-5个  
服务器角色是Consul集群的核心，负责存储集群的状态信息，执行健康检查，并协调服务注册和发现。  
服务器节点通常以多节点的方式部署，以确保高可用性和冗余。这些节点组成了Consul集群的核心，保持一致的状态信息。  
服务器节点还可以配置数据中心之间的复制和通信，以实现全局的服务发现和协调。  
### 原理  
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/4302c5f9eae74b7eb6371946c188408a.png)

了解什么是服务发现、健康检查和分布式键值存储，这是Consul的核心概念。  
服务发现及注册：  
producer启动时，会将自己的ip/host等信息通过发送请求告知consul,consul接收到producer的注册信息后，每隔10秒（默认）会向producer发送一个健康检查的请求，检查producer是否处于可用状态，防止consumer调到不可用的服务；
服务调用  
当consumer请求product时，会先从consul中拿存储的producer服务的ip和port的临时表（temp table),从表中任选一个producer的ip和port,然后根据这个ip和port,发送访问请求;此表只包含通过健康检查的producer信息，并且每隔10秒更新。  


### 服务发现
服务发现是指在分布式系统或微服务架构中，自动地发现并识别可用的服务实例。这是因为在这样的环境中，服务实例的数量和位置通常是动态的。  
服务发现系统允许服务注册它们自己，以及查询已注册的其他服务。这有助于应用程序找到其依赖的服务，并确保它们可以相互通信。  

### 健康检查
健康检查是一种机制，用于监视服务的状态和可用性。服务可以定期进行自我检查，并向服务发现系统报告它们的健康状态。  
如果服务不再健康，它可以从服务发现系统中注销，从而不再接收流量。这有助于系统自动处理故障，并确保客户端不会请求到不正常的服务。  
### 分布式键值存储  
分布式键值存储是一种数据存储系统，允许应用程序将键值对存储在一个分布式的、可扩展的数据存储中。  
这种数据存储通常具有高可用性，并可用于存储配置数据、应用程序状态、特性标志等信息。分布式键值存储还可以用于协调和同步分布式系统的操作。  
### 代理（Agent）
代理是Consul的核心组件之一，运行在每个部署了Consul的节点上。  
代理负责与其他节点通信，定期报告节点的健康状态，执行本地健康检查，以及协调服务注册和发现。  
代理还可以执行DNS或HTTP API查询，以便其他应用程序可以查找和访问已注册的服务。  
### 数据中心（Datacenter）
数据中心是一个逻辑隔离的单元，通常用于表示不同的部署区域或环境。  
在一个大型的分布式系统中，可能有多个数据中心，每个数据中心包含一组Consul代理。  
数据中心之间可以相互通信，但它们通常是独立的，可以有不同的配置和健康检查策略。  
### 服务注册（Service Registration）  
服务注册是Consul的一个功能，允许应用程序注册自己的服务实例，以便其他应用程序可以发现并与之通信。  
当一个服务启动时，它会通过Consul代理向Consul集群注册自己的服务信息，包括服务名称、IP地址、端口等。  
注册的服务信息存储在Consul的目录中，供其他应用程序查询。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/4c604bbe3ec0427496352b20fb6e0b37.png)

## 安装Consul  
下载和安装Consul，可以选择适合你操作系统的版本。   
官网：[consul](https://www.consul.io/)  
### windows（dev方式）  
目录中会得到一个consul.exe文件  
cd到对应的目录下，使用cmd启动consul:
#-dev表示开发模式运行，另外还有-server表示服务模式运行  
```
consul agent -dev -client=0.0.0.0
```
为了方便启动，也可以在consul.exe同级目录下创建一个脚本(xx.bat)来启动，脚本内容如下：  
```
consul agent -dev -client=0.0.0.00
pause
```
访问管理后台:http://localhost:8500,能看到正常界面即表示consul服务启动成功了  
tips:-client=0.0.0.0表示允许所有ip访问  
### linux
环境准备  
服务器ip             consul类型    node节点   系统  
192.168.10.101  server           server-01   centos  
192.168.10.102  server           server-02   centos  
192.168.10.103  server           server-03   centos  
192.168.10.1      client            client-01    windows  
安装  
```
yum -y install unzip
mkcir -p /usr/local/consul
unzip consul_1.7.0_linux_amd64.zip -d /usr/local/consul/
```
consul数据目录  
```
mkdir -p /usr/local/consul/data
```
启动  
注册中心服务端：  
分别在3台server上(/usr/local/consul目录)执行以下命令（-bind及-node改为对应的即可）  
```
./consul agent -server -bind=192.168.10.101 -client=0.0.0.0 -ui -bootstrap-expect=3 -data-dir=/usr/local/consul/data/ -node=server-01
```
3台server上执行以下命令：指定101为主节点  
```
./consul join 192.168.10.101
```
参数含义如下：  
-server:以服务端身份启动（注册中心）  
-bind:表示绑定以哪个ip  
-client:指定客户端访问的ip,0.0.0.0不限制客户端ip  
注册中心服务端：  
在windows下的consul.exe所在目录中执行，-data-dir对应目录先建好  
```
consul agent -client=0.0.0.0 -bind=192.168.10.1 -data-dir=D:\consul\data -node=client-01
consul join 192.168.10.101
```
linux/windows上查看集群状态  
```
./consul members  
访问  
```
访问任务一个server的8500端口即可  
如：http://192.168.0.101:8500  
### docker  
这是在window下部署consul集群的yaml文件  
加红部分：  
目录先建好，网络名改为自己的即可  
主要参数说明：  
参数名	解释  
- **-server:** 设置为 Server 类型节点，不加则为 Client 类型节点  
- **-client:** 注册或者查询等一系列客户端对它操作的 IP，默认是127.0.0.1  
- **-bootstrap-expect:** 集群期望的 Server 节点数，只有达到这个值才会选举 Leader  
- **-node:** 指定节点名称  
- **-data-dir:** 数据存放位置  
- **-retry-join:** 指定要加入的节点地址（组建集群）  
- **-ui:** 启用 UI 界面将这个转化为Markdown格式的内容  

```
version: '3.4'
services:
  consul-server1:
    image: consul
    command: agent -server -client=0.0.0.0 -bootstrap-expect=3 -node=consul-server1 -data-dir=/data
    volumes:
      - F:/consul/data/server1:/data
  consul-server2:
    image: consul
    command: agent -server -client=0.0.0.0 -retry-join=consul-server1 -node=consul-server2 -data-dir=/data
    volumes:
      - F:/consul/data/server2:/data
    depends_on:
      - consul-server1
  consul-server3:
    image: consul
    command: agent -server -client=0.0.0.0 -retry-join=consul-server1 -node=consul-server3 -data-dir=/data
    volumes:
      - F:/consul/data/server3:/data
    depends_on:
      - consul-server1
  consul-clicent1:
    image: consul
    command: agent -client=0.0.0.0 -retry-join=consul-server1 -ui -node=consul-client1 -data-dir=/data
    ports:
      - 8500:8500
    volumes:
      - F:/consul/data/client1:/data
    depends_on:
      - consul-server2
      - consul-server3
networks:
  default:
    name: hurong_www_gitee_127.0.0.1 #使用laravel项目中生成的网络
    external: true #使用已存在的网络
    #driver: bridge
```

## 编写一个简单的应用程序，并将其配置为注册到Consul

### 引入父依赖
新建consule-demo
```
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <version>2.2.4.RELEASE</version>
        </dependency>
    </dependencies>
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <spring-cloud-version>Hoxton.SR1</spring-cloud-version>
    </properties>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR10</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

```
### 新建module，service-provider
依赖
```
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

	<parent>
		<artifactId>consul-demo</artifactId>
		<groupId>org.example</groupId>
		<version>1.0-SNAPSHOT</version>
	</parent>
	<name>service-provider</name>
	<description>服务提供者</description>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-consul-discovery</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
			<version>2.2.4.RELEASE</version>

		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
			<version>2.2.4.RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<version>RELEASE</version>
			<scope>compile</scope>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<version>1.18.30</version>
		</dependency>
	</dependencies>
</project>

```
引入配置application.yml
```
server:
  port: 7070
spring:
  application:
    name: service-provider
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        register: true
        instance-id: ${spring.application.name}-01
        service-name: ${spring.application.name}
        port: ${server.port}
        prefer-ip-address: true
        ip-address: localhost

```
实体类Product：
```
package org.example.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Product implements Serializable {
    private Integer id;
    private String produceName;
    private Integer produceNum;
    private Double producePrice;
}

```
服务层ProductService
```
package org.example.service;

import org.example.entity.Product;

import java.util.List;

public interface ProductService {
    public List<Product> selectProductList();
}

```
```
package org.example.service.impl;

import org.example.entity.Product;
import org.example.service.ProductService;
import org.springframework.stereotype.Service;

import java.util.Arrays;
import java.util.List;

@Service
public class ProductServiceImpl implements ProductService {
    @Override
    public List<Product> selectProductList() {
        System.out.println("query product");
        return Arrays.asList(
                new Product(1,"小米",1,5100D),
                new Product(2,"中兴",12,5100D)
        );
    }
}

```
Controller层
```
package org.example.controller;

import org.example.entity.Product;
import org.example.service.ProductService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping(value = "/product")
public class ProductController {
    @Autowired
    ProductService productService;

    @GetMapping(value = "/list")
    public List<Product> selectProduceList(){
        return productService.selectProductList();
    }
}

```
启动文件ProviderApp
```
package org.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ProviderApp {
    public static void main(String[] args) {
        SpringApplication.run(ProviderApp.class, args);
    }
}

```
访问localhost:7070/product/list即可
### 新建module，service-consumer

依赖与service-provider一样，配置修改接口和register（是否注册）
```
server:
  port: 9090
spring:
  application:
    name: service-consumer
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        register: false
        instance-id: ${spring.application.name}-01
        service-name: ${spring.application.name}
        port: ${server.port}
        prefer-ip-address: true
        ip-address: localhost

```
实体类，Product，复制上面
Order
```
package org.example.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Order {
    private Integer id;
    private String orderNo;
    private String orderAddress;
    private Double totalPrice;
    private List<Product>  productList;
}

```
服务层
```
package org.example.service;

import org.example.entity.Order;

public interface OrderService {
    Order selectOnOrderById(Integer id);
}

```
```
package org.example.service.impl;

import org.example.entity.Order;
import org.example.entity.Product;
import org.example.service.OrderService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.ParameterizedTypeReference;
import org.springframework.http.HttpMethod;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

import java.util.List;

@Service
public class OrderServiceImpl implements OrderService {

    @Autowired
    RestTemplate restTemplate;

    @Override
    public Order selectOnOrderById(Integer id) {
        System.out.println("get order ");
        return new Order(id, "order-001", "China", 22788D, selectProductListById(id));
    }

    private List<Product> selectProductListById(Integer id) {
        ResponseEntity<List<Product>> response = restTemplate.exchange("http://service-provider/product/list",
                HttpMethod.GET, null, new ParameterizedTypeReference<List<Product>>() {
                });

        System.out.println(response.getBody());
        return response.getBody();
    }
}

```
Controller层
```
package org.example.controller;

import org.example.entity.Order;
import org.example.service.OrderService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping(value = "/order")
public class OrderController {
    @Autowired
    private OrderService orderService;


    @RequestMapping(value = "/{id}")
    public Order getOrder(@PathVariable("id") Integer id){
        return orderService.selectOnOrderById(id);
    }
}

```
启动，注意RestTemplate要注册为bean
```
package org.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class ConsumerApp {


    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApp.class, args);
    }
}

```
访问localhost:9090/order/1即可得到数据  


其他还需要学习：  
使用Consul API或CLI工具来查询注册的服务，了解如何发现其他服务。  
健康检查：  
设置健康检查，以确保注册的服务保持健康状态。  
实验并了解Consul如何自动处理不健康的服务。  
分布式键值存储：  
使用Consul的键值存储功能，存储和检索配置数据或其他键值信息。  
熟悉Consul的API和CLI命令，以便与键值存储进行交互。  
安全和认证：  
学习如何配置Consul以实现安全的通信，例如使用TLS。  
了解Consul的访问控制和身份验证机制。  
集成到实际应用中：  
将Consul集成到你的应用架构中，以实现服务发现和配置管理。  
编写脚本或使用现有工具，以自动化Consul的配置和运维任务。  
学习资源：  
阅读Consul的官方文档，这是学习的重要资源。  
参与社区和论坛，与其他使用Consul的人交流经验。  
实践项目：  
探索更复杂的用例，以更深入地理解Consul的功能。  
学习Consul可能需要一些时间，但掌握这些基本概念和技能将有助于你更好地管理分布式系统和微服务架构中的服务发现和配置管理。不断实践和探索是提高你的Consul技能的关键。  







