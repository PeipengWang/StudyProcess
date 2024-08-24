```
12:20:48 ERROR [Mx4jServiceImpl.org.springframework.scheduling.quartz.SchedulerFactoryBean#1_Worker-1] - JMX service url[159.75.251.138:-1] create has error,msg is java.io.IOException cannot be cast to javax.management.remote.JMXConnector
java.lang.ClassCastException: java.io.IOException cannot be cast to javax.management.remote.JMXConnector
        at org.smartloli.kafka.eagle.common.util.JMXFactoryUtils.connectWithTimeout(JMXFactoryUtils.java:73)
        at org.smartloli.kafka.eagle.core.factory.Mx4jServiceImpl.common(Mx4jServiceImpl.java:227)
        at org.smartloli.kafka.eagle.core.factory.Mx4jServiceImpl.bytesRejectedPerSec(Mx4jServiceImpl.java:82)
        at org.smartloli.kafka.eagle.web.quartz.MBeanQuartz.kafkaMBeanOfflineAssembly(MBeanQuartz.java:158)
        at org.smartloli.kafka.eagle.web.quartz.MBeanQuartz.brokerMbeanOffline(MBeanQuartz.java:115)
        at org.smartloli.kafka.eagle.web.quartz.MBeanQuartz.mbeanQuartz(MBeanQuartz.java:98)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.springframework.util.MethodInvoker.invoke(MethodInvoker.java:283)
        at org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean$MethodInvokingJob.executeInternal(MethodInvokingJobDetailFactoryBean.java:267)
        at org.springframework.scheduling.quartz.QuartzJobBean.execute(QuartzJobBean.java:75)
        at org.quartz.core.JobRunShell.run(JobRunShell.java:202)
        at org.quartz.simpl.SimpleThreadPool$WorkerThread.run(SimpleThreadPool.java:573)
```

有一台机器没有开启JMX