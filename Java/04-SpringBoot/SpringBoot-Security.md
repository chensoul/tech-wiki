在本 Spring Security 教程中，学习如何向我们的 Spring Boot MVC 应用程序添加默认或自定义登录表单。我们将了解默认的登录表单，并根据需求进一步自定义它。

## 1. 默认示例程序

### 1.1 Maven 依赖配置

要将 Spring Security 包含在 Spring Boot 应用程序中，我们需要 `spring-boot-starter-security` 依赖项以及其他特定于模块的依赖项。

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

在这里，我们使用了 Spring Boot 2.7.5 版本。基于此版本，Spring Boot内部将Spring Security版本解析为5.7.4。但是，如果需要，我们可以在 `pom.xml` 中覆盖这些版本，如下所示：

```xml
<properties>
    <spring-security.version>5.2.5.RELEASE</spring-security.version>
</properties>
```

如果我们使用非 spring-boot 应用程序，那么我们需要显式包含以下依赖项。

- `spring-security-core`
- `spring-security-config`
- `spring-security-web`

```xml
<properties>
    <failOnMissingWebXml>false</failOnMissingWebXml>
    <spring.version>5.2.5.RELEASE</spring.version>
</properties> 
 
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
  <version>${spring.version}</version>
</dependency>
 
<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-core</artifactId>
  <version>${spring.version}</version>
</dependency>
 
<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-config</artifactId>
  <version>${spring.version}</version>
</dependency>
 
<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-web</artifactId>
  <version>${spring.version}</version>
</dependency>
```

### 1.2 Spring Security Java 配置

让我们首先创建一个 Spring Security 配置类，通过添加@EnableWebSecurity，我们获得Spring Security和MVC集成支持：

```java
@Configuration
@EnableWebSecurity
public class SecSecurityConfig {

}
```



### 1.3 添加 Spring Security 到 Web 应用

要使用上面定义的 Spring Security 配置，我们需要将其附加到 Web 应用程序。

我们将使用 WebApplicationInitializer，因此不需要提供任何 web.xml：

```java
public class AppInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext sc) {

        AnnotationConfigWebApplicationContext root = new AnnotationConfigWebApplicationContext();
        root.register(SecSecurityConfig.class);

        sc.addListener(new ContextLoaderListener(root));

        sc.addFilter("securityFilter", new DelegatingFilterProxy("springSecurityFilterChain"))
          .addMappingForUrlPatterns(null, false, "/*");
    }
}
```

请注意，如果我们使用 Spring Boot 应用程序，则不需要此初始化程序。有关如何在 Spring Boot 中加载安全配置的更多详细信息，请参阅我们有关 [Spring Boot 安全自动配置](https://www.baeldung.com/spring-boot-security-autoconfiguration)的文章。

### 1.4 运行项目

创建项目后，运行项目。

```bash
mvn clean verify spring-boot:run 
```

在应用程序启动时，我们应该看到一个登录页面。

![img](https://howtodoinjava.com/wp-content/uploads/2022/05/Spring-security-default-login-form.jpg)

控制台日志打印作为默认安全配置的一部分随机生成的默认密码：

```
2023-08-31 14:20:58.765  INFO 84824 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 422 ms
2023-08-31 14:20:58.915  INFO 84824 --- [           main] .s.s.UserDetailsServiceAutoConfiguration : 

Using generated security password: 3bd457b0-a52d-46fd-92b7-7b8091c4d3af

2023-08-31 14:20:58.964  INFO 84824 --- [           main] o.s.s.web.DefaultSecurityFilterChain     : Will secure any request with
```

使用默认用户名 `user` 和默认密码（来自日志），我们应该能够登录该应用程序。我们可以在 `application.yml` 中覆盖这些默认值：

```yaml
spring:
  security:
    user:
      name: user
      password: password
```

现在，我们应该能够使用用户 `user` 和密码 `password` 登录。

如果我们输入的用户名和密码不正确，我们将收到“凭据错误”错误。

[![img](https://howtodoinjava.com/wp-content/uploads/2022/05/Spring-security-default-login-form-bad-credentials-error.jpg)](https://howtodoinjava.com/wp-content/uploads/2022/05/Spring-security-default-login-form-bad-credentials-error.jpg)

如果我们输入正确的凭据，我们将被重定向到应用程序的根 URL。



## 2. 基本身份验证

基本身份验证通常与在每个请求上传递凭据的无状态客户端一起使用。将其与基于表单的身份验证结合使用是很常见的，其中应用程序通过基于浏览器的用户界面和 Web 服务来使用。但是，基本身份验证以纯文本形式传输密码，因此它只能在加密传输层（例如 HTTPS）上使用。

由于每个 HTTP 请求都必须发送基本身份验证标头，因此 Web 浏览器需要将凭据缓存一段合理的时间，以避免不断提示用户输入用户名和密码。浏览器之间的缓存策略有所不同。

例如，要授权为 `user/password` ，客户端将发送：

```
Authorization: Basic dXNlcjpwYXNzd29yZA==
```

### 2.1. 默认基本身份验证配置

实现基本 HTTP 身份验证的最简单的解决方案是在 spring security 配置文件中使用 httpBasic()，如下所示。

```java
@Override
    protected void configure(final HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/public").permitAll()
                .anyRequest().authenticated()
                .and()
                .httpBasic();
    }
```

上述配置将强制用户在访问我们应用程序中的任何网页或任何其他资源之前进行身份验证。

有趣的是，我们不需要创建任何登录页面或会话管理机制。浏览器将代表我们在用户面前显示一个登录框。而且由于每个请求都包含身份验证信息，就像 HTTP 无状态机制一样，因此我们也不需要维护会话。

默认情况下，spring 将创建一个用户名为 `user` 的用户，生成的密码会打印在控制台中。它可用于对应用程序进行身份验证。

### 2.2. 自定义基本认证配置

在以下配置中，我们定制了一些内容：

- 我们使用 `InMemoryUserDetailsManager` 配置用户（用户：密码）来代替默认用户。我们可以根据需要创建任意数量的用户，并为每个用户分配所需的权限。
- 将默认密码编码器配置为 `BCryptPasswordEncoder` 。
- 保护所有 URL，除了允许所有的 `/public` 。

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private BasicAuthenticationEntryPoint authenticationEntryPoint;

    @Override
    protected void configure(final HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/public").permitAll()
                .anyRequest().authenticated()
                .and()
                .httpBasic()
                .authenticationEntryPoint(authenticationEntryPoint);
    }

    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        UserDetails user = User
                .withUsername("user")
                .password("$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG")
                .roles("USER_ROLE")
                .build();
        return new InMemoryUserDetailsManager(user);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### 2.3.自定义BasicAuthenticationEntryPoint

`ExceptionTranslationFilter` 使用身份验证入口点来开始身份验证。默认情况下，BasicAuthenticationEntryPoint 将 401 未经授权响应返回完整页面给客户端。

要自定义基本身份验证使用的默认身份验证错误页面，我们可以扩展 BasicAuthenticationEntryPoint 类。在这里我们可以设置领域名称以及发送回客户端的错误消息。

```java
@Component
public class CustomBasicAuthenticationEntryPoint
        extends BasicAuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request,
                         HttpServletResponse response,
                         AuthenticationException e) throws IOException {
        response.addHeader("WWW-Authenticate", "Basic realm=" + getRealmName());

        response.setStatus(HttpStatus.UNAUTHORIZED.value());
        response.setContentType("application/json");
        response.setCharacterEncoding("UTF-8");

        String jsonPayload = "{\"message\": \"%s\", \"timestamp\": \"%s\"}";
        String currentTime = java.util.Calendar.getInstance().getTime().toString();
        String formattedPayload = String.format(jsonPayload, e.getMessage(), currentTime);

        response.getOutputStream().write(formattedPayload.getBytes(StandardCharsets.UTF_8));
    }

    @Override
    public void afterPropertiesSet() {
        setRealmName("basic-auth");
        super.afterPropertiesSet();
    }
}
```

### 2.4. 使用基本身份验证

启动服务器并在浏览器中加载任何不受保护的 URL，出现登录窗口。请注意，这是浏览器生成的登录框，应用程序仅向浏览器提供相关标头。

输入错误的用户名和密码，这将使浏览器再次显示清除的登录框，或者在某些情况下会显示错误页面。

当我们输入正确的用户名和密码时，浏览器中会加载正确的响应。

使用 curl 进行测试。首先，让我们尝试在不提供任何安全凭证的情况下请求 /public：

```bash
curl -i http://localhost:8080/public
```

我们收到预期的 401 Unauthorized 和 Authentication Challenge：

```
HTTP/1.1 401 Unauthorized
Server: Apache-Coyote/1.1
Set-Cookie: JSESSIONID=E5A8D3C16B65A0A007CFAACAEEE6916B; Path=/; HttpOnly
WWW-Authenticate: Basic realm="Spring Security Application"
Content-Type: text/html;charset=utf-8
Content-Length: 1061
Date: Wed, 29 May 2013 15:14:08 GMT
```

现在让我们请求相同的资源，并提供访问它的凭据：

```bash
curl -i --user user:password http://localhost:8080/public
```

结果，服务器的响应是 200 OK 以及 Cookie：

```
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Set-Cookie: JSESSIONID=301225C7AE7C74B0892887389996785D; Path=/; HttpOnly
Content-Type: text/html;charset=ISO-8859-1
Content-Language: en-US
Content-Length: 90
Date: Wed, 29 May 2013 15:19:38 GMT
```

### 2.5. 单元测试

要使用基本身份验证进行身份验证，访问资源是单元测试，我们可以在使用 MockMvc 时使用 MockMvcRequestBuilders.with() 方法。

```java
@SpringBootTest(
    webEnvironment = SpringBootTest.WebEnvironment.MOCK,
    classes = {
      BasicAuthWebSecurityConfiguration.class,
      AppBasicAuthenticationEntryPoint.class,
      AppController.class
    }
)
@AutoConfigureMockMvc
class BasicAuthTest {
    @Autowired
    private MockMvc mvc;
    @Test
    void expectOKResponse_WhenAccessNotSecuredURL() throws Exception {
        ResultActions result = mvc.perform(MockMvcRequestBuilders.get("/public"))
          .andExpect(status().isOk());
    }
    @Test
    void expectUnauthorizedUser_WhenPasswordIsWrong() throws Exception {
        ResultActions result = mvc.perform(MockMvcRequestBuilders.get("/")
                .with(httpBasic("user","wrong-password")))
            .andExpect(status().isUnauthorized());
    }
    @Test
    void expectOKResponse_WhenPasswordIsCorrect() throws Exception {
        ResultActions result = mvc.perform(MockMvcRequestBuilders.get("/")
            .with(httpBasic("user", "password")))
          .andExpect(content().string("Hello World !!"));
    }
}
```

## 3. 表单登录示例

要配置基于登录表单的安全性，我们需要配置以下组件：

### 3.1. 认证提供者

身份验证提供程序负责在 UserDetailsManager 和 PasswordEncoder 实现的帮助下提供身份验证逻辑。为了简单起见，我们使用 InMemoryUserDetailsManager。

```java
@EnableWebSecurity
public class SecurityConfig {
  @Bean
  public UserDetailsService userDetailsService() {
    UserDetails user = User.builder()
        .username("user")
        .password(passwordEncoder().encode("password"))
        .roles("USER")
        .build();
    UserDetails admin = User.builder()
        .username("admin")
        .password(passwordEncoder().encode("password"))
        .roles("USER", "ADMIN")
        .build();
    return new InMemoryUserDetailsManager(user, admin);
  }
  @Bean
  public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
  }
  //Other beans
}
```

### 3.2. 提供默认登录页面

在受 Spring 安全保护的应用程序中，未经身份验证的用户将被重定向到一个表单，他们可以在其中使用其凭据进行身份验证。一旦应用程序对它们进行身份验证，它们就会被重定向到应用程序的主页。

当 Spring boot 检测到已配置基于表单的身份验证时，它会提供合理的默认值。要启用基于表单的登录，我们可以使用 HttpSecurity 类调用 formLogin() 方法。

`formLogin()` 方法返回一个 `FormLoginConfigurer<HttpSecurity>` 类型的对象，它允许我们进行更多自定义。

- `@EnableWebSecurity` 启用Spring Security的Web安全支持，并提供Spring MVC集成。
- `WebSecurityConfigurerAdapter` 提供了一组用于启用特定 Web 安全配置的方法。
- `configure(HttpSecurity http)` 用于保护需要安全的不同 URL。

```java
@EnableWebSecurity
public class SecurityConfig {
	@Bean
	public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
	  http.authorizeRequests()
	      .requestMatchers("/login").permitAll()
	      .requestMatchers("/**").authenticated()
	      .and()
	      .formLogin().permitAll();
	      return http.build();
	}
	//...
}
```

### 3.3. 自定义登录页面配置

默认的登录表单很适合启动，但在生产级应用程序中，我们必须提供自定义的登录表单和各种身份验证选项。

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(final HttpSecurity http) throws Exception {
    //@formatter:off
    http.authorizeRequests()
        .antMatchers("/login").permitAll()
        .antMatchers("/**").hasAnyRole("USER", "ADMIN")
        .antMatchers("/admin/**").hasAnyRole("ADMIN")
        .and()
          .formLogin()
          .loginPage("/login")
          .loginProcessingUrl("/process-login")
          .defaultSuccessUrl("/home")
          .failureUrl("/login?error=true")
          .permitAll()
        .and()
          .logout()
      		.logoutUrl("/perform_logout")
          .logoutSuccessUrl("/login?logout=true")
          .invalidateHttpSession(true)
          .deleteCookies("JSESSIONID")
          .permitAll()
        .and()
          .csrf()
          .disable();
    //@formatter:on
  }
  @Override
  public void configure(WebSecurity web) {
    web.ignoring()
        .antMatchers("/resources/**", "/static/**");
  }
}
```

#### 3.3.1.自定义登录页面

`.loginPage("/login")`函数在 URL `/login` 处配置自定义登录页面。我们必须定义一个 URL 映射处理程序，它将返回它的视图名称。

```java
@Controller
public class LoginController {
  @GetMapping("/login")
  public String login() {
    return "login";
  }
}
```

src/main/resources/templates/login.html 文件使用默认的 `ThymeleafViewResolver` ，其默认模板目录为 `src/main/resources/templates` 。

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org">
<head>
    <title>Please Log In</title>
</head>
<body>
<h1>Please Log In</h1>
<div th:if="${param.error}">
    Invalid username and password.
</div>
<div th:if="${param.logout}">
    You have been logged out.
</div>
<form method="post" th:action="@{/process-login}">
    <div>
        <input name="username" placeholder="Username" type="text"/>
    </div>
    <div>
        <input name="password" placeholder="Password" type="password"/>
    </div>
    <input type="submit" value="Log in"/>
</form>
</body>
</html>
```

现在，当我们运行应用程序时，默认登录页面已使用 HTML 文件更改。其余所有功能均相同。

![img](https://howtodoinjava.com/wp-content/uploads/2022/05/Spring-security-custom-login-form.jpg)

#### 3.3.2.登录处理 URL

`.loginProcessingUrl("/process-login")`函数指定自定义身份验证处理 URL，而不是默认 URL /login。

我们必须在视图文件 HTML 表单元素的 action 属性中指定自定义 URL。

#### 3.3.3.登录成功和失败的登陆 URL

我们可以使用 `defaultSuccessUrl()` 和 `failureUrl()` 方法将登录成功URL（默认为根URL）配置为另一个URL。

在给定的配置中，如果用户名/密码匹配，则请求将重定向到 `/home` ，否则将刷新登录页面并显示相应的错误消息。

```java
http.formLogin()
      .defaultSuccessUrl("/home")
      .failureUrl("/login?error=true")
      .permitAll();
```

#### 3.3.4.自定义用户名和密码字段

默认情况下，Spring Security 使用用户名字段作为“username”，密码作为“password”。如果我们在 login.html 文件中使用一些其他字段名称，那么我们可以覆盖默认字段名称。

```java
http.formLogin()
  .loginPage("/login")
  .usernameParameter("email")
  .passwordParameter("passcode")
  .permitAll()
```

现在使用新的字段名称，如下所示：

```html
<form th:action="@{/login}" method="post">
    <p>
        E-mail: <input type="email" name="email" required />
    </p>
    <p>
        Password: <input type="password" name="passcode" required />
    </p>
    <p>
        <input type="submit" value="Login" />
    </p>
</form>
```

#### 3.3.5. 登录成功和失败转发 URL

如果我们想将控件转发到任何特定的 URL，那么我们可以使用 `successForwardUrl()` 方法指定，而不是转发到根 URL。使用此方法，我们可以在用户成功登录后执行自定义逻辑，例如插入特殊的审核条目，然后转发到所需的视图。

```java
http.formLogin()
    .successForwardUrl("/login_success_handler");
```

同样，我们可以为登录失败尝试指定处理程序方法。

```java
http.formLogin()
    .failureForwardUrl("/login_failure_handler");
```

上述 URL 需要出现在某些 MVC 控制器中。

```java
@Controller
public class LoginController {
  //Other code
  @PostMapping("/login_success_handler")
  public String loginSuccessHandler() {
    //perform audit action
    return "/";
  }
  @PostMapping("/login_failure_handler")
  public String loginFailureHandler() {
    //perform audit action
    return "login";
  }
}
```

#### 3.3.6. 自定义身份验证处理程序

与登录成功和失败转发 URL 类似，我们还可以通过实现 `AuthenticationSuccessHandler` 和 `AuthenticationFailureHandler` 接口来编写身份验证成功和失败处理程序方法。

这些实现提供对 Authentication 对象的直接访问。

```java
http.formLogin()
  .successHandler(authenticationSuccessHandler())
  .failureHandler(authenticationFailureHandler());
```

在安全配置文件中定义 bean。

```java
	@Bean
  AuthenticationSuccessHandler authenticationSuccessHandler() {
    return new CustomAuthenticationSuccessHandler();
  }

  @Bean
  AuthenticationFailureHandler authenticationFailureHandler() {
    return new CustomAuthenticationFailureHandler();
  }
```

实现类有：

```java
public class CustomAuthenticationSuccessHandler
    implements AuthenticationSuccessHandler {
  @Override
  public void onAuthenticationSuccess(HttpServletRequest request,
                                      HttpServletResponse response,
                                      Authentication authentication) throws IOException, ServletException {
    System.out.println("Logged user: " + authentication.getName());
    response.sendRedirect("/");
  }
}
```



```java
public class CustomAuthenticationFailureHandler implements AuthenticationFailureHandler {
  @Override
  public void onAuthenticationFailure(HttpServletRequest httpServletRequest,
                                      HttpServletResponse httpServletResponse
      , AuthenticationException e) throws IOException {
    httpServletResponse.setStatus(HttpStatus.UNAUTHORIZED.value());
    String jsonPayload = "{\"message\" : \"%s\", \"timestamp\" : \"%s\" }";
    httpServletResponse.getOutputStream()
        .println(String.format(jsonPayload,
            e.getMessage(),
            Calendar.getInstance().getTime()));
  }
}
```

#### 3.3.7 登出处理 URL

与 Spring Security 中的其他默认值类似，实际触发注销机制的 URL 也有一个默认值 - /logout。

但是，最好更改此默认值，以确保不会发布有关使用什么框架来保护应用程序的信息：

```java
.logout()
.logoutUrl("/perform_logout")
```

#### 3.3.8 会话失效和删除 cookie

invalidateHttpSession 允许设置会话，以便在发生注销时会话不会失效（默认情况下为 true）。

```java
.logout()
.logoutUrl("/perform_logout")
.invalidateHttpSession(true)
.deleteCookies("JSESSIONID")
```

#### 3.3.9 **注销成功处理程序**

对于更高级的场景，命名空间不够灵活，Spring Context 中的 LogoutSuccessHandler bean 可以替换为自定义引用：

```java
@Bean
public LogoutSuccessHandler logoutSuccessHandler() {
    return new CustomLogoutSuccessHandler();
}

//...
.logout()
.logoutSuccessHandler(logoutSuccessHandler());
//...
```

用户成功注销时需要运行的任何自定义应用程序逻辑都可以使用自定义注销成功处理程序来实现。例如 – 一个简单的审核机制，跟踪用户触发注销时所访问的最后一个页面：

```java
@Slf4j
public class CustomLogoutSuccessHandler extends SimpleUrlLogoutSuccessHandler implements LogoutSuccessHandler {

    @Override
    public void onLogoutSuccess(final HttpServletRequest request, final HttpServletResponse response, final Authentication authentication) throws IOException, ServletException {
        final String refererUrl = request.getHeader("Referer");
        log.info(refererUrl);

        super.onLogoutSuccess(request, response, authentication);
    }
}
```

#### 3.3.10 记住我

使用 Java 设置安全配置：

```java
		.rememberMe()
    .key("uniqueAndSecret")
```

此外，可以使用 tokenValiditySeconds() 将令牌的有效时间从默认的 2 周配置为一天：

```java
		.rememberMe()
    .key("uniqueAndSecret")
    .tokenValiditySeconds(86400)
```

修改表单登录页面：

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org">
<head>
    <title>Please Log In</title>
</head>
<body>
<h1>Please Log In</h1>
<div th:if="${param.error}">
    Invalid username and password.
</div>
<div th:if="${param.logout}">
    You have been logged out.
</div>
<form method="post" th:action="@{/process-login}">
    <div>
        <input name="username" placeholder="Username" type="text"/>
    </div>
    <div>
        <input name="password" placeholder="Password" type="password"/>
    </div>
    <div>
        Remember Me:<input type="checkbox" name="remember-me" />
    </div>
    <input type="submit" value="Log in"/>
</form>
</body>
</html>
```

请注意新添加的复选框输入 - 映射到"记住我"。添加的输入足以在“记住我”活动的情况下登录。

也可以按如下方式更改此默认路径：

```java
.rememberMe().rememberMeParameter("remember-me-new")
```

当用户登录时，该机制将创建一个额外的 cookie——“记住我”cookie，包含以下数据：

- **username** – 识别登录主体
- **expirationTime** – 使 cookie 过期；默认为 2 周
- **MD5 hash** –  前 2 个值 – 用户名和过期时间，加上密码和预定义密钥

这里首先要注意的是，用户名和密码都是 cookie 的一部分——这意味着，如果其中任何一个发生更改，cookie 就不再有效。此外，还可以从 cookie 中读取用户名。

此外，重要的是要了解，如果“记住我”cookie 被捕获，则该机制可能容易受到攻击。 Cookie 在过期或凭证更改之前一直有效且可用。

#### 3.3.11 防止暴力身份验证尝试

记录来自单个 IP 地址的失败尝试次数。如果该特定 IP 的请求次数超过一定数量，它将被阻止 24 小时。

让我们首先定义一个 AuthenticationFailureListener – 监听 AuthenticationFailureBadCredentialsEvent 事件并通知我们身份验证失败：

```java
@Component
public class AuthenticationFailureListener implements 
  ApplicationListener<AuthenticationFailureBadCredentialsEvent> {

    @Autowired
    private HttpServletRequest request;

    @Autowired
    private LoginAttemptService loginAttemptService;

    @Override
    public void onApplicationEvent(AuthenticationFailureBadCredentialsEvent e) {
        final String xfHeader = request.getHeader("X-Forwarded-For");
        if (xfHeader == null || xfHeader.isEmpty() || !xfHeader.contains(request.getRemoteAddr())) {
            loginAttemptService.loginFailed(request.getRemoteAddr());
        } else {
            loginAttemptService.loginFailed(xfHeader.split(",")[0]);
        }
    }
}
```

请注意，当身份验证失败时，我们如何通知 LoginAttemptService 发起失败尝试的 IP 地址。在这里，我们从 HttpServletRequest bean 获取 IP 地址，这也为我们提供了 X-Forwarded-For 标头中转发的请求的原始地址、代理地址。

我们还注意到，X-Forwarded-For 标头是多值的，可以进行调整以轻松欺骗原始 IP。因此，我们不应该假设标头是可信的；相反，我们必须首先检查它是否包含请求的远程地址。否则，攻击者可以在标头的第一个索引处设置与自己不同的 IP，以避免阻塞自己的 IP。如果我们阻止其中一个 IP 地址，那么攻击者就可以添加另一个 IP 地址，依此类推。这意味着他可以暴力破解标头 IP 地址来欺骗请求。

LoginAttemptService：

```java
@Service
public class LoginAttemptService {
    public static final int MAX_ATTEMPT = 10;
    private LoadingCache<String, Integer> attemptsCache;

    @Autowired
    private HttpServletRequest request;

    public LoginAttemptService() {
        super();
        attemptsCache = CacheBuilder.newBuilder().expireAfterWrite(1, TimeUnit.DAYS).build(new CacheLoader<String, Integer>() {
            @Override
            public Integer load(final String key) {
                return 0;
            }
        });
    }

    public void loginFailed(final String key) {
        int attempts;
        try {
            attempts = attemptsCache.get(key);
        } catch (final ExecutionException e) {
            attempts = 0;
        }
        attempts++;
        attemptsCache.put(key, attempts);
    }

    public boolean isBlocked() {
        try {
            return attemptsCache.get(getClientIP()) >= MAX_ATTEMPT;
        } catch (final ExecutionException e) {
            return false;
        }
    }

    private String getClientIP() {
        final String xfHeader = request.getHeader("X-Forwarded-For");
        if (xfHeader != null) {
            return xfHeader.split(",")[0];
        }
        return request.getRemoteAddr();
    }
}
```

请注意，我们有一些额外的逻辑来识别客户端的原始 IP 地址。在大多数情况下，这不是必需的，但在某些网络场景中是必需的。

对于这些罕见的情况，我们使用 X-Forwarded-For 标头来获取原始 IP；这是该标头的语法：

```bash
X-Forwarded-For: clientIpAddress, proxy1, proxy2
```

修改*UserDetailsService*：

```java
@Service("userDetailsService")
@Transactional
public class MyUserDetailsService implements UserDetailsService {
 
    @Autowired
    private LoginAttemptService loginAttemptService;
 
    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        if (loginAttemptService.isBlocked()) {
            throw new RuntimeException("blocked");
        }
 
        //.....
    }
}
```

修改 CustomAuthenticationFailureHandler 以自定义新的错误消息。

```java
@Component
public class CustomAuthenticationFailureHandler extends SimpleUrlAuthenticationFailureHandler {

    @Autowired
    private MessageSource messages;

    @Autowired
    private LocaleResolver localeResolver;

    @Autowired
    private HttpServletRequest request;

    @Autowired
    private LoginAttemptService loginAttemptService;

    @Override
    public void onAuthenticationFailure(...) {
        ...

        if (loginAttemptService.isBlocked()) {
            errorMessage = messages.getMessage("auth.message.blocked", null, locale);
        }
        String errorMessage = messages.getMessage("message.badCredentials", null, locale);
        if (exception.getMessage().equalsIgnoreCase("blocked")) {
            errorMessage = messages.getMessage("auth.message.blocked", null, locale);
        }

        ...
    }

}
```



## 4. 数据库支持的表单登录

### 4.1. UserDetailsService在认证中的作用

如果我们参考 Spring 安全架构，我们可以看到验证用户提供的身份验证令牌（用户名/密码、LDAP、OTP 或任何其他技术）是由我们插入 AuthenticationProvider 的 UserDetailsService 实现来处理的。

![img](https://howtodoinjava.com/wp-content/uploads/2022/05/Spring-Security-Architecture-1024x576.jpg)

UserDetailsService 的主要职责是从缓存或底层存储中通过用户名查找用户。加载 UserDetails 后，提供程序使用配置的密码编码器将存储的密码与用户提供的密码进行匹配。

```java
UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
```

此流程帮助我们通过实现和注入适当的 UserDetailsService 实现来插入任何用户详细信息存储，而无需更改 spring 安全配置的任何其他部分。

#### 4.1.1 UserDetailsService的默认实现

AuthenticationProvider 的默认实现使用为 UserDetailsService 和 PasswordEncoder 提供的默认实现。

UserDetailsService 的默认实现仅在应用程序的内存中注册默认凭据。这些默认凭据是带有默认密码的 `"user"` ，该密码是在 Spring 上下文加载时写入应用程序控制台的随机生成的通用唯一标识符（UUID）。

```
Using generated security password: 78nh23h-sd56-4b98-86ef-dfas8f8asf8
```

请注意，UserDetailsService 始终与 `PasswordEncoder` 关联，该 `PasswordEncoder` 对提供的密码进行编码并验证密码是否与现有编码匹配。当我们替换UserDetailsService的默认实现时，我们还必须指定一个PasswordEncoder。

#### 4.1.2. 内存中的UserDetailsService

重写 UserDetailsService 的第一个非常基本的示例是 InMemoryUserDetailsManager。此类将凭据存储在内存中，然后 Spring Security 可以使用该凭据来验证传入请求。

> UserDetailsManager 扩展了 UserDetailsService 契约。除了继承的行为之外，它还提供了创建用户以及修改或删除用户密码的方法。 UserDetailsService 只负责通过用户名检索用户。

```java
	@Bean
  public UserDetailsService userDetailsService() {
    		UserDetails user = User.builder()
                .username("user")
                .password("password")
                .roles("USER")
                .build();
        UserDetails admin = User.builder()
                .username("admin")
                .password("password")
                .roles("USER", "ADMIN")
                .build();
        return new InMemoryUserDetailsManager(user, admin);
  }
  
  @Bean
  public PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
  }
```

上面的配置将 UserDetailsService 和 PasswordEncoder 类型的 bean 注册到 spring 上下文中，并且身份验证提供程序会自动使用它们。如果我们愿意的话，Spring 允许我们将用户服务和密码编码器直接设置到身份验证管理器。

之前的配置可以重写如下：

```java
@Configuration
class AppConfig extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(AuthenticationManagerBuilder auth) throws  Exception {
    UserDetailsService userDetailsService = new InMemoryUserDetailsManager();
    UserDetails user = User.withUsername("user")
        .password("password")
        .authorities("USER_ROLE")
        .build();
    userDetailsService.createUser(user);
    
    auth.userDetailsService(userDetailsService)
        .passwordEncoder(NoOpPasswordEncoder.getInstance());
  }
}
```

我们可以通过一个简单的测试来验证上面的配置。

```java
@Test
public void expectOKResponse_WhenAuthenticaionManagerIsTestedWithCorrectDetails() {
  AuthenticationManager authenticationManager = this.spring.getContext()
    .getBean(AuthenticationManager.class);
  Authentication authentication = authenticationManager
    .authenticate(UsernamePasswordAuthenticationToken
      .unauthenticated("user", "password"));
  assertThat(authentication.isAuthenticated()).isTrue();
}
```

> 请注意， `InMemoryUserDetailsManager` 不适用于生产就绪的应用程序。此类仅用于示例或概念证明。

#### 4.1.3. 数据库支持的UserDetailsService

为了从 SQL 数据库存储和检索用户名和密码，我们使用 `JdbcUserDetailsManager` 类。它通过 JDBC 直接连接到数据库。

默认情况下，它在数据库中创建两个表：

- USERS 
- AUTHORITIES 

默认 schema 文件位于 `users.ddl` 中。文件位置在常量 `JdbcDaoImpl.DEFAULT_USER_SCHEMA_DDL_LOCATION` 中给出。

要自定义默认架构并创建初始用户，我们可以编写自己的 `schema.sql` 和 `data.sql` 文件并将它们放置在应用程序的 /src/main/resources 文件夹中。当我们启动应用程序时，Spring Boot 会自动运行这些文件。

请注意，JdbcUserDetailsManager 需要一个 DataSource 来连接到数据库，因此我们也需要定义它。

```java
@EnableWebSecurity
public class AppSecurityConfig extends WebSecurityConfigurerAdapter {
  @Autowired
	private Environment env;

	@Bean
	public DataSource dataSource() {
		final DriverManagerDataSource dataSource = new DriverManagerDataSource();
		dataSource.setDriverClassName(env.getProperty("jdbc.driverClassName"));
		dataSource.setUrl(env.getProperty("jdbc.url"));
		dataSource.setUsername(env.getProperty("jdbc.user"));
		dataSource.setPassword(env.getProperty("jdbc.pass"));
		return dataSource;
	}
  
  @Bean
  public UserDetailsService jdbcUserDetailsService(DataSource dataSource) {
    JdbcUserDetailsManager users = new JdbcUserDetailsManager(dataSource);
    
    UserDetails user = User
      .withUsername("user")
      .password("password")
      .roles("USER_ROLE")
      .build();
    users.createUser(user);
    return users;
  }
  
  @Bean
  public PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
  }
}
```



### 4.2. 使用默认配置 JdbcUserDetailsManager

为了从 SQL 数据库存储和检索用户名和密码，我们使用 JdbcUserDetailsManager 类。它通过 JDBC 直接连接到数据库。请注意， `JdbcUserDetailsManager` 扩展了 JdbcDaoImpl，而 `JdbcDaoImpl` 实现了 `UserDetailsService` 接口。

```java
	@Bean
	public UserDetailsService jdbcUserDetailsService(DataSource dataSource) {
		return new JdbcUserDetailsManager(dataSource);
	}
```

### 4.3. 使用自定义配置 JdbcUserDetailsManager

默认模式很好，但它可能不适合所有用例。有时我们需要支持额外的列或不同的列以进行身份验证。

例如，我们可能要求用户通过电子邮件 ID 和密码登录。或者我们可能会要求用户提供额外的域名信息。这样的场景可能有很多。

在这些情况下，我们需要在文件 `schema.sql` 中提供自定义架构。我们可能需要插入表中的任何初始数据，我们可以将 SQL INSERT/UPDATE 查询放在 `data.sql` 文件中。将这两个文件放入应用程序的 /resources 文件夹中。

schema.sql

```sql
drop table if exists users cascade;
create table users(
  username varchar(50) not null primary key,
  password varchar(500) not null,
  enabled boolean not null
);

drop table if exists authorities;
create table authorities (
  username varchar(50) not null,
  authority varchar(50) not null,
  constraint fk_authorities_users foreign key(username) references users(username)
);
create unique index if not exists ix_auth_username on authorities (username,authority);

```

data.sql

```sql
insert into users (username, password, enabled)
  values ('user',
    '$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG',
    1);

insert into authorities (username, authority)
  values ('user', 'ROLE_USER');

insert into users (username, password, enabled)
  values ('admin',
    '$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG',
    1);

insert into authorities (username, authority)
  values ('admin', 'ROLE_ADMIN');
insert into authorities (username, authority)
  values ('admin', 'ROLE_USER');
```

现在我们需要将 `JdbcUserDetailsManager` 实例插入自动使用 AuthenticationProvider 的 Spring security。

SecurityConfig.java

```java
@EnableWebSecurity
public class SecurityConfig {
  @Autowired
  private DataSource dataSource;
  
  @Bean
  public UserDetailsService jdbcUserDetailsService(DataSource dataSource) {
    return new JdbcUserDetailsManager(dataSource);
  }
  @Bean
  public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
  }
  @Override
  protected void configure(final HttpSecurity http) throws Exception {
    //@formatter:off
    http.authorizeRequests()
        .antMatchers("/login").permitAll()
        .antMatchers("/**").hasAnyRole("USER", "ADMIN")
        .antMatchers("/admin/**").hasAnyRole("ADMIN")
        .and()
          .formLogin()
          .loginPage("/login")
          .loginProcessingUrl("/process-login")
          .defaultSuccessUrl("/home")
          .failureUrl("/login?error=true")
          .permitAll()
        .and()
          .logout()
          .logoutSuccessUrl("/login?logout=true")
          .invalidateHttpSession(true)
          .deleteCookies("JSESSIONID")
          .permitAll()
        .and()
          .csrf()
          .disable();
    //@formatter:on
  }
  @Override
  public void configure(WebSecurity web) {
    web.ignoring()
        .antMatchers("/resources/**", "/static/**");
  }
}
```

现在，如果我们使用凭据 `user:password` 或 `admin:password` 登录应用程序，我们将能够成功进行身份验证。

请注意，如果我们使用不同的列名称并且想要自定义 `UserDetailService.loadUserByUsername()` 使用的 SQL 查询，那么我们可以使用 JdbcUserDetailsManager 正确配置 SQL 查询。

```java
@Bean
public UserDetailsService jdbcUserDetailsService(DataSource dataSource) {
  String usersByUsernameQuery = "select username, password, enabled from tbl_users where username = ?";
  String authsByUserQuery = "select username, authority from tbl_authorities where username = ?";
  JdbcUserDetailsManager users = new JdbcUserDetailsManager(dataSource);
  userDetailsManager.setUsersByUsernameQuery(usersByUsernameQuery);
  userDetailsManager.setAuthoritiesByUsernameQuery(authsByUserQuery);
  return users;
}
```

### 4.4. 使用内存数据库测试表单登录

为了对登录表单功能进行单元测试，我们不应该连接应用程序的物理数据库。我们应该注入一个内存数据库并使用一些虚拟用户来测试登录表单的安全性。

一种好方法是创建测试配置并覆盖 `DataSource` bean，以便它连接到内存数据库而不是真实数据库。我们还可以根据需要创建一些用户。

在下面的示例中，我们使用应用程序的主要安全配置并将 PasswordEncoder 和 UserDetailsService 自动装配到测试配置中。此外，我们重写 `DataSource` bean 和 `AuthenticationManagerBuilder` 以添加一些用户。

```java
@Configuration
@Order(101)
class TestConfiguration extends WebSecurityConfigurerAdapter {
  @Autowired
  PasswordEncoder passwordEncoder;
  @Autowired
  UserDetailsService userDetailsService;
  
  @Bean
  public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.H2)
        .addScript(JdbcDaoImpl.DEFAULT_USER_SCHEMA_DDL_LOCATION)
        .build();
  }
  
  @Override
  protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    JdbcUserDetailsManager mgr = (JdbcUserDetailsManager) userDetailsService;
    var user = User.withUsername("user")
        .password(passwordEncoder.encode("password"))
        .roles("USER")
        .build();
    var admin = User.withUsername("admin")
        .password(passwordEncoder.encode("password"))
        .roles("ADMIN", "USER")
        .build();
    mgr.createUser(user);
    mgr.createUser(admin);
    auth.userDetailsService(mgr)
        .passwordEncoder(passwordEncoder);
  }
}
```

现在我们可以使用这个 `TestConfiguration` 类来测试登录表单功能。在以下测试中，我们使用 formLogin() 方法来处理提供的用户详细信息的身份验证，并使用authenticated() 和 unauthenticated() 方法测试结果。

```java
@ExtendWith({SpringTestContextExtension.class})
public class LoginFormWithInMemoryDatabaseTest {
  public final SpringTestContext spring = new SpringTestContext(this);
  @Autowired
  private MockMvc mvc;
  
  @BeforeEach
  public void setup() {
    spring.register(TestConfiguration.class, SecurityConfig.class).autowire();
  }
  
  @Test
  void contextLoads() throws Exception {
  }
  
  @Test
  void verifyUserIsUnauthenticated_WhenNotLoggedIn() throws Exception {
    mvc.perform(MockMvcRequestBuilders.get("/home"))
        .andExpect(unauthenticated());
  }
  
  @Test
  void verifyUserIsUnAuthenticated_WhenSubmitInCorrectDetails() throws Exception {
    mvc
        .perform(formLogin("/process-login")
            .user("user")
            .password("wrong-password"))
        .andExpect(unauthenticated());
  }
  
  @Test
  void verifyUserIsUnAuthenticated_WhenSubmitCorrectDetails() throws
      Exception {
    mvc
        .perform(formLogin("/process-login")
            .user("user")
            .password("password"))
        .andExpect(authenticated().withRoles("USER"));
  }
  
  @Test
  void verifyUserRoleIsAdmin_WhenSubmitAdminCredentials() throws Exception {
    mvc
        .perform(formLogin("/process-login")
            .user("admin")
            .password("password"))
        .andExpect(authenticated().withRoles("ADMIN", "USER"));
  }
}
```

