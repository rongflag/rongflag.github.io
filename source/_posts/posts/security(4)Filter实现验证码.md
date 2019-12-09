---
title: security(4)Filter实现验证码
url: security4
date: 2019-12-09 23:58:00
tags:
  - security
categories:
  - security
---





用验证嘛来

这章主要说一下security的filter。这里以验证码的filter作为实例进行验证

有的只能靠debug来看顺序

| 方法                             | 说明         |
| -------------------------------- | ------------ |
| addFilterBefore(filterA,filterB) | 添加A在B在前 |
| addFilter(filter)                | 添加一个     |
| addFilterAfter(filterA,filterB)  | 添加A在B在后 |
| addFilterAt()                    | 在同一顺序   |



<!-- more -->

全文内容

pom.xml

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.1</version>
        </dependency>
        <dependency>
            <groupId>com.github.penggle</groupId>
            <artifactId>kaptcha</artifactId>
            <version>2.3.2</version>
        </dependency>


        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

kaptcha配置

```java
@Configuration
public class CaptchaConfig {

    @Bean
    public Producer captcha() {
        // 配置图形验证码的基本参数
        Properties properties = new Properties();
        // 图片宽度
        properties.setProperty("kaptcha.image.width", "150");
        // 图片长度
        properties.setProperty("kaptcha.image.height", "50");
        // 字符集
        properties.setProperty("kaptcha.textproducer.char.string", "0123456789");
        // 字符长度
        properties.setProperty("kaptcha.textproducer.char.length", "4");
        Config config = new Config(properties);
        // 使用默认的图形验证码实现，当然也可以自定义实现
        DefaultKaptcha defaultKaptcha = new DefaultKaptcha();
        defaultKaptcha.setConfig(config);
        return defaultKaptcha;
    }

}

```

```java
@Controller
public class CaptchaController {
    @Autowired
    private Producer captchaProducer;
    @GetMapping("/captcha.jpg")
    public void getCaptcha(HttpServletRequest request, HttpServletResponse response) throws IOException {
        // 设置内容类型
        response.setContentType("image/jpeg");
        // 创建验证码文本
        String capText = captchaProducer.createText();
        // 将验证码文本设置到session
        request.getSession().setAttribute("captcha", capText);
        // 创建验证码图片
        BufferedImage bi = captchaProducer.createImage(capText);
        // 获取响应输出流
        ServletOutputStream out = response.getOutputStream();
        // 将图片验证码数据写到响应输出流
        ImageIO.write(bi, "jpg", out);
        // 推送并关闭响应输出流
        try {
            out.flush();
        } finally {
            out.close();
        }
    }

}
```

filter

```java
public class VerificationCodeFilter extends OncePerRequestFilter {

    private AuthenticationFailureHandler authenticationFailureHandler = new SecurityAuthenticationFailureHandler();

    @Override
    protected void doFilterInternal(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, FilterChain filterChain) throws ServletException, IOException {
        // 非登录请求不校验验证码
        if (!"/login".equals(httpServletRequest.getRequestURI())) {
            filterChain.doFilter(httpServletRequest, httpServletResponse);
        } else {
            try {
                verificationCode(httpServletRequest);
                filterChain.doFilter(httpServletRequest, httpServletResponse);
            } catch (VerificationCodeException e) {
                authenticationFailureHandler.onAuthenticationFailure(httpServletRequest, httpServletResponse, e);
            }
        }
    }

    public void verificationCode (HttpServletRequest httpServletRequest) throws VerificationCodeException {
        String requestCode = httpServletRequest.getParameter("captcha");
        HttpSession session = httpServletRequest.getSession();
        String savedCode = (String) session.getAttribute("captcha");
        if (!StringUtils.isEmpty(savedCode)) {
            // 随手清除验证码，不管是失败还是成功，所以客户端应在登录失败时刷新验证码
            session.removeAttribute("captcha");
        }
        // 校验不通过抛出异常
        if (StringUtils.isEmpty(requestCode) || StringUtils.isEmpty(savedCode) || !requestCode.equals(savedCode)) {
            throw new VerificationCodeException();
        }
    }
}
```

Security

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/admin/**").hasRole("admin")
                .antMatchers("/user/**").hasRole("user")
                .antMatchers("/api/**").permitAll()
                .antMatchers("/captcha.jpg").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .loginPage("/myLogin.html")
                .loginProcessingUrl("/login")
                .permitAll()
                .and()
                .cors().and().csrf().disable();

        http.addFilterBefore(new VerificationCodeFilter(),UsernamePasswordAuthenticationFilter.class);
//        http.addFilter()
//        http.addFilterAfter()
//        http.addFilterA
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
}
```

```html
<div class="login">
            <h2>Acced Form</h2>
            <div class="login-top">
                <h1>LOGIN FORM</h1>
                <form action="/login" method="post">
                    <input type="text" name="username" placeholder="username" />
                    <input type="password" name="password" placeholder="password" />
                    <div style="display: flex;">
                        <!-- 新增图形验证码的输入框 -->
                        <input type="text" name="captcha" placeholder="captcha" />
                        <!-- 图片指向图形验证码API -->
                        <img src="/captcha.jpg" alt="captcha" height="50px" width="150px" style="margin-left: 20px;">
                    </div>
                    <div class="forgot">
                        <a href="#">forgot Password</a>
                        <input type="submit" value="Login" >
                    </div>
                </form>
            </div>
            <div class="login-bottom">
                <h3>New User &nbsp;<a href="#">Register</a>&nbsp Here</h3>
            </div>
        </div>
```

