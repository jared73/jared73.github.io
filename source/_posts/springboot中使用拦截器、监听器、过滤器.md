
---
title: springboot中使用拦截器、监听器、过滤器
date: 2018-10-12 16:40:52
tags: 拦截
---

&emsp;&emsp;拦截器、过滤器、监听器在web项目中很常见，这里对springboot中怎么去使用做一个总结.

**1. 拦截器(Interceptor)**

   &emsp;&emsp;我们需要对一个类实现HandlerInterceptor接口， 默认会实现其中的三个方法，preHandle,postHandle ,afterCompletion，其中preHandle实在Controller方法调用之前执行，postHandle是在请求处理之后进行调用，但是在视图渲染之前，即Controller方法调用之后执行，afterCompletion方法是在整个请求结束之后被调用，也是在DispatcherServlet渲染了对应识图之后执行，主要用于资源清理工作。

```
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 需要config配置类配合，看config目录
 * 路径只有走DispatcherServlet，才会被拦截，默认静态资源不会被拦截
 * @author jared
 */
public class MyInterceptor implements HandlerInterceptor{
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        //controller方法调用之前
        System.out.println("我没进controller");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        //请求处理之后进行调用，但是在视图被渲染之前，即Controller方法调用之后
        System.out.println("我进拦截器了");
    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        //在整个请求结束之后被调用，也就是在DispatcherServlet 渲染了对应的视图之后执行，主要是用于进行资源清理工作
    }
}
```


  &emsp;&emsp;路径只有走DispacherServlet，才会被拦截，默认静态资源不会被拦截，拦截器需要注册，一般需要在config中注册该拦截器，才能在项目中使用。

```
import com.aits.interceptor.MyInterceptor;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
@Configuration
public class MyWebAppConfigurer extends WebMvcConfigurerAdapter{
    /**
     * 添加拦截器
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor()).addPathPatterns("/*");
        super.addInterceptors(registry);
    }
}

```


**2. 过滤器**

&emsp;&emsp;过滤器需要实现ServletContextListener，需要springboot主方法类加入@ServletComponentScan注解。

```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

/**
 * 会过滤到地址栏为以下参数的请求
 * 需要主类加入@ServletComponentScan
 */
@WebFilter(filterName = "firstFilter",urlPatterns = {
  "/test/*","/hello/*"
})
public class FirstFilter implements Filter {
    private static Logger LOG= LoggerFactory.getLogger(FirstFilter.class);
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest req= (HttpServletRequest) servletRequest;
        String requestURI=req.getRequestURI();
        LOG.info("过滤到的请求-->"+requestURI);
    }

    @Override
    public void destroy() {

    }
}

```



```
@SpringBootApplication
//开启servlet注解
@ServletComponentScan
public class JaredMain {

    public static void main(String[] args) {
        SpringApplication.run(JaredMain.class, args);
    }

}
```

**3. 监听器**


```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
import javax.servlet.annotation.WebListener;

/**
 * 需要主类加入
 * @ServletComponentScan
 */
@WebListener
public class FirstListener implements ServletContextListener{
    private static Logger LOG= LoggerFactory.getLogger(FirstListener.class);
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        LOG.info("firstListener  初始化...");
    }

    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        LOG.info("FirstListener  销毁...");
    }
}

```

- **区别**





1. 拦截器是基于java的反射机制的，而过滤器是基于函数回调
2. 过滤器依赖与servlet容器，而拦截器不依赖与servlet容器
3. 拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用
4. 拦截器可以访问action上下文、值栈里的对象，而过滤器不能
5. 在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次



- **用途**



1. 过滤器，主要的用途是过滤字符编码、做一些业务逻辑判断等。

1. 监听器，做一些初始化的内容添加工作、设置一些基本的内容、比如一些参数或者是一些固定的对象等等

1. 拦截器，在面向切面编程中应用的，就是在你的service或者一个方法前调用一个方法，或者在方法后调用一个方法。是基于JAVA的反射机制
