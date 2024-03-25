# nginx动静分离
为了提高网站的响应速度，减轻程序服务器（Tomcat，Jboss等）的负载，对于静态资源，如图片、js、css等文件，可以在反向代理服务器中进行缓存，这样浏览器在请求一个静态资源时，代理服务器就可以直接处理，而不用将请求转发给后端服务器。对于用户请求的动态文件，如servlet、jsp，则转发给Tomcat，Jboss服务器处理，这就是动静分离。即动态文件与静态文件的分离。
##  动静分离原理
动静分离可通过location对请求url进行匹配，将网站静态资源（HTML，JavaScript，CSS，img等文件）与后台应用分开部署，提高用户访问静态代码的速度，降低对后台应用访问。通常将静态资源放到nginx中，动态资源转发到tomcat服务器中。
[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-6DOYO4Fb-1690195463244)(vx_images/152464491826923.png)]
## 配置
```
    location / {
           proxy_pass http://static_server;
     }
    location ~  .*\.(css)$  {
         root   /usr/share/nginx/dss;
     }
     location ~ .*\.(htm|html|gif|jpg|jpeg|png|gif|bmp|swf|ioc|rar|zip|txt|flv|mid|doc|ppt|pdf|xls|mp3|wma) {
         proxy_pass http://static_server;
         expires 5d;
     }
     location ~ .*\.jsp$ {
         proxy_pass http://tomcat_server;
         expires 1h;
     }
```



