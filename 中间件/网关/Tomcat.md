# Tomcat与Java Web

## Tomcat的server.xml的配置

```
<Server>
    <Service>
        <Connector />
        <Connector />
        <Engine>
            <Host>
                <Context /><!-- 现在常常使用自动部署，不推荐配置Context元素，Context小节有详细说明 -->
            </Host>
        </Engine>
    </Service>
</Server>
```

server.xml的每一个元素都对应了Tomcat中的一个组件；通过对xml文件中元素的配置，可以实现对Tomcat中各个组件的控制。

### 元素分类

server.xml文件中的元素可以分为以下4类：
（1）顶层元素：<Server>和<Service>
<Server>元素是整个配置文件的根元素，<Service>元素则代表一个Engine元素以及一组与之相连的Connector元素。
（2）连接器：<Connector>
<Connector>代表了外部客户端发送请求到特定Service的接口；同时也是外部客户端从特定Service接收响应的接口。
（3）容器：<Engine><Host><Context>
容器的功能是处理Connector接收进来的请求，并产生相应的响应。Engine、Host和Context都是容器，但它们不是平行的关系，而是父子关系：Engine包含Host，Host包含Context。一个Engine组件可以处理Service中的所有请求，一个Host组件可以处理发向一个特定虚拟主机的所有请求，一个Context组件可以处理一个特定Web应用的所有请求。
（4）内嵌组件：可以内嵌到容器中的组件。实际上，Server、Service、Connector、Engine、Host和Context是最重要的最核心的Tomcat组件，其他组件都可以归为内嵌组件。

## 核心组件

1、Server
Server元素在最顶层，代表整个Tomcat容器，因此它必须是server.xml中唯一一个最外层的元素。一个Server元素中可以有一个或多个Service元素。
在第一部分的例子中，在最外层有一个<Server>元素，shutdown属性表示关闭Server的指令；port属性表示Server接收shutdown指令的端口号，设为-1可以禁掉该端口。
Server的主要任务，就是提供一个接口让客户端能够访问到这个Service集合，同时维护它所包含的所有的Service的声明周期，包括如何初始化、如何结束服务、如何找到客户端要访问的Service。
2、Service
Service的作用，是在Connector和Engine外面包了一层，把它们组装在一起，对外提供服务。一个Service可以包含多个Connector，但是只能包含一个Engine；其中Connector的作用是从客户端接收请求，Engine的作用是处理接收进来的请求。
在第一部分的例子中，Server中包含一个名称为“Catalina”的Service。实际上，Tomcat可以提供多个Service，不同的Service监听不同的端口，后文会有介绍。
3、Connector
Connector的主要功能，是接收连接请求，创建Request和Response对象用于和请求端交换数据；然后分配线程让Engine来处理这个请求，并把产生的Request和Response对象传给Engine。
