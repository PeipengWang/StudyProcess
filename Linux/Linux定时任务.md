# 定时任务crontab
能够循环执行任务计划  
当用户使用crontab这个命令建立计划任务后，该项任务就会被记录到var/spool/cron中，而且是以账号来作为判断依据的。  
并且，cron每次执行的任务都会被记录到/var/log/cron这个日志文件中  
## 基本语法
```
crontab [-u username] [-l|-e|-r]
-u 只有root才能执行这个任务，帮助选择的用户建立或删除任务
-l 查看所有的任务列表
-e 查看crontab的任务内容
-r 删除所有的crontab的任务内容
```
## 编辑新增任务
默认情况下，任何用户只要不被列入/etc/cron.deny当中，就可以直接执行crontab -e去编辑自己的例行性命令。编辑完成之后直接 ：wq退出即可。  
例如每个5月1号23点59分发送一封邮件：  
59 23 1 5 * mail kiki < /home/dmtsai/lover.txt  
符号表示：  
*（星号） 每一分钟/小时/......   
, （逗号）分隔时间段 例如每3点和6点，3，6  
- （减号）时间段， 例如8-12  8到12点的每一个小时都执行  
/n (斜线) 每五分钟执行一次  */5 * * * * *  

## 删除计划任务  
我们使用命令：crontab -e 进入crontab命令的计划服务配置界面  
直接使用“dd”，删除计划任务  
输入命令“：wq!”保存并退出  




