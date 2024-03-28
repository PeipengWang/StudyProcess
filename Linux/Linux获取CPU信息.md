# 获取CPU数目
cat /proc/cpuinfo| grep "cpu cores"| uniq  

总核数 = 物理CPU个数 X 每颗物理CPU的核数   
总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数  
# 查看物理CPU个数  
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l  

# 查看每个物理CPU中core的个数(即核数)  
cat /proc/cpuinfo| grep "cpu cores"| uniq  
# 查看逻辑CPU的个数  
cat /proc/cpuinfo| grep "processor"| wc -l  
# 查看CPU信息（型号）  
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c  
# 查看内 存信息  
 cat /proc/meminfo  

# 读取cpu的温度  
cpu0：  
cat /sys/class/thermal/thermal_zone0/temp  
cpu1：  
cat /sys/class/thermal/thermal_zone1/temp  
信息读取，数据除以1000就是温度℃，如下为86.55℃  
 
# 获取CPU运行频率
cat /sys/devices/system/cpu/cpufreq/policy0/cpuinfo_cur_freq  
或者  
cat /proc/cpuinfo | grep -v "grep" | grep "cpu MHz" | tail -1 | grep '{print $4}  
# 查看内存状态  
free命令用来查看内存状态  
对于输出的第一行，我们先纵向看，可以发现除去第一列，后面一共有六列，分别为total、used、free、shared、buffers、cached  
              total        used        free      shared  buff/cache   available  
Mem:        8010100     2058440      653504     1034376     5298156     4611304  
Swap:       8257532       36464     8221068
total：物理内存大小，就是机器实际的内存  
used：已使用的内存大小，这个值包括了 cached 和 应用程序实际使用的内存  
free：未被使用的内存大小  
shared：共享内存大小，是进程间通信的一种方式  
buffers：被缓冲区占用的内存大小  
cached：被缓存占用的内存大小  
 

