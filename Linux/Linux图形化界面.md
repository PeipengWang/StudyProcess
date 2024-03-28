# Linux图像化界面
xclock不能显示时钟  
1. vncserver  
如果报错 command not found，则  
a.yum install tigervnc-server  
b.然后再设置密码  
c.执行 vncserver  

2.根据vncserver执行的结果执行以下操作  
a.export DISPLAY=local:0.0  
b.echo $DISPLAY  
c.xhost +  
d.xclock  

3.如果执行xclock报错，command not found，则  
yum install xorg-x11-apps  

xhost 是用来控制X server访问权限的。  

通常当你从hostA登陆到hostB上运行hostB上的应用程序时，做为应用程序来说，hostA是client,  
但是作为图形来说，是在hostA上显示的，需要使用hostA的Xserver,所以hostA是server.(执行xhost +后，得到提示“access control disabled, clients can connect from any host”)  
因此在登陆到hostB前，  
需要在hostA上运行xhost +来使其它用户能够访问hostA的Xserver.  

xhost + 是使所有用户都能访问Xserver.  
xhost + ip使ip上的用户能够访问Xserver.  
xhost + nis:user@domain使domain上的nis用户user能够访问  
xhost + inet:user@domain使domain上的inet用户能够访问。  
/home/weblogic/Oracle/Middleware/Oracle_Home/user_projects/domains/base_domain  
