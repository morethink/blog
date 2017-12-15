---
title: SpringMVC 知识整理
date: 2017-12-12
tags:
categories: SpringMVC
---
# SpringMVC架构设计

**MVC是一种架构模式**，它把业务的实现和展示相分离。

![](https://images.morethink.cn/ddbf6d71cefe533f850b5466b583cfc0.png "SpringMVC架构")

<!-- more -->

# SpringMVC与struts2的区别

1. **Struts2是类级别的拦截**， 一个类对应一个request上下文，**SpringMVC是方法级别的拦截**，一个方法对应一个request上下文，而方法同时又跟一个url对应,所以说从架构本身上SpringMVC就容易实现restful url,而struts2的架构实现起来要费劲，因为Struts2中Action的一个方法可以对应一个url，而其类属性却被所有方法共享，这也就无法用注解或其他方式标识其所属方法了。
2. **springmvc可以进行单例开发**，并且建议使用单例开发，struts2通过类的成员变量接收参数，无法使用单例，只能使用多例。
3. 由于Struts2需要针对每个request进行封装，把request，session等servlet生命周期的变量封装成一个一个Map，供给每个Action使用，并保证线程安全，所以在原则上，是比较耗费内存的。
4. **拦截器实现机制上**，Struts2有以自己的interceptor机制，SpringMVC用的是独立的AOP方式，这样导致Struts2的配置文件量还是比SpringMVC大。
5. servlet和filter的区别了。
6. SpringMVC集成了Ajax，使用非常方便，只需一个注解`@ResponseBody`就可以实现，然后直接返回响应文本即可，而Struts2拦截器集成了Ajax，在Action中处理时一般必须安装插件或者自己写代码集成进去，使用起来也相对不方便。
7. SpringMVC验证支持JSR303，处理起来相对更加灵活方便，而Struts2验证比较繁琐，感觉太烦乱。
8. spring MVC和Spring是无缝的。从这个项目的管理和安全上也比Struts2高（当然Struts2也可以通过不同的目录结构和相关配置做到SpringMVC一样的效果，但是需要xml配置的地方不少）。
9. 设计思想上，Struts2更加符合OOP的编程思想， SpringMVC就比较谨慎，在servlet上扩展。
10. SpringMVC开发效率和性能高于Struts2。
11. SpringMVC可以认为已经100%零配置。


# SpringAOP整合SpringMVC

spring容器不注册controller层组件，controller组件由springMVC容器单独注册。

```xml
// applicationContext.xml
<context:component-scan base-package="com.shuyun.channel">
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller" />
    <context:exclude-filter type="annotation" expression="org.springframework.web.bind.annotation.RestController" />
    <context:exclude-filter type="annotation" expression="org.springframework.web.bind.annotation.ControllerAdvice" />
</context:component-scan>

// springmvc-servlet.xml
<context:component-scan base-package="com.shuyun.channel" use-default-filters="false">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller" />
    <context:include-filter type="annotation" expression="org.springframework.web.bind.annotation.RestController" />
    <context:include-filter type="annotation" expression="org.springframework.web.bind.annotation.ControllerAdvice" />
</context:component-scan>
```
**注意**：

关于`<context:annotation-config /> `和 `<context:component-scan />`:

**component-scan会自动加上annotation-config功能，有了component-scan不用再写annotation-config了**。参见spring官方reference

# 配置层次化Spring容器


参考 [配置层次化Spring容器](http://blog.csdn.net/skyhitnow/article/details/43731493)

我们知道，在开发基于spring的Web应用时，通常使用两个IoC容器，一个是由DispatchServlet初始化的WebApplicationContext,一个是由ContextLoaderListener初始化的ApplicationContext。对于Spring容器，Spring的官方参考手册详细地讲解了依赖注入的配置方式，对于容器本身的配置和多个容器之间的关系却不曾提及。于是，很多人以为在一个应用中只有一个全局的Spring容器，或者不了解MVC使用的WebApplicationContext和根ApplicationContext的关系。

通过查看Spring的源码和API,发现Spring可以配置为多个容器，容器之间可以配置为层级关系，一个根容器可以配置许多子容器，子容器还可以配置子容器，从而形成一个单根的层次化结构。对于该容器结构中的每个容器，在其中查找特定的bean时，会首先在本容器内查找，如果找到对应的bean,就返回该bean；如果没有找到，就会从直接父容器中去查找，依此类推，直到根容器为止。

从上面的示例中可以看到，从子容器中可以取得父容器中配置的bean，而父容器中不能够取得子容器中的bean。

在Spring MVC中WebApplicationContext配置为根ApplicationContext的子容器，所以，MVC使用的容器中能够取得根ApplicationContext中的bean。**在一个web程序中可以配置多个DispatchSerlvet，每个Servlet对应一个容器**，所有这些容器都作为根容器的子容器，这样，我们就可以把通用的bean放在根容器中，而针对特定DispatchServlet的bean，可以放在各自的子容器中。

# 文件上传

- Controller的方法中需要接受一个Spring MVC提供的MultipartFile接口作为方法的参数，该参数接收前台表单type为file提交的对象，使用`@RequestParam`注解指明参数，那么Spring就会自动将表单传递过来的对象的类型转换为MultipartFile类型。

- MultipartFile中提供了getName()、getSize()、getByte()
getContentType()、isEmpty()、getInputStream()、getOriginalFilename()方法来访问文件。getOriginalFilename()方法是获取最初文件名，即本地文件名。
- 在Controller方法中使用FileUtils下的copyInputStreamToFile(InputStream in,File file)方法来完成文件的拷贝.第一个参数是文件拷贝源的输入流,直接使用MultipartFile下的getInputStream()方法.第二个参数是文件将要保存的位置.


```java
@RequestMapping("/doUpload")
public Result doUpload(@RequestParam("file") MultipartFile file) throws IOException {
    if (!file.isEmpty()) {
        FileUtils.copyInputStreamToFile(file.getInputStream(), new File("E://", file.getOriginalFilename()));
    }
    return ResultUtil.SUCCESS_RESULT;
}
```

# Jackson

- [How to enable pretty print JSON output (Jackson)](https://www.mkyong.com/java/how-to-enable-pretty-print-json-output-jackson/)
- [Jackson 2 – Convert Java Object to / from JSON](https://www.mkyong.com/java/jackson-2-convert-java-object-to-from-json/)
- [SpringMVC关于json、xml自动转换的原理研究(附带源码分析)](http://www.cnblogs.com/fangjian0423/p/springMVC-xml-json-convert.html)

# SpringMVC拦截器

拦截器好比你要去取经，那么，你就必须经过九九八十一关，主要用来解决请求的共性问题，如：乱码问题、权限验证问题等

实现SpringMVC拦截器的三个步骤
1. 创建一个实现HandlerInterceptor接口，并实现接口的方法的类
2. 将创建的拦截器注册到SpringMVC的配置文件中实现注册
    ```xml
    <mvc：interceptors>
    <bean class="路径下的类">
    </mvc：interceptors>
    ```
3. 配置拦截器的拦截规则：
    ```xml
    <mvc：interceptors>
       <mvc：interceptor>
              <mvc:mapping path="拦截的action">
              <bean class="路径下的类">
       </mvc：interceptor>
    </mvc：interceptors>
    ```

拦截器中三个方法的介绍：
1. preHandle()方法是否将当前请求拦截下来。（返回true请求继续运行，返回false请求终止（包括action层也会终止），Object arg代表被拦截的目标对象。）
2. postHandle()方法的ModelAndView对象可以改变发往的视图或修改发往视图的信息。
3. afterCompletion()方法表示视图显示之后在执行该方法。（一般用于资源的销毁）


## 拦截器和过滤器

共同：他们都是用来检查程序的共同场景，只不过拦截器是面向Action的，过滤器是面向整个web应用的。
1. 解决权限验证问题
2. 解决乱码问题

拦截器和过滤器的区别：
1. 拦截器是基于java的反射机制的，而过滤器是基于函数回调。
2. 拦截器不依赖与servlet容器，过滤器依赖与servlet容器。
3. 拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用。
4. 拦截器可以访问action上下文、**值栈** 里的对象，而过滤器不能访问。
5. 在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次。
6. **拦截器可以获取IOC容器中的各个bean，而过滤器就不行，这点很重要，在拦截器里注入一个service，可以调用业务逻辑**。

## 拦截器方法的作用顺序

![](https://images.morethink.cn/a23b0321f130d0b064094c2b9e1e15a6.png "拦截器方法的作用顺序")

拦截器的其它实现方式：
1. 拦截器的类还可以通过实现WebRequestInterceptor（HandlerInterceptor）接口来编写。
2. 向SpringMVC框架注册的写法不变。
3. 弊端：preHandler方法没有返回值，不能终止请求。


Ps：建议使用功能更强大的实现方式，实现HandlerInterceptor接口。


# Spring4增加功能

Spring4主要在Web服务方面有下面两个方面提升：
1. 控制器使用`@ResponseBody`和 `@RestController`。
2. 异步调用。


# Spring整合Struts2

Spring默认是单例，Struts2默认是多实例的。

如果是spring配置文件中的 bean的名字的话就是spring创建，那么单实例还是多实例就由spring的action Bean中的业务逻辑控制器类是否配置为scope=”prototype”，有就是多实例的，没有就是单实例的，顺序是先从spring中找，找不到再从struts配置文件中找。

1. 对于无Spring插件（Struts2-spring-plugin-XXX.jar）的整合方式，需要在spring的action Bean中加业务逻辑控制器类配scope="prototype"。
    ```xml
    <bean id="user" class="modle.User" scope="prototype"/>
    ```
2. 对于有Spring插件（Struts2-spring-plugin-XXX.jar）的整合方式：反编译StrutsSpringObjectFactory以及相关的代码才发现，如果在struts action的配置文件 `<action name=".." class=".."/>` 中class写的如果是完整的包名和类名的话就是struts创建action对象，也就是多实例的；

**参考文档**：

1. [史上最全最强SpringMVC详细示例实战教程](http://www.cnblogs.com/sunniest/p/4555801.html)
2. [Spring MVC快速入门](https://www.tianmaying.com/tutorial/spring-mvc-quickstart)
3. [Web MVC framework - Part VI. The Web](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html)
4. [Spring 注解学习手札（七） 补遗——@ResponseBody，@RequestBody，@PathVariable](http://snowolf.iteye.com/blog/1628861)
5. [SpringMVC4.1之Controller层最佳实践](https://github.com/kuitos/kuitos.github.io/issues/9)
