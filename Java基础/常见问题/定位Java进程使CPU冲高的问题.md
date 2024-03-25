# 定位线程高问题
1.使用top 定位到占用CPU高的进程PID
top
2、获取线程信息，并找到占用CPU高的线程。
ps -mp pid -o THREAD,tid,time | sort -rn
3、将需要的线程ID转换为16进制格式
printf "%x\n" tid
4、打印线程的堆栈信息
jstack pid |grep tid -A 30
