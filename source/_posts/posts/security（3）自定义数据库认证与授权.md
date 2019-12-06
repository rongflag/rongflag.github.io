---
title: security（3）自定义数据库认证与授权
url: security3
date: 2019年12月6日, PM 10:25:07
tags:
  - security
categories:
  - security
---

# security（3）自定义数据库认证与授权
先创建数据库的表

```sql
CREATE TABLE `security-study`.`user2`  (
  `id` int(12) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL,
  `password` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL,
  `roles` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL COMMENT '多个用,隔开',
  `enabled` tinyint(1) NULL DEFAULT 1 COMMENT '1可用0不可用',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci ROW_FORMAT = Dynamic;

INSERT INTO `security-study`.`users2`(`id`, `username`, `password`, `roles`, `enabled`) VALUES (1, 'admin', '123', 'ROLE_admin,ROLE_user', 1);
INSERT INTO `security-study`.`users2`(`id`, `username`, `password`, `roles`, `enabled`) VALUES (2, 'user', '123', 'ROLE_user', 1);

```

## 1. 创建自定义UserDetails

Security其实过程的的中间件是 UserDetails

UserDetails源码

```java
public interface UserDetails extends Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();

    String getPassword();

    String getUsername();

    boolean isAccountNonExpired();

    boolean isAccountNonLocked();

    boolean isCredentialsNonExpired();

    boolean isEnabled();
}
```

创建User   User实现UserDetails

```java
@Data
public class User  implements UserDetails{

    private Long id;

    private String username;

    private String password;

    private boolean enabled;

    private String roles;

    private List<GrantedAuthority> authorities;

    @Override
    public Collection<GrantedAuthority> getAuthorities() {
        return this.authorities;
    }

    public void setAuthorities( List<GrantedAuthority> authorities) {
        this.authorities = authorities;
    }
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }
}
```

## 2.创建service和mapper

> 加配置 @MapperScan("com.rongflag.chapter3.mapper")

mapper

```java
@Component
public interface UsersMapper {

    @Select("select * from users2 where username=#{username}")
     public User getUser(@Param("username") String username);
}
```

service

这里需要实现UserDetailsService 让sercurity使用这个来进行认证

```java
@Service
public class MyUserDetailsService implements UserDetailsService {

    @Autowired
    private UsersMapper usersMapper;
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = usersMapper.getUserByUsername(username);
        if(user == null){
            throw new UsernameNotFoundException("未找到该用户："+username);
        }
        String roles = user.getRoles();
        List<GrantedAuthority> grantedAuthorities = this.createAuthorities(roles);
//        系统自带的转换权限 但是用自己的更好控制
//        List<GrantedAuthority> grantedAuthorities = AuthorityUtils.commaSeparatedStringToAuthorityList(user.getRoles());
        user.setAuthorities(grantedAuthorities);

        return user;
    }

    /***
     * @Descripton 自行转换权限
     * @param
     * @rerturn 
     * @date  
     * @auther aihuxi
     */
      
    private List<GrantedAuthority> createAuthorities(String roles) {
        List<GrantedAuthority>lists = new ArrayList<>();
        if(roles != null && !"".equals(roles)){
            String[] roleArr = roles.split(",");
            for (String role:roleArr ){
                lists.add(new SimpleGrantedAuthority(role));
            }
        }
        return lists;
    }


}
```

## 3.创建security和controller

和前面是一样的

```java
@EnableWebSecurity(debug = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/admin/**").hasRole("admin")
                .antMatchers("/user/**").hasRole("user")
                .antMatchers("/api/**").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin()
        ;

    }
}
```

```java
@RestController
@RequestMapping("user")
public class UserController {

    @GetMapping("/hello")
    public String sayHello(){
        return "user say hello";
    }
}

@RestController
@RequestMapping("api")
public class ApiController {

    @GetMapping("/hello")
    public String sayHello(){
        return "api say hello";
    }
}

@RestController
@RequestMapping("admin")
public class AdminController {

    @GetMapping("/hello")
    public String sayHello(){
        return "admin say hello";
    }
}
```

这样就按照原来的验证即可

## 4.踩坑记录

1. 配置需要的权限是 “admin”,"user" 但是数据库只能保存“ROLE_admin”,"ROLE_user"，带有**ROLE_**

因为在securityConfig中

```
.antMatchers("/admin/**").hasRole("admin")
```

往下看源码

org.springframework.security.config.annotation.web.configurers.ExpressionUrlAuthorizationConfigurer#hasRole

```java

 private static String hasRole(String role) {
        Assert.notNull(role, "role cannot be null");
        if (role.startsWith("ROLE_")) {
            throw new IllegalArgumentException("role should not start with 'ROLE_' since it is automatically inserted. Got '" + role + "'");
        } else {
            return "hasRole('ROLE_" + role + "')";
        }
    }
```

其实上一章也说过了，在自己初始化的时候会调用方法 自动加上 **ROLE_**

1.  enabled 字段的问题

enabled 是security验证自带字段  1是true 0是false  

其实自己使用最好是不使用这个字段，不容易控制

这个字段也是我在自己试的时候踩坑出来的