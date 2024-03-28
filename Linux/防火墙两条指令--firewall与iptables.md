@[TOC](iptables与firewall的命令的使用)
## 安装
查看是否已安装：  

CentOS：rpm -qa | grep iptables  
安装（一般系统是集成的）：  

CentOS 6：yum install -y iptables  
Iptables 的配置文件  
路径：vim /etc/sysconfig/iptables  
## firewall指令基本使用  
查防火墙状态： systemctl status firewalld  
开启防火墙：systemctl start firewalld.service  
firewall-cmd --permanent --zone=public --remove-service=ssh    #限制远程连接
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="10.137.196.0/22" port protocol="tcp" port="22" accept"      #添加22端口白名单  
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="10.224.0.0/16" port protocol="tcp" port="8080" accept"      #添加8080端口白名单  
firewall-cmd --reload    #生效设置  
如果之前没有设置白名单，8080端口一直在应用，可以查一下/etc/firewalld/zones/public.xml文件  
看是否添加有<port protocol="tcp" port="8080"/>这句，如果有，请删除，  
再执行firewall-cmd --reload  
1:查看防火状态  
systemctl status firewalld  
service  iptables status  
2:暂时关闭防火墙  
systemctl stop firewalld  
service  iptables stop  
3:永久关闭防火墙  
systemctl disable firewalld  
chkconfig iptables off  
4:重启防火墙  
systemctl enable firewalld  
service iptables restart    
5:永久关闭后重启  
//暂时还没有试过  
chkconfig iptables on  

## iptables  
iptables 是 Linux 防火墙系统的重要组成部分，iptables 的主要功能是实现对网络数据包进出设备及转发的控制。当数据包需要进入设备、从设备中流出或者由该设备转发、路由时，都可以使用 iptables 进行控制  
### 简介  
iptables 是集成在 Linux 内核中的包过滤防火墙系统。使用 iptables 可以添加、删除具体的过滤规则，iptables 默认维护着 4 个表和 5 个链，所有的防火墙策略规则都被分别写入这些表与链中。  
“四表”是指 iptables 的功能，默认的 iptable s规则表有 filter 表（过滤规则表）、nat 表（地址转换规则表）、mangle（修改数据标记位规则表）、raw（跟踪数据表规则表）：  
filter 表：控制数据包是否允许进出及转发，可以控制的链路有 INPUT、FORWARD 和 OUTPUT。  
nat 表：控制数据包中地址转换，可以控制的链路有 PREROUTING、INPUT、OUTPUT 和 POSTROUTING。  
mangle：修改数据包中的原数据，可以控制的链路有 PREROUTING、INPUT、OUTPUT、FORWARD 和 POSTROUTING。  
raw：控制 nat 表中连接追踪机制的启用状况，可以控制的链路有 PREROUTING、OUTPUT。  
“五链”是指内核中控制网络的 NetFilter 定义的 5 个规则链。每个规则表中包含多个数据链：INPUT（入站数据过滤）、OUTPUT（出站数据过滤）、FORWARD（转发数据过滤）、PREROUTING（路由前过滤）和POSTROUTING（路由后过滤），防火墙规则需要写入到这些具体的数据链中。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/b9a200f4557948c68b82022f58db3f67.png)

可以看出，如果是外部主机发送数据包给防火墙本机，数据将会经过 PREROUTING 链与 INPUT 链；如果是防火墙本机发送数据包到外部主机，数据将会经过 OUTPUT 链与 POSTROUTING 链；如果防火墙作为路由负责转发数据，则数据将经过 PREROUTING 链、FORWARD 链以及 POSTROUTING 链。  
### 命令

重启服务命令 ：service iptables restart  
查看服务状态： service iptables status  
设置开启默认启动： chkconfig --level 345 iptables on  
清除所有规则(慎用)  
iptables -F  
iptables -X  
iptables -Z  
#### 查看 IPTABLES 版本¶
rpm -q iptables  
这里返回的结果类似于 iptables-1.4.7-9.el6.i686。  
#### 查看当前 IPTABLES 规则¶
service iptables status  
或者  
iptables -L --line-numbers  
iptables -L  
详细 ipables -nvL  
各参数的含义为：  
-L 表示查看当前表的所有规则，默认查看的是 filter 表，如果要查看 nat 表，可以加上 -t nat 参数。  
-n 表示不对 IP 地址进行反查，加上这个参数显示速度将会加快。  
-v 表示输出详细信息，包含通过该规则的数据包数量、总字节数以及相应的网络接口  
#### 添加规则
格式 iptables [-AI 链名] [-io 网络接口] [-p 协议] [-s 来源IP/网域] [-d 目标IP/网域] -j [ACCEPT|DROP|REJECT|LOG  
端口启动检查 netstat -ano|grep 8443，如果显示内容，表示端口监听了   
选项与参数：  
-AI 链名：针对某的链进行规则的 “插入” 或 “累加”  
-A ：新增加一条规则，该规则增加在原本规则的最后面。例如原本已经有四条规则，使用 -A 就可以加上第五条规则！  
-I ：插入一条规则。如果没有指定此规则的顺序，默认是插入变成第一条规则。例如原本有四条规则，使用 -I 则该规则变成第一条，而原本四条变成 2~5 号链 ：有 INPUT, OUTPUT, FORWARD 等，此链名称又与 -io 有关，请看底下。  
-io 网络接口：设定封包进出的接口规范  
-i ：封包所进入的那个网络接口，例如 eth0, lo 等接口。需与 INPUT 链配合；  
-o ：封包所传出的那个网络接口，需与 OUTPUT 链配合；  
-p 协定：设定此规则适用于哪种封包格式。主要的封包格式有： tcp, udp, icmp 及 all 。  
-s 来源 IP/网域：设定此规则之封包的来源项目，可指定单纯的 IP 或包括网域，例如：IP：192.168.0.100，网域：192.168.0.0/24, 192.168.0.0/255.255.255.0 均可。若规范为『不许』时，则加上 ! 即可，例如：-s ! 192.168.100.0/24 表示不许 192.168.100.0/24 之封包来源。  
-d 目标 IP/网域：同 -s ，只不过这里指的是目标的 IP 或网域。  
-j ：后面接动作，主要的动作有接受(ACCEPT)、丢弃(DROP)、拒绝(REJECT)及记录(LOG)  

例如：iptables -A INPUT -p tcp -s 10.56.71.139/24 --dport 8443 -j ACCEPT  
iptables -I INPUT -i lo -j ACCEPT #允许本地回环接口(即运行本机访问本机)  
iptables -I INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT # 允许已建立的或相关连的通行  
iptables -I OUTPUT -j ACCEPT #允许所有本机向外的访问  
iptables -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT # 允许访问 22 端口  
iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT #允许访问 80 端口  
iptables -A INPUT -p tcp -m tcp --dport 8080 -j ACCEPT #允许访问 8080 端口  
iptables -A INPUT -p tcp -m tcp --dport 21 -j ACCEPT #允许 FTP 服务的 21 端口  
iptables -A INPUT -p tcp -m tcp --dport 20 -j ACCEPT #允许 FTP 服务的 20 端口  
iptables -I INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT #允许 ping  
iptables -I INPUT -j REJECT #禁止其他未允许的规则访问（使用该规则前一定要保证 22 端口是开着，不然就连 SSH 都会连不上）  
iptables -I FORWARD -j REJECT  
#### 修改规则
 修改规则  
在修改规则时需要使用-R参数。 【例 4】把添加在第 6 行规则的 DROP 修改为 ACCEPT。首先需要使用 su 命令，切换当前用户到 root 用户，然后在终端页面输入如下命令：  
 iptables -R INPUT 6 -s 194.168.1.5 -j ACCEPT  
#### 删除某条规则
iptables -D INPUT 16  
16为行数  
#### 加入白名单  
白名单加入  
iptables -A whitelist -s 10.235.5.86  -p all -j ACCEPT  
input加入  
iptables -A INPUT -s   -p tcp -j ACCEPT  
#### 备份与还原
保存配置命令：service iptables save 或者 /etc/rc.d/init.d/iptables save  
1、iptables-save命令  
2、iptables-restore命令  
#### 定义策略
定义iptables默认策略（policy）  
语法  
iptables [-t nat] -P [INPUT,OUTPUT,FORWARD] [ACCEPT,DROP]
选项与参数：
-P：定义策略。
ACCEPT：该数据包可接受。  
DROP：该数据包直接丢弃，不会让Client端知道为何被丢弃。  
iptables -P INPUT DROP
tcpdump -i ens3 tcp port 8443 and  host 10.56.71.139  

FORWARD : 过滤所有不是本地产生的并且目的地不是本地（所谓本地就是防火墙了）的包  
INPUT : 针对那些目的地是本地的包  
OUTPUT : 过滤所有本地生成的包的  

### 其他
-t：指定需要维护的防火墙规则表 filter、nat、mangle或raw。在不使用 -t 时则默认使用 filter 表。  
COMMAND：子命令，定义对规则的管理。  
chain：指明链表。  
CRETIRIA：匹配参数。  
ACTION：触发动作  
iptables 命令常用的选项及各自的功能如表 2 所示  
```
COMMAND如下所示：  
-A  添加防火墙规则  
-D  删除防火墙规则  
-I   插入防火墙规则  
-F  清空防火墙规则  
-L  列出添加防火墙规则    
-R  替换防火墙规则  
-Z  清空防火墙数据表统计信息  
-P  设置链默认规则  
iptables 命令常用匹配参数及各自的功能。  
[!]-p    匹配协议，! 表示取反  
[!]-s    匹配源地址  
[!]-d    匹配目标地址
[!]-i     匹配入站网卡接口
[!]-o   匹配出站网卡接口
[!]--sport       匹配源端口
[!]--dport      匹配目标端口
[!]--src-range       匹配源地址范围
[!]--dst-range       匹配目标地址范围
[!]--limit                四配数据表速率
[!]--mac-source    匹配源MAC地址
[!]--sports             匹配源端口
[!]--dports            匹配目标端口
[!]--stste               匹配状态（INVALID、ESTABLISHED、NEW、RELATED)
[!]--string              匹配应用层字串

ACCEPT  允许数据包通过
DROP     丢弃数据包
REJECT   拒绝数据包通过
LOG        将数据包信息记录 syslog 曰志
DNAT     目标地址转换
SNAT      源地址转换
MASQUERADE   地址欺骗
REDIRECT      重定向
```


常用命令    
iptables -A INPUT -p tcp -s 10.56.71.139/24 --dport 8443 -j ACCEPT
iptables -A INPUT -p tcp --dport 8443 -j DROP  
1、iptables -P INPUT DROP  设置默认策略是禁用  
2、iptables -A INPUT -s 192.168.2.0/24 -j ACCEPT 设置所要允许的规则  
在尾部添加允许所有数据包通行的规则  
iptables -A FORWARD -j ACCEPT  
```
/home/oracle/oracle12c
/home/weblogic/Oracle/Middleware/Oracle_Home/user_projects/domains/base_domain


iptables -A INPUT -p tcp -m tcp --dport 7001 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 7021 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 5556 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 7002 -j ACCEPT
```
