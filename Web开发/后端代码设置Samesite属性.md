# Samesite属性设置
目的：防御CSRF
Cookie的属性SameSite如果不配置或者配置为none，则存在CSRF风险。  
SameSite的取值可以为：  
（1）unset（默认）。这种情况浏览器可能会采用自己的策略。  
（2）none。存在CSRF风险。  
（2）lax。大多数情况也是不发送第三方 Cookie，但是导航到目标网址的 Get 请求除外。  
（3）strict。完全禁止第三方 Cookie，跨站点时，任何情况下都不会发送 Cookie。换言之，只有当前网页的 URL 与请求目标一致，才会带上 Cookie  
进展：  
方式一：tomcat8 及以上可以在 /conf/context.xml 文件里面的`<Context>`节点里加一个子节点  
```
<Context>
   <CookieProcessor sameSiteCookies="lax" />
</Context
```
方式二：在过滤器中设置响应头  
```
        String cookieHeader = req.getHeader("Cookie");
        if(cookieHeader != null) {      
        	 String[] cookiesHeaders = cookieHeader.split(";");
             boolean firstHeader = true;
         	 for (String header : cookiesHeaders) {
         		header = header.trim();
         		if(header != null) {
         			String[] headStr = header.split("=");
             		if(headStr[0].equals("名称")&&firstHeader) {
             			firstHeader = false;
             			resp.setHeader("Set-Cookie", String.format("%s; %s", header, "Path=/;HttpOnly=True;Secure=true;SameSite=Lax;"));
             		 }
         		}
              }
        }
```
