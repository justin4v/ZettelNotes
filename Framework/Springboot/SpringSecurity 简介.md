#Todo #Springboot #SpringSecurity

# 框架
![[SpringSecurity主要流程.svg]]

# 使用示例
## 启动

导入spring-boot-starter-security启动器后，Spring Security已经生效，默认拦截全部请求，如果用户没有登录，跳转到内置登录页面。

在浏览器输入：`http://localhost:8080/` 进入Spring Security内置登录页面

用户名：user。

密码：项目启动，打印在控制台中。


## 自定义用户名和密码

修改 **「application.yml」**  文件
- 静态用户，一般只在内部网络认证中使用，如：内部服务器1，访问服务器2  
```yaml
spring:  
  security:  
    user:  
      name: test  # 通过配置文件，设置静态用户名  
      password: test # 配置文件，设置静态登录密码  
```

# UserDetailsService
- 没有配置的时候，账号和密码是由Spring Security定义生成的。
- 实际项目中账号和密码都是从数据库中查询出来的。所以需要 **自定义逻辑控制认证逻辑** 。需要实现UserDetailsService 接口

- *UserDetailsService*
```java
/**
 * Core interface which loads user-specific data.
 * <p>
 * It is used throughout the framework as a user DAO and is the strategy used by the
 * {@link org.springframework.security.authentication.dao.DaoAuthenticationProvider
 * DaoAuthenticationProvider}.
 *
 * <p>
 * The interface requires only one read-only method, which simplifies support for new
 * data-access strategies.
 *
 * @see org.springframework.security.authentication.dao.DaoAuthenticationProvider
 * @see UserDetails
 *
 * @author Ben Alex
 */
public interface UserDetailsService {

	/**
	 * Locates the user based on the username. In the actual implementation, the search
	 * may possibly be case sensitive, or case insensitive depending on how the
	 * implementation instance is configured. In this case, the <code>UserDetails</code>
	 * object that comes back may have a username that is of a different case than what
	 * was actually requested..
	 *
	 * @param username the username identifying the user whose data is required.
	 *
	 * @return a fully populated user record (never <code>null</code>)
	 *
	 * @throws UsernameNotFoundException if the user could not be found or the user has no
	 * GrantedAuthority
	 */
	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}

```

```java
@Component  
public class UserSecurity implements UserDetailsService {  
  
    @Autowired  
    private UserService userService;  
  
    @Override  
    public UserDetails loadUserByUsername(String userName) throws UsernameNotFoundException {  
  
        User user = userService.login(userName);  
        System.out.println(user);  
        if (null==user){  
            throw new UsernameNotFoundException("用户名错误");  
        }  
        org.springframework.security.core.userdetails.User result =  
                new org.springframework.security.core.userdetails.User(  
                        userName,user.getPassword(), AuthorityUtils.createAuthorityList()  
                );  
        return result;  
    }  
  
}  
```


# PasswordEncoder 密码解析器

## PasswordEncoder

- **「PasswordEncoder」** 是 SpringSecurity 的密码解析器，用于**密码校验、加密** 。
- 自定义登录逻辑时需要在容器注入 PaswordEncoder 的bean。
- SpringSecurity 定义了很多 **PasswordEncoder** 的实现满足密码加密、密码校验需求。

### 自定义密码解析器

1.  实现 PasswordEncoder  接口
```java
/**  
 * 凭证匹配器，用于做认证流程的凭证校验使用的类型  
 * 其中有2个核心方法  
 * 1. encode - 把明文密码，加密成密文密码  
 * 2. matches - 校验明文和密文是否匹配  
 * */  
public class MyMD5PasswordEncoder implements PasswordEncoder {  
  
    /**  
     * 加密  
     * @param charSequence  明文字符串  
     * @return  
     */  
    @Override  
    public String encode(CharSequence charSequence) {  
        try {  
            MessageDigest digest = MessageDigest.getInstance("MD5");  
            return toHexString(digest.digest(charSequence.toString().getBytes()));  
        } catch (NoSuchAlgorithmException e) {  
            e.printStackTrace();  
            return "";  
        }  
    }  
  
    /**  
     * 密码校验  
     * @param charSequence 明文，页面收集密码  
     * @param s 密文 ，数据库中存放密码  
     * @return  
     */  
    @Override  
    public boolean matches(CharSequence charSequence, String s) {  
        return s.equals(encode(charSequence));  
    }  
  
     /**  
     * @param tmp 转16进制字节数组  
     * @return 饭回16进制字符串  
     */  
    private String toHexString(byte [] tmp){  
        StringBuilder builder = new StringBuilder();  
        for (byte b :tmp){  
            String s = Integer.toHexString(b & 0xFF);  
            if (s.length()==1){  
                builder.append("0");  
            }  
            builder.append(s);  
        }  
  
        return builder.toString();  
  
    }  
}  
```

2. 在配置类中指定自定义密码凭证匹配器

```java
/**  
  * 加密  
  * @return 加密对象  
  * 如需使用自定义密码凭证匹配器 返回自定义加密对象  
  * 例如: return new MD5PasswordEncoder();   
  */  
@Bean  
public PasswordEncoder passwordEncoder() {  
    return new BCryptPasswordEncoder(); //Spring Security 自带  
}  
```

# 登录配置
- **转发（forward）**：
	- 客户端发出请求；
	- 服务器接收到该请求，**做出相应的处理后跳转（返回）到新页面**；
	- 在服务器端完成转发。
- **重定向（redirect）**：
	- 客户端先发出请求；
	- 服务器接收到该请求，接收到之后不直接做处理；
	- **返回给 Explore 新的请求地址**，让 Explore 发出新的请求到服务器端；
	- 服务端处理之后返回给客户端（游览器完成重定向）。



## 方式一 转发

```java
http.formLogin()  
    .usernameParameter("name") // 设置请求参数中，用户名参数名称。 默认username  
    .passwordParameter("pswd") // 设置请求参数中，密码参数名称。 默认password  
    .loginPage("/toLogin") // 当用户未登录的时候，跳转的登录页面地址是什么？ 默认 /login  
    .loginProcessingUrl("/login") // 用户登录逻辑请求地址是什么。 默认是 /login  
    .failureForwardUrl("/failure"); // 登录失败后，请求转发的位置。Security请求转发使用Post请求。默认转发到：loginPage?error  
    .successForwardUrl("/toMain"); // 用户登录成功后，请求转发到的位置。Security请求转发使用POST请求。 
``` 

## 方式二 ：重定向

http.formLogin()  
    .usernameParameter("name") // 设置请求参数中，用户名参数名称。 默认username  
    .passwordParameter("pswd") // 设置请求参数中，密码参数名称。 默认password  
    .loginPage("/toLogin") // 当用户未登录的时候，跳转的登录页面地址是什么？ 默认 /login  
    .loginProcessingUrl("/login") // 用户登录逻辑请求地址是什么。 默认是 /login  
 .defaultSuccessUrl("/toMain",true); //用户登录成功后，响应重定向到的位置。GET请求。必须配置绝对地址。  
  .failureUrl("/failure"); // 登录失败后，重定向的位置。  

## 方式三：自定义登录处理器

### 自定义登录失败逻辑处理器

```java
/*自定义登录失败处理器*/  
public class MyAuthenticationFailureHandler implements AuthenticationFailureHandler {  
    private  String url;  
    private boolean isRedirect;  
  
  
    public MyAuthenticationFailureHandler(String url, boolean isRedirect) {  
        this.url = url;  
        this.isRedirect = isRedirect;  
    }  
  
    @Override  
    public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {  
        if (isRedirect){  
            httpServletResponse.sendRedirect(url);  
        }else {  
            httpServletRequest.getRequestDispatcher(url).forward(httpServletRequest,httpServletResponse);  
        }  
    }  
  
//get set 方法 省略  

```

### 自定义登录成功逻辑处理器

```java
/**  
 * 自定义登录成功后处理器  
 * 转发重定向，有代码逻辑实现  
 * */  
public class MyAuthenticationSuccessHandler implements AuthenticationSuccessHandler {  
    private String url;  
    private boolean isRedirect;  
  
    public MyAuthenticationSuccessHandler(String url, boolean isRedirect) {  
        this.url = url;  
        this.isRedirect = isRedirect;  
    }  
  
    /**  
     * @param request 请求对象 request.getRequestDispatcher.forward()  
     * @param response 响应对象 response.sendRedirect()  
     * @param authentication 用户认证成功后的对象。其中报换用户名权限结合，内容是  
     *                       自定义UserDetailsService  
     * */  
    @Override  
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {  
        if (isRedirect){  
            response.sendRedirect(url);  
        }else {  
            request.getRequestDispatcher(url).forward(request,response);  
        }  
    }  
  
//get set 方法 省略     
http.formLogin()  
    .usernameParameter("name") // 设置请求参数中，用户名参数名称。 默认username  
    .passwordParameter("pswd") // 设置请求参数中，密码参数名称。 默认password  
    .loginPage("/toLogin") // 当用户未登录的时候，跳转的登录页面地址是什么？ 默认 /login  
    .loginProcessingUrl("/login") // 用户登录逻辑请求地址是什么。 默认是 /login  
```


# 登录相关配置类

```java
@Configuration  
@EnableWebSecurity  
public class SecurityConfig extends WebSecurityConfigurerAdapter {  
  
    @Autowired  
    private  UserSecurity userSecurity;  
    @Autowired  
    private PersistentTokenRepository persistentTokenRepository;  
  
  
    /**  
     * 加密  
     * @return 加密对象  
     * 如需使用自定义加密逻辑 返回自定义加密对象  
     * return new MD5PasswordEncoder(); return new SimplePasswordEncoder();  
     */  
    @Bean  
    public PasswordEncoder passwordEncoder() {  
        return new BCryptPasswordEncoder(); //Spring Security 自带  
    }  
  
    @Override  
    protected void configure(HttpSecurity http) throws Exception {  
        // 配置登录请求相关内容。  
        http.formLogin()  
            .loginPage("/toLogin") // 当用户未登录的时候，跳转的登录页面地址是什么？ 默认 /login  
            .usernameParameter("name") // 设置请求参数中，用户名参数名称。 默认username  
            .passwordParameter("pswd") // 设置请求参数中，密码参数名称。 默认password  
            .loginProcessingUrl("/login") //设置登录 提交表单数据访问请求地址  
            .defaultSuccessUrl("/toMain")     
            .failureUrl("/toLogin");  
         //.successForwardUrl("/toMain")  
         //.failureForwardUrl("/toLogin");  
            //.successHandler(new LoginSuccessHandler("/toMain", true)) //自定义登录成功处理器  
                //.failureHandler(new LoginErrorHandler("/toLogin", true));  
  
        http.authorizeRequests()  
            //.antMatchers("/toLogin").anonymous() //只能匿名用户访问  
            .antMatchers("/toLogin", "/register", "/login", "/favicon.ico").permitAll() // /toLogin请求地址，可以随便访问。  
            .antMatchers("/**/*.js").permitAll() // 授予所有目录下的所有.js文件可访问权限  
            .regexMatchers(".*[.]css").permitAll() // 授予所有目录下的所有.css文件可访问权限  
            .anyRequest().authenticated(); // 任意的请求，都必须认证后才能访问。  
  
  
        // 配置退出登录  
        http.logout()  
                .invalidateHttpSession(true) // 回收HttpSession对象。退出之前调用HttpSession.invalidate() 默认 true  
                .clearAuthentication(true) // 退出之前，清空Security记录的用户登录标记。 默认 true  
                // .addLogoutHandler() // 增加退出处理器。  
                .logoutSuccessUrl("/") // 配置退出后，进入的请求地址。 默认是loginPage?logout  
                .logoutUrl("/logout"); // 配置退出登录的路径地址。和页面请求地址一致即可。  
  
        // 关闭CSRF安全协议。  
        // 关闭是为了保证完整流程的可用。  
        http.csrf().disable();  
    }  
  
  
   @Bean  
   public PersistentTokenRepository persistentTokenRepository(DataSource dataSource){  
        JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();  
        jdbcTokenRepository.setDataSource(dataSource);  
        //jdbcTokenRepository.setCreateTableOnStartup(true);  
        return jdbcTokenRepository;  
    }  
}  
```

#  角色权限

> **「hasAuthority(String)」** 判断角色是否具有特定权限

```java
http.authorizeRequests().antMatchers("/main1.html").hasAuthority("admin")  
```



> **「hasAnyAuthority(String ...)」** 如果用户具备给定权限中某一个，就允许访问

```java
http.authorizeRequests().antMatchers("/admin/read").hasAnyAuthority("xxx","xxx")   
```


> **「hasRole(String)」** 如果用户具备给定角色就允许访问。否则出现403

```java
//请求地址为/admin/read的请求，必须登录用户拥有'管理员'角色才可访问  
http.authorizeRequests().antMatchers("/admin/read").hasRole("管理员")   
```

> **「hasAnyRole(String ...)」** 如果用户具备给定角色的任意一个，就允许被访问

```java
//用户拥有角色是管理员 或 访客 可以访问 /guest/read  
http.authorizeRequests().antMatchers("/guest/read").hasAnyRole("管理员", "访客")  
```


> **「hasIpAddress(String)」** 请求是指定的IP就运行访问

```java
//ip 是127.0.0.1 的请求 可以访问/ip  
http.authorizeRequests().antMatchers("/ip").hasIpAddress("127.0.0.1")  
```

## 403 权限不足页面处理

1. 编写类实现接口**AccessDeniedHandler**

```java
/**  
 * @describe  403 权限不足  
 * @author: AnyWhere  
 * @date 2021/4/18 20:57  
 */  
@Component  
public class MyAccessDeniedHandler implements AccessDeniedHandler {  
    @Override  
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException e)   
            throws IOException, ServletException {  
  
        response.setStatus(HttpServletResponse.SC_OK);  
  
        response.setContentType("text/html;charset=UTF-8");  
  
        response.getWriter().write(  
                "<html>" +  
                        "<body>" +  
                        "<div style='width:800px;text-align:center;margin:auto;font-size:24px'>" +  
                        "权限不足，请联系管理员" +  
                        "</div>" +  
                        "</body>" +  
                        "</html>"  
  
        );  
  
        response.getWriter().flush();//刷新缓冲区  
    }  
}  
```

2. 配置类中配置 exceptionHandling
```java
// 配置403访问错误处理器。  
http.exceptionHandling().accessDeniedHandler(myAccessDeniedHandler);
```

## RememberMe 配置
```java
@Configuration  
@EnableWebSecurity  
public class SecurityConfig extends WebSecurityConfigurerAdapter {  
  @Override  
  protected void configure(HttpSecurity http) throws Exception {  
    //配置记住密码  
    http.rememberMe()  
        .rememberMeParameter("remember-me") // 修改请求参数名。 默认是remember-me  
        .tokenValiditySeconds(14*24*60*60) // 设置记住我有效时间。单位是秒。默认是14天  
        .rememberMeCookieName("remember-me") // 修改remember me的cookie名称。默认是remember-me  
        .tokenRepository(persistentTokenRepository) // 配置用户登录标记的持久化工具对象。  
        .userDetailsService(userSecurity); // 配置自定义的UserDetailsService接口实现类对象  
  
  }  
  @Bean  
  public PersistentTokenRepository persistentTokenRepository(DataSource dataSource){  
     JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();  
     jdbcTokenRepository.setDataSource(dataSource);  
     //jdbcTokenRepository.setCreateTableOnStartup(true);  
     return jdbcTokenRepository;  
  }  
}     
```


# Spring Security 注解

## @Secured

> 角色校验 ,请求到来访问控制单元方法时必须包含XX角色才能访问
> 
> 角色必须添加ROLE_前缀

```java
  @Secured({"ROLE_管理员","ROLE_访客"})  
  @RequestMapping("/toMain")  
  public String toMain(){  
      return "main";  
  }  
```

- 使用注解@Secured需要在配置类中添加注解 `@EnableGlobalMethodSecurity(securedEnabled = true)  ` 使 `@Secured` 注解生效

## @PreAuthorize

> 权限检验,请求到来访问控制单元之前必须包含xx权限才能访问，控制单元方法执行前进行角色校验

```java

   /**  
     * [ROLE_管理员, admin:read, admin:write, all:login, all:logout, all:error, all:toMain]  
     * @PreAuthorize   角色 、权限 校验 方法执行前进行角色校验  
     *  
     *  hasAnyAuthority()   
     *  hasAuthority()  
     *  
     *  hasPermission()  
     *  
     *  
     *  hasRole()     
     *  hasAnyRole()  
     * */  
  
    @PreAuthorize("hasAnyRole('ROLE_管理员','ROLE_访客')")  
    @RequestMapping("/toMain")  
    @PreAuthorize("hasAuthority('admin:write')")  
    public String toMain(){  
        return "main";  
    }  
```


## @PostAuthorize
> 权限检验,请求到来访问控制单元之后必须包含xx权限才能访问 ，控制单元方法执行完后进行角色校验

```java
/**  
     * [ROLE_管理员, admin:read, admin:write, all:login, all:logout, all:error, all:toMain]  
     * @PostAuthorize  角色 、权限 校验 方法执行后进行角色校验  
     *  
     *  hasAnyAuthority()  
     *  hasAuthority()  
     *  hasPermission()  
     *  hasRole()  
     *  hasAnyRole()  
     * */  
    @PostAuthorize("hasRole('ROLE_管理员')")  
    @RequestMapping("/toMain")  
    @PreAuthorize("hasAuthority('admin:write')")  
    public String toMain(){  
        return "main";  
    }  
```
- 使用 `@PreAuthorize` 和 `@PostAuthorize` 需要在配置类中配置注解 `@EnableGlobalMethodSecurity(prePostEnabled = true) ` 才能生效


## Spring Security 整合Thymeleaf 进行权限校验
```xml
<dependency>  
      <groupId>org.springframework.boot</groupId>  
      <artifactId>spring-boot-starter-thymeleaf</artifactId>  
</dependency>  
  
<dependency>  
     <groupId>org.thymeleaf.extras</groupId>  
     <artifactId>thymeleaf-extras-springsecurity5</artifactId>  
</dependency>
```


# Spring Security中CSRF

## 什么是CSRF?

CSRF（Cross-site request forgery）跨站请求伪造，也被称为“One Click Attack” 或者Session Riding。通过伪造用户请求访问受信任站点的非法请求访问。

跨域：只要网络协议，ip地址，端口中任何一个不相同就是跨域请求。

客户端与服务进行交互时，由于http协议本身是无状态协议，所以引入了cookie进行记录客户端身份。在cookie中会存放session id用来识别客户端身份的。在跨域的情况下，session id可能被第三方恶意劫持，通过这个session id向服务端发起请求时，服务端会认为这个请求是合法的，可能发生很多意想不到的事情。

## 通俗解释

CSRF就是别的网站非法获取我们网站Cookie值，我们项目服务器是无法区分到底是不是我们的客户端，只有请求中有Cookie，认为是自己的客户端，所以这个时候就出现了CSRF。

# 参考
1. [Spring Security 学习笔记，看了必懂！ (qq.com)](https://mp.weixin.qq.com/s?src=11&timestamp=1663117899&ver=4043&signature=UsN5purUNjQ2rb9dFTvgMTP9EMiZ6jKVFpsSdDWuu6mu0MHUIxLbMjKb1qmRktvaDqTRuG5uNFL3uOh0s7aUQYlkWFR8BDr5jOSTprIpuLrjRZKVyk9f2eUO6uUTFXyz&new=1)