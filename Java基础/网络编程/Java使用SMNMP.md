# SNMP
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
随着网络技术的飞速发展，在网络不断普及的同时也给网络管理带来了一些问题：    
网络设备数量成几何级数增加，使得网络管理员对设备的管理变得越来越困难；同时，网络作为一个复杂的分布式系统，其覆盖地域不断扩大，也使得对这些设备进行实时监控和故障排查变得极为困难。  
网络设备种类多种多样，不同设备厂商提供的管理接口（如命令行接口）各不相同，这使得网络管理变得愈发复杂。  
在这种背景下，SNMP应运而生，SNMP是广泛应用于TCP/IP网络的网络管理标准协议，该协议能够支持网络管理系统，用以监测连接到网络上的设备是否有任何引起管理上关注的情况。通过“利用网络管理网络”的方式：  
网络管理员可以利用SNMP平台在网络上的任意节点完成信息查询、信息修改和故障排查等工作，工作效率得以提高。  
屏蔽了设备间的物理差异，SNMP仅提供最基本的功能集，使得管理任务与被管理设备的物理特性、网络类型相互独立，因而可以实现对不同设备的统一管理，管理成本低。  
设计简单、运行代价低，SNMP采用“尽可能简单”的设计思想，其在设备上添加的软件/硬件、报文的种类和报文的格式都力求简单，因而运行SNMP给设备造成的影响和代价都被最小化。  
SNMP的基本组件  
SNMP基本组件包括网络管理系统NMS（Network Management System）、代理进程（Agent）、被管对象（Managed Object）和管理信息库MIB（Management Information Base）  
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
云计算管理：SNMP协议可以用于云计算环境中的网络管理和监控，帮助管理员及时发现和解决云环境中的问题。  
## 原理
SNMP有两个内容，其一是其本身，专门负责管理节点，其二是一个Trap，用于监测报警。  
通俗的理解，SNMP可以看作是一个C/S结构。在客户机中，一般会部署一个snmpd的守护进程，而在服务端（管理端）会下载一个snmp工具包，这个包中包含了许多用于管理客户端网络节点的的工具，例如get，set，translate等等。下图可能会帮你更加清晰的理解这个概念：  
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/60ba4387a2c4468694ca76c6f10126b8.png)

上图中，161表示的是双方进行通信时所用的默认端口号，被管理端会打开一个守护进程，负责监听161端口发来的请求；管理端会提供一个SNMP工具包，利用工具包中的命令可以向
被管理端的161端口发送请求包，以获取响应。  
除此之外，管理端还会开启一个SNMPTrapd守护进程，用于接受被管理端向自己的162端口发送来的snmptrap请求，这一机制主要用于被管理端的自动报警中，一旦被管理端的某个节点出现故障，系统自动会发送snmptrap包，从而远端的系统管理员可以及时的知道问题。  
## 怎么用
### JAVA实现客户端  
要使用SNMP来发送一个消息，需要使用SNMP协议中的TRAP消息。TRAP消息是SNMP协议中的一种通知消息，可以用于向管理系统发送网络设备的告警信息或状态变更信息。  
1、引入依赖  
```
<dependency>
    <groupId>org.snmp4j</groupId>
    <artifactId>snmp4j</artifactId>
    <version>2.8.5</version>
</dependency>

```
2、客户端代码  
```
import java.io.IOException;

import org.snmp4j.CommunityTarget;
import org.snmp4j.PDU;
import org.snmp4j.Snmp;
import org.snmp4j.TransportMapping;
import org.snmp4j.event.ResponseEvent;
import org.snmp4j.mp.SnmpConstants;
import org.snmp4j.smi.Address;
import org.snmp4j.smi.GenericAddress;
import org.snmp4j.smi.OID;
import org.snmp4j.smi.OctetString;
import org.snmp4j.smi.UdpAddress;
import org.snmp4j.smi.VariableBinding;
import org.snmp4j.transport.DefaultUdpTransportMapping;

public class SNMPClient {

    public static void main(String[] args) throws IOException {
        // 创建UDP传输映射
        TransportMapping<? extends Address> transport = new DefaultUdpTransportMapping();

        // 创建SNMP对象
        Snmp snmp = new Snmp(transport);

        // 添加监听器
        transport.listen();

        // 创建目标设备
        CommunityTarget target = new CommunityTarget();
        target.setCommunity(new OctetString("public"));
        target.setAddress(new UdpAddress("127.0.0.1/161"));
        target.setRetries(2);
        target.setTimeout(1500);
        target.setVersion(SnmpConstants.version2c);

        // 创建PDU
        PDU pdu = new PDU();
        pdu.add(new VariableBinding(new OID("1.3.6.1.2.1.1.1.0"))); // 添加请求的OID

        // 发送请求
        ResponseEvent event = snmp.send(pdu, target);

        // 处理响应
        if (event != null && event.getResponse() != null) {
            PDU response = event.getResponse();
            if (response.getErrorStatus() == PDU.noError) {
                // 获取响应的变量绑定列表
                for (VariableBinding vb : response.getVariableBindings()) {
                    System.out.println(vb.getOid() + " = " + vb.getVariable());
                }
            } else {
                System.out.println("Error: " + response.getErrorStatusText());
            }
        } else {
            System.out.println("Error: Timeout");
        }

        // 关闭SNMP
        snmp.close();
    }

}

```
其中PDU（Protocol Data Unit）是SNMP中传输的数据单元，它用于在SNMP管理器和被管理设备之间传输信息。PDU中包含了SNMP消息的各种参数，例如SNMP版本、消息类型、OID（Object Identifier）等信息。PDU还可以包含多个变量绑定，每个变量绑定由一个OID和对应的值组成，用于传递管理信息。PDU可以分为以下几种类型：GetRequest、GetNextRequest、GetResponse、SetRequest、Trap等。  
### 服务端实现
```
package server;
import org.snmp4j.MessageDispatcher;
import org.snmp4j.MessageDispatcherImpl;
import org.snmp4j.Snmp;
import org.snmp4j.TransportMapping;
import org.snmp4j.mp.MPv3;
import org.snmp4j.security.*;
import org.snmp4j.smi.*;
import org.snmp4j.transport.DefaultTcpTransportMapping;
import org.snmp4j.transport.DefaultUdpTransportMapping;
import org.snmp4j.util.MultiThreadedMessageDispatcher;
import org.snmp4j.util.ThreadPool;

public class server1 implements Runnable {

    private Snmp snmp = null;
    private Address listenAddress = null;
    private ThreadPool threadPool = null;
    private MessageDispatcher messageDispatcher = null;
    private MultiThreadedMessageDispatcher multiThreadedDispatcher = null;

    public server1(String ipAddress) throws Exception {
        listenAddress = GenericAddress.parse(ipAddress);
    }

    private void init() throws Exception {
        // 创建线程池和消息分发器
        threadPool = ThreadPool.create("SNMPTrapReceiver", 10);
        messageDispatcher = new MessageDispatcherImpl();
        multiThreadedDispatcher = new MultiThreadedMessageDispatcher(threadPool, messageDispatcher);
        // 创建传输映射
        TransportMapping<?> transport;
        if (listenAddress instanceof UdpAddress) {
            transport = new DefaultUdpTransportMapping((UdpAddress) listenAddress);
        } else {
            transport = new DefaultTcpTransportMapping((TcpAddress) listenAddress);
        }
        // 创建SNMP实例
        snmp = new Snmp(multiThreadedDispatcher, transport);
        // 创建本地引擎ID
        OctetString localEngineID = new OctetString(MPv3.createLocalEngineID());
        // 创建USM实例
        USM usm = new USM(SecurityProtocols.getInstance(), localEngineID, 0);
        // 创建一个SNMPv3用户
        UsmUser user = new UsmUser(new OctetString("myuser"), AuthMD5.ID, new OctetString("mypassword"), PrivDES.ID, new OctetString("myprivkey"));
        // 将用户添加到USM中
        usm.addUser(user.getSecurityName(), user);
        // 创建SNMP对象
        Snmp snmp = new Snmp(new DefaultUdpTransportMapping());
        // 设置USM实例
        snmp.getUSM().removeAllUsers();
        snmp.getUSM().addUser(user.getSecurityName(), user);
        // 开始监听
        snmp.listen();
    }

    @Override
    public void run() {
        try {
            init();
            System.out.println("SNMP Trap Receiver listening on " + listenAddress.toString());
            while (true) {
                Thread.sleep(5000);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws Exception {
        server1 trap = new server1("udp:0.0.0.0/8080");
        new Thread(trap).start();
    }
}

```
