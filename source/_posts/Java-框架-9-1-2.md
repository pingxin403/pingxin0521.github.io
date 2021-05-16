---
title:  Spring Security 原理
date: 2019-11-02 17:18:59
tags:
 - Java
 - 框架
categories:
 - Java
 - 框架
---

#### 短信验证码登录

Spring Security默认只提供了账号密码的登录认证逻辑，所以要实现手机短信验证码登录认证功能，我们需要模仿Spring Security账号密码登录逻辑代码来实现一套自己的认证逻辑。
<!--more-->

**短信验证码生成**

创建基础类SmsCode:

```java
@Data
public class SmsCode {

    private String code;
    private LocalDateTime expireTime;

    public SmsCode(String code, int expireIn) {
        this.code = code;
        this.expireTime = LocalDateTime.now().plusSeconds(expireIn);
    }

    boolean isExpire() {
        return LocalDateTime.now().isAfter(expireTime);
    }
}
```

SmsCode对象包含了两个属性：code验证码和expireTime过期时间。isExpire方法用于判断短信验证码是否已过期。

创建生成短信验证码接口：

```java
@Log
@RestController
public class ValidateController {

    public final static String SESSION_KEY_IMAGE_CODE = "SESSION_KEY_IMAGE_CODE";
    public final static String SESSION_KEY_SMS_CODE = "SESSION_KEY_SMS_CODE";
    private SessionStrategy sessionStrategy = new HttpSessionSessionStrategy();

    @GetMapping("/code/sms")
    public void createSmsCode(HttpServletRequest request, String mobile) {
        SmsCode smsCode = createSMSCode();
        sessionStrategy.setAttribute(new ServletWebRequest(request), SESSION_KEY_SMS_CODE + mobile, smsCode);

        log.info("登录验证为：" + smsCode.getCode());
    }

    private SmsCode createSMSCode() {
        String code = RandomStringUtils.randomNumeric(6);
        return new SmsCode(code, 60);
    }
   ....
}
```

这里我们使用createSMSCode方法生成了一个6位的纯数字随机数，有效时间为60秒。然后通过SessionStrategy对象的setAttribute方法将短信验证码保存到了Session中，对应的key为SESSION_KEY_SMS_CODE。保存到session 是为了后期验证校验。

**改造登录页**

在登录页添加

```jsx
<form class="login-page" action="/login/mobile" method="post">
    <div class="form">
        <h3>短信验证码登录</h3>
        <input type="text" placeholder="手机号" name="mobile" value="17777777777" required="required"/>
        <span style="display: inline">
            <input type="text" name="smsCode" placeholder="短信验证码" style="width: 50%;"/>
            <!--<a href="/code/sms?mobile=17777777777">发送验证码</a>-->
            <button onclick="requestButton()" type="button">发送验证码</button>
        </span>
        <br/>

        <button type="submit">登录</button>
    </div>
</form>
```

```xml
<script>
    function requestButton() {
        alert("hello world")
        $.get("/code/sms?mobile=17777777777");
    }
</script>
```

**添加短信验证码认证**

在Spring Security中，使用用户名密码认证的过程大致如下图所示：

![QqLLc9.png](https://s2.ax1x.com/2019/12/19/QqLLc9.png)

Spring Security使用UsernamePasswordAuthenticationFilter过滤器来拦截用户名密码认证请求，将用户名和密码封装成一个UsernamePasswordToken对象交给AuthenticationManager处理。AuthenticationManager将挑出一个支持处理该类型Token的AuthenticationProvider（这里为DaoAuthenticationProvider，AuthenticationProvider的其中一个实现类）来进行认证，认证过程中DaoAuthenticationProvider将调用UserDetailService的loadUserByUsername方法来处理认证，如果认证通过（即UsernamePasswordToken中的用户名和密码相符）则返回一个UserDetails类型对象，并将认证信息保存到Session中，认证后我们便可以通过Authentication对象获取到认证的信息了。

由于Spring Security并没用提供短信验证码认证的流程，所以我们需要仿照上面这个流程来实现：

![QqOi1H.png](https://s2.ax1x.com/2019/12/19/QqOi1H.png)



在这个流程中，我们自定义了一个名为SmsAuthenticationFitler的过滤器来拦截短信验证码登录请求，并将手机号码封装到一个叫SmsAuthenticationToken的对象中。在Spring Security中，认证处理都需要通过AuthenticationManager来代理，所以这里我们依旧将SmsAuthenticationToken交由AuthenticationManager处理。接着我们需要定义一个支持处理SmsAuthenticationToken对象的SmsAuthenticationProvider，SmsAuthenticationProvider调用UserDetailService的loadUserByUsername方法来处理认证。与用户名密码认证不一样的是，这里是通过SmsAuthenticationToken中的手机号去数据库中查询是否有与之对应的用户，如果有，则将该用户信息封装到UserDetails对象中返回并将认证后的信息保存到Authentication对象中。

为了实现这个流程，我们需要定义SmsAuthenticationFitler、SmsAuthenticationToken和SmsAuthenticationProvider，并将这些组建组合起来添加到Spring Security中。下面我们来逐步实现这个过程。

**定义SmsAuthenticationToken**

```java
public class SmsAuthenticationToken extends AbstractAuthenticationToken {

    private static final long serialVersionUID = 8151829567809593110L;

    private final Object principal;

    public SmsAuthenticationToken(String mobile) {
        super(null);
        this.principal = mobile;
        super.setAuthenticated(false);
    }

    public SmsAuthenticationToken(Collection<? extends GrantedAuthority> authorities, Object principal) {
        super(authorities);
        this.principal = principal;
        super.setAuthenticated(true);
    }

    @Override
    public Object getCredentials() {
        return null;
    }

    @Override
    public Object getPrincipal() {
        return this.principal;
    }
}
```

SmsAuthenticationToken包含一个principal属性，从它的两个构造函数可以看出，在认证之前principal存的是手机号，认证之后存的是用户信息。UsernamePasswordAuthenticationToken原来还包含一个credentials属性用于存放密码，这里不需要就去掉了。

**定义SmsAuthenticationFilter**

```java
public class SmsAuthenticationFilter extends AbstractAuthenticationProcessingFilter {

    private static final String MOBILE_KEY = "mobile";
    private boolean postOnly = true;

    public SmsAuthenticationFilter() {
        super(new AntPathRequestMatcher("/login/mobile", "POST"));
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request,
                                                HttpServletResponse response) throws AuthenticationException, IOException, ServletException {

        if (postOnly && !request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException("Authentication method not support: " + request.getMethod());
        }

        String mobile = request.getParameter(MOBILE_KEY);

        if (mobile == null) {
            mobile = "";
        }

        SmsAuthenticationToken authRequest = new SmsAuthenticationToken(mobile);

        setDetails(request, authRequest);

        return this.getAuthenticationManager().authenticate(authRequest);
    }

    protected void setDetails(HttpServletRequest request, SmsAuthenticationToken authRequest) {
        authRequest.setDetails(authenticationDetailsSource.buildDetails(request));
    }
}
```

构造函数中指定了当请求为/login/mobile，请求方法为POST的时候该过滤器生效。mobileParameter属性值为mobile，对应登录页面手机号输入框的name属性。attemptAuthentication方法从请求中获取到mobile参数值，并调用SmsAuthenticationToken的SmsAuthenticationToken(String mobile)构造方法创建了一个SmsAuthenticationToken。下一步就如流程图中所示的那样，SmsAuthenticationFilter将SmsAuthenticationToken交给AuthenticationManager处理。

**定义SmsAuthenticationProvider**

```java
@Data
public class SmsAuthenticationProvider implements AuthenticationProvider {

    private UserDetailsService userDetailsService;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        SmsAuthenticationToken smsAuthenticationToken = (SmsAuthenticationToken) authentication;
        UserDetails userDetails = userDetailsService.loadUserByUsername((String) authentication.getPrincipal());

        if (userDetails == null) {
            throw new InternalAuthenticationServiceException("未找到与该手机对应的用户");
        }

        SmsAuthenticationToken authenticationResult = new SmsAuthenticationToken(userDetails.getAuthorities(), userDetails);
        authenticationResult.setDetails(smsAuthenticationToken.getDetails());
        return authenticationResult;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return SmsAuthenticationToken.class.isAssignableFrom(aClass);
    }
}
```

其中supports方法指定了支持处理的Token类型为SmsAuthenticationToken，authenticate方法用于编写具体的身份认证逻辑。在authenticate方法中，我们从SmsAuthenticationToken中取出了手机号信息，并调用了UserDetailService的loadUserByUsername方法。该方法在用户名密码类型的认证中，主要逻辑是通过用户名查询用户信息，如果存在该用户并且密码一致则认证成功；而在短信验证码认证的过程中，该方法需要通过手机号去查询用户，如果存在该用户则认证通过。认证通过后接着调用SmsAuthenticationToken的SmsAuthenticationToken(Object principal, Collection<? extends GrantedAuthority> authorities)构造函数构造一个认证通过的Token，包含了用户信息和用户权限。

你可能会问，为什么这一步没有进行短信验证码的校验呢？实际上短信验证码的校验是在SmsAuthenticationFilter之前完成的，即只有当短信验证码正确以后才开始走认证的流程。所以接下来我们需要定一个过滤器来校验短信验证码的正确性。

**定义SmsCodeFilter**

```java
@Component
public class SmsCodeFilter extends OncePerRequestFilter {

    @Autowired
    private AuthenticationFailureHandler authenticationFailureHandler;

    private SessionStrategy sessionStrategy = new HttpSessionSessionStrategy();

    @Override
    protected void doFilterInternal(HttpServletRequest httpServletRequest,
                                    HttpServletResponse httpServletResponse, FilterChain filterChain) throws ServletException, IOException {
        if (StringUtils.equalsIgnoreCase("/login/mobile", httpServletRequest.getRequestURI())
                && StringUtils.equalsIgnoreCase(httpServletRequest.getMethod(), "post")) {
            try {
                validateCode(new ServletWebRequest(httpServletRequest));
            } catch (ValidateCodeException e) {
                authenticationFailureHandler.onAuthenticationFailure(httpServletRequest, httpServletResponse, e);
                return;
            }
        }
        filterChain.doFilter(httpServletRequest, httpServletResponse);
    }

    private void validateCode(ServletWebRequest servletWebRequest) throws ServletRequestBindingException {
        String smsCode = ServletRequestUtils.getStringParameter(servletWebRequest.getRequest(), "smsCode");
        String mobile = ServletRequestUtils.getStringParameter(servletWebRequest.getRequest(), "mobile");

        SmsCode codeInSession = (SmsCode) sessionStrategy.getAttribute(servletWebRequest, ValidateController.SESSION_KEY_SMS_CODE + mobile);

        if (StringUtils.isBlank(smsCode)) {
            throw new ValidateCodeException("验证码不能为空");
        }

        if (codeInSession == null) {
            throw new ValidateCodeException("验证码不存在");
        }

        if (codeInSession.isExpire()) {
            sessionStrategy.removeAttribute(servletWebRequest, ValidateController.SESSION_KEY_SMS_CODE + mobile);
            throw new ValidateCodeException("验证码已经过期");
        }

        if (!StringUtils.equalsIgnoreCase(codeInSession.getCode(), smsCode)) {
            throw new ValidateCodeException("验证码不正确");
        }

        sessionStrategy.removeAttribute(servletWebRequest, ValidateController.SESSION_KEY_SMS_CODE + mobile);
    }
}
```

**关联配置，配置生效**

在定义完所需的组件后，我们需要进行一些配置，将这些组件组合起来形成一个和上面流程图对应的流程。创建一个配置类SmsAuthenticationConfig：

```java
@Component
public class SmsAuthenticationConfig extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity> {

    @Autowired
    private AuthenticationFailureHandler authenticationFailureHandler;
    @Autowired
    private AuthenticationSuccessHandler authenticationSuccessHandler;
    @Autowired
    private UserDetailService userDetailService;

    @Override
    public void configure(HttpSecurity httpSecurity) throws Exception {
        SmsAuthenticationFilter authenticationFilter = new SmsAuthenticationFilter();
        authenticationFilter.setAuthenticationManager(httpSecurity.getSharedObject(AuthenticationManager.class));
        authenticationFilter.setAuthenticationSuccessHandler(authenticationSuccessHandler);
        authenticationFilter.setAuthenticationFailureHandler(authenticationFailureHandler);

        SmsAuthenticationProvider smsAuthenticationProvider = new SmsAuthenticationProvider();
        smsAuthenticationProvider.setUserDetailsService(userDetailService);

        httpSecurity.authenticationProvider(smsAuthenticationProvider)
                .addFilterAfter(authenticationFilter, UsernamePasswordAuthenticationFilter.class);
    }
}
```

在流程中第一步需要配置SmsAuthenticationFilter，分别设置了AuthenticationManager、AuthenticationSuccessHandler和AuthenticationFailureHandler属性。这些属性都是来自SmsAuthenticationFilter继承的AbstractAuthenticationProcessingFilter类中。

第二步配置SmsAuthenticationProvider，这一步只需要将我们自个的UserDetailService注入进来即可。

最后调用HttpSecurity的authenticationProvider方法指定了AuthenticationProvider为SmsAuthenticationProvider，并将SmsAuthenticationFilter过滤器添加到了UsernamePasswordAuthenticationFilter后面。

到这里我们已经将短信验证码认证的各个组件组合起来了，最后一步需要做的是配置短信验证码校验过滤器，并且将短信验证码认证流程加入到Spring Security中。在SecurityConfig的configure方法中添加如下配置：

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private MyAuthenticationSuccessHandler authenticationSuccessHandler;
    @Autowired
    private MyAuthenticationFailureHandler authenticationFailureHandler;
    @Autowired
    private ValidateCodeFilter validateCodeFilter;
    @Autowired
    private SmsCodeFilter smsCodeFilter;
    @Autowired
    private SmsAuthenticationConfig smsAuthenticationConfig;

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.addFilterBefore(validateCodeFilter, UsernamePasswordAuthenticationFilter.class) // 添加验证码校验过滤器
                .addFilterBefore(smsCodeFilter, UsernamePasswordAuthenticationFilter.class)  // 添加短信验证码校验过滤器
                .formLogin() // 表单登录
                .loginPage("/login.html")       // 登录跳转url
//                .loginPage("/authentication/require")
                .loginProcessingUrl("/login")   // 处理表单登录url
                .successHandler(authenticationSuccessHandler)
                .failureHandler(authenticationFailureHandler)
                .and()
                .authorizeRequests()            // 授权配置
                .antMatchers("/login.html", "/css/**", "/authentication/require", "/code/image","/code/sms").permitAll()  // 无需认证
                .anyRequest()                   // 所有请求
                .authenticated()                // 都需要认证
                .and().csrf().disable()
                .apply(smsAuthenticationConfig);  // 将短信验证码认证配置加到 Spring Security 中
    }
}
```

**测试**

重启项目，访问[http://localhost:9090/hello](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8080%2Fhello),跳转到登录页，点击发送验证码，控制台输出如下：

```
登录验证为：835177
```

输入验证码，登录成功，跳转到hello接口

### security原理分析

#### springSecurity过滤器链

springSecurity 采用的是责任链的设计模式，它有一条很长的过滤器链。现在对这条过滤器链的各个进行说明

1. WebAsyncManagerIntegrationFilter：将Security上下文与Spring Web中用于处理异步请求映射的 WebAsyncManager 进行集成。
2. SecurityContextPersistenceFilter：在每次请求处理之前将该请求相关的安全上下文信息加载到SecurityContextHolder中，然后在该次请求处理完成之后，将SecurityContextHolder中关于这次请求的信息存储到一个“仓储”中，然后将SecurityContextHolder中的信息清除
    例如在Session中维护一个用户的安全信息就是这个过滤器处理的。
3. HeaderWriterFilter：用于将头信息加入响应中
4. CsrfFilter：用于处理跨站请求伪造
5. LogoutFilter：用于处理退出登录
6. UsernamePasswordAuthenticationFilter：用于处理基于表单的登录请求，从表单中获取用户名和密码。默认情况下处理来自“/login”的请求。从表单中获取用户名和密码时，默认使用的表单name值为“username”和“password”，这两个值可以通过设置这个过滤器的usernameParameter 和 passwordParameter 两个参数的值进行修改。
7. DefaultLoginPageGeneratingFilter：如果没有配置登录页面，那系统初始化时就会配置这个过滤器，并且用于在需要进行登录时生成一个登录表单页面。
8. BasicAuthenticationFilter：检测和处理http basic认证
9. RequestCacheAwareFilter：用来处理请求的缓存
10. SecurityContextHolderAwareRequestFilter：主要是包装请求对象request
11. AnonymousAuthenticationFilter：检测SecurityContextHolder中是否存在Authentication对象，如果不存在为其提供一个匿名Authentication
12. SessionManagementFilter：管理session的过滤器
13. ExceptionTranslationFilter：处理 AccessDeniedException 和 AuthenticationException 异常
14. FilterSecurityInterceptor：可以看做过滤器链的出口
15. RememberMeAuthenticationFilter：当用户没有登录而直接访问资源时, 从cookie里找出用户的信息, 如果Spring Security能够识别出用户提供的remember me cookie, 用户将不必填写用户名和密码, 而是直接登录进入系统，该过滤器默认不开启。

#### security配置

```java
 
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailService).passwordEncoder(new BCryptPasswordEncoder());
    }
    @Override
    public void configure(WebSecurity web) throws Exception {

        web.ignoring().antMatchers("/resources/**/*.html", "/resources/**/*.js");
    }
    @Override
    protected void configure(HttpSecurity http) throws Exception {
       http.formLogin().loginPage("/login_page").passwordParameter("username").passwordParameter("password").loginProcessingUrl("/sign_in").permitAll()
               .and().authorizeRequests().antMatchers("/test").hasRole("test")
               .anyRequest().authenticated().accessDecisionManager(accessDecisionManager())
               .and().logout().logoutSuccessHandler(new MyLogoutSuccessHandler())
               .and().csrf().disable();
        http.addFilterAt(getAuthenticationFilter(),UsernamePasswordAuthenticationFilter.class);
        http.exceptionHandling().accessDeniedHandler(new MyAccessDeniedHandler());
        http.addFilterAfter(new MyFittler(), LogoutFilter.class);
    }
}

```

1. configure(AuthenticationManagerBuilder auth) 说明

   AuthenticationManager的建造器，配置AuthenticationManagerBuilder 会让security自动构建一个AuthenticationManager（该类的功能参考流程图）；如果想要使用该功能你需要配置一个UserDetailService和passwordEncoder。userDetailsService用于在认证器中根据用户传过来的用户名查找一个用户，passwordEncoder用于密码的加密与比对，我们存储用户密码的时候用passwordEncoder.encode()加密存储，在认证器里会调用passwordEncoder.matches()方法进行密码比对。

   如果重写了该方法，security会启用DaoAuthenticationProvider这个认证器，该认证就是先调用UserDetailsService.loadUserByUsername然后使用passwordEncoder.matches()进行密码比对，如果认证成功成功则返回一个Authentication对象

2. configure(WebSecurity web)说明

   这个配置方法用于配置静态资源的处理方式，可使用ant匹配规则

3. configure(HttpSecurity http) 说明

   这个配置方法是最关键的方法，也是最复杂的方法。我们慢慢掰开来说

   ```java
   http.formLogin().loginPage("/login_page").passwordParameter("username").passwordParameter("password").loginProcessingUrl("/sign_in").permitAll()
   ```

   这是配置登陆相关的操作从方法名可知，配置了登录页请求路径，密码属性名，用户名属性名，和登陆请求路径，permitAll()代表任意用户可访问

   ```java
   http.authorizeRequests().antMatchers("/test").hasRole("test").anyRequest().authenticated().accessDecisionManager(accessDecisionManager());
   ```

   以上配置是权限相关的配置，配置了一个“/test” url该有什么权限才能访问，anyRequest()表示所有请求，authenticated()表示已登录用户，accessDecisionManager（）表示绑定在url上的鉴权管理器

   为了对比，现在贴出另一个权限配置清单

   ```bash
   http.authorizeRequests().antMatchers("/tets_a/**","/test_b/**").hasRole("test").antMatchers("/a/**","/b/**").authenticated().accessDecisionManager(accessDecisionManager())
   ```

   我们可以看到权限配置的自由度很高，鉴权管理器可以绑定到任意url上；而且可以硬编码各种url权限;

   ```java
   http.logout().logoutUrl("/logout").logoutSuccessHandler(new MyLogoutSuccessHandler())
   ```

   登出相关配置，这里配置了登出url和登出成功处理器

   ```java
   http.exceptionHandling().accessDeniedHandler(new MyAccessDeniedHandler());
   ```

   上面代码是配置鉴权失败的处理器

   ```java
   http.addFilterAfter(new MyFittler(), LogoutFilter.class);
   http.addFilterAt(getAuthenticationFilter(),UsernamePasswordAuthenticationFilter.class);
   ```

   上面代码展示如何在过滤器链中插入自己的过滤器，addFilterBefore加在对应的过滤器之前addFilterAfter之后，addFilterAt加在过滤器同一位置，事实上框架原有的Filter在启动HttpSecurity配置的过程中，都由框架完成了其一定程度上固定的配置，是不允许更改替换的。根据测试结果来看，调用addFilterAt方法插入的Filter，会在这个位置上的原有Filter之前执行。

   注：关于HttpSecurity使用的是链式编程，其中http.xxxx.and.yyyyy这种写法和http.xxxx;http.yyyy写法意义一样。

#### 自定义authenticationManager和accessDecisionManager

重写authenticationManagerBean()方法，并构造一个authenticationManager



```java
@Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        ProviderManager authenticationManager = new ProviderManager(Arrays.asList(getMyAuthenticationProvider(),daoAuthenticationProvider()));
        return authenticationManager;
    }
```

定义构造AccessDecisionManager的方法并在配置类中调用，配置参考 configure(HttpSecurity http) 说明

```java
public AccessDecisionManager accessDecisionManager(){
        List<AccessDecisionVoter<? extends Object>> decisionVoters
                = Arrays.asList(
                new MyExpressionVoter(),
                new WebExpressionVoter(),
                new RoleVoter(),
                new AuthenticatedVoter());
        return new UnanimousBased(decisionVoters);

    }
```

投票管理器会收集投票器投票结果做统计，最终结果大于等于0代表通过；每个投票器会返回三个结果：-1（反对），0（通过），1（赞成）。

#### security 权限用户系统说明

1. UserDetails:security中的用户接口，我们自定义用户类要实现该接口，各个属性的含义自行百度

2. GrantedAuthority:security中的用户权限接口，自定义权限需要实现该接口

   ```
   @Data
   public class MyGrantedAuthority implements GrantedAuthority {
       private String authority;
   }
   ```

   authority权限字段，需要注意的是在config中配置的权限会被加上ROLE_前缀，比如我们的配置authorizeRequests().antMatchers("/test").hasRole("test")，配置了一个“test”权限但我们存储的权限字段（authority）应该是“ROLE_test”

3. UserDetailsService

   security用户service，自定义用户服务类需要实现该接口

   ```java
   @Service
   public class MyUserDetailService implements UserDetailsService {
       @Override
       public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
         return.....
       }
   }
   ```

   loadUserByUsername的作用在上文中已经说明；

4. SecurityContextHolder

   用户在完成登陆后security会将用户信息存储到这个类中，之后其他流程需要得到用户信息时都是从这个类中获得，用户信息被封装成SecurityContext ，而实际存储的类是SecurityContextHolderStrategy ，默认的SecurityContextHolderStrategy 实现类是ThreadLocalSecurityContextHolderStrategy 它使用了ThreadLocal来存储了用户信息。

   手动填充SecurityContextHolder示例：

   ```
   UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken("test","test",list);
   SecurityContextHolder.getContext().setAuthentication(token);
   ```

   对于token鉴权的系统

   我们就可以验证token后手动填充SecurityContextHolder，填充时机只要在执行投票器之前即可，或者干脆可以在投票器中填充，然后在登出操作中清空SecurityContextHolder。

#### security扩展说明

可扩展的有

- 鉴权失败处理器：security鉴权失败默认跳转登陆页面，我们可以
- 验证器
- 登陆成功处理器
- 投票器
- 自定义token处理过滤器
- 登出成功处理器
- 登陆失败处理器
- 自定义UsernamePasswordAuthenticationFilter

1. 鉴权失败处理器

   security鉴权失败默认跳转登陆页面，我们可以实现AccessDeniedHandler接口，重写handle()方法来自定义处理逻辑；然后参考配置类说明将处理器加入到配置当中

   ```java
   public class MyAccessDeniedHandler implements AccessDeniedHandler {
       @Override
       public void handle(HttpServletRequest httpServletRequest, HttpServletResponse response, AccessDeniedException e) throws IOException, ServletException {
           response.setContentType("application/json;charset=UTF-8");
           PrintWriter writer = response.getWriter();
           writer.write("{\"code\":\"403\",\"msg\":\"没有权限\"}");
           writer.close();
       }
   }
   ```

2. 验证器

   实现AuthenticationProvider接口来实现自己验证逻辑。需要注意的是在这个类里面就算你抛出异常，也不会中断验证流程，而是算你验证失败，我们由流程图知道，只要有一个验证器验证成功，就算验证成功，所以你需要留意这一点

   ```java
   public class MyAuthenticationProvider  implements AuthenticationProvider {
       private UserDetailsService userDetailsService;
       private BCryptPasswordEncoder bCryptPasswordEncoder;
   
       public MyAuthenticationProvider(UserDetailsService userDetailsService, BCryptPasswordEncoder bCryptPasswordEncoder) {
           this.userDetailsService = userDetailsService;
           this.bCryptPasswordEncoder = bCryptPasswordEncoder;
       }
   
       @Override
       public Authentication authenticate(Authentication authentication) throws AuthenticationException {
   //       这里写验证逻辑
           return null;
       }
   
       @Override
       public boolean supports(Class<?> aClass) {
           return true;
       }
   }
   ```

3. 登陆成功处理器

   在security中验证成功默认跳转到上一次请求页面或者路径为"/"的页面，我们同样可以自定义：继承SimpleUrlAuthenticationSuccessHandler这个类或者实现AuthenticationSuccessHandler接口。我这里建议采用继承的方式；SimpleUrlAuthenticationSuccessHandler是默认的处理器，采用继承可以契合里氏替换原则，提高代码的复用性和避免不必要的错误。

   ```java
   ublic class MyAuthenticationSuccessHandler extends SimpleUrlAuthenticationSuccessHandler {
       @Override
       public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
           //随便写点啥
       }
   }
   ```

4. 投票器

   投票器可继承WebExpressionVoter或者实现AccessDecisionVoter<FilterInvocation\>接口；WebExpressionVoter是security默认的投票器；我这里同样建议采用继承的方式；添加到配置的方式参考 配置类说明章节；

   注意：投票器vote方法返回一个int值；-1代表反对，0代表弃权，1代表赞成；投票管理器收集投票结果，如果最终结果大于等于0则放行该请求。

   ```java
   
   public class MyExpressionVoter extends WebExpressionVoter {
       @Override
       public int vote(Authentication authentication, FilterInvocation fi, Collection<ConfigAttribute> attributes) {
   //        这里写鉴权逻辑
           System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
           return 1 ;
       }
   }
   ```

5. 自定义token处理过滤器

   自定义token处理器继承自可OncePerRequestFilter或者GenericFilterBean或者Filter都可以，在这个处理器里面需要完成的逻辑是：获取请求里的token，验证token是否合法然后填充SecurityContextHolder，虽然说过滤器只要添加在投票器之前就可以；但我这里还是建议添加在http.addFilterAfter(new MyFittler(), LogoutFilter.class);

   ```java
   
   
   public class MyFittler extends OncePerRequestFilter {
       @Override
       protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
           String token1 = request.getHeader("token");
           if (token1==null){
   
           }
   
           ArrayList<GrantedAuthority> list = new ArrayList<>();
           GrantedAuthority grantedAuthority = new GrantedAuthority() {
               @Override
               public String getAuthority() {
                   return "test";
               }
           };
           list.add(grantedAuthority);
           UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken("test","test",list);
           SecurityContextHolder.getContext().setAuthentication(token);
           filterChain.doFilter(request, response);
       }
   }
   ```

6. 登出成功处理器

   实现LogoutSuccessHandler接口

   ```java
   public class MyLogoutSuccessHandler implements LogoutSuccessHandler {
       @Override
       public void onLogoutSuccess(HttpServletRequest httpServletRequest, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
           final PrintWriter writer = response.getWriter();
   
           writer.write("{\"code\":\"200\",\"msg\":\"登出成功\"}");
           writer.close();
       }
   }
   ```

7. 登陆失败处理器

   登陆失败默认跳转到登陆页，我们同样可以自定义。继承SimpleUrlAuthenticationFailureHandler 或者实现AuthenticationFailureHandler；建议采用继承。

   ```java
   public class MyUrlAuthenticationFailureHandler extends SimpleUrlAuthenticationFailureHandler {
       @Override
       public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
           response.setContentType("application/json;charset=UTF-8");
           final PrintWriter writer = response.getWriter();
           if(exception.getMessage().equals("坏的凭证")){
               writer.write("{\"code\":\"401\",\"msg\":\"登录失败,用户名或者密码有误\"}");
               writer.close();
           }else {
               writer.write("{\"code\":\"401\",\"msg\":\"登录失败,"+exception.getMessage()+"\"}");
               writer.close();
           }
   
       }
   }
   ```

8. 自定义UsernamePasswordAuthenticationFilter

   我们自定义UsernamePasswordAuthenticationFilter可以极大提高我们security的灵活性（比如添加验证验证码是否正确的功能），所以我这里是建议自定义UsernamePasswordAuthenticationFilter；

   我们直接继承UsernamePasswordAuthenticationFilter，然后在配置类中初始化这个过滤器，给这个过滤器添加登陆失败处理器，登陆成功处理器，登陆管理器，登陆请求url

   ```java
   
   public class MyUsernamePasswordAuthenticationFilte extends UsernamePasswordAuthenticationFilter {
       private RedisService redisService;
       private boolean postOnly = true;
   
       public MyUsernamePasswordAuthenticationFilte(RedisService redisService){
           this.redisService=redisService;
       }
   
       @Override
       public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        
           //你可以在这里做验证码校验，校验不通过抛出AuthenticationException()即可
               super.attemptAuthentication(request,response);
       }
   }
   ```

**配置**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    RedisService redisService;
    @Autowired
    MyUserDetailService userDetailService;
  
    @Override
    public void configure(WebSecurity web) throws Exception {

        web.ignoring().antMatchers("/resources/**/*.html", "/resources/**/*.js",
                "/resources/**/*.css", "/resources/**/*.txt",
                "/resources/**/*.png", "/**/*.bmp", "/**/*.gif", "/**/*.png", "/**/*.jpg", "/**/*.ico");
//        super.configure(web);
    }


    @Override
    protected void configure(HttpSecurity http) throws Exception {
//        配置登录页等 permitAll表示任何权限都能访问
       http.formLogin().loginPage("/login_page").passwordParameter("username").passwordParameter("password").loginProcessingUrl("/sign_in").permitAll()
               .and().authorizeRequests().antMatchers("/test").hasRole("test")
//               任何请求都被accessDecisionManager() 的鉴权器管理
               .anyRequest().authenticated().accessDecisionManager(accessDecisionManager())
//               登出配置
               .and().logout().logoutUrl("/logout").logoutSuccessHandler(new MyLogoutSuccessHandler())
//               关闭csrf
               .and().csrf().disable();
       http.authorizeRequests().antMatchers("/tets_a/**","/test_b/**").hasRole("test").antMatchers("/a/**","/b/**").authenticated().accessDecisionManager(accessDecisionManager())
//      加自定义过滤器
        http.addFilterAt(getAuthenticationFilter(),UsernamePasswordAuthenticationFilter.class);
//        配置鉴权失败的处理器
        http.exceptionHandling().accessDeniedHandler(new MyAccessDeniedHandler());
        http.addFilterAfter(new MyFittler(), LogoutFilter.class);

    }


    MyUsernamePasswordAuthenticationFilte getAuthenticationFilter(){
        MyUsernamePasswordAuthenticationFilte myUsernamePasswordAuthenticationFilte = new MyUsernamePasswordAuthenticationFilte(redisService);
        myUsernamePasswordAuthenticationFilte.setAuthenticationFailureHandler(new MyUrlAuthenticationFailureHandler());
        myUsernamePasswordAuthenticationFilte.setAuthenticationSuccessHandler(new MyAuthenticationSuccessHandler());
        myUsernamePasswordAuthenticationFilte.setFilterProcessesUrl("/sign_in");
        myUsernamePasswordAuthenticationFilte.setAuthenticationManager(getAuthenticationManager());
        return myUsernamePasswordAuthenticationFilte;
    }
    MyAuthenticationProvider getMyAuthenticationProvider(){
        MyAuthenticationProvider myAuthenticationProvider = new MyAuthenticationProvider(userDetailService,new BCryptPasswordEncoder());
        return myAuthenticationProvider;
    }
    DaoAuthenticationProvider daoAuthenticationProvider(){
        DaoAuthenticationProvider daoAuthenticationProvider = new DaoAuthenticationProvider();
        daoAuthenticationProvider.setPasswordEncoder(new BCryptPasswordEncoder());
        daoAuthenticationProvider.setUserDetailsService(userDetailService);
        return daoAuthenticationProvider;
    }
    protected AuthenticationManager getAuthenticationManager()  {
        ProviderManager authenticationManager = new ProviderManager(Arrays.asList(getMyAuthenticationProvider(),daoAuthenticationProvider()));
        return authenticationManager;
    }

    public AccessDecisionManager accessDecisionManager(){
        List<AccessDecisionVoter<? extends Object>> decisionVoters
                = Arrays.asList(
                new MyExpressionVoter(),
                new WebExpressionVoter(),
                new RoleVoter(),
                new AuthenticatedVoter());
        return new UnanimousBased(decisionVoters);

    }
}
```

#### 总结

对于security的扩展配置关键在于`configure(HttpSecurity http)`方法；扩展认证方式可以自定义`authenticationManager`并加入自己验证器，在验证器中抛出异常不会终止验证流程；扩展鉴权方式可以自定义`accessDecisionManager`然后添加自己的投票器并绑定到对应的url（url 匹配方式为ant）上，投票器`vote(Authentication authentication, FilterInvocation fi, Collection attributes)`方法返回值为三种：-1 0 1，分别表示反对弃权赞成；

对于token认证的校验方式，可以暴露一个获取的接口，或者重写`UsernamePasswordAuthenticationFilter`过滤器和扩展登陆成功处理器来获取token，然后在`LogoutFilter`之后添加一个自定义过滤器，用于校验和填充SecurityContextHolder

security的处理器大部分都是重定向的，我们的项目如果是前后端分离的话，我们希望无论什么情况都返回json,那么就需要重写各个处理器了。