# idea远程debug
修改catalina.sh文件  
JAVA_OPTS="$JAVA_OPTS -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=8080"  
远程服务器防火墙端口放行  
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT  
重启出现如下表示8080端口被监听  
OpenJDK 64-Bit Server VM warning: ignoring option PermSize=256M; support was removed in 8.0  
Listening for transport dt_socket at address: 8080  

idea配置
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=8080  
