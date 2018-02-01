# Filter

在SpringBoot中使用过滤器有两种方式

## 使用注解直接注入

``` java
package com.jiangyx.filter;

import org.springframework.stereotype.Component;

import javax.servlet.*;
import java.io.IOException;
import java.util.Date;

@Component
public class TimeFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("初始化");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("time filter start");
        long start = new Date().getTime();
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("time filter耗时为:" + (new Date().getTime() - start));
        System.out.println("time filter stop");
    }

    @Override
    public void destroy() {
        System.out.println("销毁");
    }
}

```

直接定义上述Filter后SpringBoot会自动识别实现Filter的类，自动的进行注册，缺点是无法自定义拦截的url

## 使用@Bean注解注入

``` java

@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean timeFilter() {
        FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        TimeFilter timeFilter = new TimeFilter();
        registrationBean.setFilter(timeFilter);

        List<String> urls = new ArrayList<>();
        // 设置拦截的URL
        urls.add("/*");
        registrationBean.setUrlPatterns(urls);
        return registrationBean;
    }
}

```

通过上述注册即可对对应的URL进行过滤

# Inteceptor

``` java
package com.jiangyx.interceptor;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Date;

@Component
public class TimeInterceptor implements HandlerInterceptor {
    @Override
    // 方法运行前处理
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        System.out.println("time interceptor preHandle");
        httpServletRequest.setAttribute("time", new Date().getTime());
        // 如果返回false则不会条用控制器
        return true;
    }

    @Override
    // 方法执行正确后运行
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("time interceptor postHandle");
        long start = (long) httpServletRequest.getAttribute("time");
        System.out.println("time interceptor 耗时:" + (new Date().getTime() - start));
    }

    @Override
    // 无论方法是否正确都会运行
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        System.out.println("time interceptor afterCompletion");
        long start = (long) httpServletRequest.getAttribute("time");
        System.out.println("time interceptor 耗时:" + (new Date().getTime() - start));
    }
}

```

使用上述代码进行拦截器的创建。缺点是无法获取到方法中的参数。

原因是因为在分发请求的Servlet中，参数的拼装在拦截器的preHandle方法之后

``` java
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {

    @Autowired
    private TimeInterceptor timeInterceptor;

    @Override
    // 注册拦截器
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(timeInterceptor);
    }

    @Bean
    public FilterRegistrationBean timeFilter() {
        FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        TimeFilter timeFilter = new TimeFilter();
        registrationBean.setFilter(timeFilter);

        List<String> urls = new ArrayList<>();
        urls.add("/*");
        registrationBean.setUrlPatterns(urls);
        return registrationBean;
    }
}

```

# Aspect

``` java
@Aspect
@Component
public class TimeAspect {

    // @Around设置切面的执行时机
    // @Around中的表达式设置了切面的触发范围
    @Around("execution(* com.jiangyx.controller.UserController.*(..))")
    public Object handleControllerMethod(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("time aspect start");
        Object[] objs = pjp.getArgs();
        Arrays.stream(objs).forEach(item -> System.out.println(item.toString()));
        long start = new Date().getTime();
        Object o = pjp.proceed();
        System.out.println("time aspect耗时为:" + (new Date().getTime() - start));
        System.out.println("tim aspect stop");
        return o;
    }
}

```

# 总结

三种拦截方式的顺序是

filter > interceptor > aspect > 控制器中的方法

如果控制器抛出异常的响应顺序

aspect > @ControllerAdvice > interceptor > filter