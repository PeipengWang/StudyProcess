# JDK获取网卡信息为null问题
调用jdk代码获取节点网络信息返回null或空对象，重启虚机后恢复正常，希望帮助定位获取网络信息失败原因  
NetworkInterface.getNetworkInterfaces  
测试过程中进行CPU冲高，持续10分钟，之后恢复正常，但调用jdk代码获取网络信息返回null或空对象，重启虚机后才返回正确的值。希望帮助定位返回null或空对象原因。  


  
经分析JDK本地代码会调用Windows操作系统iphlpapi.h的中GetIfTable和GetIpAddrTable等API。  
 
单独编写C++代码调用GetIfTable函数的测试用例在问题环境上运行，该函数返回时间在5秒以上，而其他正常环境的返回时间是毫秒级。  

启动时频繁使用socket通信包括产生随机数使用熵池会调用GetIfTable函数，导致启动很慢。

1、网络配置问题：

检查虚拟机的网络配置是否正确。确保虚拟机具有正确的网络设置，包括IP地址、子网掩码、网关等。  
如果虚拟机使用的是虚拟网络适配器，请确保它已正确配置并且虚拟网络适配器正常工作。  
资源问题：  

您提到在测试期间CPU占用率过高，可能导致系统资源不足。高负载可能会影响网络操作的正常执行。确保虚拟机有足够的资源可供使用，特别是CPU和内存。  
监视系统资源使用情况，以查看是否存在资源瓶颈。您可以使用工具如top、htop或任务管理器来监视系统资源。  
2、网络故障：  

虚拟机所在的物理网络或路由器可能会发生故障或临时中断。在网络故障期间，NetworkInterface.getNetworkInterfaces可能会返回null或空对象。  
检查物理网络的稳定性，并确保虚拟机能够正常连接到网络。  

3、缓存问题：

有时候Java中的一些网络信息可能会被缓存起来，而且这些缓存可能在一段时间后过期。在虚拟机高负载期间，可能会导致缓存中的信息不一致。  
尝试在代码中使用NetworkInterface.getNetworkInterfaces之前调用NetworkInterface.getNetworkInterfaces刷新缓存。  
4、虚拟机问题：  
虚拟机本身可能存在问题，可能需要考虑升级虚拟机软件或修复虚拟机配置。  
日志和异常：  
检查应用程序的日志以查看是否有与网络接口相关的异常信息。这可以帮助您更好地了解问题的根本原因。  


通过 API 文档可知，getInetAddresses 方法返回绑定到该网卡的所有的 IP 地址。（虽然一个网络接口确实可以绑定多个 IP 地址，然而通常情况下，一个网络接口都是只对应一个 IP 地址）


getInterfaceAddresses 方法的作用与 getInetAddresses 方法类似，但是返回的是一个绑定到该网络接口的所有 InterfaceAddress 的集合。InterfaceAddress 也是 JDK1.6 之后添加的类，相比于 InetAddress 的区别在于它除了具有一个 IP 地址（InetAddress），还包括了该地址对应的广播地址和掩码长度。
