---
title: security（2）初识认证与授权
url: security2
date: 2019-12-06 22:04:00
tags:
  - security
categories:
  - security
---



# security（2）初识认证与授权

这里会基础的见一下认证和授权基于**内存**和**数据库**

<!-- more -->

pom.xml(为了方便就把数据库配置一起加进去了)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.10.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.rongflag</groupId>
    <artifactId>chapter2</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>chapter2</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

application.yml

```yaml
spring:
  datasource:
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc://localhost:3306/security-study?useUnicode=true&characterEncoding=utf-8&useSSL=false

```

security配置

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

初始化3个controller

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



## 1.内存

通过InMemoryUserDetailsManager 去构建两个人

| 用户名 | 密码   | 权限           |
| ------ | ------ | -------------- |
| user   | 123456 | user           |
| admin  | 123456 | admin     user |



```java
 @Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(User.withUsername("user").password("123456").roles("user").build());
        manager.createUser(User.withUsername("admin").password("123456").roles("user", "admin").build());
        return manager;
    }

@Bean
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
```

即可以通过不同的url去验证权限是否成功了。



## 2.JDBC

创建两张表 users authorities

```mysql
CREATE TABLE `security-study`.`users`  (
  `username` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `password` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL,
  `enabled` tinyint(1) NULL DEFAULT NULL,
  PRIMARY KEY (`username`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci ROW_FORMAT = Dynamic;


CREATE TABLE `security-study`.`authorities`  (
  `username` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `authority` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  INDEX `u_name`(`username`) USING BTREE,
  CONSTRAINT `u_name` FOREIGN KEY (`username`) REFERENCES `security-study`.`users` (`username`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci ROW_FORMAT = Dynamic;
```

和上面基本类似

```java
  @Autowired
    private DataSource dataSource;
    @Override
    @Bean
    public UserDetailsService userDetailsService() {
        JdbcUserDetailsManager manager = new JdbcUserDetailsManager();
        manager.setDataSource(dataSource);
        manager.createUser(User.withUsername("user").password("123").roles("user").build());
        manager.createUser(User.withUsername("admin").password("123").roles("user", "admin").build());
        return manager;
    }
```

启动的时候自动将构建的数据向users 和 authorities里面插入  这是自动的不用手加。

```sql
INSERT INTO `security-study`.`users`(`username`, `password`, `enabled`) VALUES ('user', '123', 1);
INSERT INTO `security-study`.`users`(`username`, `password`, `enabled`) VALUES ('admin', '123', 1);
INSERT INTO `security-study`.`authorities`(`username`, `authority`) VALUES ('user', 'ROLE_user');
INSERT INTO `security-study`.`authorities`(`username`, `authority`) VALUES ('admin', 'ROLE_admin');
INSERT INTO `security-study`.`authorities`(`username`, `authority`) VALUES ('admin', 'ROLE_user');

```





> 这里重复启动的时候数据库数据不会删除 会报错 需要把数据库中数据清掉

这里需要注意一点 自动加的时候 role里面会自动加上 ROLE_  看源码

```java
   public User.UserBuilder roles(String... roles) {
            List<GrantedAuthority> authorities = new ArrayList(roles.length);
            String[] var3 = roles;
            int var4 = roles.length;

            for(int var5 = 0; var5 < var4; ++var5) {
                String role = var3[var5];
                Assert.isTrue(!role.startsWith("ROLE_"), () -> {
                    return role + " cannot start with ROLE_ (it is automatically added)";
                });
                authorities.add(new SimpleGrantedAuthority("ROLE_" + role));
            }

            return this.authorities((Collection)authorities);
        }
```

实验方式和上面类似

