项目开发中经常要用到一些定时任务，例如每天在固定时间分析一次前一天的日志信息。Spring为我们提供了异步执行任务调度方式，提供了TaskExecutor、TaskSheduler接口。
两个注解：@EnableSheduling、@Sheduled
首先要开启定时任务的注解@EnableSheduling

```
package com.uestc.springboot_task;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@EnableScheduling //开启基于注解的定时任务
@SpringBootApplication
public class SpringbootTaskApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootTaskApplication.class, args);
    }

}

```
然后设置定时任务：

```java
package com.uestc.springboot_task.service;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

@Service
public class ScheduledService {

    /*秒 分 时 日 月 周几
    * 0 * * * * MON-FRI
    * 每个位置代表一个时间的是指
    * */
    @Scheduled(cron = "0 * * * * MON-FRI" )
    public void hello(){
        System.out.println("定时任务处理中。。。。");
    }
}

```
其中cron来设置表达式来设置启动时间，上面设置的是周一到周五每一分钟的0秒就会启动一次，执行一次任务。表达式设置如下：
例如枚举  "，"  设置没一分钟的1,2,3,4秒钟执行一次 1,2,3,4  *  *  *  *  *
 区间 ： 1-4  *  *  *  *  *
 设置步长每4s打印一次：1/4  *  *  *  *  *
 特别注意星期可以用数字0-7，其中0和7都代表周日，或者用SUN-SAT来表示。
 可以在这里借助工具自动生成corn：[corn自动生成器](http://www.bejson.com/othertools/cron/)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201130135932303.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)

输出结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201130135959757.png)



