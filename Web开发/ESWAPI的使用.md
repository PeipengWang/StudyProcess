# ESAPI的使用
依赖jar包
esapi-2.2.3.1.jar
log4j-1.2.17.jar

配置文件
1、ESAPI.properties
```
ESAPI.printProperties=true

# 这个地方是配置ESAPI的实现类，项目中用到那个就选择性配置即可
ESAPI.Encoder=org.owasp.esapi.reference.DefaultEncoder
ESAPI.Validator=org.owasp.esapi.reference.DefaultValidator

# 以下是选择性配置，可以在官网找到对应的介绍
ESAPI.Logger=org.owasp.esapi.logging.java.JavaLogFactory
Encoder.AllowMultipleEncoding=false
Encoder.AllowMixedEncoding=false
Encoder.DefaultCodecList=HTMLEntityCodec,PercentCodec,JavaScriptCodec
Logger.ApplicationName=ExampleApplication
Logger.LogEncodingRequired=false
Logger.LogApplicationName=true
Logger.LogServerIP=true
Logger.LogFileName=ESAPI_logging_file
Logger.MaxLogFileSize=10000000
Logger.UserInfo=true
Logger.ClientInfo=true

```
2、 validation.properties
```


ESAPI.printProperties=true

# 这个地方是配置ESAPI的实现类，项目中用到那个就选择性配置即可
ESAPI.Encoder=org.owasp.esapi.reference.DefaultEncoder
ESAPI.Validator=org.owasp.esapi.reference.DefaultValidator

# 以下是选择性配置，可以在官网找到对应的介绍
ESAPI.Logger=org.owasp.esapi.logging.java.JavaLogFactory
Encoder.AllowMultipleEncoding=false
Encoder.AllowMixedEncoding=false
Encoder.DefaultCodecList=HTMLEntityCodec,PercentCodec,JavaScriptCodec
Logger.ApplicationName=ExampleApplication
Logger.LogEncodingRequired=false
Logger.LogApplicationName=true
Logger.LogServerIP=true
Logger.LogFileName=ESAPI_logging_file
Logger.MaxLogFileSize=10000000
Logger.UserInfo=true
Logger.ClientInfo=true

```
3、 esapi-java-logging.properties
```
handlers= java.util.logging.ConsoleHandler.level= INFO
java.util.logging.ConsoleHandler.level = INFO
java.util.logging.ConsoleHandler.formatter = java.util.logging.SimpleFormatter
java.util.logging.SimpleFormatter.format=[%1$tF %1$tT] [%3$-7s] %5$s %n

```
使用：
```
import org.owasp.esapi.ESAPI;

public class decode {
    public static void main(String[] args) {
        String safe = ESAPI.encoder().encodeForHTML("<script>alert('xss')</script>");
        System.out.println(safe);
    }
}

```
在项目中测试：
在LoginAction类execute()中插入
```
System.out.println( ESAPI.encoder().encodeForHTML("test......"+"<script>"));
```
可以运行成功