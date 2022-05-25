> 了解SpringMvc的执行流程及组件，方便在开发过程中如果出现了问题，就可以直到在哪出现的问题，从而缩小查找范围，有的放矢地去解决。

## SpringMvc的执行流程

​	SpringMvc 的执行流程主要逻辑封装在 DispatcherServlet # doDispatch 方法中，处理流程大概如下：

![image-20211107183529932](C:\Users\lsk\AppData\Roaming\Typora\typora-user-images\image-20211107183529932.png)

分步骤描述为：

 	1. 发送请求，请求会先到 DispatcherServlet 控制器，执行doService方法；
 	2. 设置上下文属性，执行 doDispatch 方法；
 	3. 在HandlerMappering 处理器映射器列表中获取 HandlerExecutionChain 执行器链，在返回的HandlerExecutionChain 执行器链中包括 HandlerInterceptor 拦截器 和 Handler 处理器；

![image-20211107183127897](C:\Users\lsk\AppData\Roaming\Typora\typora-user-images\image-20211107183127897.png)

4. 通过 Handler 处理器获取到 对应的 HandlerAdapter 适配器 （通过遍历适配器列表找到匹配的适配器）

![image-20211107183049069](C:\Users\lsk\AppData\Roaming\Typora\typora-user-images\image-20211107183049069.png)

5. 通过适配器执行handler方法，即目标接口方法，返回ModelAndView ；
6. 处理处理默认的视图；
7. 调用执行器链中的拦截器的后置方法；
8. 执行processDispatchResult方法，处理请求的返回结果；



## SpringMvc的各组件分析

### DispatcherServlet

> 前端控制器，用于控制请求流转，调度等工作。

### HandlerMapping

> 即处理器映射器，处理器即是指常见的Controller。主要负责根据request的请求找到对应的Handler处理器及Interceptor拦截器，将它们封装在 HandlerExecutionChain 中 返回给前端控制器，即DispatcherServlet

#### BeanNameUrlHandlerMapping

根据url与Spring 容器中定义的bean的name进行匹配，从而在Spring容器中获取到对应的bean实例；筛选出 Name 或者 别名以 "/" 开头的 Bean ，将这些 Bean 注册为 “Handler”，实现 URL 映射。

```xml
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" id="beanNameUrlHandlerMapping"/>
```

#### SimpleUrlHandlerMapping

​	与 BeanNameUrlHandlerMapping 类似，可以统一配置bean的id 与 url的映射；



#### RequestMappingHandlerMapping

​	比较常用的注解 @RequestMapping 对应的映射器，处理注解中的url 与 handler的对应关系；



### HandlerAdapter

> 处理器适配器，用于执行具体Handler的接口；使用Adpapter适配器来执行Handler处理器是因为SpringMvc对于Handler没做什么限制，处理器可以以任意合理的方式来表现，可以是个类，也可以是个方法。从方法的签名中也看到Handler的类型是Object的；

接口定义很简单，只有三个方法：

```java
public interface HandlerAdapter {
    // 判断是否可以使用某个Handler
    boolean supports(Object handler);

	// 执行Handler，返回视图	
    @Nullable
    ModelAndView handle(HttpServletRequest var1, HttpServletResponse var2, Object handler) throws Exception;

	// 获取资源的Last-Modified,Last-Modified 指资源最后一次的修改时间
    long getLastModified(HttpServletRequest var1, Object handler);
}
```

​	从上面SpringMvc执行的流程分析中可以发现，获取HandlerAdapter的逻辑很简单，遍历所有HandlerAdapter，找到第一个能处理此Handler的Adapter就会被使用。



#### SimpleControllerHandlerAdapter

​	实现了Controller接口的Handler会在此适配器被调用，与下面的 HttpRequestHandlerAdapter 类似；

#### HttpRequestHandlerAdapter

​	HttpRequestHandler 的适配器，实现 HttpRequestHandler的子类即可被该适配器使用，执行HttpRequestHandler 的handleRequest 方法；

```java
public class HttpRequestHandlerAdapter implements HandlerAdapter {
    public HttpRequestHandlerAdapter() {
    }

	// 判断handler 是否为 HttpRequestHandler 类型； 
    public boolean supports(Object handler) {
        return handler instanceof HttpRequestHandler;
    }

    @Nullable
    public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    	// 强转为 HttpRequestHandler，执行 handleRequest
        ((HttpRequestHandler)handler).handleRequest(request, response);
        return null;
    }

    public long getLastModified(HttpServletRequest request, Object handler) {
        return handler instanceof LastModified ? ((LastModified)handler).getLastModified(request) : -1L;
    }
}
```



#### RequestMappingHandlerAdapter

​	该适配器继承于抽象类 AbstractHandlerMethodAdapter，实现了InitializingBean，实现初始化bean；主要处理@RequestMapping 注解类型对应的handler方法；



### ViewResolver

​	视图解析器的接口定义，根据视图名 及 本地化参数解析出对应的视图View。其中Locale用于国际化支持。

```java
public interface ViewResolver {
    @Nullable
    View resolveViewName(String viewName, Locale local) throws Exception;
}
```

#### UrlBasedViewResolver

​	简单的url视图解析器，通过拼接url的方式来解析视图；下面是该视图解析器的常用配置：

```xml
<bean class="org.springframework.web.servlet.view.UrlBasedViewResolver">
    <property name="prefix" value="/WEB-INF/page/"/>
    <property name="suffix" value=".jsp"/>
    <property name="viewClass" value="org.springframework.web.servlet.view.InternalResourceView"/>
</bean>
```

其中，prefix 属性表示前缀，suffix 表示后缀，viewClass 属性表示解析成哪种视图，InternalResourceView 算是用的比较多的。


#### InternalResourceViewResolver

​	内部资源视图解析器，是 UrlBasedViewResolver 的子类。 InternalResourceViewResolver  会利用 RequestDispatcher 在服务器端把请求forward到/WEB-INF/下。



## 写在最后

​	在SpringMvc中存在多个 HandlerMapping、HandlerAdapter、ViewResolver组件，多个组件一般都是一条单独的链结构，通过实现Ordered接口进行排序，顺序越小说明优先级越高。获取对应组件时，采用的是责任链的设计模式，通过循环各个组件链，直到有一个组件能够使用。

​	关于SpringMvc的执行流程，查看 DispatcherServlet 的 doService 方法就可大概了解SpringMvc的执行流程了。





参考文章：

https://blog.csdn.net/w372426096/article/details/78431547
