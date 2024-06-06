
## 是什么
SNMP（Simple Network Management Protocol）是一种用于管理网络设备的协议。它是一种标准化的协议，被用于监控和管理网络设备，包括路由器、交换机、服务器、打印机和其他设备。  
SNMP协议的基本组成部分包括：  
管理站（Management Station）：通常是一个PC或服务器，用于监控和管理网络设备。  
管理代理（Management Agent）：运行在网络设备上的软件，负责管理和监控网络设备，并提供信息给管理站。  
MIB（Management Information Base）：一个层次结构的数据库，包含了网络设备的信息。MIB的每个节点都对应着网络设备的一个属性或状态。  
SNMP协议：用于在管理站和管理代理之间传输信息的协议。SNMP消息包含了操作类型、对象标识符和值。  
SNMP协议定义了一系列操作，可以通过这些操作获取、设置和监控网络设备的属性和状态。这些操作包括：  
1、GET：获取一个属性或状态的值。  
2、SET：设置一个属性或状态的值。  
3、GETNEXT：获取下一个节点的值。  
4、GETBULK：获取一组节点的值。  
5、TRAP：向管理站发送一个警告或通知。  

SNMP协议的最新版本是SNMPv3，它提供了安全性和访问控制机制，以保护网络设备的安全性和保密性。  
## SNMPv3与SNMPv2的不同点  
SNMPv3和SNMPv2是SNMP协议的不同版本，它们之间有一些重要的区别：  
安全性：SNMPv3增加了对安全性的支持，包括认证和加密机制。SNMPv3支持多种认证和加密方式，可以提供更高的安全性。  
访问控制：SNMPv3引入了访问控制模型，可以对用户和用户组进行授权管理，控制对MIB数据的访问权限。  
消息格式：SNMPv3增加了一些新的消息格式，包括通知消息（Notification）和加密消息（Encrypted PDU）。通知消息可以在设备状态发生变化时通知管理者，加密消息可以保护管理信息的安全性。  
管理信息库（MIB）：SNMPv3支持更多的MIB变量，包括新的系统和安全相关变量。  
用户管理：SNMPv3支持更多的用户管理功能，包括用户认证、用户组管理、用户密码策略和用户活动跟踪。  
总的来说，SNMPv3比SNMPv2更加安全和灵活，可以更好地满足现代网络管理的需求。但是，由于SNMPv3相对复杂，它的部署和配置也需要更多的工作和注意。  
## 为什么用SNMP  
### 优势  
1、简单易用：SNMP协议是一种简单易用的协议，使用方便，易于管理。  
2、标准化：SNMP协议是一种标准化的协议，被广泛应用于网络管理领域。  
3、实时监控：SNMP协议可以实时监控网络设备的性能和状态，及时发现和解决问题。   
4、集中管理：SNMP协议可以通过集中管理工具对网络设备进行统一管理，提高管理效率和管理水平。  
5、可扩展性：SNMP协议具有良好的可扩展性，可以适应不同的网络管理需求和应用场景。  
### 应用
网络设备管理：SNMP协议是一种常用的网络设备管理协议，用于监控和管理路由器、交换机、服务器、打印机等网络设备。  
网络性能监控：SNMP协议可以实时监控网络设备的性能指标，如CPU利用率、内存使用率、网络流量等，帮助管理员及时发现和解决性能问题。  
安全管理：SNMP协议可以用于安全管理，监控网络设备的安全状态，及时发现和防范安全威胁。  
系统管理：SNMP协议可以用于监控和管理操作系统、数据库、应用程序等系统和软件，保证系统正常运行和高效管理。  
云计算管理：SNMP协议可以用于云计算环境中的网络管理和监控，帮助管理员及时发现和解决云环境中的问题。IB（Management Information Base）。如图所示他们共同构成SNMP的管理模型，在SNMP的体系结构中都起着至关重要的作用。  
##  SNMP原理介绍  
SNMP有两个内容，其一是其本身，专门负责管理节点，其二是一个Trap，用于监测报警。  
通俗的理解，SNMP可以看作是一个C/S结构。在客户机中，一般会部署一个snmpd的守护进程，而在服务端（管理端）会下载一个snmp工具包，这个包中包含了许多用于管理客户端网络节点的的工具，例如get，set，translate等等。下图可能会帮你更加清晰的理解这个概念：  
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/a248b86f0c504810a510acdab7b1af31.png)

上图中，161表示的是双方进行通信时所用的默认端口号，被管理端会打开一个守护进程，负责监听161端口发来的请求；管理端会提供一个SNMP工具包，利用工具包中的命令可以向  
被管理端的161端口发送请求包，以获取响应。  
除此之外，管理端还会开启一个SNMPTrapd守护进程，用于接受被管理端向自己的162端口发送来的snmptrap请求，这一机制主要用于被管理端的自动报警中，一旦被管理端的某个节点出现故障，系统自动会发送snmptrap包，从而远端的系统管理员可以及时的知道问题。  
##  实际运用  
基本指令  
1、snmpwalk： snmpwalk 用于获取 SNMP 设备上的数据，它遍历设备的 SNMP 树，并返回特定对象标识符（OID）的值，通常用于查询设备的信息和状态。使用 snmpwalk 可以获取有关设备的详细信息，例如接口、系统信息、传感器状态等。以下是 snmpwalk 的一般用法：  
```
nmpwalk -v SNMP_VERSION -c COMMUNITY_STRING TARGET_HOST OID
SNMP_VERSION：SNMP 版本，通常是 "1"（SNMPv1）或 "2c"（SNMPv2c）。
COMMUNITY_STRING：SNMP 社区字符串，用于身份验证。
TARGET_HOST：目标设备的主机名或 IP 地址。
OID：要查询的对象标识符。
```
例如，要获取设备的系统信息，你可以运行：  
```
snmpwalk -v 2c -c public 192.168.1.1 1.3.6.1.2.1.1
```
2、snmptrap： snmptrap 用于生成和发送 SNMP 陷阱（trap）到 SNMP 管理器，这些陷阱通常表示设备上发生的重要事件或告警。snmptrap 常用于监控设备状态变化和异常事件。以下是 snmptrap 的一般用法：  
```
snmptrap -v SNMP_VERSION -c COMMUNITY_STRING TARGET_HOST TRAP_OID [OID_VALUE] [OPTIONS]  
SNMP_VERSION：SNMP 版本，通常是 "1"（SNMPv1）或 "2c"（SNMPv2c）。  
COMMUNITY_STRING：SNMP 社区字符串，用于身份验证。  
TARGET_HOST：目标 SNMP 管理器的主机名或 IP 地址。  
TRAP_OID：陷阱的对象标识符，表示事件类型。  
OID_VALUE：可选，与陷阱相关的 OID 值。  
OPTIONS：可选，包括发送陷阱的其他选项，如 -p（指定陷阱端口）等。  
```
例如，要生成并发送一个告警陷阱，你可以运行：  
```
snmptrap -v 2c -c public 192.168.1.1 1.3.6.1.6.3.1.1.5.1
```
3、snmpget： 用于获取单个 SNMP 对象的值。你可以指定要查询的 OID，它将返回相应的值。  
```
snmpget -v SNMP_VERSION -c COMMUNITY_STRING TARGET_HOST OID
```
例如：  

```
snmpget -v 2c -c public 192.168.1.1 1.3.6.1.2.1.1.1.0
```
4、snmpset： 用于设置 SNMP 对象的值。它允许你修改设备上的特定 OID 的值。  
```
snmpset -v SNMP_VERSION -c COMMUNITY_STRING TARGET_HOST OID TYPE VALUE
```
例如：  
```
snmpset -v 2c -c private 192.168.1.1 1.3.6.1.2.1.1.6.0 s "New Location"
```
5、snmpbulkwalk： 类似于 snmpwalk，但它使用了 SNMP Bulk Protocol（SNMPv2c）来提高效率，特别对于大型数据集。  

```
snmpbulkwalk -v 2c -c public 192.168.1.1 1.3.6.1.2.1.2
```
6、snmpinform： 用于向 SNMP 管理器发送 SNMP INFORM 消息，这是一种更可靠的通知机制，它需要管理器进行确认。  
```
snmpinform -v 2c -c community 192.168.1.1 1.3.6.1.6.3.1.1.5.1
```





