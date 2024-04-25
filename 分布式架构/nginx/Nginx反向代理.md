反向代理与正向代理区别  
　　正向代理：如果把局域网外的 Internet 想象成一个巨大的资源库，则局域网中的客户端要访问 Internet，则需要通过代理服务器来访问，这种代理服务就称为正向代理。  
　　反向代理：其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了  

## 反向代理的几个标签  
###  标签指令-location  
loacation 匹配顺序  
1）location 匹配格式规则前缀有如下几种：  
···
= 开头：表示精确匹配  
^~ 开头：注意这不是一个正则表达式，它的目的是优于正则表达式的匹配；如果该 location 是最佳匹配，则不再进行正则表达式检测。  
~ 开头：表示区分大小写的正则匹配;  
~* 开头：表示不区分大小写的正则匹配  
!~ && !~*：表示区分大小写不匹配的正则和不区分大小写的不匹配的正则  
···
2）location 如果不带前缀就是普通字符串匹配，比如：  
/uri/ 普通字符串匹配  
/ 绝对路径根目录匹配，如果没有其它匹配，任何请求都会匹配到  
3）匹配的搜索顺序优先级如下（从上到下优先级依次递减）：  
注意：  
当有匹配成功时，立刻停止匹配，按照当前匹配规则处理请求  
优先搜索并不意味着优先命中！  
字符串匹配优先搜索，但是只是记录下最长的匹配 ( 如果 ^~ 是最长的匹配，则会直接命中，停止搜索正则 )，然后继续搜索正则匹配，如果有正则匹配，则命中正则匹配，如果没有正则匹配，则命中最长的字符串匹配。  
```
首先匹配 =  
其次匹配 ^~  
再其次按照配置文件的顺序进行正则匹配  
最后是交给 / 进行通用匹配  
```
(location =) > (location 完整路径) > (location ^~ 路径) > (location ~,~* 正则顺序) > (location 部分起始路径) > (/)  
4）location 是否以“／”结尾  
在 ngnix 中 location 进行的是模糊匹配  
没有“/”结尾时，location/abc/def 可以匹配 /abc/defghi 请求，也可以匹配 /abc/def/ghi 等  
而有“/”结尾时，location/abc/def/ 不能匹配 /abc/defghi 请求，只能匹配 /abc/def/anything 这样的请求  
### 标签指令--proxy_pass
proxy_pass 代理规则（是否以“／”结尾）  
（1）配置 proxy_pass 时，当在后面的 url 加上了 /，相当于是绝对路径，则 Nginx 不会把 location 中匹配的路径部分加入代理 uri。  
[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-qFeSCTj6-1690195783164)(vx_images/165721769748951.png)]  
比如下面配置，我们访问 http://IP/proxy/test.html，最终代理到 URL 是 http://127.0.0.1/test.html  
原文:Nginx - 反向代理location与proxy_pass配置规则总结（附样例）  
（2）如果配置 proxy_pass 时，后面没有 /，Nginx 则会把匹配的路径部分加入代理 uri。  
比如下面配置，我们访问 http://IP/proxy/test.html，最终代理到 URL 是 http://127.0.0.1/proxy/test.html  

原文:Nginx - 反向代理location与proxy_pass配置规则总结（附样例）  
###  标签指令--proxy_set_header
Nginx proxy_set_header：即允许重新定义或添加字段传递给代理服务器的请求头。该值可以包含文本、变量和它们的组合  
在没有定义proxy_set_header时会继承之前定义的值。默认情况下，只有两个字段被重定义：  
```
proxy_set_header Host $proxy_host;
proxy_set_header Connection close;
```
如果启用缓存，来自之前请求的头字段“If-Modified-Since”, “If-Unmodified-Since”, “If-None-Match”, “If-Match”, “Range”, 和 “If-Range” 将不会被代理服务器传递。  
一个不会变化的“Host”头请求字段可通过如下方式被传递：
```
proxy_set_header Host       $http_host;  
```
然后，当字段不在请求头中就无法传递了，在这种情况下，可通过设置Host变量，将需传递值赋给Host变量  

proxy_set_header Host       $host;  
此外，服务器名称和端口一起通过代理服务器传递  
```
proxy_set_header Host       $host:$proxy_port;
```
如果请求头的存在空的字段将不会通过代理服务器传递  
```
proxy_set_header Accept-Encoding "";
```
简而言之，proxy_set_header 就是可设置请求头-并将头信息传递到服务器端，不属于请求头的参数中也需要传递时，重定义下即可！  
