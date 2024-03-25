# linux启动jar包的方式,直接运行与守护进程运行
通常我们开发好的程序需要打成war/jar包,在linux运行,war包好说直接丢在tomcat中即可,如果开发好的程序为jar包的话,方式比较多

```

## 直接启动(java-jar xxx.jar)

```shell
java -jar shareniu.jar
```

特点：当前ssh窗口被锁定，可按CTRL + C打断程序运行，或直接关闭窗口，程序退出
## 后台启动(java -jar xxx.jar &)

```shell
java -jar xxx.jar &
```

&代表在后台运行。
特定：当前ssh窗口不被锁定，但是当窗口关闭时，程序中止运
## nohup命令无启动,无日志(nohup java -jar xxx.jar &)

```shell
nohup java -jar xxx.jar &
```

nohup 意思是不挂断运行命令,当账户退出或终端关闭时,程序仍然运行
当用 nohup 命令执行作业时，缺省情况下该作业的所有输出被重定向到nohup.out的文件中，除非另外指定了输出文件
## nohup命令无启动,有日志(nohup java -jar xxx.jar &)

```shell
nohup java -jar xxx.jar >msg.log 2>&1 &;
```

//nohup命令的作用就是让程序在后台运行，不用担心关闭连接进程断掉的问题了（推荐使用)

nohup是no hang up的缩写，就是不挂断的意思。
nohup命令运行由Command参数和任何相关的Arg参数指定的命令，忽略所有挂断（SIGHUP）信号。
如果你正在运行一个进程，而且你觉得在退出帐户时该进程还不会结束，那么可以使用nohup命令。
该命令可以在你退出帐户/关闭终端之后继续运行相应的进程。
例如我们断开SSH连接都不会影响他的运行,注意了nohup没有后台运行的意思,&才是后台运行。

```shell
nohup java -jar xxx.jar >msg.log 2>&1 &;
```

命令解释
0： stdin (standard input)   标准输入
1： stdout (standard output) 标准输出
2： stderr (standard error)  标准错误 

可通过jobs命令查看后台运行任务
jobs
那么就会列出所有后台执行的作业，并且每个作业前面都有个编号。
后台任务回调到前台
如果想将某个作业调回前台控制，只需要 fg + 编号即可。
fg 23