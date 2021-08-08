---
title: Spring Boot + Spring Security 应用开启接口跨域
date: 2019-04-07 11:19:33
tags:
---



## 1.为何要提供跨域资源服务

最近在写的一个项目后端使用了springboot和springsecurity，前端使用Vue和axios（发送ajax）。为了实现前后端分离，后端要开启`CORS（跨站资源共享）`，保证后端能对不同站点下的前端提供服务。
>由于浏览器的同源策略，跨域请求一般会被拒绝，导致ajax无法访问到后端资源。  
>跨域的几种场景，可以判断自己的网站是否跨域：  
>1. 域名不同
>2. 域名相同，端口不同
>3. 域名相同，协议不同，如http和https  

了解了为什么要开启CORS后可以开始编写开启CORS的代码了。
## 2.Spring Boot配置里开启CORS

springboot开启CORS只需在配置里添加CORS的支持，直接编写配置类即可：
```Java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

@Configuration
public class CorsConfig {
    private CorsConfiguration buildConfig() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*"); // 允许的域名
        corsConfiguration.addAllowedHeader("*"); // 允许的请求头
        corsConfiguration.addAllowedMethod("*"); // 允许的方法（post、get等）
        return corsConfiguration;
    }

    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", buildConfig()); // 对接口配置跨域设置
        return new CorsFilter(source);
    }
}
```
## 3.Spring Security配置

由于使用Spring Security后请求都会被它拦截，需要对security也进行配置：
```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    ...

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                // 如果有允许匿名的url，填在下面
                .anyRequest()
                .permitAll()
                .and()
                .formLogin()
                .loginProcessingUrl("/user/login")
                // 自定义处理器，对ajax请求响应json数据
                .successHandler(new AuthenticationSuccessHandler() {
                    @Override
                    public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
                        httpServletResponse.setContentType("application/json;charset=utf-8");
                        PrintWriter out = httpServletResponse.getWriter();
                        // 将结果转换成Json数据，Result是自己编写的结果类
                        JSONObject result = JSONObject.fromObject(new Result(Result.SUCCESS, "登录成功！"));
                        out.write(result.toString());
                        out.flush();
                        out.close();
                    }
                })
                // 自定义处理器，对ajax请求响应json数据
                .failureHandler(new AuthenticationFailureHandler() {
                    @Override
                    public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
                        httpServletResponse.setContentType("application/json;charset=utf-8");
                        PrintWriter out = httpServletResponse.getWriter();
                        // 将结果转换成Json数据，Result是自己编写的结果类
                        JSONObject result = JSONObject.fromObject(new Result(Result.ERROR, "登录失败！"));
                        out.write(result.toString());
                        out.flush();
                        out.close();
                    }
                })
                .usernameParameter("username")
                .passwordParameter("password")
                .permitAll()
                .and()
                .logout()
                .logoutUrl("/logout")
                .permitAll();
        // 开启CORS，关闭CSRF跨域
        http.cors().and().csrf().disable();
    }

    @Override
    public void configure(WebSecurity web) {
        // 设置拦截忽略文件夹，可以对静态资源放行
        web.ignoring().antMatchers("/css/**", "/js/**", "/js/**", "/lib/**", "/upload/**");
    }
}
```
这里有两个坑。
## 3.1设置CORS和CSRF
一是，下面这一句必须要设置，否则请求会直接403 Forbidden
```
http.cors().and().csrf().disable();
```
## 3.2自定义登录处理器

二是，SpringSecurity通常配置有`http.formLogin().successForwardUrl().failureForwardUrl()`这几个配置，但是这些配置都是基于服务器渲染页面的模式，在前后端分离项目中登录只用发送携带登录信息的ajax，服务器响应结果即可，开启这些跳转配置会使ajax无法正常获取响应。  

自定义处理器，登录成功响应对应的结果的Json数据,写到`HttpServletResponse`中:
```java
new AuthenticationSuccessHandler() {
    @Override
    public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
        httpServletResponse.setContentType("application/json;charset=utf-8");
        PrintWriter out = httpServletResponse.getWriter();
        // 将结果转换成Json数据，Result是自己编写的结果类
        JSONObject result = JSONObject.fromObject(new Result(Result.SUCCESS, "登录成功！"));
        out.write(result.toString());
        out.flush();
        out.close();
    }
}
```

## 4.前端设置
在跨域请求的时候，通常会自动发送两个请求。第一个请求方法为OPTION，目的是探测后端是否支持跨域，如果支持跨域再发送真实请求。这里是自动发送不用自己编写代码。  

然后在发送登录请求的时候，会遇到前端ajax发送了username和password参数，但是后端收不到参数而验证失败。这里的原因是，前端ajax比如axios通常以json格式发送数据，请求头中`content-type`为`application/json`，但是springsecurity的登录不支持这种格式所以导致错误。  

解决方法有两种，一是重写SpringSecurity的登录Filter，使其支持json类型数据，操作比较复杂网上有很多相关博客这里就不提了。二是在登录时前端设置数据格式为SpringSecurity支持的application/x-www-form-urlencoded，相关代码如下：
```javascript
let params = new URLSearchParams();
params.append("username", username);
params.append("password", password);
this.axios.post(url, params);
```
## 5.总结
至此CORS就已经开启，前端就可以愉快地使用ajax访问后端了。