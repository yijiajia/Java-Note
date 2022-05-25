



* 拦截器
* 全局声明
* 文件上传/下载
* HttpMessageConverter
* 服务器推送与异步
* SpringMvc的测试；



SpringMvc 除了常用的Controller 控制器以外，还有很多组件是很好用的。



## 自定义拦截器

​	拦截器可以处理在请求前后的业务逻辑，类似 Servlet 的Filter。实现上，一般是继承 HandlerInterceptorAdapter 类 或者 实现 HandlerInterceptor 即可实现自定义的拦截器。

​	然后通过在实现 WebMvcConfigurer 的实现类中重写 addInterceptors 方法，将自定义的拦截器bean注册到SpringMvc的拦截器链中。



* 继承 HandlerInterceptorAdapter 类，自定义拦截器

```java
public class XssInterceptor extends HandlerInterceptorAdapter {

    /**
     * 请求前处理
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("拦截器前置方法");
        long beginTime = System.currentTimeMillis();
        request.setAttribute("beginTime",beginTime);
        return super.preHandle(request, response, handler);
    }

    /**
     * 请求后处理
     * @param request
     * @param response
     * @param handler  执行方法
     * @param modelAndView 模型视图
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

        long beginTime = (Long)request.getAttribute("beginTime");

        request.removeAttribute("beginTime");

        System.out.println("请求耗时 :" + (System.currentTimeMillis() - beginTime) );

    }
}
```



* 在mvc的配置类中，重写addInterceptor 方法，注册自定义的拦截器；

```java
@EnableWebMvc
@Configuration
@ComponentScan(value = "com.example.controller")
public class WebMvcConfig implements WebMvcConfigurer {

    /**
     * 视图解析器的bean配置
     * @return
     */
    @Bean
    public ViewResolver viewResolver() {
        System.out.println("viewResolver init");
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        /**
         * 如果需要在jsp中使用jstl标签的话，需要加上这个视图，且要用这个引用，否则会报找不到方法的500错误
         * 项目中使用JSTL，SpringMVC会把视图由InternalView转换为JstlView。
         * 若使用Jstl的fmt标签，需要在SpringMVC的配置文件中配置国际化资源文件。
         * 需要引入 jstl.jar和standard.jar
         */
//        resolver.setViewClass(org.springframework.web.servlet.view.JstlView.class);
        resolver.setPrefix("/WEB-INF/page/");
        resolver.setSuffix(".jsp");
        resolver.setExposeContextBeansAsAttributes(true);
        return resolver;
    }


    @Bean
    public XssInterceptor initXssInterceptor() {
        return new XssInterceptor();
    }

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    /**
     * 静态资源的配置
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        System.out.println("addResourceHandlers init....");
        registry.addResourceHandler("/css/**").addResourceLocations("/WEB-INF/statics/css/");
        registry.addResourceHandler("/js/**").addResourceLocations("/WEB-INF/statics/js/");
        registry.addResourceHandler("/image/**").addResourceLocations("WEB-INF/statics/image/");
    }

    /**
     * 注册拦截器
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(initXssInterceptor());
    }
}
```





## 全局声明配置

​	通过使用注解@ControllerAdvice 可以将对于控制器的全局配置都放在一个位置里，注解了@Controller 的类的方法可使用@ExceptionHandler、@InitBinder、@ModelAttritbute 注解到方法上，这对所有注解了@RequestMapping 的控制器内的方法有效。

* @ExceptionHandler：用于全局处理控制器里的异常；
* @InitBinder ：用来设置 WebDataBinder，WebDataBinder 用来自动绑定前台请求参数到Model 中；
* @ModelAttribute：用于绑定键值对到Model对象中，在 @ControllerAdvice中注解表示所有的@RequestMappering注解的方法都能获取到该键值对；



常用于处理全局异常的配置。



## 文件的上传与下载

​	



##  类型转换器Convertor

​	Convertor 是SpringMvc 提供给我们使用的类型转换器接口，通过自定义的类型转换器就可实现在请求之前对参数进行类型转换；



### 自定义类型转换器

​	假设需要自定义请求的格式，需要将特殊的String字符串解析成Java Bean，那么就可以利用自定义类型转换器做请求参数的解析并转化为bean。

#### 1. 实现**Converter**接口



实现 Formatter 接口，重写parse 和 print 方法。

Formatter 只能将String 转换为另一种Java类型。



JavaBean的格式是这样的。

```java
public class Student {

    private int sid;

    private String name;

    private int age;

    private LocalDate createTime;
    
    // 省略 set get...
}
```



#### 1. 实现 Formatter

* StringToLocalDateFormatter 类型转换器

  目的是将请求参数中的String日期转换为LocalDate的类型。

```java
public class StringToLocalDateFormatter implements Formatter<LocalDate> {

    private DateTimeFormatter formatter;
    private String datePattern;

    public StringToLocalDateFormatter(String datePattern) {
        this.datePattern = datePattern;
        this.formatter = DateTimeFormatter.ofPattern(datePattern);
    }

    /**
     * 利用指定的 Local 将一个String解析成目标类型
     */
    @Override
    public LocalDate parse(String source, Locale time) throws ParseException {
        System.out.println("StringToLocalDateFormatter converter..");
        return LocalDate.parse(source,DateTimeFormatter.ofPattern(datePattern));
    }

    @Override
    public String print(LocalDate date, Locale locale) {
        return date.format(formatter);
    }
}
```



#### 2. Java配置新增自定义的类型转换器

```java
@EnableWebMvc
@Configuration
@ComponentScan(value = "com.example.controller")
public class WebMvcConfig implements WebMvcConfigurer {
	
	
	@Bean
    public StringToLocalDateFormatter stringToLocalDateFormatter() {
        return new StringToLocalDateFormatter("yyyy-MM-dd");
    }
    
    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addFormatter(stringToLocalDateFormatter());
    }

}
```



#### 3. 接口请求

```java
@RestController
@RequestMapping("/student")
public class StudentController {

    @GetMapping(value = "/converter")
    public String getStudentInfo(Student student) {
        System.out.println(student);
        return "success：" + student;
    }
}
```



当使用 localhost:8080/student/converter?sid=122&name=zhangsan&age=34&createTime=2021-11-07 访问接口时，即可将String 的日期转换为LocalDate的格式，同时注入Student的对象中。



## 相关Resolver 

​	Resolver 解析器，除开常见的视图解析器（`InternalResourceViewResolver`) 以外，参数解析器 以及 返回值解析器也是SpringMvc 常用的组件。

### HandlerMethodArgumentResolver

请求参数解析器，包含以下两个方法：

* supportsParameter：判断是否支持解析；返回true表示进入解析方法；
* resolveArgument：解析参数，注入值；

### 以获取用户登录态为例

#### 1. 自定义注解

@Login，方法注解，注解在方法上表明需要进行登录拦截

```java
@Target(value ={ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Login {
}
```

@LoginUser，参数注解，用于解析请求注入到方法参数

```java
/**
 * 登录用户信息
*/
@Target(value = {ElementType.PARAMETER})  //作用在参数上
@Retention(RetentionPolicy.RUNTIME)  //运行时检查
public @interface LoginUser {

}
```



#### 2. 自定义拦截器

自定义权限（token）拦截器，拦截方法上注解了 @Login的方法，解析请求头的数据，放入到请求属性中，用于后面的参数解析注入；

```java
/**
 * 权限(token)验证
 */
@Component
public class AuthorizationInterceptor extends HandlerInterceptorAdapter {


    @Autowired
    private JwtUtils jwtUtils;

    public static final String USER_KEY = "userId";

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Login annotation;
        if(handler instanceof HandlerMethod) {
            annotation = ((HandlerMethod) handler).getMethodAnnotation(Login.class);
        }else{
            return true;
        }

        if(annotation == null){
            return true;
        }

        //获取用户凭证
        String token = request.getHeader(jwtUtils.getHeader());
        if(StringUtils.isBlank(token)){
            token = request.getParameter(jwtUtils.getHeader());
        }

        //凭证为空
        if(StringUtils.isBlank(token)){
            throw new RRException(jwtUtils.getHeader() + "不能为空", HttpStatus.UNAUTHORIZED.value());
        }

        Claims claims = jwtUtils.getClaimByToken(token);
        if(claims == null || jwtUtils.isTokenExpired(claims.getExpiration())){
            throw new RRException(jwtUtils.getHeader() + "失效，请重新登录", HttpStatus.UNAUTHORIZED.value());
        }

        //设置userId到request里，后续根据userId，获取用户信息
        request.setAttribute(USER_KEY, Integer.valueOf(claims.getSubject()));

        return true;
    }

}
```



#### 3. 自定义方法参数解析器

```java
/**
 * 有@LoginUser注解的方法参数，注入当前登录用户
 */
@Component
@Slf4j
public class LoginUserHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {

    @Autowired
    private UserService userService;

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.getParameterType().isAssignableFrom(User.class) && parameter.hasParameterAnnotation(LoginUser.class);
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer container,
                                  NativeWebRequest request, WebDataBinderFactory factory) throws Exception {
        //获取用户ID
        Object object = request.getAttribute(AuthorizationInterceptor.USER_KEY, RequestAttributes.SCOPE_REQUEST);
        if(object == null){
            return null;
        }

        log.info("获取到的用户ID为{}",object);
        //获取用户信息
        User user = userService.getUserById((Integer)object);

        return user;
    }
}
```



4. mvc配置 自定义拦截器和参数解析器

```java
@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {

    @Autowired
    private AuthorizationInterceptor authorizationInterceptor;
    @Autowired
    private LoginUserHandlerMethodArgumentResolver loginUserHandlerMethodArgumentResolver;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 拦截user路径下的请求
        registry.addInterceptor(authorizationInterceptor).addPathPatterns("/user/**");
    }

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(loginUserHandlerMethodArgumentResolver);
    }
}
```



5. 在Handler上使用注解，实现用户登录态的自动注入；

```java
@Login
@GetMapping(value = "/user/devices")
public R getDeviceList(@LoginUser User user){
	// do something
}
```

注解了 @LoginUser 的 User对象，将从请求头中自动注入当前登录态的用户信息。





### HandlerMethodReturnValueHandler

> 返回值解析处理器



## 服务器推送与异步





## 测试

