---
title: 番外篇HttpSecurity配置意思
url: security_f1
date: 2019-12-08 15:14:23
tags:
  - security
categories:
  - security
---



Security番外篇

这篇主要说的是 HttpSecurity各种配置的方法及意义。这个用到哪里就会说到哪里

<!-- more -->

```java
 http.authorizeRequests()
                .antMatchers("/admin/api/**").hasIpAddress("")
                .antMatchers("/admin/api/**").hasRole("admin")
                .antMatchers("/user/api/**").hasRole("user")
                .antMatchers("/app/api/**", "/captcha.jpg").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .failureHandler(new SecurityAuthenticationFailureHandler())
                .successHandler(new SecurityAuthenticationSuccessHandler())
                .loginPage("/myLogin.html")
                .loginProcessingUrl("/login")
                .permitAll()
                .and()
                .csrf()
                .disable();
```

其实我们需要好好考虑一下HttpSecurity那么多配置方法到底代表什么意思。我们一起来说一下常用的方法吧。

| **方法**                                                 | **说明**                                                     | 返回值                |
| -------------------------------------------------------- | ------------------------------------------------------------ | --------------------- |
| **openidLogin()**                                        | 用于基于 OpenId 的验证                                       | OpenIDLoginConfigurer |
| **headers()**                                            | 将安全标头添加到响应                                         |                       |
| **cors()**                                               | 配置跨域资源共享（ CORS ）                                   |                       |
| **sessionManagement()**                                  | 允许配置会话管理                                             |                       |
| **portMapper()**                                         | 允许配置一个PortMapper(HttpSecurity#(getSharedObject(class)))，其他提供SecurityConfigurer的对象使用 PortMapper 从 HTTP 重定向到 HTTPS 或者从 HTTPS 重定向到 HTTP。默认情况下，Spring Security使用一个PortMapperImpl映射 HTTP 端口8080到 HTTPS 端口8443，HTTP 端口80到 HTTPS 端口443 |                       |
| **jee()**                                                | 配置基于容器的预认证。 在这种情况下，认证由Servlet容器管理   |                       |
| **x509()**                                               | 配置基于x509的认证                                           |                       |
| **rememberMe**                                           | 允许配置“记住我”的验证                                       |                       |
| <a href="#authorizeRequests">**authorizeRequests()**</a> | 允许基于使用HttpServletRequest限制访问                       |                       |
| **requestCache()**                                       | 允许配置请求缓存                                             |                       |
| **exceptionHandling()**                                  | 允许配置错误处理                                             |                       |
| **securityContext()**                                    | 在HttpServletRequests之间的SecurityContextHolder上设置SecurityContext的管理。 当使用WebSecurityConfigurerAdapter时，这将自动应用 |                       |
| **servletApi()**                                         | 将HttpServletRequest方法与在其上找到的值集成到SecurityContext中。 当使用WebSecurityConfigurerAdapter时，这将自动应用 |                       |
| **csrf()**                                               | 添加 CSRF 支持，使用WebSecurityConfigurerAdapter时，默认启用 |                       |
| **logout()**                                             | 添加退出登录支持。当使用WebSecurityConfigurerAdapter时，这将自动应用。默认情况是，访问URL”/ logout”，使HTTP Session无效来清除用户，清除已配置的任何#rememberMe()身份验证，清除SecurityContextHolder，然后重定向到”/login?success” |                       |
| **anonymous()**                                          | 允许配置匿名用户的表示方法。 当与WebSecurityConfigurerAdapter结合使用时，这将自动应用。 默认情况下，匿名用户将使用org.springframework.security.authentication.AnonymousAuthenticationToken表示，并包含角色 “ROLE_ANONYMOUS” |                       |
| **formLogin()**                                          | 指定支持基于表单的身份验证。如果未指定FormLoginConfigurer#loginPage(String)，则将生成默认登录页面 |                       |
| **oauth2Login()**                                        | 根据外部OAuth 2.0或OpenID Connect 1.0提供程序配置身份验证    |                       |
| **requiresChannel()**                                    | 配置通道安全。为了使该配置有用，必须提供至少一个到所需信道的映射 |                       |
| **httpBasic()**                                          | 配置 Http Basic 验证                                         |                       |
| **addFilterAt()**                                        | 在指定的Filter类的位置添加过滤器                             |                       |
| and()                                                    | 重新获得HttpSercurity                                        |                       |
| permitAll()                                              | 不允许匿名访问，不需要权限                                   |                       |

1. <a href="#authorizeRequests">authorizeRequests</a>

   返回值： ExpressionInterceptUrlRegistry

2. <a href="#formLogin">formLogin    </a>         

   返回值 FormLoginConfigurer

3. <a href="#cors">cors</a>       

   返回值  CorsConfigurer

4. sessionManagement    

   返回值 SessionManagementConfigurer

5. <a href="#httpBasic">httpBasic</a>        

   返回值 HttpBasicConfigurer

6.  rememberMe                    

   返回值 RememberMeConfigurer

7. <a href="#csrf">csrf</a>

   返回值 CsrfConfigurer

8. jee() 

   返回值：JeeConfigurer 

9.  <a href="#logout">logout</a>

   返回值： LogoutConfigurer

10. anonymous

    返回值：AnonymousConfigurer

11. exceptionHandling

    返回值：ExceptionHandlingConfigurer

12. userDetailsService(UserDetailsService userDetailsService)

    返回值：HttpSecurity

13. servletApi

    返回值：ServletApiConfigurer

14. x509() 

    返回值：X509Configurer 

15. openidLogin()

    返回值：OpenIDLoginConfigurer 

16. requestCache 

    返回值：RequestCacheConfigurer

17. headers() 

    返回值：HeadersConfigurer 

18. oauth2Login

    返回值：OAuth2LoginConfigurer

19. oauth2Client

    返回值：OAuth2ClientConfigurer

20. oauth2ResourceServer

    返回值：OAuth2ResourceServerConfigurer

21. requiresChannel

    返回值：ChannelRequestMatcherRegistry

22. portMapper  

    返回值：PortMapperConfigurer 

23. beforeConfigure

    返回值：void

24. performBuild

    返回值：DefaultSecurityFilterChain

25. authenticationProvider

    返回值：HttpSecurity

26. securityContext 

    返回值：SecurityContextConfigurer  

27. getAuthenticationRegistry

    返回值：AuthenticationManagerBuilder

28. addFilterAfter(Filter filter, Class<? extends Filter> afterFilter)

    返回值：HttpSecurity

29. addFilterBefore(Filter filter,Class<? extends Filter> beforeFilter)、

    返回值：HttpSecurity

30. addFilter(Filter filter)

    返回值：HttpSecurity

31.  addFilterAt(Filter filter, Class<? extends Filter> atFilter)

    返回值：HttpSecurity

32. requestMatchers（基本不直接使用）

    返回值 RequestMatcherConfigurer

33. requestMatcher(RequestMatcher requestMatcher)（基本不直接使用）

    返回值：HttpSecurity

34. antMatcher(String antPattern)（基本不直接使用）

    返回值：HttpSecurity

35. mvcMatcher(String mvcPattern)（基本不直接使用）

    返回值：HttpSecurity

36. regexMatcher(String pattern)（基本不直接使用）

    返回值：HttpSecurity

    



## <a name="authorizeRequests">1.authorizeRequests()</a>

执行这个方法后，主要是配置各种请求路径，资源与权限的关系；

### 1.配置路径

| 方法            | 说明         |
| --------------- | ------------ |
| anyRequest      | 所有请求     |
| antMatchers     | 配置请求路径 |
| mvcMatchers     |              |
| regexMatchers   |              |
| requestMatchers |              |

执行了这个方法其实才获得了ExpressionUrlAuthorizationConfigurer，下面是介绍：

> ​	Adds URL based authorization based upon SpEL expressions to an application. 
>
> ​	基于应用程序的SpEL表达式的dds基于URL的授权。
>
> At least one {@link org.springframework.web.bind.annotation.RequestMapping} needs to be mapped
>
> to {@link ConfigAttribute}'s for this {@link SecurityContextConfigurer} to have meaning. 
>
> 至少需要映射一个RequestMapping使SecurityContextConfigurer具有ConfigAttribute意义
>
> 



### 2. 配置所需权限操作ExpressionUrlAuthorizationConfigurer



```java
	static final String permitAll = "permitAll";
	private static final String denyAll = "denyAll";
	private static final String anonymous = "anonymous";
	private static final String authenticated = "authenticated";
	private static final String fullyAuthenticated = "fullyAuthenticated";
	private static final String rememberMe = "rememberMe";
```

方法

```
   .antMatchers("/user/api/**").hasRole("user")
   .antMatchers("/admin/api/**").hasIpAddress("")
   .antMatchers("/app/api/**", "/captcha.jpg").permitAll() # 放弃验证但是必须要登录
   .anyRequest().authenticated() # 所有验证
```



| 方法            | 说明 |
| --------------- | ---- |
| hasAnyRole      |      |
| hasRole         |      |
| hasAuthority    |      |
| hasAnyAuthority |      |
| hasIpAddress    |      |
| authenticated   |      |





## 2.<a name="formLogin">formLogin</a>

配置成功 失败的路径和页面

```java
  .failureHandler(new SecurityAuthenticationFailureHandler())
                .successHandler(new SecurityAuthenticationSuccessHandler())
                .loginPage("/myLogin.html")
                .loginProcessingUrl("/login")
                .defaultSuccessUrl("")
                .authenticationDetailsSource(null)
                .failureForwardUrl("")
                .failureUrl("")
                .permitAll()
```



## 3. <a name="cors">cors</a>

跨域

## 5.<a name="httpBasic">httpBasic</a>

asic身份认证，是HTTP 1.0中引入的认证方案之一。虽然方案比较古老，同时存在安全缺陷，但由于实现简单，所以仍有部分场景下在使用，例如Tomcat自带的有个manager项目，访问这个项目需要Basic认证

**默认页面弹出个框**

## 7.<a name="csrf">csrf</a>

禁用csrf 保护功能

## 9.<a name="logout">logout</a>

退出配置  默认的是“logout”

```java
 .addLogoutHandler(null)
                .clearAuthentication()
                .deleteCookies()
                .logoutRequestMatcher()
                .logoutSuccessHandler()
                .logoutSuccessUrl()
                .logoutUrl()
                .defaultLogoutSuccessHandlerFor()
                .invalidateHttpSession()
```

