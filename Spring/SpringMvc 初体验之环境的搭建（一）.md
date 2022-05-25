# SpringMvc 初体验

> SpringMvc的接入虽然挺简单的吧，但经常没弄挺容易忘记是咋配置的；有些关键的信息还是得记一下，以防忘记。当然直接接入SpringBoot会更简单些。

接入Springmvc 主要包括三个步骤：

1. 在web.xml中配置DispatcherServlet；
* 创建SpringMvc的xml配置文件；
* 编写Controller 和 view视图；



可以采用xml配置和Java配置来构建SpringMvc的环境，也可以混合使用。

## Xml 配置



### 1.  新建Java Web 项目，配置web.xml 文件



在web项目中的web.xml配置 DispatcherServlet，该Servlet 是SpringMvc的入口；理解为一个Servlet的转发类吧

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <servlet>
        <servlet-name>demo</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 配置servlet的属性 -->
        <init-param>
            <!-- 如果不指定contextConfigLocation，默认会去WEB-INF/[servlet-name]-servlet.xml 读取mvc配置  -->
            <param-name>contextConfigLocation</param-name>
            <!-- 这里指定路径为 springmvc.xml下 -->
            <param-value>classpath:/springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>demo</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    
       <!-- 引入bean的配置文件 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:application-context.xml</param-value>
    </context-param>

    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>

</web-app>
```





### 2. 创建SpringMvc的xml配置文件；

> 前面在web.xml 中指定了springmvc配置文件的位置，在该位置中新建xml文件，里面配置了springmvc常用的配置属性；



需要注意的是，在beans 标签内需配置好相应组件的声明（项目有用到哪个就需要配置哪个），如下面声明的mvc 和 context 的配置声明。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
>

    <!-- mvc注解驱动-->
    <mvc:annotation-driven/>
    <!-- context bean的扫描路径-->
    <context:component-scan base-package="controller"/>

</beans>
```



​	`xmlns:xxx` 配置 可用于xml编写的提示，可利用idea提示出具体的配置是什么；

​	`xsi:schemaLocation` 配置用于运行时mvc找对应配置的声明，这里使用了mvc的配置，如果没有声明mvc的xsd配置的话，在运行时会报 "通配符的匹配很全面, 但无法找到元素 'mvc:annotation-driven' 的声明"。



### 3. 编写Controller 和 view视图；

配置好web.xml 及 springmvc的配置后，编写Controller 控制器 及 视图。

* IndexController

```java
package controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller
public class IndexController {

    @RequestMapping(value = "/hello", method = RequestMethod.HEAD)
    public String hello() {
        return "hello";
    }

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String index() {
        return "index.jsp";
    }

}
```



### 4. 部署到Tomcat中运行



* 检查路径是否有配置正确，如web.xml的路径，资源路径等；

  [![4sBSaV.png](https://z3.ax1x.com/2021/09/25/4sBSaV.png)](https://imgtu.com/i/4sBSaV)

* 配置tomcat 

[![4s1Uc8.png](https://z3.ax1x.com/2021/09/25/4s1Uc8.png)](https://imgtu.com/i/4s1Uc8)

* 在Project Structure ，检查是否有Artifacts 应用，没有的话add一下

[![4s12cT.png](https://z3.ax1x.com/2021/09/25/4s12cT.png)](https://imgtu.com/i/4s12cT)

* 在tomcat配置的Deployment 配置这个Artifacts 应用；

[![4s3SUA.png](https://z3.ax1x.com/2021/09/25/4s3SUA.png)](https://imgtu.com/i/4s3SUA)



在这个界面上检查下这个应用的路径是否没问题，这里只有一个web应用，那么指向根目录即可。

[![4s3EDg.png](https://z3.ax1x.com/2021/09/25/4s3EDg.png)](https://imgtu.com/i/4s3EDg)



启动检查就ok了。



## Java 配置



### 1. 配置 WebConfig，代替web.xml

​	实现 AbstractAnnotationConfigDispatcherServletInitializer 抽象类，从名字上可以看出这个抽象类的作用是前端控制器，也即 DispatcherServlet 的注解配置初始化类。即可以在该类中引入SpringMvc Java的配置。

```java
/**
 * 代替 web.xml配置的 Java配置
 */
public class WebConfig extends AbstractAnnotationConfigDispatcherServletInitializer {

    /**
     * bean配置的配置类
     * @return
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        System.out.println("getRootConfigClasses 。。。。。");
        return new Class<?>[] {ApplicationContextConfig.class};
    }

    /**
     * DispatcherServlet 的配置类，如本应用的 springmvc.xml
     * @return
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] {WebMvcConfig.class};
    }

    /**
     * servlet的映射路径
     * @return
     */
    @Override
    protected String[] getServletMappings() {
        return new String[] {"/"}; // 映射根目录以下的路径，即全路径
    }
}
```



### 2. Mvc 的配置，代替spring-mvc.xml

​	在Spring 5.0 前，Mvc的Java 配置是通过继承 WebMvcConfigurerAdapter 抽象类实现的。

​	在Spring 5.0后，Mvc采用了Java8编写，可以直接实现抽象类WebMvcConfigurerAdapter 的父类 WebMvcConfigurer 接口就行，因为Java8后接口是可以有默认方法的，也就是default关键字的描述。

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
        resolver.setViewClass(org.springframework.web.servlet.view.JstlView.class);
        resolver.setPrefix("/WEB-INF/page/");
        resolver.setSuffix(".jsp");
        resolver.setExposeContextBeansAsAttributes(true);
        return resolver;
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
}
```



### 3. 配置bean扫描的配置类

配置扫描bean的包路径，这里使用了excludeFilters属性去排除 EnableWebMvc 这个注解，表示不需要将有这个注解的类扫描成bean。

```java
@ComponentScan(basePackages = {"com.example"},excludeFilters = {@ComponentScan.Filter (type = FilterType.ANNOTATION,value = EnableWebMvc.class)})
@Configuration
public class ApplicationContextConfig {
}

```



剩下步骤跟 xml 的是一样的，只是这里将xml的配置改为用Java配置了。

