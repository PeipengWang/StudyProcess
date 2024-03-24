# SNMP的配置与日志
SNMP 配置文件： 大多数系统上的 SNMP 配置信息存储在配置文件中。通常，SNMP 的主要配置文件是 /etc/snmp/snmpd.conf 或 /etc/snmp/snmpd.conf。你可以使用文本编辑器查看这些文件。示例命令：
cat /etc/snmp/snmpd.conf
SNMP 日志文件： SNMP 代理通常会生成日志文件，以记录各种事件和活动。SNMP 日志文件的位置和名称取决于系统，但通常可以在 /var/log/snmpd.log 或类似位置找到。你可以使用文本编辑器或 cat 命令查看日志文件的内容。示例命令：
cat /var/log/snmpd.log

查看系统日志： SNMP 代理的日志信息也可能记录在系统的通用系统日志中，如 /var/log/messages 或 /var/log/syslog。你可以使用文本编辑器或 cat 命令查看系统日志中的 SNMP 相关信息。
SNMP 监视工具： 有一些专门的 SNMP 监视工具，如 Cacti、Nagios 或 Zabbix，它们可以帮助你监视和可视化 SNMP 设备的性能和事件信息。这些工具通常提供更丰富的 SNMP 日志和报告。

# 自定义mib
https://blog.csdn.net/yiyu89/article/details/123984547
# snmpdconf文件详解
https://www.cnblogs.com/zhming26/p/5461193.html
其实配制一个 snmpd.conf 文件不算太难， 
（ 1 ）首选是定义一个共同体名 (community) ，这里是 public ，及可以访问这个 public 的用户名（ sec name ），这里是 notConfigUser 。 Public 相当于用户 notConfigUser 的密码：）
com2sec notConfigUser  default       public
（ 2 ）定义一个组名（ groupName ）这里是 notConfigGroup ，及组的安全级别，把 notConfigGroup 这个用户加到这个组中。   
            groupName      securityModel securityName 
group   notConfigGroup   v1           notConfigUser   
group   notConfigGroup   v2c           notConfigUser 

  （ 3 ）定义一个可操作的范围 (view) 名，   这里是 all ，范围是  .1 
    #       name           incl/excl     subtree         mask(optional) 
       view  all             included     .1 
 （ 4 ）定义 notConfigUser 这个组在 all 这个 view 范围内可做的操作，这时定义了 notConfigUser 组的成员可对.1 这个范围做只读操作。 
     #       group          context sec.model sec.level prefix read   write  notif 
access  notConfigGroup ""      any       noauth    exact  all  none no