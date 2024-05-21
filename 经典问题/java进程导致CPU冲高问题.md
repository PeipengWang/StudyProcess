1、查看tomcat进程id

    ps aux|grep tomcat

2、生成线程堆栈文件

     jstack -l 24769 > statck.log  
     
3、top命令看该进程中占用cpu最高的线程

     top -H -p 24769，其中24769是tomcat的进程pid，所有该进程的线程都列出来了，左边一列是线程的id，是十进制

4、看看哪个线程cpu占用最多，然后将该线程的id转换为16进制，

     比如id为24857的线程，转换为16进制后是6119，如果转换为16进制后，有字母，字母必须小写

5 在第2步生成的线程堆栈文件中查询nid为6119的信息如下便能定位到占用cpu高的代码位置了。

查看cpu的命令也可以使用：sar -u 1 5 表示每隔1秒打印一次，总共打印5次

OOM  
设置参数  
JAVA_OPTS="-server -Xms2000M -Xmx2000M -XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=256M -Djava.awt.headless=true -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/zxin10/was/tomcat/logs/outMemError.hprof"

参数说明

（1）-XX:+HeapDumpOnOutOfMemoryError参数表示当JVM发生OOM时，自动生成DUMP文件。

（2）-XX:HeapDumpPath=${目录}参数表示生成DUMP文件的路径，也可以指定文件名称，例如：-XX:HeapDumpPath=${目录}/java_heapdump.hprof。如果不指定文件名，默认为：java_<pid>_<date>_<time>_heapDump.hprof。

1、top -H -p 24110 查看各个线程的内存占用情况，其中24110是tomcat进程pid

2、生成dump文件，又叫堆转储文件
1）生成堆转储文件方法

jmap -dump:format=b,file=meme.hprof  tomcat进程id

文件名通常为*.hprof

用mat或者jprofile工具分析
