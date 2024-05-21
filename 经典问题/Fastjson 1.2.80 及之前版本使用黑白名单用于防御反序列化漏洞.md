# Fastjson 1.2.80 及之前版本使用黑白名单用于防御反序列化漏洞  

影响版本：Fastjson 1.2.80 及之前版本  
Fastjson 是阿里巴巴开源的 Java 对象和 JSON 格式字符串的快速转换的工具库。  
Fastjson 1.2.80 及之前版本使用黑白名单用于防御反序列化漏洞，经研究该防御策略在特定条件下可绕过默认 autoType 关闭限制，攻击远程服务器，风险影响较大。建议 Fastjson 用户尽快采取安全措施保障系统安全。  
影响版本：<=1.2.80  
使用版本：1.1.46  
在1.2.68之后的版本，在1.2.68版本中，fastjson增加了safeMode的支持。safeMode打开后，完全禁用autoType。所有的安全修复版本sec10也支持SafeMode配置。  
fastjson在1.2.68及之后的版本中引入了safeMode，配置safeMode后，无论白名单和黑名单，都不支持autoType，可杜绝反序列化Gadgets类变种攻击（关闭autoType注意评估对业务的影响）。  
autotype 是 Fastjson 中的一个重要机制，粗略来说就是用于设置能否将 JSON 反序列化成对象。  
但是其实从源码实现来看，即使没有开启 autoype，也有一些情况下是可以反序列化出对象来的，具体的情况在后面再分析。  
远古版本 Fastjson < 1.2.10  
启动方式：  
1、在代码中配置  
ParserConfig.getGlobalInstance().setSafeMode(true);   
2、加上JVM启动参数  
-Dfastjson.parser.safeMode=true  
3、通过fastjson.properties文件配置。  
通过类路径的fastjson.properties文件来配置，配置方式如下：  
fastjson.parser.safeMode=true  

选择方式1    
 

选择方式2  
 加上JVM启动参数  
  -Dfastjson.parser.safeMode=true  
