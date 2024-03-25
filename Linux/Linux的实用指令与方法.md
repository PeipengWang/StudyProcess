下文主要是一些工作中零碎的常用指令与方法
#  实用指令与方法（部分）
## linux长时间保持ssh连接
这个问题的原因是：设置检测时间太短，或者没有保持tcp长连接。

解决步骤：

步骤1：打开sshd配置文件（/etc/ssh/sshd_config）
步骤2：修改三个参数
        ClientAliveInterval 600      
        ClientAliveCountMax 10
       TCPKeepAlive yes
 重启
 CentOS6操作系统
service sshd restart
CentOS7/EulerOS操作系统
 systemctl restart sshd

## SCP指令
scp 跨机远程拷贝
scp是secure copy的简写，用于在Linux下进行远程拷贝文件的命令，和它类似的命令有cp，不过cp只是在本机进行拷贝不能跨服务器，而且scp传输是加密的。可能会稍微影响一下速度。两台主机之间复制文件必需得同时有两台主机的复制执行帐号和操作权限。

scp命令参数

```
-1 强制scp命令使用协议ssh1
-2 强制scp命令使用协议ssh2
-4 强制scp命令只使用IPv4寻址
-6 强制scp命令只使用IPv6寻址
-B 使用批处理模式（传输过程中不询问传输口令或短语）
-C 允许压缩。（将-C标志传递给ssh，从而打开压缩功能）
-p 留原文件的修改时间，访问时间和访问权限。
-q 不显示传输进度条。
-r 递归复制整个目录。
-v 详细方式显示输出。scp和ssh(1)会显示出整个过程的调试信息。这些信息用于调试连接，验证和配置问题。
-c cipher 以cipher将数据传输进行加密，这个选项将直接传递给ssh。
-F ssh_config 指定一个替代的ssh配置文件，此参数直接传递给ssh。
-i identity_file 从指定文件中读取传输时使用的密钥文件，此参数直接传递给ssh。
-l limit 限定用户所能使用的带宽，以Kbit/s为单位。
-o ssh_option 如果习惯于使用ssh_config(5)中的参数传递方式，
-P port 注意是大写的P, port是指定数据传输用到的端口号
-S program 指定加密传输时所使用的程序。此程序必须能够理解ssh(1)的选项。
```

## ssh连接的日志查看
linux下登录日志在下面的目录里：
cd /var/log
查看ssh用户的登录日志：
less secure

## 杀死所有tomcat进程
ps -ef|grep tomcat|grep -v 'grep'|awk '{print $2}' |xargs kill -9
##  查看域名对应的ip
输入ping+域名，查询域名ip地址，或者输入nslookup+域名，查询域名ip地址。
查本机局域网IP方法，输入ipconfig，查询本机ip。
IP查询域名方法，输入nslookup+IP ，查询ip地址域名。
用网站查询方法，网上搜索IP反查域名的网站。
粘贴IP地址，点击查询，就可得到网站信息。

## tcpdump
默认启动
tcpdump
普通情况下，直接启动tcpdump将监视第一个网络接口上所有流过的数据包。
监视指定网络接口的数据包
tcpdump -i eth1
如果不指定网卡，默认tcpdump只会监视第一个网络接口，一般是eth0，下面的例子都没有指定网络接口。　
监视指定主机的数据包
打印所有进入或离开sundown的数据包.
tcpdump host sundown
也可以指定ip,例如截获所有210.27.48.1 的主机收到的和发出的所有的数据包
tcpdump host 210.27.48.1 
## 监视指定主机和端口的数据包
如果想要获取主机210.27.48.1接收或发出的telnet包，使用如下命令
tcpdump tcp port 23 and host 210.27.48.1
对本机的udp 123 端口进行监视 123 为ntp的服务端口
tcpdump udp port 123 

tcpdump tcp -i eth1 -t -s 0 -c 100 and dst port ! 22 and src net 192.168.1.0/24 -w ./target.cap

(1)tcp: ip icmp arp rarp 和 tcp、udp、icmp这些选项等都要放到第一个参数的位置，用来过滤数据报的类型
(2)-i eth1 : 只抓经过接口eth1的包
(3)-t : 不显示时间戳
(4)-s 0 : 抓取数据包时默认抓取长度为68字节。加上-S 0 后可以抓到完整的数据包
(5)-c 100 : 只抓取100个数据包
(6)dst port ! 22 : 不抓取目标端口是22的数据包
(7)src net 192.168.1.0/24 : 数据包的源网络地址为192.168.1.0/24
(8)-w ./target.cap : 保存成cap文件，方便用ethereal(即wireshark)分析

tcpdump tcp -i ens3  -s 0  and dst port  22 and src net 10.56.68.82  -w ./target.cap

## linux获取硬盘I/O繁忙度
linux获取硬盘I/O繁忙度
iostat -x 1
这将每秒钟显示一次磁盘I/O统计信息，包括磁盘读写速度、I/O请求队列的长度、磁盘使用率以及I/O利用率等信息。其中，-x参数表示输出扩展统计信息。
iostat -d -x
查看硬盘I/O繁忙度的数值
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %ut
iostat 命令输出的设备的I/O统计信息，其中包括每秒读写请求的数量（r/s和w/s）、每秒读写合并的请求数量（rrqm/s和wrqm/s）、每秒读写的数据量（rkB/s和wkB/s）、平均I/O请求大小（avgrq-sz）、请求队列长度的平均值（avgqu-sz）等指标。其中，%util 指标表示设备的I/O繁忙度，也就是设备处理I/O请求的时间占总时间的百分比，该指标的值越高，表示设备的I/O繁忙度越高。通常认为，当设备的I/O繁忙度超过80%时，就需要考虑对该设备进行优化或升级。

## usermod指令
usermod命令 用于修改用户的基本信息。usermod 命令不允许你改变正在线上的使用者帐号名称。当 usermod 命令用来改变user id，必须确认这名user没在电脑上执行任何程序。你需手动更改使用者的 crontab 档。也需手动更改使用者的 at 工作档。采用 NIS server 须在server上更动相关的NIS设定。

语法
usermod(选项)(参数)
选项
-c<备注>：修改用户帐号的备注文字；
-d<登入目录>：修改用户登入时的目录，只是修改/etc/passwd中用户的家目录配置信息，不会自动创建新的家目录，通常和-m一起使用；
-m<移动用户家目录>:移动用户家目录到新的位置，不能单独使用，一般与-d一起使用。
-e<有效期限>：修改帐号的有效期限；
-f<缓冲天数>：修改在密码过期后多少天即关闭该帐号；
-g<群组>：修改用户所属的群组；
-G<群组>；修改用户所属的附加群组；
-l<帐号名称>：修改用户帐号名称；
-L：锁定用户密码，使密码无效；
-s<shell>：修改用户登入后所使用的shell；
-u<uid>：修改用户ID；
-U:解除密码锁定。
## 环境变量配置
【环境变量配置的三个方法】
如想将一个路径加入到$PATH中，可以像下面这样做： 
1. 控制台中,不赞成使用这种方法，因为换个shell，你的设置就无效了，因此这种方法仅仅是临时使用，以后要使用的时候又要重新设置，比较麻烦。
 这个只针对特定的shell; 
$ PATH="$PATH:/my_new_path"    （关闭shell，会还原PATH）
2. 修改/etc/profile文件,如果你的计算机仅仅作为开发使用时推荐使用这种方法，因为所有用户的shell都有权使用这些环境变量，可能会给系统带来安全性问题。 这里是针对所有的用户的,所有的shell; 
$ vi /etc/profile 
在里面加入: 
export PATH="$PATH:/my_new_path" 
使用source命令使修改立刻生效：  
source  /etc/profile
3. 修改.bashrc文件,这种方法更为安全，它可以把使用这些环境变量的权限控制到用户级别,这里是针对某一个特定的用户，如果你需要给某个用户权限使用这些环境变量，你只需要修改其个人用户主目录下的.bashrc文件就可以了。
$ vi /root/.bashrc 
在里面加入： 

export PATH="$PATH:/my_new_path" 

source  /root/.bashrc

后两种方法一般需要重新注销系统才能生效，也可以使用source 命令，使修改的配置立刻
##  更改软链接-python为例
1.linux的软连接存放位置
cd /usr/bin

2.查看现有python的软连接指向的版本
ls -al *python*

3.删除旧的软连接
rm python

4.建立新的软连接
ln -s python3.5 python

5.查看软连接版本
python -V
##  查看端口是否被占用
1.使用lsof
lsof -i:端口号查看某个端口是否被占用
2.使用netstat
使用netstat -anp|grep 80
## linux长时间保持ssh连接
这个问题的原因是：设置检测时间太短，或者没有保持tcp长连接。

解决步骤：

步骤1：打开sshd配置文件（/etc/ssh/sshd_config）
步骤2：修改三个参数
        ClientAliveInterval 600      
        ClientAliveCountMax 10
       TCPKeepAlive yes
 重启
 CentOS6操作系统
service sshd restart
CentOS7/EulerOS操作系统
 systemctl restart sshd


##  查看域名对应的ip
输入ping+域名，查询域名ip地址，或者输入nslookup+域名，查询域名ip地址。
查本机局域网IP方法，输入ipconfig，查询本机ip。
IP查询域名方法，输入nslookup+IP ，查询ip地址域名。
用网站查询方法，网上搜索IP反查域名的网站。
粘贴IP地址，点击查询，就可得到网站信息。

## tcpdump
默认启动
tcpdump
普通情况下，直接启动tcpdump将监视第一个网络接口上所有流过的数据包。
监视指定网络接口的数据包
tcpdump -i eth1
如果不指定网卡，默认tcpdump只会监视第一个网络接口，一般是eth0，下面的例子都没有指定网络接口。　
监视指定主机的数据包
打印所有进入或离开sundown的数据包.
tcpdump host sundown
也可以指定ip,例如截获所有210.27.48.1 的主机收到的和发出的所有的数据包
tcpdump host 210.27.48.1 
## 监视指定主机和端口的数据包
如果想要获取主机210.27.48.1接收或发出的telnet包，使用如下命令
tcpdump tcp port 23 and host 210.27.48.1
对本机的udp 123 端口进行监视 123 为ntp的服务端口
tcpdump udp port 123 

tcpdump tcp -i eth1 -t -s 0 -c 100 and dst port ! 22 and src net 192.168.1.0/24 -w ./target.cap

(1)tcp: ip icmp arp rarp 和 tcp、udp、icmp这些选项等都要放到第一个参数的位置，用来过滤数据报的类型
(2)-i eth1 : 只抓经过接口eth1的包
(3)-t : 不显示时间戳
(4)-s 0 : 抓取数据包时默认抓取长度为68字节。加上-S 0 后可以抓到完整的数据包
(5)-c 100 : 只抓取100个数据包
(6)dst port ! 22 : 不抓取目标端口是22的数据包
(7)src net 192.168.1.0/24 : 数据包的源网络地址为192.168.1.0/24
(8)-w ./target.cap : 保存成cap文件，方便用ethereal(即wireshark)分析

tcpdump tcp -i ens3  -s 0  and dst port  22 and src net 10.56.68.82  -w ./target.cap

## linux获取硬盘I/O繁忙度
linux获取硬盘I/O繁忙度
iostat -x 1
这将每秒钟显示一次磁盘I/O统计信息，包括磁盘读写速度、I/O请求队列的长度、磁盘使用率以及I/O利用率等信息。其中，-x参数表示输出扩展统计信息。
iostat -d -x
查看硬盘I/O繁忙度的数值
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %ut
iostat 命令输出的设备的I/O统计信息，其中包括每秒读写请求的数量（r/s和w/s）、每秒读写合并的请求数量（rrqm/s和wrqm/s）、每秒读写的数据量（rkB/s和wkB/s）、平均I/O请求大小（avgrq-sz）、请求队列长度的平均值（avgqu-sz）等指标。其中，%util 指标表示设备的I/O繁忙度，也就是设备处理I/O请求的时间占总时间的百分比，该指标的值越高，表示设备的I/O繁忙度越高。通常认为，当设备的I/O繁忙度超过80%时，就需要考虑对该设备进行优化或升级。

##  查看端口是否被占用
1.使用lsof
lsof -i:端口号查看某个端口是否被占用
2.使用netstat
使用netstat -anp|grep 80