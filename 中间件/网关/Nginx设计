# Nginx底层设计

## 介绍

Nginx（发音为“engine-x”）是一个免费、开源、高性能的Web服务器和反向代理软件。它被开发用于处理高流量网站并作为负载均衡器、内容缓存和Web服务器。
Nginx旨在轻巧高效，专注于可伸缩性和性能。它采用事件驱动、异步、非阻塞的架构，能高效地同时处理多个连接。这使它成为高流量网站和应用程序的热门选择。
Nginx可用于提供静态文件，也可将请求代理到其他Web服务器，如Apache或Tomcat。它还可以用作负载均衡器，将传入流量分布到多个服务器上。Nginx还支持SSL/TLS加密，可配置为提供安全的HTTPS连接。
Nginx高度可配置，具有模块化架构，使开发人员可以通过添加第三方模块来扩展其功能。其配置文件使用简单的语法，易于阅读，这使得管理员更容易配置和维护。
Nginx适用于Linux、Unix、Windows和macOS操作系统。它被广泛用于流行的网站，包括Netflix、Dropbox、WordPress和GitHub等。

## 进程模式

Nginx可以分为单进程模式和多进程模式，进程运行可分为daemon模式和非damen模式。
Nginx支持两种进程模式：单进程（master-worker）模式和多进程（多工作进程）模式。
在单进程模式下，Nginx只有一个主进程（master process），主进程管理着多个工作进程（worker process），每个工作进程处理客户端请求和响应。主进程负责读取配置文件、启动和停止工作进程，并且在必要时重新启动工作进程。这种模式通常用于处理低流量的网站或服务。
在多进程模式下，Nginx启动多个工作进程来处理客户端请求和响应。每个工作进程都是独立的，它们可以并行地处理客户端请求，从而提高了并发性能。多进程模式通常用于处理高流量的网站或服务。
在多进程模式下，还可以使用多种策略来管理工作进程，例如round-robin、IP hash、least connections等等。这些策略可以根据不同的负载均衡需求来选择最优的工作进程。
无论是单进程模式还是多进程模式，Nginx都采用非阻塞I/O模型，能够高效地处理并发连接和请求。这使得Nginx成为高性能Web服务器和反向代理软件的热门选择。

### linux系统的damen模式

在计算机操作系统中，Daemon是指一种长期运行的后台进程或服务，通常在系统启动时自动启动，并在系统运行期间一直运行，直到系统关闭或服务停止。
Daemon进程通常不与用户直接交互，它们在后台默默地运行，处理各种任务和服务请求，例如网络服务、数据处理、系统监控、定时任务等等。
Daemon进程通常以daemon进程模式运行，这种模式下，它们会脱离终端控制，并将自己的进程组ID设置为一个新的会话组ID，这样可以避免终端中断信号（SIGHUP）的影响，并使得它们可以在后台持续运行。
在Linux系统中，可以使用nohup命令将一个进程设置为daemon进程模式，并将其输出重定向到一个日志文件中，以便后续查看和管理。例如，下面的命令将一个进程（PID为12345）设置为daemon进程模式，并将其输出重定向到log.txt文件中：
nohup command > log.txt 2>&1 &
在Web服务器中，daemon进程通常用于后台服务和任务，例如Nginx等Web服务器就支持以daemon进程模式运行。它们在后台持续监听和处理客户端请求，并提供高性能和高可靠性的Web服务。
接下来详细介绍nginx的damen模式

## Nginx的damen模式

Nginx的daemon模式可以使其在后台持续运行，提供高性能和高可靠性的Web服务，并且可以通过配置文件和命令行参数进行灵活的配置和管理。
在daemon模式下，Nginx进程会脱离终端控制，并以后台服务的方式运行，持续监听和处理客户端请求，并提供高性能和高可靠性的Web服务。
Nginx支持两种daemon模式：普通daemon模式和master-worker daemon模式。
在普通daemon模式下，Nginx进程会在启动时脱离终端控制，并且将自己的进程ID（PID）写入到指定的PID文件中。在后续运行中，Nginx会持续监听和处理客户端请求，并将日志输出到指定的日志文件中。可以使用以下命令将Nginx以普通daemon模式运行：

```
nginx -c /path/to/nginx.conf -p /path/to/nginx/dir -g "daemon on;"
```

其中
-c参数指定Nginx的配置文件路径
-p参数指定Nginx的运行目录
-g参数指定Nginx的全局配置。
daemon on表示启用daemon模式。

在master-worker daemon模式下，Nginx进程会在启动时创建一个master进程和多个worker进程。master进程负责读取配置文件、启动和停止worker进程，并在必要时重新启动worker进程。worker进程负责处理客户端请求和响应。这种模式通常用于处理高流量的网站或服务。可以使用以下命令将Nginx以master-worker daemon模式运行：

```
nginx -c /path/to/nginx.conf -p /path/to/nginx/dir
```

在Nginx的配置文件中，可以使用daemon on指令来启用daemon模式，例如：

```
worker_processes 4;
daemon on;
```

其中，worker_processes指令指定Nginx启动的worker进程数量，daemon on指令表示启用daemon模式。

## Nginx的单进程模式和多进程模式

单进程模式适用于小型网站和应用场景，可以降低系统的负载和资源占用；而多进程模式适用于高负载和大型网站和应用场景，可以提高系统的并发性能和处理能力。在实际使用中，可以根据需求和实际情况进行选择和配置。
单进程模式：
在单进程模式下，Nginx只会创建一个master进程，该进程负责接收和处理所有客户端请求，同时也负责读取配置文件和管理worker进程的启动和关闭。由于只有一个进程在处理请求，因此单进程模式可以降低系统负载和资源占用，适用于小型的网站和应用场景。
使用单进程模式可以通过如下命令行参数进行配置：

```
nginx -c /path/to/nginx.conf -p /path/to/nginx/dir -g "worker_processes 1;"
```

其中，
-c参数指定Nginx的配置文件路径
-p参数指定Nginx的运行目录
-g参数指定Nginx的全局配置。
worker_processes指定启动的worker进程数，可以设置为1以使用单进程模式。
多进程模式：
在多进程模式下，Nginx会创建一个master进程和多个worker进程。master进程负责读取配置文件，启动和管理worker进程，以及处理客户端请求的分发和管理；worker进程则负责实际处理客户端请求。由于多个进程并行处理请求，多进程模式可以提高系统的并发性能和处理能力，适用于高负载和大型的网站和应用场景。
使用多进程模式可以通过如下命令行参数进行配置：

```
nginx -c /path/to/nginx.conf -p /path/to/nginx/dir
```

其中，-c参数指定Nginx的配置文件路径，-p参数指定Nginx的运行目录。在配置文件中，可以通过worker_processes指令指定启动的worker进程数，例如：

```
worker_processes 4;
```

表示启动4个worker进程。

总的来说，单进程模式适用于小型网站和应用场景，可以降低系统的负载和资源占用；而多进程模式适用于高负载和大型网站和应用场景，可以提高系统的并发性能和处理能力。在实际使用中，可以根据需求和实际情况进行选择和配置。
需要注意
**Nginx的daemon模式和多进程模式是两个概念，它们并不冲突，可以同时使用**
前者是指以守护进程的方式运行Nginx，后者是指以多进程的方式运行Nginx。在实际使用中，可以根据需求和实际情况进行选择和配置，也可以同时使用两种模式。

# HTTP模块

Nginx的HTTP模块是Nginx中一个核心的模块，用于处理HTTP协议相关的请求和响应。HTTP模块包括HTTP核心模块、HTTP代理模块、HTTP FastCGI模块、HTTP uWSGI模块、HTTP SCGI模块、HTTP memcached模块等。
 HTTP核心模块
HTTP核心模块是Nginx的HTTP模块中最基本和最核心的模块，提供了处理HTTP请求和响应的基本功能。主要包括HTTP请求的解析、HTTP请求的处理、HTTP响应的生成和发送等功能。HTTP核心模块是Nginx的HTTP模块中最核心的部分，也是其他HTTP模块的基础。
HTTP代理模块
HTTP代理模块是Nginx的HTTP模块中重要的模块之一，主要用于反向代理功能。通过HTTP代理模块，Nginx可以将客户端的HTTP请求转发给其他Web服务器进行处理，并将处理结果返回给客户端。HTTP代理模块可以支持反向代理、负载均衡、缓存等功能。
HTTP FastCGI模块
HTTP FastCGI模块是Nginx的HTTP模块中一个重要的模块，主要用于与FastCGI进程通信，通过FastCGI进程处理动态内容的请求。通过HTTP FastCGI模块，Nginx可以将客户端的HTTP请求发送给FastCGI进程进行处理，并将处理结果返回给客户端。
HTTP uWSGI模块
HTTP uWSGI模块是Nginx的HTTP模块中一个用于与uWSGI进程通信的模块，通过uWSGI进程处理动态内容的请求。与HTTP FastCGI模块类似，HTTP uWSGI模块可以将客户端的HTTP请求发送给uWSGI进程进行处理，并将处理结果返回给客户端。
HTTP SCGI模块
HTTP SCGI模块是Nginx的HTTP模块中一个用于与SCGI进程通信的模块，通过SCGI进程处理动态内容的请求。HTTP SCGI模块可以将客户端的HTTP请求发送给SCGI进程进行处理，并将处理结果返回给客户端。
HTTP memcached模块
HTTP memcached模块是Nginx的HTTP模块中一个用于与memcached服务通信的模块，主要用于实现静态内容的缓存。通过HTTP memcached模块，Nginx可以将客户端的HTTP请求发送给memcached服务进行处理，并将处理结果返回给客户端。

## HTTP模块在收到请求后的处理流程

1、HTTP请求解析
Nginx首先会对客户端发送的HTTP请求进行解析，包括解析HTTP请求头、HTTP请求体等信息。
2、配置文件匹配
Nginx会根据配置文件中的规则匹配请求的URL，找到匹配的虚拟主机，并将请求转发到相应的虚拟主机。
3、访问控制
Nginx可以通过配置文件中的规则对请求进行访问控制，如限制IP地址、限制请求方法等。
4、缓存处理
如果配置了缓存，Nginx会先查找缓存是否存在请求对应的内容，如果存在，则直接返回缓存的结果，不会再次请求后端服务。
5、负载均衡
如果配置了负载均衡，Nginx会根据负载均衡算法将请求分配给后端的服务器进行处理。
6、反向代理
如果配置了反向代理，Nginx会将请求转发给后端的服务器进行处理，并将处理结果返回给客户端。
7、处理请求
如果请求的URL匹配了静态文件，Nginx会直接返回文件的内容；如果请求需要处理动态内容，Nginx会将请求转发给FastCGI、uWSGI、SCGI进程等进行处理，并将处理结果返回给客户端。
8、响应处理
Nginx会将后端服务器返回的响应进行处理，包括响应头的设置、响应体的处理等。
9、响应发送
最后，Nginx将响应发送给客户端。



