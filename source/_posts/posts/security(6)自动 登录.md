---
title: security(6)自动 登录
url: security6
date: 2019-12-13 00:00:00
tags:
  - security
categories:
  - security
---



自动登录很简单cookie和databse。。拉胯了，拉胯了 以后在说吧

<!-- more -->

全文内容

# 配置

数据库用户这些和以前一样就不说了。po出配置吧，但是都需要cookie



```java
@EnableWebSecurity(debug = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Value("${spring.security.remember-me.key}")
    private String rememberKey;

    @Autowired
    private DataSource dataSource;

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        JdbcTokenRepositoryImpl tokenRepository = new JdbcTokenRepositoryImpl();

        tokenRepository.setDataSource(dataSource);

        http.authorizeRequests()
                .antMatchers("/admin/**")
                .hasRole("admin")
                .antMatchers("/user/**")
                .hasRole("user")
                .antMatchers("/api/**")
                .permitAll()
                .anyRequest()
                .authenticated()
                .and()
                .formLogin()
                .permitAll()
                .and()
                .rememberMe()
                .userDetailsService(userDetailsService)
//                这个可以重写自定记录的逻辑  实现AbstractRememberMeServices
//                .rememberMeServices(myremenberService)
                // 1. 散列加密方案 这个可要可不要
                .key(rememberKey)
                // 7天有效期  不设置默认两周
                .tokenValiditySeconds(60 * 60 * 24 * 7)
                // 2. 持久化令牌方案  不配置就是cookie 配置了就是数据库
//                .tokenRepository(tokenRepository)
                .and()
                .logout()
                .logoutUrl("/myLogout")
                // 注销成功，重定向到该路径下
                .logoutSuccessUrl("/")
                .invalidateHttpSession(true)
                .and()
                .csrf()
                .disable();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
}
```

## 关注的源码

### 1. RememberMeConfigurer

这里默认 默认TokenBasedRememberMeServices

```java
private AbstractRememberMeServices createRememberMeServices(H http, String key)
			throws Exception {
		return this.tokenRepository == null
				? createTokenBasedRememberMeServices(http, key)
				: createPersistentRememberMeServices(http, key);
	}


private AbstractRememberMeServices createTokenBasedRememberMeServices(H http,
			String key) {
		UserDetailsService userDetailsService = getUserDetailsService(http);
		return new TokenBasedRememberMeServices(key, userDetailsService);
	}

private PersistentTokenRepository tokenRepository = new InMemoryTokenRepositoryImpl()；
    这里是
```



```
private AbstractRememberMeServices createRememberMeServices(H http, String key)
			throws Exception {
		return this.tokenRepository == null
				? createTokenBasedRememberMeServices(http, key)
				: createPersistentRememberMeServices(http, key);
	}
```

所有重写都可以根据这个里面需要的进行配置

```java
private static final String DEFAULT_REMEMBER_ME_NAME = "remember-me";
	private AuthenticationSuccessHandler authenticationSuccessHandler;
	private String key;
	private RememberMeServices rememberMeServices;
	private LogoutHandler logoutHandler;
	private String rememberMeParameter = DEFAULT_REMEMBER_ME_NAME;
	private String rememberMeCookieName = DEFAULT_REMEMBER_ME_NAME;
	private String rememberMeCookieDomain;
	private PersistentTokenRepository tokenRepository;
	private UserDetailsService userDetailsService;
	private Integer tokenValiditySeconds;
	private Boolean useSecureCookie;
	private Boolean alwaysRemember;
```



### 1. AbstractRememberMeServices

```java
protected abstract UserDetails processAutoLoginCookie(String[] cookieTokens,
			HttpServletRequest request, HttpServletResponse response)
			throws RememberMeAuthenticationException, UsernameNotFoundException;
```



### 1.1 TokenBasedRememberMeServices

生成散列加密的部分makeTokenSignature

```java

// 在成功的验证后生成新的token
protected UserDetails processAutoLoginCookie(String[] cookieTokens,
			HttpServletRequest request, HttpServletResponse response) {

		if (cookieTokens.length != 3) {
			throw new InvalidCookieException("Cookie token did not contain 3"
					+ " tokens, but contained '" + Arrays.asList(cookieTokens) + "'");
		}

		long tokenExpiryTime;

		try {
			tokenExpiryTime = new Long(cookieTokens[1]).longValue();
		}
		catch (NumberFormatException nfe) {
			throw new InvalidCookieException(
					"Cookie token[1] did not contain a valid number (contained '"
							+ cookieTokens[1] + "')");
		}

		if (isTokenExpired(tokenExpiryTime)) {
			throw new InvalidCookieException("Cookie token[1] has expired (expired on '"
					+ new Date(tokenExpiryTime) + "'; current time is '" + new Date()
					+ "')");
		}

		// Check the user exists.
		// Defer lookup until after expiry time checked, to possibly avoid expensive
		// database call.

		UserDetails userDetails = getUserDetailsService().loadUserByUsername(
				cookieTokens[0]);

		// Check signature of token matches remaining details.
		// Must do this after user lookup, as we need the DAO-derived password.
		// If efficiency was a major issue, just add in a UserCache implementation,
		// but recall that this method is usually only called once per HttpSession - if
		// the token is valid,
		// it will cause SecurityContextHolder population, whilst if invalid, will cause
		// the cookie to be cancelled.
		String expectedTokenSignature = makeTokenSignature(tokenExpiryTime,
				userDetails.getUsername(), userDetails.getPassword());

		if (!equals(expectedTokenSignature, cookieTokens[2])) {
			throw new InvalidCookieException("Cookie token[2] contained signature '"
					+ cookieTokens[2] + "' but expected '" + expectedTokenSignature + "'");
		}

		return userDetails;
	}

// 散列加密
protected String makeTokenSignature(long tokenExpiryTime, String username,
			String password) {
		String data = username + ":" + tokenExpiryTime + ":" + password + ":" + getKey();
		MessageDigest digest;
		try {
			digest = MessageDigest.getInstance("MD5");
		}
		catch (NoSuchAlgorithmException e) {
			throw new IllegalStateException("No MD5 algorithm available!");
		}

		return new String(Hex.encode(digest.digest(data.getBytes())));
	}

// 成功过后
public void onLoginSuccess(HttpServletRequest request, HttpServletResponse response,
			Authentication successfulAuthentication) {

		String username = retrieveUserName(successfulAuthentication);
		String password = retrievePassword(successfulAuthentication);

		// If unable to find a username and password, just abort as
		// TokenBasedRememberMeServices is
		// unable to construct a valid token in this case.
		if (!StringUtils.hasLength(username)) {
			logger.debug("Unable to retrieve username");
			return;
		}

		if (!StringUtils.hasLength(password)) {
			UserDetails user = getUserDetailsService().loadUserByUsername(username);
			password = user.getPassword();

			if (!StringUtils.hasLength(password)) {
				logger.debug("Unable to obtain password for user: " + username);
				return;
			}
		}

		int tokenLifetime = calculateLoginLifetime(request, successfulAuthentication);
		long expiryTime = System.currentTimeMillis();
		// SEC-949
		expiryTime += 1000L * (tokenLifetime < 0 ? TWO_WEEKS_S : tokenLifetime);

		String signatureValue = makeTokenSignature(expiryTime, username, password);

		setCookie(new String[] { username, Long.toString(expiryTime), signatureValue },
				tokenLifetime, request, response);

		if (logger.isDebugEnabled()) {
			logger.debug("Added remember-me cookie for user '" + username
					+ "', expiry: '" + new Date(expiryTime) + "'");
		}
	}
```

