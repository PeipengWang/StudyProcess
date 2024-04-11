# Spring+Quartz的定时器使用
## 1、引入依赖
```
<properties>
      <!-- spring版本号 -->
      <spring.version>5.0.2.RELEASE</spring.version>
      <failOnMissingWebXml>false</failOnMissingWebXml>
  </properties>
<!-- spring核心包 -->
<dependency>
        <groupId>org.quartz-scheduler</groupId>
        <artifactId>quartz</artifactId>
        <version>2.1.1</version>
    </dependency>
 <!-- spring核心包 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>
```
## 2、 编写任务类
Spring与Quartz的结合使用，其任务类有两种方式——任务类继承QuartzJobBean类、任务类不继承特定类。

任务类继承QuartzJobBean类，需要重写executeInternal方法：
```

package org.my.SpringQuartz;
import java.text.SimpleDateFormat;
import java.util.Date;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.springframework.scheduling.quartz.QuartzJobBean;
public class SpringQuartzJob extends QuartzJobBean {
    @Override
    protected void executeInternal(JobExecutionContext arg0) throws JobExecutionException { 
        SimpleDateFormat sdf  = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println("这是SpringQuartzJob定时任务...任务类继承QuartzJobBean,当前时间："+sdf.format(new Date()));  
    }
}
```
或者
任务类不继承特定类，POJO，方法名自定义：

```
package org.my.SpringQuartz;
import java.text.SimpleDateFormat;
import java.util.Date;
public class NotExtendSpringQuartzJob {
    public void dosomething(){
        SimpleDateFormat sdf  = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println("这是NotExtendSpringQuartzJob定时任务...任务类不继承特定类，是POJO,现在时间："+sdf.format(new Date()));
    }
}
```
## 3、Spring配置
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
                        http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context-4.0.xsd
                        http://www.springframework.org/schema/mvc
                        http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">
<!-- 定时任务配置（任务类继承特定的类） 开头 -->
    <!-- 定时任务bean,任务执行类 -->
    <bean name="springQuartzJob" class="org.springframework.scheduling.quartz.JobDetailFactoryBean" >
        <property name="jobClass" value="org.my.SpringQuartz.SpringQuartzJob" />
        <!-- 任务类的属性map,可以设置参数，我这里没有属性 -->
        <!-- <property name="jobDataAsMap">
            <map>
            <entry key="timeout" value="0" />
            </map>
        </property>   -->
    </bean>
    <!-- 任务执行类对应的触发器：第一种（按照一定的时间间隔触发） -->
    <bean name="springQuartzJobTrigger1" class="org.springframework.scheduling.quartz.SimpleTriggerFactoryBean">
        <property name="jobDetail" ref="springQuartzJob" />
        <property name="startDelay" value="0" /><!-- 调度工厂实例化后，经过0秒开始执行调度 -->
        <property name="repeatInterval" value="30000" /><!-- 每30秒调度一次 -->
    </bean>
    <!-- 任务执行类对应的触发器：第二种（按照指定时间触发，如每天12点） -->
    <bean name="springQuartzJobTrigger2" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
        <property name="jobDetail" ref="springQuartzJob" />
        <!-- 每天21:37运行一次 -->
        <property name="cronExpression" value="0 37 21 * * ?" />
    </bean>
    <!-- 调度工厂 -->
    <bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
        <property name="triggers">
        <list>
            <ref bean="springQuartzJobTrigger1" />
            <ref bean="springQuartzJobTrigger2" />
        </list>
        </property>
    </bean>
<!-- 定时任务配置（任务类继承特定的类） 结尾 -->
<!-- 定时任务配置（任务类不继承特定的类）开头 -->
    <!-- 定时任务bean,任务执行类 -->
     <bean id="notExtendSpringQuartzJob"  class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
        <property name="targetObject">  <!-- 指定任务类 -->
            <bean class="org.my.SpringQuartz.NotExtendSpringQuartzJob" />
        </property>
        <property name="targetMethod" value="dosomething" />  <!-- 制定任务类中的方法 -->
        <property name="concurrent" value="false" /><!-- 作业不并发调度 -->
    </bean>
    <!-- 任务执行类对应的触发器：第一种（按照一定的时间间隔触发） -->
    <bean name="notExtendSpringQuartzJobTrigger1" class="org.springframework.scheduling.quartz.SimpleTriggerFactoryBean">
        <property name="jobDetail" ref="notExtendSpringQuartzJob" />
        <property name="startDelay" value="0" /><!-- 调度工厂实例化后，经过0秒开始执行调度 -->
        <property name="repeatInterval" value="30000" /><!-- 每30秒调度一次 -->
    </bean>
    <!-- 任务执行类对应的触发器：第二种（按照指定时间触发，如每天12点） -->
    <bean name="notExtendSpringQuartzJobTrigger2" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
        <property name="jobDetail" ref="notExtendSpringQuartzJob" />
        <!-- 每天21:37运行一次 -->
        <property name="cronExpression" value="0 37 21 * * ?" />
    </bean>
    <!-- 调度工厂 -->
    <bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
        <property name="triggers">
        <list>
            <ref bean="notExtendSpringQuartzJobTrigger1" />
            <ref bean="notExtendSpringQuartzJobTrigger2" />
        </list>
        </property>
    </bean>
<!-- 定时任务配置（任务类不继承特定的类）结尾 -->
</beans>
```
