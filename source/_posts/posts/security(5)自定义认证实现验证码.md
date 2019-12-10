---
title: security(5)自定义认证实现验证码
url: security5
date: 2019-12-10 11:23:00
tags:
  - security
categories:
  - security
---

## 1. 基本介绍
你好世界

<!-- more -->

全文内容



这里使用**自定义认证**来实现验证码，也就是实现Filter的另外一种方式，其实我也不太熟:joy:,将就看吧。

[TOC]



在Security验证对象其实就是主体**（Principal）**，主体包含了所有能够经过验证而获得系统访问权限的用户，设备或者其他系统。



```java
public interface Authentication extends Principal, Serializable {
	// 主体权限列表
	Collection<? extends GrantedAuthority> getAuthorities();
	// 获取主体凭证 通常为用户密码
	Object getCredentials();
	// 获取主体携带的详细信息
	Object getDetails();
	// 获取主体，通常是用户名
	Object getPrincipal();
	// 主体是否验证成功
	boolean isAuthenticated();

	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```

```java
public interface Authentication extends Principal, Serializable {
	// 获得主体权限
	Collection<? extends GrantedAuthority> getAuthorities();
	// 
	Object getCredentials();
	// 
	Object getDetails();
	// 
	Object getPrincipal();
	// 
	boolean isAuthenticated();
	// 
	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```

AuthenticationProvider 是Security的一个验证过程

```java
public interface AuthenticationProvider {

	// 验证过程 成功返回一个验证完成Authentication
	Authentication authenticate(Authentication authentication)
			throws AuthenticationException;

	// 是否支持验证当前的Authentication类型
	boolean supports(Class<?> authentication);
}

```

验证AuthenticationProvider是靠ProviderManager来管理

```java
public class ProviderManager implements AuthenticationManager, MessageSourceAware,
		InitializingBean {
	=====================================================================================


	private AuthenticationEventPublisher eventPublisher = new NullEventPublisher();
     // 多个验证的过程
	private List<AuthenticationProvider> providers = Collections.emptyList();
	protected MessageSourceAccessor messages = SpringSecurityMessageSource.getAccessor();
	private AuthenticationManager parent;
	private boolean eraseCredentialsAfterAuthentication = true;
==============================================
	// 验证的过程
	public Authentication authenticate(Authentication authentication)
			throws AuthenticationException {
		Class<? extends Authentication> toTest = authentication.getClass();
		AuthenticationException lastException = null;
		AuthenticationException parentException = null;
		Authentication result = null;
		Authentication parentResult = null;
		boolean debug = logger.isDebugEnabled();

		for (AuthenticationProvider provider : getProviders()) {
			if (!provider.supports(toTest)) {
				continue;
			}
			
			try {
                // 验证通过就跳出循环
				result = provider.authenticate(authentication);
				if (result != null) {
					copyDetails(authentication, result);
					break;
				}
			}
			catch (AccountStatusException | InternalAuthenticationServiceException e) {
				prepareException(e, authentication);
				// SEC-546: Avoid polling additional providers if auth failure is due to
				// invalid account status
				throw e;
			} catch (AuthenticationException e) {
				lastException = e;
			}
		}

		if (result == null && parent != null) {
			// Allow the parent to try.
			try {
				result = parentResult = parent.authenticate(authentication);
			}
			catch (ProviderNotFoundException e) {
				// ignore as we will throw below if no other exception occurred prior to
				// calling parent and the parent
				// may throw ProviderNotFound even though a provider in the child already
				// handled the request
			}
			catch (AuthenticationException e) {
				lastException = parentException = e;
			}
		}

		if (result != null) {
			if (eraseCredentialsAfterAuthentication
					&& (result instanceof CredentialsContainer)) {
				// Authentication is complete. Remove credentials and other secret data
				// from authentication
				((CredentialsContainer) result).eraseCredentials();
			}

			// If the parent AuthenticationManager was attempted and successful than it will publish an AuthenticationSuccessEvent
			// This check prevents a duplicate AuthenticationSuccessEvent if the parent AuthenticationManager already published it
			if (parentResult == null) {
				eventPublisher.publishAuthenticationSuccess(result);
			}
			return result;
		}

		// Parent was null, or didn't authenticate (or throw an exception).

		if (lastException == null) {
			lastException = new ProviderNotFoundException(messages.getMessage(
					"ProviderManager.providerNotFound",
					new Object[] { toTest.getName() },
					"No AuthenticationProvider found for {0}"));
		}

		// If the parent AuthenticationManager was attempted and failed than it will publish an AbstractAuthenticationFailureEvent
		// This check prevents a duplicate AbstractAuthenticationFailureEvent if the parent AuthenticationManager already published it
		if (parentException == null) {
			prepareException(lastException, authentication);
		}

		throw lastException;
	}
=====================================================
}

```



## 2.AuthenticationProvider

验证其实就是AuthenticationProvider，所以这里需要自定义验证。其中Security，包含了基本的几种：

1. HTTP层面的认证技术，包括HTTP基本认证和HTTP摘要认证
2. 基于LDAP的认证技术
3. 聚焦于证明用户身份的OpenID认证技术
4. 聚焦于授权的OAuth认证技术
5. 系统内维护的用户名和密码认证技术

其中最广泛的的就是**系统内维护的用户名和密码认证技术**，这里因为需要和数据库交互，Security提供了一个抽象的AuthenticationProvider,**AbstractUserDetailsAuthenticationProvider**:

```java
public abstract class AbstractUserDetailsAuthenticationProvider implements
		AuthenticationProvider, InitializingBean, MessageSourceAware {

// 附加认证过程 -- 核心方法
	protected abstract void additionalAuthenticationChecks(UserDetails userDetails,
			UsernamePasswordAuthenticationToken authentication)
			throws AuthenticationException;


	// 认证过程
	public Authentication authenticate(Authentication authentication)
			throws AuthenticationException {
		Assert.isInstanceOf(UsernamePasswordAuthenticationToken.class, authentication,
				() -> messages.getMessage(
						"AbstractUserDetailsAuthenticationProvider.onlySupports",
						"Only UsernamePasswordAuthenticationToken is supported"));

		// Determine username
		String username = (authentication.getPrincipal() == null) ? "NONE_PROVIDED"
				: authentication.getName();

		boolean cacheWasUsed = true;
		UserDetails user = this.userCache.getUserFromCache(username);

		if (user == null) {
			cacheWasUsed = false;

			try {
                // 先检索用户
				user = retrieveUser(username,
						(UsernamePasswordAuthenticationToken) authentication);
			}
			catch (UsernameNotFoundException notFound) {
				logger.debug("User '" + username + "' not found");

				if (hideUserNotFoundExceptions) {
					throw new BadCredentialsException(messages.getMessage(
							"AbstractUserDetailsAuthenticationProvider.badCredentials",
							"Bad credentials"));
				}
				else {
					throw notFound;
				}
			}

			Assert.notNull(user,
					"retrieveUser returned null - a violation of the interface contract");
		}

		try {
            // 检查用户账户是否可用
			preAuthenticationChecks.check(user);
            // 附加认证
			additionalAuthenticationChecks(user,
					(UsernamePasswordAuthenticationToken) authentication);
		}
		catch (AuthenticationException exception) {
			if (cacheWasUsed) {
				// There was a problem, so try again after checking
				// we're using latest data (i.e. not from the cache)
				cacheWasUsed = false;
				user = retrieveUser(username,
						(UsernamePasswordAuthenticationToken) authentication);
				preAuthenticationChecks.check(user);
				additionalAuthenticationChecks(user,
						(UsernamePasswordAuthenticationToken) authentication);
			}
			else {
				throw exception;
			}
		}
		// 检查用户密码是否过期
		postAuthenticationChecks.check(user);

		if (!cacheWasUsed) {
			this.userCache.putUserInCache(user);
		}

		Object principalToReturn = user;

		if (forcePrincipalAsString) {
			principalToReturn = user.getUsername();
		}
		// 返回一个认证通过的Authentication
		return createSuccessAuthentication(principalToReturn, authentication, user);
	}
	
	// 检索用户 （核心方法）
	protected abstract UserDetails retrieveUser(String username,
			UsernamePasswordAuthenticationToken authentication)
			throws AuthenticationException;
    
    // 此认证过程支持UsernamePasswordAuthenticationToken及衍生对象
    public boolean supports(Class<?> authentication) {
		return (UsernamePasswordAuthenticationToken.class
				.isAssignableFrom(authentication));
	}

	
}

```

还有一个重要的实现类**DaoAuthenticationProvider**，主要实现了密码的编码



## 3.实际编码

因为这个我们需要HttpRequest  所以需要**MyWebAuthenticationDetailsSource**，然后补充到**MyWebAuthenticationDetails**实际验证验证码，最后通过**MyAuthenticationProvider**也就**DaoAuthenticationProvider**来执行。



```java
@Component
public class MyAuthenticationProvider extends DaoAuthenticationProvider {


    // 构造方法注入UserDetailService和PasswordEncoder
    public MyAuthenticationProvider(UserDetailsService userDetailsService, PasswordEncoder passwordEncoder) {
        this.setUserDetailsService(userDetailsService);
        this.setPasswordEncoder(passwordEncoder);
    }

    @Override
    protected void additionalAuthenticationChecks(UserDetails userDetails, UsernamePasswordAuthenticationToken usernamePasswordAuthenticationToken) throws AuthenticationException {
        MyWebAuthenticationDetails details = (MyWebAuthenticationDetails) usernamePasswordAuthenticationToken.getDetails();
        // 验证码验证逻辑
        String imageCode = details.getImageCode();
        String savedImageCode = details.getSavedImageCode();
        // 检验图形验证码
        if (StringUtils.isEmpty(imageCode) || StringUtils.isEmpty(savedImageCode) || !imageCode.equals(savedImageCode)) {
            throw new VerificationCodeException();
        }
        // 调用父类进行密码验证
        super.additionalAuthenticationChecks(userDetails, usernamePasswordAuthenticationToken);
    }
}

```

```java
public class MyWebAuthenticationDetails extends WebAuthenticationDetails {

    private String imageCode;

    private String savedImageCode;

    public String getImageCode() {
        return imageCode;
    }

    public String getSavedImageCode() {
        return savedImageCode;
    }

    // 补充用户提交的验证码和session保存的验证码
    public MyWebAuthenticationDetails(HttpServletRequest request) {
        super(request);
        this.imageCode = request.getParameter("captcha");
        HttpSession session = request.getSession();
        this.savedImageCode = (String) session.getAttribute("captcha");
        if (!StringUtils.isEmpty(this.savedImageCode)) {
            // 随手清除验证码，不管是失败还是成功，所以客户端应在登录失败时刷新验证码
            session.removeAttribute("captcha");
        }
    }

}
```

```java
@Component
public class MyWebAuthenticationDetailsSource implements AuthenticationDetailsSource<HttpServletRequest, WebAuthenticationDetails> {

    @Override
    public WebAuthenticationDetails buildDetails(HttpServletRequest request) {
        return new MyWebAuthenticationDetails(request);
    }

}
```

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private AuthenticationDetailsSource<HttpServletRequest, WebAuthenticationDetails> myWebAuthenticationDetailsSource;

    @Autowired
    private AuthenticationProvider authenticationProvider;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 应用AuthenticationProvider
        auth.authenticationProvider(authenticationProvider);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/admin/api/**").hasRole("ADMIN")
                .antMatchers("/user/api/**").hasRole("USER")
                .antMatchers("/app/api/**", "/captcha.jpg").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .authenticationDetailsSource(myWebAuthenticationDetailsSource)
                .failureHandler(new SecurityAuthenticationFailureHandler())
                .successHandler(new SecurityAuthenticationSuccessHandler())
                .loginPage("/myLogin.html")
                .loginProcessingUrl("/login")
                .permitAll()
                .and()
                .csrf().disable();;


    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
}
```

其他的代码就是上一章的代码。



## 4.对比一下Filter和Provider

因为我们密码**UsernamePasswordAuthenticationFilter**是Filter。

Filter:

1. 这个是Servlet层面的，简单易理解；
2. 可自定义顺序；

Provider：

​	1.交给Security来控制，感觉更优雅；

​	2.顺序是必须要有UserDetails,才能继续。（个人暂时理解，不确定）

如果没有什么特殊要求还是用**Filter**吧

