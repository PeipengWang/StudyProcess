目的：禁用 URL 重写功能

　　当程序需要为某个客户端的请求创建一个session时，服务器首先检查这个客户端的请求里是否已包含了一个session标识（称为sessionid），如果已包含则说明以前已经为此客户端创建过session，服务器就按照sessionid把这个session检索出来使用（检索不到，会新建一个），如果客户端请求不包含sessionid，则为此客户端创建一个session并且生成一个与此session相关联的sessionid，sessionid的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个sessionid将被在本次响应中返回给客户端保存。  
      保存这个sessionid的方式可以采用cookie，这样在交互过程中浏览器可以自动的按照规则把这个标识发送给服务器。一般这个cookie的名字都是类似于Ssessionid。但cookie可以被人为的禁止，则必须有其他机制以便在cookie被禁止时仍然能够把sessionid传递回服务器。  
     经常被使用的一种技术叫做URL重写，就是把sessionid直接附加在URL路径的后面。还有一种技术叫做表单隐藏字段。就是服务器会自动修改表单，添加一个隐藏字段，以便在表单提交时能够把sessionid传递回服务，这回暴露生成的sessionid。
在 tomcat 中 context.xml 中设置设置禁用 URL 重写功能。  
或者打开cookie  
