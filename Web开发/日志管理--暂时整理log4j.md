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
#### 配置输出位置  
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
## log4j2  
依赖  
```
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.14.0</version>
</dependency>

```

