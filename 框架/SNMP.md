# SNMP
下载
http://sourceforge.net/projects/net-snmp/files/net-snmp/5.7.1/  


./configure --prefix=/usr/local/snmp --with-mib-modules='ucd-snmp/diskio ip-mib/ipv4InterfaceTable'
make  &&  make install
mkdir /usr/local/snmp/etc
cp EXAMPLE.conf /usr/local/snmp/etc/snmpd.conf
https://cloud.tencent.com/developer/article/1366176
ps aux | grep snmp | grep -v grep |awk '{print $2}'| xargs kill
netstat -an |grep 161
/usr/local/snmp/sbin/snmpd -c /usr/local/snmp/etc/snmpd.conf
snmpget -v 2c -c public localhost sysName.0"或者"snmpget -v 2c -c public 本机的ip地址 sysName.0
snmpget -v 2c -c public 本机的ip地址 .1.3.6.1.2.1.1.5.0
　snmpget -v 2c -c public localhost sysName.0
　　　　snmpget -v 2c -c public 127.0.0.1 sysName.0
　　　　snmpget -v 2c -c public 192.168.1.229 sysName.0
　　　　snmpget -v 2c -c public localhost .1.3.6.1.2.1.1.5.0
　　　　snmpget -v 2c -c public 127.0.0.1 .1.3.6.1.2.1.1.5.0
　　　　snmpget -v 2c -c public 192.168.1.229 .1.3.6.1.2.1.1.5.0
　　　　iptables –L –n
iptables -I INPUT -p udp --dport 161 -j ACCEPT
iptables save
Snmputil.exe
snmputil get 192.168.1.229 public .1.3.6.1.2.1.1.5.0
如果安装后想卸载SNMP，可以执行"make uninstall"命令，卸载步骤如下：

　　1、使用"netstat -an |grep 161"查看snmp服务是否启动

　　2、如果已经启动就使用命令"ps aux | grep snmp | grep -v grep |awk '{print $2}'| xargs kill"关闭snmp的相关服务

　　3、进入SNMP源码目录(net-snmp-5.7.1)

　　4、执行"make uninstall"命令卸载SNMP

　　5、删除/usr/local下的snmp目录以及snmp目录里面的所有文件，usr/local/snmp是SNMP的安装路径
