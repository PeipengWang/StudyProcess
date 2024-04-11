@[TOC](SpringBoot的异步任务处理)
# 为什么要用异步任务

异步(async)是相对于同步(sync)而言的，很好理解。同步就是一件事一件事的执行。只有前一个任务执行完毕，才能执行后一个任务。而异步比如：

```
   public String setTimeout(){
        asyncService.hello();
        return "success";
    }
    public class AsyncService {
    public void hello(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("数据处理中。。。");
    }
}
```

setTimeout就是这个任务会顺序执行等待3000ms，再返回数据，但是对于这个函数来说，如果等待没有意义，一直这样等下去就太傻了，所以利用注解方式的异步任务来直接跳过这个任务，在3000ms的时候再来执行这个hello函数。
# SpringBoot注解方式的异步任务使用方法
首先在主函数中开启异步注解，让异步注解变为可使用状态：@EnableAsync

```java
package com.uestc.springboot_task;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableAsync;

@EnableAsync //开启异步注解
@SpringBootApplication
public class SpringbootTaskApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootTaskApplication.class, args);
    }

}

```
然后将一个方法设置为异步注解方法： @Async

```java
package com.uestc.springboot_task.service;

import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

//异步任务
@Service
public class AsyncService {
    //注解为异步方法
    @Async
    public void hello(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("数据处理中。。。");
    }
}

```
这样一个简单的异步注解的配置就完成了，编写一个控制层来测试

```java
package com.uestc.springboot_task.controller;

import com.uestc.springboot_task.service.AsyncService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AsyncController {

    @Autowired
    AsyncService asyncService;

    @GetMapping("/hello")
    public String hello(){
        asyncService.hello();
        return "success";
    }
}

```
测试结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201130134101605.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
首先success完成响应，然后控制台打印出结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201130134145815.png)
