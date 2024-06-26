# jmeter工具使用
[官方下载](https://jmeter.apache.org/download_jmeter.cgi)
安装好jdk后，下载之后直接运行即可  
## 基本流程

1、首先添加线程组  
线程组：JMeter是由Java实现的，并且使用一个Java线程来模拟一个用户，因此线程组（Thread Group）就是指一组用户的意思，换句话说一个线程组就是一组虚拟用户（virtual users），这些虚拟用户用来模拟访问被测系统。  
1）线程数：这里就是指虚拟用户数，默认的输入是“1”，则表明模拟一个虚拟用户访问被测系统，如果想模拟100个用户，则此处输入100。  
2）Ramp-Up Period (in seconds): 虚拟用户增长时长。不明白别着急，xmeter君给你举个栗子：比如你测试的是一个考勤系统，那么实际用户登录使用考勤系统的时候并不是大家喊1、2、3 - 走起，然后一起登录。实际使用场景可能是9点钟上班，那么从8:30开始，考勤系统会陆陆续续有人开始登录，直到9:10左右，那么如果完全按照用户的使用场景，设计该测试的时候此处应输入40（分钟）* 60（秒）= 2400。但是实际测试一般不会设置如此长的Ramp-Up时间，原因嘛，难道你做一次测试要先等上40分钟做登录操作？一般情况下，可以估计出登录频率最高的时间长度，比如此处可能从8:55到9:00登录的人最多，那这里设置成300秒，如果“线程数”输入为100，则意味着在5分钟内100用户登录完毕。  
3）循环次数：该处设置一个虚拟用户做多少次的测试。默认为1，意味着一个虚拟用户做完一遍事情之后，该虚拟用户停止运行。如果选中“永远”，则意味着测试运行起来之后就根本停不下来了，除非你把它强制咔嚓。  
2、接下来需要设置一下“HTTP请求” 取样器（Sampler）的属性  
没啥需要介绍的，跟postman一样就行  
可以去“选项” > “Log Viewer”看看运行的日志  
3、添加结果监听器  
右击“线程组” > “监听器” > “察看结果树”来查看性能测试过程中请求和响应信息。添加完毕后，保存测试脚本，再次运行。  




