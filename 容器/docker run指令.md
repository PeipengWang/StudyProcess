# docker run命令
通过 docker run 命令创建一个新的容器。
## Docker容器运行语法
 docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
## docker run常用选项
d, --detach=false 指定容器运行于前台还是后台，默认为 false。
-i, --interactive=false 打开 STDIN，用于控制台交互。
-t, --tty=false  分配 tty 设备，该可以支持终端登录，默认为 false。
-u, --user=""  指定容器的用户。
-a, --attach=[]   登录容器（必须是以 docker run -d 启动的容器）。
-w, --workdir=""   指定容器的工作目录。
-c, --cpu-shares=0   设置容器 CPU 权重，在 CPU 共享场景使用。
-e, --env=[]    指定环境变量，容器中可以使用该环境变量。
-m, --memory=""  指定容器的内存上限。
-P, --publish-all=false   指定容器暴露的端口。
-p, --publish=[]  指定容器暴露的端口。
-h, --hostname=""  指定容器的主机名。
-v, --volume=[]   给容器挂载存储卷，挂载到容器的某个目录。
–volumes-from=[]  给容器挂载其他容器上的卷，挂载到容器的某个目录。
–cap-add=[]  添加权限。
–cap-drop=[]  删除权限。
–cidfile=""  运行容器后，在指定文件中写入容器 PID 值，一种典型的监控系统用法。
–cpuset=""  设置容器可以使用哪些 CPU，此参数可以用来容器独占 CPU。
–device=[]   添加主机设备给容器，相当于设备直通。
–dns=[]  指定容器的 dns 服务器。
–dns-search=[]   指定容器的 dns 搜索域名，写入到容器的 /etc/resolv.conf 文件。
–entrypoint=""  覆盖 image 的入口点。
–env-file=[]  指定环境变量文件，文件格式为每行一个环境变量。
–expose=[]   指定容器暴露的端口，即修改镜像的暴露端口。
–link=[]   指定容器间的关联，使用其他容器的 IP、env 等信息。
–lxc-conf=[]  指定容器的配置文件，只有在指定 --exec-driver=lxc 时使用。
–name=""   指定容器名字，后续可以通过名字进行容器管理，links 特性需要使用名字。
–net=“bridge” 器网络设置：
1. bridge 使用 docker daemon 指定的网桥。
2. host //容器使用主机的网络。
3. container:NAME_or_ID >//使用其他容器的网路，共享 IP 和 PORT 等网络资源。
4. none 容器使用自己的网络（类似–net=bridge），但是不进行配置。
–privileged=false  指定容器是否为特权容器，特权容器拥有所有的 capabilities。
–restart=“no” 指定容器停止后的重启策略:
1. no：容器退出时不重启。
2. on-failure：容器故障退出（返回值非零）时重启。
3. always：容器退出时总是重启。
–rm=false   指定容器停止后自动删除容器(不支持以 docker run -d 启动的容器)。
–sig-proxy=true 设置由代理接受并处理信号，但是 SIGCHLD、SIGSTOP 和 SIGKILL 不能被代理。
/bin/echo “echo ......”在启动的容器里执行的命令
/bin/bash  在启动的容器里执行的命令。
docker run -i -t centos /bin/bash
我们使用 docker run -i -t 命令以交互式方式运行 Docker。
docker run -d centos /bin/bash -c "while true; do echo hello world; sleep 3; done"