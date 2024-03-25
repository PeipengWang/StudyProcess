# 修改DNS
inux电脑修改DNS配置（以redhat为例，其他系统一般一样，如有报错可自行搜索百度）：
1、登陆Linux服务器，输入cat /etc/resolv.conf命令，查看机器DNS配置是否正确，如果是旧的dns需要修改
2、输入如何查看DNS配置，一般通过cat /etc/resolv.conf
nameserver ip
顺序一为首选，二为备用（具体请参考：000 公司内网DNS配置规范）然后按esc键退出编辑界面，输入英文冒号:，wq即可。
3、修改完毕后，用nslookup命令验证一下配置的DNS能否正常解析域名：
4.若修改后用nslookup命令解析调用的dns server还是旧服务器，请检查网卡/etc/network/interface参数


另一种方法：
linux查看DNS配置的方法：
       登录Linux系统查看：cat /etc/resolv.conf
确定网卡名称ifconfig
/etc/sysconfig/network-scripts/ifcfg-网卡名称
vim /etc/sysconfig/network-scripts/ifcfg-eth0
添加规则
DNS1=114.114.114.114
DNS2=8.8.8.8


service network restart #重启网络使配置生效


重启网络
service network restart