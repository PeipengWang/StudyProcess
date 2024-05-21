1、在nginx上加头部
配置原因：请求转发到 应用后，应用重定向了，导致头确实，需要在nginx中配置如下：
proxy_set_header Host $http_host;

2、在tomcat中配置静态资源路径访问
```
<Context path="lbs-9ebd9753-ad50-4b2d-a17b-ba703a16950d/v1/uniportal" docBase="/home/zxin10/was/tomcat/webapps/uniportal" />
