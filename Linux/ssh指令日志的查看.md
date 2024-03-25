# ssh基本指令
1、强制登录
```
ssh -t username@remote_host 
```
-t 表示 Force pseudo-tty allocation，
2、ssh连接到其他端口
SSH 默认连接到目标主机的 22 端口上，可以使用-p选项指定端口号
```
$ ssh -p 10022 user@hostname
```
3、使用ssh在远程主机执行一条命令并显示到本地, 然后继续本地工作
直接连接并在后面加上要执行的命令就可以了
```
$ ssh pi@10.42.0.47 ls -l
```
4、在远程主机运行一个图形界面的程序
使用ssh的-X选项，然后主机就会开启 X11 转发功能
```
$ ssh -X feiyu@222.24.51.147
```

5、如何配置 SSH
SSH 的配置文件在 /etc/ssh/sshd_config 中，你可以看到端口号, 空闲超时时间等配置项。
6、构建 ssh 密钥对
使用 ssh-keygen -t +算法 ，现在大多数都使用rsa或者dsa算法。
```
$ ssh-keygen -t rsa
```
7、查看是否已经添加了对应主机的密钥
使用-F选项
```
$ ssh-keygen -F 222.24.51.147
```
8、删除主机密钥
使用-R选项，也可以在~/.ssh/known_hosts文件中手动删除
```
$ ssh-keygen -R 222.24.51.147
```
9、绑定源地址
如果你的客户端有多于两个以上的 IP 地址，你就不可能分得清楚在使用哪一个 IP 连接到 SSH 服务器。为了解决这种情况，我们可以使用 -b 选项来指定一个IP 地址。这个 IP 将会被使用做建立连接的源地址。
```
$ ssh -b 192.168.0.200  root@192.168.0.103
```
10、对所有数据请求压缩
使用 -C 选项，所有通过 SSH 发送或接收的数据将会被压缩，并且任然是加密的。
```
$ ssh -C root@192.168.0.103
```
11、打开调试模式
因为某些原因，我们想要追踪调试我们建立的 SSH 连接情况。SSH 提供的 -v 选项参数正是为此而设的。其可以看到在哪个环节出了问题。 -vv -vvv 更具体
```
$ ssh -v root@192.168.0.103
```
12、设置ssh连接不断开
其它命令行客户端，通过配置 ServerAliveInterval 来实现，在 ~/.ssh/config 中加入： ServerAliveInterval=30。表示ssh客户端每隔30秒给远程主机发送一个no-op包，no-op是无任何操作的意思，这样远程主机就不会关闭这个SSH会话。
vim ~/.ssh/config，然后新增
Host *
    ServerAliveInterval 60
60秒就好了，而且基本去连的机器都保持，所以配置了*，如果有需要针对某个机器，可以自行配置为需要的serverHostName。
13、SSH数据传输
SSH不仅可以用于远程主机登录，还可以直接在远程主机上执行操作。
```
$ ssh user@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub
```
单引号中间的部分，表示在远程主机上执行的操作；后面的输入重定向，表示数据通过SSH传向远程主机。
这就是说，SSH可以在用户和远程主机之间，建立命令和数据的传输通道，因此很多事情都可以通过SSH来完成。
下面看几个例子。
【例1】
```
将$HOME/src/目录下面的所有文件，复制到远程主机的$HOME/src/目录。
$ cd && tar czv src | ssh user@host 'tar xz'
```
【例2】
将远程主机$HOME/src/目录下面的所有文件，复制到用户的当前目录。
```
$ ssh user@host 'tar cz src' | tar xzv
```
【例3】
查看远程主机是否运行进程httpd。
```
 $ ssh user@host 'ps ax | grep [h]ttpd'
```




配置文件详细说明
```
Port 2
“Port”设置sshd监听的端口号。
 
ListenAddress 192.168.1.1
“ListenAddress”设置sshd服务器绑定的IP地址。
 
HostKey /etc/ssh/ssh_host_key
 
“HostKey”设置包含计算机私人密匙的文件。
 
ServerKeyBits 1024
“ServerKeyBits”定义服务器密匙的位数。
 
LoginGraceTime 600
“LoginGraceTime”设置如果用户不能成功登录，在切断连接之前服务器需要等待的时间（以秒为单位）。
 
ClientAliveInterval 300（默认为0）
这个参数的是意思是每5分钟，服务器向客户端发一个消息，用于保持连接
 
 
KeyRegenerationInterval 3600
“KeyRegenerationInterval”设置在多少秒之后自动重新生成服务器的密匙（如果使用密匙）。重新生成密匙是为了防止用盗用的密匙解密被截获的信息。
 
PermitRootLogin no
“PermitRootLogin”设置root能不能用ssh登录。这个选项一定不要设成“yes”。
 
IgnoreRhosts yes
“IgnoreRhosts”设置验证的时候是否使用“rhosts”和“shosts”文件。
 
IgnoreUserKnownHosts yes
“IgnoreUserKnownHosts”设置ssh daemon是否在进行RhostsRSAAuthentication安全验证的时候忽略用户的“$HOME/.ssh/known_hosts”
 
StrictModes yes
“StrictModes”设置ssh在接收登录请求之前是否检查用户家目录和rhosts文件的权限和所有权。这通常是必要的，因为新手经常会把自己的目录和文件设成任何人都有写权限。
 
X11Forwarding no
“X11Forwarding”设置是否允许X11转发。
 
PrintMotd yes
“PrintMotd”设置sshd是否在用户登录的时候显示“/etc/motd”中的信息。
 
SyslogFacility AUTH
“SyslogFacility”设置在记录来自sshd的消息的时候，是否给出“facility pre”。
 
LogLevel INFO
“LogLevel”设置记录sshd日志消息的层次。INFO是一个好的选择。查看sshd的man帮助页，已获取更多的信息。
 
RhostsAuthentication no
“RhostsAuthentication”设置只用rhosts或“/etc/hosts.equiv”进行安全验证是否已经足够了。
 
RhostsRSAAuthentication no
“RhostsRSA”设置是否允许用rhosts或“/etc/hosts.equiv”加上RSA进行安全验证。
 
RSAAuthentication yes
“RSAAuthentication”设置是否允许只有RSA安全验证。
 
PasswordAuthentication yes
“PasswordAuthentication”设置是否允许口令验证。
 
PermitEmptyPasswords no
“PermitEmptyPasswords”设置是否允许用口令为空的帐号登录。
```
# ssh连接的日志查看
linux下登录日志在下面的目录里：
cd /var/log
查看ssh用户的登录日志：
less secure
