# nginx基本操作
## 启动操作 
nginx  

当然我们可以只用-c选项制定配置文件，不指定的话就是使用默认的配置  

nginx -c [path]  

## 停止操作
 立即停止  

 nginx -s stop   

或者  

平滑停止  

nginx -s quit  

或者  

kill TERM | INT | QUIT PID  

或者（不建议这么停止服务）  

kill -9 PID  

## 平滑重启服务
nginx -s reload  

或者  

kill HUP PID  


## 基本指令
···
nginx -h：查看帮助
nginx -v：查看nginx的版本
nginx -V：查看版本和nginx的配置选项
nginx -t：测试配置文件的正确性
Nginx -T: 测试配置文件，并显示配置文件（这个命令可以快速查看配置文件）
nginx -q：测试配置文件，但是只显示错误信息
nginx -s：发送信号，下面详细介绍
nginx -p：设置前缀
nginx -c：设置配置文件
nginx -g：附加配置文件路径
···
