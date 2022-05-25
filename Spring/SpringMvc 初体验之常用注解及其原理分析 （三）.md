## SpringMvc 常用的注解



### @Controller

​	标记@Controller的类被称作为控制器类，使其无需实现Controller即可表示为控制器类，告知 DispatcherServlet 前端控制器这是一个Controller对象。

​	要使Spring 容器认识它，则需要在配置文件中标记该bean；

```xml
<!-- 方式 1，单独标记 -->
<bean class="com.example.controller.IndexController"/>
<!-- 方式 2，Controller 包的扫描 -->
<context:component-scan base-package="com.example.controller"/>
```



#### @RequestMapping

​	标记URL与方法之间的关系，映射请求Url对应处理的Handler；

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
    // 映射名，一般不用设置
    String name() default "";

    // 请求路径，可配置多个路径，与path功能相同
    @AliasFor("path")
    String[] value() default {};

    // 请求路径，与value功能相同
    @AliasFor("value")
    String[] path() default {};

    // 请求方法，可多种类型
    RequestMethod[] method() default {};

    // 指定参数类型
    String[] params() default {};

    // 指定请求头
    String[] headers() default {};

    // 指定数据请求的格式，如application/json
    String[] consumes() default {};

    // 指定返回的内容类型，如application/json
    String[] produces() default {};
}
```

注：属性中的@AliasFor注解表明这个互为别名，即path属性 跟 value 属性的作用是一致的。

需要知道的是，@RequestMapping 上的属性类似于说明对这个请求的限制，也即这个请求需满足RequestMapping 的属性内容，才能被该Handler 处理。

```java
@RequestMapping(value = {"requestMapping1","requestMapping2"}
            ,method = {RequestMethod.GET,RequestMethod.POST}
            ,consumes = "application/x-www-form-urlencoded"
            ,produces = "application/json"
            ,headers = {"Content-Type=application/json"} )
    @ResponseBody
    public String requestMapping() {
        return "success";
    }

```

在上面的例子中，requestMapping 方法 指定了  url="/requestMapping1"  和 "requestMapping2" 都可访问到该请求。同时要求请求方法为 GET 或 POST的一种，数据请求的格式可以为 consumes 指定的 application/x-www-form-urlencoded，也可以为 headers 里表示的application/json。在返回给客户端的内容类型为 application/json。



#### @RestController

​	@Controller 与 @ResponseBody 的组合注解，标记在类上表示这个类为控制器类，同时返回的内容为字符串；

#### @RequestParam

​	标记url请求的参数，用于方法参数上的注解，解析请求参数名对应的值绑定到方法参数上。

```java
@Target({ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestParam {
    
    // 参数名
    @AliasFor("name")
    String value() default "";

    // 参数名，与value一致
    @AliasFor("value")
    String name() default "";

    // 是否必须，如果为false，表示可不传该参数，默认为null，如果参数是基本数据类型的变量，此时会报空指针异常
    boolean required() default true;

    // 参数的默认值，如果没传该参数，将采用该默认值，默认值可以使用SpEL表达式
    String defaultValue() default "\n\t\t\n\t\t\n\ue000\ue001\ue002\n\t\t\t\t\n";
}
```

使用示例：

```Java
 @RequestMapping(value = "list",method = RequestMethod.GET)
    public String getUserList(@RequestParam(required = false,defaultValue = "0",value = "page") int pageNum, @RequestParam(required = false,defaultValue = "10",value = "limit") int limit ) {
        return userService.getUserList(pageNum,limit);
    }
```



事实上，不加 @RequestParam 注解，只要请求参数与方法参数名一致也是能绑定的。不过使用注解可以灵活控制参数，例如控制参数是否必须，默认值，以及参数别名等。



#### @PathVariable

​	标记请求路径变量的注解，结合@RequestMapping 上的SpEl表达式使用，用于方法参数上。适用于链接上的动态参数标记。

```java
 @RequestMapping(value = "info/{uid}",method = RequestMethod.GET)
 public String getUserInfoById(@PathVariable(value = "uid") String uid) {
       return uid;
 }
```

当请求参数与方法参数名一致时，可省略@PathVariable 上的值。



#### @ResponseBody

​	作用是将Java对象转为特定的数据格式，如Json 、xml等，直接输出数据到response对象的body部分。使用该注解不会将方法的返回值解析为跳转路径，而RequestBody

#### @RequestBody

​	作用是将 HTTP 请求正文插入方法中，使用适合的 HttpMessageConverter 将请求体写入某个对象。使用该注解会解析request请求中body部分的数据，由默认的HttpMessageConverter负责解析，将数据绑定在参数对象上。

​	一般用于处理非 `Content-Type: application/x-www-form-urlencoded`编码格式的数据，比如：`application/json`、`application/xml`等类型的数据。



参考链接：https://blog.csdn.net/weixin_38004638/article/details/99655322



#### @PostMapping

​	相当于 @RequestMapping(method = RequestMethod.POST)，接收Post 请求；



### @ModelAttribute

​	主要的作用是将数据添加到模型对象中，用于视图页面显示。在不同的位置上使用该注解，作用也会不同。

#### 1. 作用在方法上

@ModelAttribute 注解的方法会在Controller 每个方法执行前都会被调用，有点类型 Junit的@Before 方法。如果有返回值，则自动将返回值加入到ModelMap中。

* 注解在返回值为void的方法上。

  一般需要在方法上主动使用Model对象将属性set到模型对象中去；

  ```java
    @ModelAttribute
      public void addModelAttribute(String name, Model model) {
          model.addAttribute("name",name);
      }
  ```

* 注解有返回值的方法上。

  使用 @ModelAttribute  可以将返回值添加到 mvc的模型对象中；默认的key为返回对象类型的小写；

  ```java
  //    @ModelAttribute // 如果使用该注解的话，相当于在模型视图中有 "string":name 这样的属性
      @ModelAttribute("name") // 使用该注解会将返回值添加到模型视图中，即 "name" : name
      public String addModelAttribute(@RequestParam String name) {
          return name;
      }
  ```

* 与@RequestMapping 一起使用

  ```java
  	@RequestMapping("/model")
      @ModelAttribute("name")
      public String name(String name) {
          return name;
      }
  ```

  以上面的例子来说，访问 localhost:port/model?name=zhangsan 会访问到 项目根目录下的 model.jsp 文件（假设后缀为.jsp），同时会将参数name添加到模型视图中。即在model.jsp页面上使用该表达式将输出 zhangsan。 

  注：即使当前 Controller 中有 @ResquestBody 注解也会转发到对应的页面；
  
  ```jsp
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
  <head>
      <title>Title</title>
      ${name}
  </head>
  <body>
  
  </body>
  </html>
  ```

#### 2. 作用在参数上

​	运用在参数上，会将客户端传递过来的参数按名称注入到指定对象中，并且会将这个对象自动加入ModelMap中，便于View层使用；

用在方法的入参上依次做如下操作：

- 从隐含对象中获取隐含的模型数据

- 将请求参数绑定到隐含对象中

- 将隐含对象传入到入参

- 将入参绑定到Model

  

### @CookieValue

​	用于获取Cookie值的注解。

```java
  @GetMapping("verify")
    public String verifyUser(@CookieValue("_TOKEN") String token) {
        // check token
        return "";
    }
```

上例中，即是获取请求中的_TOKEN的cookie；



#### @SessionAttributes

> 1. 若希望在多个请求之间共用数据，则可以在控制器类上标注一个 @SessionAttributes,配置需要在session中存放的数据范围，Spring MVC将存放在model中对应的数据暂存到HttpSession 中。
>
> 2. @SessionAttributes只能使用在类定义上。
>
> 3. @SessionAttributes 除了可以通过属性名指定需要放到会 话中的属性外，还可以通过模型属性的对象类型指定哪些模型属性需要放到会话中 例如：
>
>    @SessionAttributes(types=User.class)会将model中所有类型为 User的属性添加到会话中。
>    @SessionAttributes(value={“user1”, “user2”}) 会将model中属性名为user1和user2的属性添加到会话中。
>    @SessionAttributes(types={User.class, Dept.class}) 会将model中所有类型为 User和Dept的属性添加到会话中。
>    @SessionAttributes(value={“user1”,“user2”},types={Dept.class})会将model中属性名为user1和user2以及类型为Dept的属性添加到会话中。
>
> 4. **value和type之间是并集关系**



@SessionAttributes 只能使用在类定义上。

```java
@Controller
@SessionAttributes("user")
public class ModelController {

    @ModelAttribute("user")
    public User initUser(){
        User user = new User();
        user.setName("default");
        return user;
    }

}
```

@SessionAttribute是用于获取已经存储的session数据，并且作用在方法的层面上。

```java
   @RequestMapping("/session")
    public String session(@SessionAttribute("user") User user){
        // do something
        return "index";
    }
```



 ## 注解原理分析

