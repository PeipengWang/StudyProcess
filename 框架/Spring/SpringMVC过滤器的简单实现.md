# SpringMVC过滤器和过滤器的简单实现
过滤器实际上是servlet里面的内容，在很多地方时可以通用的
需要再web.xml里配置

```xml
    <filter>
        <filter-name>tokenFilter</filter-name>
        <filter-class>com.wpp.filter.TokenFilter</filter-class>
        <init-param>
            <param-name>allowedMethods</param-name>
            <param-value>GET,POST</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>tokenFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```
代码实现：基本逻辑是实现Filter这个servlet里面的接口，重写doFilter代码，过滤成功则执行chain.doFilter(request, response);这意味着要执行下一个过滤器

```java
package com.wpp.filter;

import org.springframework.stereotype.Component;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

@Component
public class TokenFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("初始化"+this.getClass());
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        System.out.println("经过过滤器");
        System.out.println(req.getSession().getId());
        System.out.println(req.getSession().getAttribute("admin"));
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {
        System.out.println("销毁"+this.getClass());
    }
}

```

# 拦截器
这里的拦截器是SpringMVC里面的内容，需要实现HandlerInceptor接口
java代码

```java
package com.wpp.interceptor;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;


public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MyInterceptor 拦截器执行postHandle()方法");
        HttpSession session = request.getSession();
        // 检查用户是否已登录
        if (session.getAttribute("admin") == null) {
            // 用户未登录，重定向到登录页面或返回错误信息
            response.sendRedirect(request.getContextPath() + "/");
            return false;
        }

        // 用户已登录，允许继续访问
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("MyInterceptor 拦截器执行afterCompletion方法");
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("MyInterceptor 拦截器执行postHandle()方法");
    }
}

```
因为是Spring里面的，这个需要再spring的xml配置文件里配置

```xml
    <!-- 配置拦截器 -->
    <mvc:interceptors>
        <mvc:interceptor>
            <!-- 配置拦截器作用的路径 -->
            <mvc:mapping path="/**" />
            <!-- 配置不需要拦截作用的路径 -->
            <mvc:exclude-mapping path="/login" />
            <!-- 定义<mvc:interceptor>元素中，表示匹配指定路径的请求才进行拦截 -->
            <bean class="com.wpp.interceptor.LoginInterceptor" />
        </mvc:interceptor>
   </mvc:interceptors>
```
还有一种通过注解配置方式来进行的，基本实现是

```java
package com.wpp.config;


import com.wpp.interceptor.GlobalInceptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MyWebMvcConfig implements WebMvcConfigurer {

    @Autowired
    GlobalInceptor globalInceptor;
    /**
     * 将自定义拦截器作为bean写入配置
     * @return
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {

        //拦截处理操作的匹配路径
        //放开静态拦截
        registry.addInterceptor(globalInceptor)
                //拦截所有路径
                .addPathPatterns("/**")
                //排除路径
                .excludePathPatterns("/login", "/exit", "/get_cpacha")
                //排除静态资源拦截
                .excludePathPatterns("/xadmin/**");

    }

}
```
