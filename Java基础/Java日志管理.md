# 日志管理
## 作用：
设置日志级别，决定什么日志信息应该被输出、什么日志信息应该被忽略。
## 基本工具
见的日志管理用具有:JDK logging（配置文件：logging.properties） 和log4j(配置文件：log4j.properties) 。
日志工具有很多，应用程序这个框架用这个，另外一个框架用另外一个日志。配置日志就很麻烦。
各自日志提供各自的，
Self4j这个工具提供一个接口，用来管理日志工具，加那个日志的jar包，就使用哪个日志。
SLF4J，即简单日志门面（Simple Logging Facade for Java），不是具体的日志解决方案，它只服务于各种各样的日志系统。按照官方的说法，SLF4J是一个用于日志系统的简单Facade，允许最终用户在部署其应用时使用其所希望的日志系统。实际上，SLF4J所提供的核心API是一些接口以及一个LoggerFactory的工厂类。从某种程度上，SLF4J有点类似JDBC，不过比JDBC更简单，在JDBC中，你需要指定驱动程序，而在使用SLF4J的时候，不需要在代码中或配置文件中指定你打算使用那个具体的日志系统。如同使用JDBC基本不用考虑具体数据库一样，SLF4J提供了统一的记录日志的接口，只要按照其提供的方法记录即可，最终日志的格式、记录级别、输出方式等通过具体日志系统的配置来实现，因此可以在应用中灵活切换日志系统。如果你开发的是类库或者嵌入式组件，那么就应该考虑采用SLF4J，因为不可能影响最终用户选择哪种日志系统。在另一方面，如果是一个简单或者独立的应用，确定只有一种日志系统，那么就没有使用SLF4J的必要。假设你打算将你使用log4j的产品卖给要求使用JDK 1.4 Logging的用户时，面对成千上万的log4j调用的修改，相信这绝对不是一件轻松的事情。但是如果开始便使用SLF4J，那么这种转换将是非常轻松的事情。
## log4j
下载jar包：log4j-1.2.17.jar
Log4j由三个重要的组件构成：日志信息的优先级，日志信息的输出目的地，日志信息的输出格式。日志信息的优先级从高到低有ERROR、WARN、 INFO、DEBUG，分别用来指定这条日志信息的重要程度；日志信息的输出目的地指定了日志将打印到控制台还是文件中；而输出格式则控制了日志信息的显 示内容。
### 基本使用测试
#### 配置文件
log4j.properties
```
log4j.rootLogger=trace,Console,File，logfile //注意包含了很多
## mina 设置日志发送到控制台
log4j.appender.Console=org.apache.log4j.ConsoleAppender 
log4j.appender.Console.layout=org.apache.log4j.PatternLayout 
log4j.appender.Console.layout.ConversionPattern=%d{yyyy-MM-dd HH\:mm\:ss,SSS} %-5p %c{1} %x - %m%n
## File 设置日志输出到指定大小的文件 
log4j.appender.File =org.apache.log4j.RollingFileAppender
log4j.appender.File.Threshold=DEBUG   //不继承父类的
log4j.appender.File.File=./log/mina.log  //路径
log4j.appender.File.MaxFileSize=5120KB  //大小
log4j.appender.File.MaxBackupIndex=10
log4j.appender.File.layout=org.apache.log4j.PatternLayout
log4j.appender.File.layout.ConversionPattern=[VAMS][%d] %p | %m | [%t] %C.%M(%L)%n
## logFile 设置日志输出到指定路劲
log4j.appender.logFile=org.apache.log4j.FileAppender 
log4j.appender.logFile.Threshold=DEBUG
log4j.appender.logFile.ImmediateFlush=true （表示所有消息都会被立即输出）
log4j.appender.logFile.Append=true  （rue表示消息增加到指定文件中，默认就是true）
log4j.appender.logFile.File=./log/test.log 
log4j.appender.logFile.layout=org.apache.log4j.PatternLayout 
log4j.appender.logFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
log4j.appender.logFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
```
#### 测试代码
```
import org.apache.log4j.Logger;
import org.apache.log4j.PropertyConfigurator;

public class testLogger {
    public static void main(String[] args) {
        String path = System.getProperty("user.dir");
        PropertyConfigurator.configure(path+ "/src/log4j.properties"); //读取配置文件路径
        Logger logger = Logger.getLogger(testLogger.class);  //加载本类
        logger.debug(" debug ");     //设置此处debug级别输出的内容
        logger.error(" error ");     //设置此处error级别输出的内容
        logger.trace(" trace ");
        logger.info(" info ");
        logger.warn(" warn ");

    }
}

```
#### 测试结果
```
2021-11-24 16:43:01,788 DEBUG testLogger  -  debug 
2021-11-24 16:43:01,791 ERROR testLogger  -  error 
2021-11-24 16:43:01,791 TRACE testLogger  -  trace 
2021-11-24 16:43:01,791 INFO  testLogger  -  info 
2021-11-24 16:43:01,791 WARN  testLogger  -  warn 
```
先有一个结果，然后对整个代码进行详细的解析
首先，
### 配置方式（properties方式）
对于日志的配置，本质上要解决定义问题：输出什么？输出在哪里？格式是什么？由此分别对这三个问题进行配置。
#### 配置根Logger
其语法为：
log4j.rootLogger = [ level ] , appenderName, appenderName, …
从这里可以得出两个信息：
1、level 是日志记录的优先级：分为OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL或者定义的级别。通过在这里定义的级别，可以控制到应用程序中相应级别的日志信息的开关。比如在这里定 义了INFO级别，则应用程序中所有DEBUG级别的日志信息将不被打印出来。 
2、appenderName是定义的输出位置的名称，这里代表可以有多个位置与名称。
例如：我们把INFO层级以及以上的信息输出到Console和File;
即输出到控制台和本地硬盘文件，其中 Console 与 File 都是自定义命名。
```
log4j.rootLogger=INFO, Console ,File 
    
#Console 
log4j.appender.Console=org.apache.log4j.ConsoleAppender 
log4j.appender.Console.layout=org.apache.log4j.PatternLayout 
log4j.appender.Console.layout.ConversionPattern=%d [%t] %-5p [%c] - %m%n
    
#File
log4j.appender.File = org.apache.log4j.FileAppender
log4j.appender.File.File = d://log4j2.log    //自己的硬盘路径
log4j.appender.File.layout = org.apache.log4j.PatternLayout
log4j.appender.File.layout.ConversionPattern =%d [%t] %-5p [%c] - %m%n
```
这一句设置以为着所有的log都输出
如果为log4j.rootLogger=WARN, 则意味着只有WARN,ERROR,FATAL被输出，DEBUG,INFO将被屏蔽掉
#### 配置输出方式
```
org.apache.log4j.RollingFileAppender(滚动文件，自动记录最新日志)
org.apache.log4j.ConsoleAppender (控制台)
org.apache.log4j.FileAppender (文件)
org.apache.log4j.DailyRollingFileAppender (每天产生一个日志文件)
org.apache.log4j.WriterAppender (将日志信息以流格式发送到任意指定的地方)
```
在上述介绍的输出位置中已经进行初步了解对于日志的输出位置，接下来将会具体了解几种输出位置的定义方式。
首先需要知道配置输出位置的键值对应的来源。
对于键格式为：log4j.appender.appenderName(rootLogger里自定义名字)
对于值格式，Log4j提供的appender有以下几种：
**org.apache.log4j.ConsoleAppender（控制台）方式**
            name:指定Appender的名字.
             target:SYSTEM_OUT 或 SYSTEM_ERR,一般只设置默认:SYSTEM_OUT.
             PatternLayout:输出格式，不设置默认为:%m%n.
**org.apache.log4j.FileAppender（文件）**
            name:指定Appender的名字.
            fileName:指定输出日志的目的文件带全路径的文件名.
            PatternLayout:输出格式，不设置默认为:%m%n.
**org.apache.log4j.RollingFileAppender（文件大小到达指定尺寸的时候产生一个新的文件）**
            name:指定Appender的名字.
　　　  fileName:指定输出日志的目的文件带全路径的文件名.
　　　  PatternLayout:输出格式，不设置默认为:%m%n.
　　　  filePattern:指定新建日志文件的名称格式.
　　　  Policies:指定滚动日志的策略，就是什么时候进行新建日志文件输出日志.
　　      TimeBasedTriggeringPolicy:Policies子节点，基于时间的滚动策略，interval属性用来指定多久滚动一次，默认是1 hour。modulate=true用来调整时间：比如现在是早上3am，interval是4，那么第一次滚动是在4am，接着是8am，12am...而不是7am.
　　　  SizeBasedTriggeringPolicy:Policies子节点，基于指定文件大小的滚动策略，size属性用来定义每个日志文件的大小.
　　　  DefaultRolloverStrategy:用来指定同一个文件夹下最多有几个日志文件时开始删除最旧的，创建新的(通过max属性),默认是7个文件。
**org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件）**
**org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方）**
#### 配置输出格式
Layout：日志输出格式，Log4j提供的layout有以下几种：
org.apache.log4j.HTMLLayout（以HTML表格形式布局），
org.apache.log4j.PatternLayout（可以灵活地指定布局模式），
org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串），
org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等等信息）
(1)HTMLLayout选项：
LocationInfo=true：输出java文件名称和行号，默认值是false。
Title=My Logging： 默认值是Log4J Log Messages。
(2)PatternLayout选项：
ConversionPattern=%m%n：设定以怎样的格式显示消息。
格式化符号说明：
%p：输出日志信息的优先级，即DEBUG，INFO，WARN，ERROR，FATAL。
%d：输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，如：%d{yyyy/MM/dd HH:mm:ss,SSS}。
%r：输出自应用程序启动到输出该log信息耗费的毫秒数。
%t：输出产生该日志事件的线程名。
%l：输出日志事件的发生位置，相当于%c.%M(%F:%L)的组合，包括类全名、方法、文件名以及在代码中的行数。例如：test.TestLog4j.main(TestLog4j.java:10)。
%c：输出日志信息所属的类目，通常就是所在类的全名。
%M：输出产生日志信息的方法名。
%F：输出日志消息产生时所在的文件名称。
%L:：输出代码中的行号。
%m:：输出代码中指定的具体日志信息。
%n：输出一个回车换行符，Windows平台为"rn"，Unix平台为"n"。
%x：输出和当前线程相关联的NDC(嵌套诊断环境)，尤其用到像java servlets这样的多客户多线程的应用中。
%%：输出一个"%"字符。
另外，还可以在%与格式字符之间加上修饰符来控制其最小长度、最大长度、和文本的对齐方式。如：
1) c：指定输出category的名称，最小的长度是20，如果category的名称长度小于20的话，默认的情况下右对齐。
2)%-20c："-"号表示左对齐。
3)%.30c：指定输出category的名称，最大的长度是30，如果category的名称长度大于30的话，就会将左边多出的字符截掉，但小于30的话也不会补空格。

## log4j 的几种配置方式
```
Log4j配置文件实现了输出到控制台、文件、回滚文件、发送日志邮件、输出到数据库日志表、自定义标签等全套功能。
log4j.rootLogger=DEBUG,console,dailyFile,im
log4j.additivity.org.apache=true
# 控制台(console)
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.ImmediateFlush=true
log4j.appender.console.Target=System.err
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n

# 日志文件(logFile)
log4j.appender.logFile=org.apache.log4j.FileAppender
log4j.appender.logFile.Threshold=DEBUG
log4j.appender.logFile.ImmediateFlush=true
log4j.appender.logFile.Append=true
log4j.appender.logFile.File=D:/logs/log.log4j
log4j.appender.logFile.layout=org.apache.log4j.PatternLayout
log4j.appender.logFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 回滚文件(rollingFile)
log4j.appender.rollingFile=org.apache.log4j.RollingFileAppender
log4j.appender.rollingFile.Threshold=DEBUG
log4j.appender.rollingFile.ImmediateFlush=true
log4j.appender.rollingFile.Append=true
log4j.appender.rollingFile.File=D:/logs/log.log4j
log4j.appender.rollingFile.MaxFileSize=200KB
log4j.appender.rollingFile.MaxBackupIndex=50
log4j.appender.rollingFile.layout=org.apache.log4j.PatternLayout
log4j.appender.rollingFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 定期回滚日志文件(dailyFile)
log4j.appender.dailyFile=org.apache.log4j.DailyRollingFileAppender
log4j.appender.dailyFile.Threshold=DEBUG
log4j.appender.dailyFile.ImmediateFlush=true
log4j.appender.dailyFile.Append=true
log4j.appender.dailyFile.File=D:/logs/log.log4j
log4j.appender.dailyFile.DatePattern='.'yyyy-MM-dd
log4j.appender.dailyFile.layout=org.apache.log4j.PatternLayout
log4j.appender.dailyFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 应用于socket
log4j.appender.socket=org.apache.log4j.RollingFileAppender
log4j.appender.socket.RemoteHost=localhost
log4j.appender.socket.Port=5001
log4j.appender.socket.LocationInfo=true
# Set up for Log Factor 5
log4j.appender.socket.layout=org.apache.log4j.PatternLayout
log4j.appender.socket.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# Log Factor 5 Appender
log4j.appender.LF5_APPENDER=org.apache.log4j.lf5.LF5Appender
log4j.appender.LF5_APPENDER.MaxNumberOfRecords=2000
# 发送日志到指定邮件
log4j.appender.mail=org.apache.log4j.net.SMTPAppender
log4j.appender.mail.Threshold=FATAL
log4j.appender.mail.BufferSize=10
log4j.appender.mail.From = xxx@mail.com
log4j.appender.mail.SMTPHost=mail.com
log4j.appender.mail.Subject=Log4J Message
log4j.appender.mail.To= xxx@mail.com
log4j.appender.mail.layout=org.apache.log4j.PatternLayout
log4j.appender.mail.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 应用于数据库
log4j.appender.database=org.apache.log4j.jdbc.JDBCAppender
log4j.appender.database.URL=jdbc:mysql://localhost:3306/test
log4j.appender.database.driver=com.mysql.jdbc.Driver
log4j.appender.database.user=root
log4j.appender.database.password=
log4j.appender.database.sql=INSERT INTO LOG4J (Message) VALUES('=[%-5p] %d(%r) --> [%t] %l: %m %x %n')
log4j.appender.database.layout=org.apache.log4j.PatternLayout
log4j.appender.database.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n

# 自定义Appender
log4j.appender.im = net.cybercorlin.util.logger.appender.IMAppender
log4j.appender.im.host = mail.cybercorlin.net
log4j.appender.im.username = username
log4j.appender.im.password = password
log4j.appender.im.recipient = corlin@cybercorlin.net
log4j.appender.im.layout=org.apache.log4j.PatternLayout
log4j.appender.im.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
```

## log4j.xml
xml方式配置
如下的配置文件，仅列出了三种文件输出目的地的配置，他们相对比较常用，其中值得特别注意的是记录器的 name 和 additivity 这两个属性的作用，详情往下看注解，最好自己试试
### 例一：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
 
<log4j:configuration>
 
    <!-- 将日志信息输出到控制台 -->
    <appender name="ConsoleAppender" class="org.apache.log4j.ConsoleAppender">
        <!-- 设置日志输出的样式 -->
        <layout class="org.apache.log4j.PatternLayout">
            <!-- 设置日志输出的格式 -->
            <param name="ConversionPattern" value="[%d{yyyy-MM-dd HH:mm:ss:SSS}] [%-5p] [method:%l]%n%m%n%n" />
        </layout>
        <!--过滤器设置输出的级别-->
        <filter class="org.apache.log4j.varia.LevelRangeFilter">
            <!-- 设置日志输出的最小级别 -->
            <param name="levelMin" value="WARN" />
            <!-- 设置日志输出的最大级别 -->
            <param name="levelMax" value="ERROR" />
            <!-- 设置日志输出的xxx，默认是false -->
            <param name="AcceptOnMatch" value="true" />
        </filter>
    </appender>
 
    <!-- 将日志信息输出到文件，但是当文件的大小达到某个阈值的时候，日志文件会自动回滚 -->
    <appender name="RollingFileAppender" class="org.apache.log4j.RollingFileAppender">
        <!-- 设置日志信息输出文件全路径名 -->
        <param name="File" value="D:/log4j/RollingFileAppender.log" />
        <!-- 设置是否在重新启动服务时，在原有日志的基础添加新日志 -->
        <param name="Append" value="true" />
        <!-- 设置保存备份回滚日志的最大个数 -->
        <param name="MaxBackupIndex" value="10" />
        <!-- 设置当日志文件达到此阈值的时候自动回滚，单位可以是KB，MB，GB，默认单位是KB -->
        <param name="MaxFileSize" value="10KB" />
        <!-- 设置日志输出的样式 -->
        <layout class="org.apache.log4j.PatternLayout">
            <!-- 设置日志输出的格式 -->
            <param name="ConversionPattern" value="[%d{yyyy-MM-dd HH:mm:ss:SSS}] [%-5p] [method:%l]%n%m%n%n" />
        </layout>
    </appender>
 
    <!-- 将日志信息输出到文件，可以配置多久产生一个新的日志信息文件 -->
    <appender name="DailyRollingFileAppender" class="org.apache.log4j.DailyRollingFileAppender">
        <!-- 设置日志信息输出文件全路径名 -->
        <param name="File" value="D:/log4j/DailyRollingFileAppender.log" />
        <!-- 设置日志每分钟回滚一次，即产生一个新的日志文件 -->
        <param name="DatePattern" value="'.'yyyy-MM-dd-HH-mm'.log'" />
        <!-- 设置日志输出的样式 -->
        <layout class="org.apache.log4j.PatternLayout">
            <!-- 设置日志输出的格式 -->
            <param name="ConversionPattern" value="[%d{yyyy-MM-dd HH:mm:ss:SSS}] [%-5p] [method:%l]%n%m%n%n" />
        </layout>
    </appender>
    
    <!--
     注意：
     1：当additivity="false"时，root中的配置就失灵了，不遵循缺省的继承机制
     2：logger中的name非常重要，它代表记录器的包的形式，有一定的包含关系，试验表明
        2-1：当定义的logger的name同名时，只有最后的那一个才能正确的打印日志
        2-2：当对应的logger含有包含关系时，比如：name=test.log4j.test8 和 name=test.log4j.test8.UseLog4j，则2-1的情况是一样的
        2-3：logger的name表示所有的包含在此名的所有记录器都遵循同样的配置，name的值中的包含关系是指记录器的名称哟！注意啦！
     3：logger中定义的level和appender中的filter定义的level的区间取交集
     4：如果appender中的filter定义的 levelMin > levelMax ，则打印不出日志信息
     -->
     
    <!-- 指定logger的设置，additivity指示是否遵循缺省的继承机制-->
    <logger name="test.log4j.test8.UseLog4j" additivity="false">
        <level value ="WARN"/>
        <appender-ref ref="DailyRollingFileAppender"/>
    </logger>
 
    <!--指定logger的设置，additivity指示是否遵循缺省的继承机制 -->
    <logger name="test.log4j.test8.UseLog4j_" additivity="false">
        <level value ="ERROR"/>
        <appender-ref ref="RollingFileAppender"/>
    </logger>
 
    <!-- 根logger的设置-->
    <root>
        <level value ="INFO"/>
        <appender-ref ref="ConsoleAppender"/>
        <!--<appender-ref ref="DailyRollingFileAppender"/>-->
    </root>
 
</log4j:configuration
```
### 例二：
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
<log4j:configuration xmlns:log4j='http://jakarta.apache.org/log4j/' >

    <appender name="myConsole" class="org.apache.log4j.ConsoleAppender">
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="[%d{dd HH:mm:ss,SSS\} %-5p] [%t] %c{2\} - %m%n" />
        </layout>

        <!--过滤器设置输出的级别-->   
        <filter class="org.apache.log4j.varia.LevelRangeFilter">
            <param name="levelMin" value="debug" />
            <param name="levelMax" value="warn" />
            <param name="AcceptOnMatch" value="true" />
        </filter>
    </appender>

 
    <appender name="myFile" class="org.apache.log4j.RollingFileAppender">
        <param name="File" value="D:/output.log" /><!-- 设置日志输出文件名 -->
        <!-- 设置是否在重新启动服务时，在原有日志的基础添加新日志 -->
        <param name="Append" value="true" />
        <param name="MaxBackupIndex" value="10" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%p (%c:%L)- %m%n" />
        </layout>
    </appender>

 
    <appender name="activexAppender" class="org.apache.log4j.DailyRollingFileAppender">
        <param name="File" value="E:/activex.log" />
        <param name="DatePattern" value="'.'yyyy-MM-dd'.log'" />
        <layout class="org.apache.log4j.PatternLayout">
         <param name="ConversionPattern" value="[%d{MMdd HH:mm:ss SSS\} %-5p] [%t] %c{3\} - %m%n" />
        </layout>
    </appender>

 
    <!-- 指定logger的设置，additivity指示是否遵循缺省的继承机制-->
    <logger name="com.runway.bssp.activeXdemo" additivity="false">
        <priority value ="info"/>
        <appender-ref ref="activexAppender" />
    </logger>

    <!-- 根logger的设置-->
    <root>
        <priority value ="debug"/>
        <appender-ref ref="myConsole"/>
        <appender-ref ref="myFile"/>
    </root>

</log4j:configuration>
```
### xml节点参数
#### xml declaration and DTD

xml配置文件的头部包括两个部分：xml声明和DTD声明。头部的格式如下：
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
#### log4j:configuration (root element)
xmlns:log4j [#FIXED attribute] : 定义log4j的名字空间，取定值"http://jakarta.apache.org/log4j/"

appender [* child] : 一个appender子元素定义一个日志输出目的地
logger [* child] : 一个logger子元素定义一个日志写出器
root [? child] : root子元素定义了root logger

#### appender
appender元素定义一个日志输出目的地。
name [#REQUIRED attribute] : 定义appender的名字，以便被后文引用
class [#REQUIRED attribute] : 定义appender对象所属的类的全名
param [* child] : 创建appender对象时传递给类构造方法的参数
layout [? child] : 该appender使用的layout对象
#### layout
layout元素定义与某一个appender相联系的日志格式化器。
class [#REQUIRED attribute] : 定义layout对象所属的类的全名
param [* child] : 创建layout对象时传递给类构造方法的参数
#### logger
logger元素定义一个日志输出器。
name [#REQUIRED attribute] : 定义logger的名字，以便被后文引用
additivity [#ENUM attribute] : 取值为"true"（默认）或者"false"，是否继承父logger的属性
level [? child] : 定义该logger的日志级别
appender-ref [* child] : 定义该logger的输出目的地
#### root
root：基础日志配置、包括使用的输出器、日志级别等；
param [* child] : 创建root logger对象时传递给类构造方法的参数
level [? child] : 定义root logger的日志级别
appender-ref [* child] : 定义root logger的输出目的地
#### category
category：自定义输出配置；
logger、category 用法一致，可以配置通过 additivity 属性标记是否集成 root 配置；
以上三个总结
```
    <!--自定义日志输出配置，additivity=是否继承 root 配置-->
    <category name="testCategoryLogger" additivity="false">
        <level value="INFO"></level>
        <appender-ref ref="testCategoryAppender"></appender-ref>
        <appender-ref ref="stdout"></appender-ref>
    </category>
    <!--自定义日志输出配置，additivity=是否继承 root 配置-->
    <logger name="testLogger" additivity="false">
        <!--级别-->
        <level value="INFO"></level>
        <!--输出器-->
        <appender-ref ref="testAppender"></appender-ref>
        <appender-ref ref="stdout"></appender-ref>
    </logger>
    <!--基础日志输出配置-->
    <root>
        <level value="INFO"/>
        <!--输出器-->
        <appender-ref ref="stdout"/>
        <appender-ref ref="fileAppender"/>
```
#### level
level元素定义logger对象的日志级别。
class [#IMPLIED attribute] : 定义level对象所属的类，默认情况下是"org.apache.log4j.Level类
value [#REQUIRED attribute] : 为level对象赋值。可能的取值从小到大依次为"all"、"debug"、"info"、"warn"、"error"、"fatal"和"off"。当值为"off"时表示没有任何日志信息被输出
param [* child] : 创建level对象时传递给类构造方法的参数
#### appender-ref
appender-ref元素引用一个appender元素的名字，为logger对象增加一个appender。
ref [#REQUIRED attribute] : 一个appender元素的名字的引用
appender-ref元素没有子元素
#### param
param元素在创建对象时为类的构造方法提供参数。它可以成为appender、layout、filter、errorHandler、level、categoryFactory和root等元素的子元素。
name and value [#REQUIRED attributes] : 提供参数的一组名值对
param元素没有子元素

#### 在xml文件中配置appender和layout
创建不同的Appender对象或者不同的Layout对象要调用不同的构造方法。可以使用param子元素来设定不同的参数值。
创建ConsoleAppender对象
ConsoleAppender的构造方法不接受其它的参数。
... ... ... ...
<appender name="console.log" class="org.apache.log4j.ConsoleAppender">
  <layout ... >
     ... ...

  </layout>
</appender>
... ... ... ...
#### 创建FileAppender对象

可以为FileAppender类的构造方法传递两个参数：File表示日志文件名；Append表示如文件已存在，是否把日志追加到文件尾部，可能取值为"true"和"false"（默认）。
... ... ... ...

<appender name="file.log" class="org.apache.log4j.FileAppender">
  <param name="File" value="/tmp/log.txt" />
  <param name="Append" value="false" />
  <layout ... >

    ... ...

  </layout>

</appender>
... ... ... ...

#### 创建RollingFileAppender对象
除了File和Append以外，还可以为RollingFileAppender类的构造方法传递两个参数：MaxBackupIndex备份日志文件的个数（默认是1个）；MaxFileSize表示日志文件允许的最大字节数（默认是10M）。
... ... ... ...

<appender name="rollingFile.log" class="org.apache.log4j.RollingFileAppender">
  <param name="File" value="/tmp/rollingLog.txt" />
  <param name="Append" value="false" />
  <param name="MaxBackupIndex" value="2" />
  <param name="MaxFileSize" value="1024" />
  <layout ... >
    ... ...
  </layout>
</appender>
... ... ... ...
#### 创建PatternLayout对象
可以为PatternLayout类的构造方法传递参数ConversionPattern。
... ... ... ...
<layout class="org.apache.log4j.PatternLayout>
  <param name="Conversion" value="%d [%t] %p - %m%n" />
</layout>
## log4j2
### 与log4j的区别
log4j是通过一个.properties的文件作为主配置文件的，而现在的log4j 2则已经弃用了这种方式，采用的是.xml，.json或者.jsn这种方式来做，可能这也是技术发展的一个必然性，毕竟properties文件的可阅读性真的是有点差。

依赖
```
<dependency>
     <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
     <version>2.5</version>
</dependency>
<dependency>
     <groupId>org.apache.logging.log4j</groupId>
     <artifactId>log4j-api</artifactId>
     <version>2.5</version>
</dependency>

```
log4j和log4j 2的包路径是不同的，Apache为了区分，包路径都更新了，这样大家甚至可以在一个项目中使用2个版本的日志输出哦！
log4j想要生效，我们需要在web.xml中进行配置，
log4j2就比较简单，以maven工程为例，我们只需要把log4j2.xml放到工程resource目录下就行了。大家记住一个细节点，是log4j2.xml，而不是log4j.xml，xml名字少个2都不行

和log4j相比，主要有这么一些变化，
首先整体结构上变化很大，appender、logger都被集中到了各自的一个根节点下。
xml各节点的名称也采用了新设计，名称直接就是有用信息，不再是之前appender xxx="xxx", param xxx="xxx"的形式。
然后一些属性，包括fileName等，只能作为节点属性配置，不能像log4j那样配置成子节点。
此外，log4j2归档时支持压缩，在RollingFile节点的filePattern属性里将文件名后缀写成gz,zip等压缩格式，log4j2会自动选择相应压缩算法进行压缩。
### log4j2的文件格式
(1).根节点
Configuration有两个属性:status和monitorinterval,有两个子节点:Appenders和Loggers(表明可以定义多个Appender和Logger)。
status用来指定log4j本身的打印日志的级别.
monitorinterval用于指定log4j自动重新配置的监测间隔时间，单位是s,最小是5s.
(2).Appenders节点，
常见的有三种子节点:Console、RollingFile、File.
Console节点用来定义输出到控制台的Appender.
name:指定Appender的名字.
target:SYSTEM_OUT 或 SYSTEM_ERR,一般只设置默认:SYSTEM_OUT.
Patt　ernLayout:输出格式，不设置默认为:%m%n.
　　File节点用来定义输出到指定位置的文件的Appender.
　　　　name:指定Appender的名字.
　　　　fileName:指定输出日志的目的文件带全路径的文件名.
　　　　PatternLayout:输出格式，不设置默认为:%m%n.
　　RollingFile节点用来定义超过指定大小自动删除旧的创建新的的Appender.
　　　　name:指定Appender的名字.
　　　　fileName:指定输出日志的目的文件带全路径的文件名.
　　　　PatternLayout:输出格式，不设置默认为:%m%n.
　　　　filePattern:指定新建日志文件的名称格式.
　　　　Policies:指定滚动日志的策略，就是什么时候进行新建日志文件输出日志.
　　　　TimeBasedTriggeringPolicy:Policies子节点，基于时间的滚动策略，interval属性用来指定多久滚动一次，默认是1 hour。modulate=true用来调整时间：比如现在是早上3am，interval是4，那么第一次滚动是在4am，接着是8am，12am...而不是7am.
　　　　SizeBasedTriggeringPolicy:Policies子节点，基于指定文件大小的滚动策略，size属性用来定义每个日志文件的大小.
　　　　DefaultRolloverStrategy:用来指定同一个文件夹下最多有几个日志文件时开始删除最旧的，创建新的(通过max属性)。
ThresholdFilter属性：onMatch表示匹配设定的日志级别后是DENY还是ACCEPT，onMismatch表示不匹配设定的日志级别是DENY还是ACCEPT还是NEUTRAL

(3).Loggers节点
常见的有两种:Root和Logger.
Root节点用来指定项目的根日志，如果没有单独指定Logger，那么就会默认使用该Root日志输出
level:日志输出级别，共有8个级别，按照从低到高为：All < Trace < Debug < Info < Warn < Error < Fatal < OFF.
Logger节点用来单独指定日志的形式，比如要为指定包下的class指定不同的日志级别等。
　　　　level:日志输出级别，共有8个级别，按照从低到高为：All < Trace < Debug < Info < Warn < Error < Fatal < OFF.
　　　　name:用来指定该Logger所适用的类或者类所在的包全路径,继承自Root节点.
　　　　AppenderRef：Logger的子节点，用来指定该日志输出到哪个Appender,如果没有指定，就会默认继承自Root.如果指定了，那么会在指定的这个Appender和Root的Appender中都会输出，此时我们可以设置Logger的additivity="false"只在自定义的Appender中进行输出。
(4).关于日志level.
　　共有8个级别，按照从低到高为：All < Trace < Debug < Info < Warn < Error < Fatal < OFF.
　　All:最低等级的，用于打开所有日志记录.
　　Trace:是追踪，就是程序推进以下，你就可以写个trace输出，所以trace应该会特别多，不过没关系，我们可以设置最低日志级别不让他输出.
　　Debug:指出细粒度信息事件对调试应用程序是非常有帮助的.
　　Info:消息在粗粒度级别上突出强调应用程序的运行过程.
　　Warn:输出警告及warn以下级别的日志.
　　Error:输出错误信息日志.
　　Fatal:输出每个严重的错误事件将会导致应用程序的退出的日志.
　　OFF:最高等级的，用于关闭所有日志记录.
　　程序会打印高于或等于所设置级别的日志，设置的日志等级越高，打印出来的日志就越少。

配置参数解释
%d{HH:mm:ss.SSS} 表示输出到毫秒的时间
%t 输出当前线程名称
### 例子一：
```
<?xml version="1.0" encoding="UTF-8"?>
 <!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
 <!--Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，你会看到log4j2内部各种详细输出-->
 <!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数-->
 <configuration status="WARN" monitorInterval="30">
     <!--先定义所有的appender-->
     <appenders>
     <!--这个输出控制台的配置-->
         <console name="Console" target="SYSTEM_OUT">
         <!--输出日志的格式-->
             <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
         </console>
     <!--文件会打印出所有信息，这个log每次运行程序会自动清空，由append属性决定，这个也挺有用的，适合临时测试用-->
     <File name="log" fileName="log/test.log" append="false">
        <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M - %msg%xEx%n"/>
     </File>
     <!-- 这个会打印出所有的info及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
         <RollingFile name="RollingFileInfo" fileName="${sys:user.home}/logs/info.log"
                      filePattern="${sys:user.home}/logs/$${date:yyyy-MM}/info-%d{yyyy-MM-dd}-%i.log">
             <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->        
             <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
             <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
             <Policies>
                 <TimeBasedTriggeringPolicy/>
                 <SizeBasedTriggeringPolicy size="100 MB"/>
             </Policies>
         </RollingFile>
         <RollingFile name="RollingFileWarn" fileName="${sys:user.home}/logs/warn.log"
                      filePattern="${sys:user.home}/logs/$${date:yyyy-MM}/warn-%d{yyyy-MM-dd}-%i.log">
             <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/>
             <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
             <Policies>
                 <TimeBasedTriggeringPolicy/>
                 <SizeBasedTriggeringPolicy size="100 MB"/>
             </Policies>
         <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件，这里设置了20 -->
             <DefaultRolloverStrategy max="20"/>
         </RollingFile>
         <RollingFile name="RollingFileError" fileName="${sys:user.home}/logs/error.log"
                      filePattern="${sys:user.home}/logs/$${date:yyyy-MM}/error-%d{yyyy-MM-dd}-%i.log">
             <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
             <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
             <Policies>
                 <TimeBasedTriggeringPolicy/>
                 <SizeBasedTriggeringPolicy size="100 MB"/>
             </Policies>
         </RollingFile>
     </appenders>
     <!--然后定义logger，只有定义了logger并引入的appender，appender才会生效-->
     <loggers>
         <!--过滤掉spring和mybatis的一些无用的DEBUG信息-->
         <logger name="org.springframework" level="INFO"></logger>
         <logger name="org.mybatis" level="INFO"></logger>
         <root level="all">
             <appender-ref ref="Console"/>
             <appender-ref ref="RollingFileInfo"/>
             <appender-ref ref="RollingFileWarn"/>
             <appender-ref ref="RollingFileError"/>
         </root>
     </loggers>
 </configuration>
```
### 例子二
```

<?xml version="1.0" encoding="UTF-8"?>
<!--
    status : 这个用于设置log4j2自身内部的信息输出,可以不设置,当设置成trace时,会看到log4j2内部各种详细输出。
    因此我们直接设置成OFF
 -->
<Configuration status="OFF">
    <!-- 配置输出端  -->
    <Appenders>
        <!-- 输出到控制台  -->
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="[%-level]%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %logger{36} - %msg%n"/>
        </Console>
        <!-- 输出到文件 -->
        <!--
            name:           输出端的名字
            fileName:       指定当前日志文件的位置和文件名称
            filePattern:    指定当发生自动封存日志时，文件的转移和重命名规则
            这个filePatten结合下面的TimeBasedTriggeringPolicy一起使用，可以实现控制日志按天生成文件.
            自动封存日志的策略可以设置时间策略和文件大小策略（见下面的Policies配置）
            时间策略：
                文件名_%d{yyyy-MM-dd}_%i.log  这里%d表示自动封存日志的单位是天
                如果下面的TimeBasedTriggeringPolicy的interval设为1,
                表示每天自动封存日志一次;那么就是一天生成一个文件。
            文件大小策略：
                如果你设置了SizeBasedTriggeringPolicy的size的话，
                超过了这个size就会再生成一个文件，这里的%i用来区分的
            %d{yyyy-MM-dd}会自动替代为日期，如2017-06-30
        -->
        <RollingFile name="RollingFileInfo" fileName="/log/guitool_info.log"
                     filePattern="/log/%d{yyyy-MM-dd}/guitool_%d{yyyy-MM-dd}_%i.log">
            <!-- 只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch） -->
            <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
            <!-- 输出的格式  -->
            <PatternLayout pattern="[%d{yyyy-MM-dd HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
            <!--
                Policies：自动封存日志策略，表示日志什么时候应该产生新日志，
                可以有时间策略和大小策略等，并且：只有满足一个策略，就好生成一个新的文件。
                这里使用的是时间和大小都使用了，每隔1天产生新的日志文件
                如果果今天的文件大小到了设定的size，则会新生成一个文件，上面的%i就表示今天的第几个文件
             -->
            <Policies>
                <TimeBasedTriggeringPolicy interval="1"/>
                <SizeBasedTriggeringPolicy size="20MB"/>
            </Policies>
            <!--
                DefaultRolloverStrategy属性如不设置，
                则默认为最多同一文件夹下7个文件，这里设置了20
             -->
            <DefaultRolloverStrategy max="20"/>
        </RollingFile>
    </Appenders>
    <!-- 配置Loggers  -->
    <Loggers>
        <!--
            Logger： 用于指定部分包的日志级别
                日志级别局部的会覆盖全局的
                比如这里hibernate的级别设为debug，而控制台没有设级别，那么控制台会打印debug级别的日志
                而输出到文件这个输出端设置了info级别，那么hibernate的debug级别的日志还是看不了。
                所以最终输出的级别和输出端设置的级别是有关系的。
            name: 包名
            level：日志级别
            additivity：是否冒泡，既在当前logger的输出端输出日志后
                             是否需要在父输出端上输出该日志，默认为 true。
                             如果设为false，则必须配置AppendRef。
         -->
        <Logger name="org.hibernate" level="debug" additivity="true"/>
        <!-- 这个root是配置全局日志级别和输出端功能和老版的log4j中根的配置是一样的 -->
        <Root level="info">
            <!-- 这里引用上面定义的输出端，千万不要漏了。 -->
            <AppenderRef ref="Console"/>
            <!--<AppenderRef ref="RollingFileInfo"/>-->
        </Root>
    </Loggers>
</Configuration>
```


### Appenders:包含以下标签

FileAppender 　　 普通地输出到本地文件
FlumeAppender　　 将几个不同源的日志汇集、集中到一处
RewriteAppender 　　对日志事件进行掩码或注入信息
RollingFileAppender　　对日志文件进行封存
RoutingAppender　　在输出地之间进行筛选路由
SMTPAppender　　将LogEvent发送到指定邮件列表
SocketAppender　　将LogEvent以普通格式发送到远程主机
SyslogAppender　　将LogEvent以RFC 5424格式发送到远程主机
AsynchAppender 　　将一个LogEvent异步地写入多个不同输出地
ConsoleAppender　　将LogEvent输出到控制台
FailoverAppender　　维护一个队列，系统将尝试向队列中的Appender依次输出LogEvent，直到有一个成功为止
##  log4j.xml通过shell脚本升级到log4j2.xml的流程
1、完全解析log4j.xml
awk指令解析xml文件：
①头部
②解析标签<appender，结束标志 </appender>
    存放数据结构：待确定
    内部子标签与值：
        name，class
   子标签与值：
 <!-- 设置日志信息输出文件全路径名 -->
     <param name="File" value="${catalina.base}/logs/WEBOSLog.log"/>
      <!-- 设置是否在重新启动服务时，在原有日志的基础添加新日志 -->
     <param name="Append" value="true"/>
      <!-- 设置当日志文件达到此阈值的时候自动回滚，单位可以是KB，MB，GB，默认单位是KB -->
     <param name="MaxFileSize" value="5242880" />
      <!-- 设置保存备份回滚日志的最大个数 -->
     <param name="MaxBackupIndex" value="9" />
     <!-- 设置日志输出的样式 -->
     <layout class="com.zte.webos.logger.logFormatter"/>
③解析多个appender标签
    存放数据结构：待确定
④解析<category标签
    内部子标签：name（键）
     外部子标签：<priority   <appender-ref

2、新建log4j2.xml
可以直接将log4j.xml重命名，将第一行以下的数据全部删除
3、将解析好的log4j.xml的数据对应生成到log4j2.xml文件中
根据对应的标签，通过轮询的方式插入数据到xml文件中
首先检测轮询到的列表是<appender还是<category（还有其他很多父标签）
得出轮询到的列表中某数据（键）对应的所有的标签数据
然后依次填充内部标签和子标签
4、删除log4j.xml
## log4j.xml通过shell脚本升级到log4j2.xml的问题：
1、shell缺少对于xml文件进行解析的工具，只能自己写，难以保证质量
2、业务的自行配置是难以进行预测的，需要对log4j所有可能的配置进行考虑
3、解析出的存储结构需要确认，列表和哈希表能嵌套几层
4、没有实例可以参考
5、工作量巨大

## 简便方法（仅适用于当前的配置）
1、搭建起整体不变的结构，包括头部，package，configuration，logger，root标签结构
2、获取所有输出文件的名称
awk  '/<\/*appender name=\/*/{gsub(/[[:space:]]*\/*class=\"org.apache.log4j.RollingFileAppender\">/,"");print $2}' log4j.xml
3、获取对应的输出文件：





