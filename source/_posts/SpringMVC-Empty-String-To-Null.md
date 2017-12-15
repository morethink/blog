---
title: SpringMVC空字符串转为null
date: 2017-02-23
tags: 
categories: SpringMVC
---

# 空字符串转为null

现在我遇到这样一个需求，那就是我想要吧前端传过来的值变为空，因为所谓前端的校验，其实都不是校验，如果前端传给后台一个表单，可是表单未填入值，我们后台进行判断的时候 **既需要判断null，同时需要判断是否为`""`, 并且如果你不希望数据库插入的是空字符串，而是`null`**，那么转换和插入的就很麻烦

<!-- more -->

```java
if (manager.getUsername().equals("") || manager.getUsername() == null) {
            throw new ErrorException("用户名未填写");
        }
        if (manager.getPassword().equals("") || manager.getPassword() == null) {
            throw new ErrorException("密码未填写");
        }
```

可是如果你在SpringMVC进行参数赋值处理之前就能把`""`转换为`null`，那么你就只需要进行如下判断，并且插入数据库的一直是空值

```java
if (manager.getUsername() == null) {
            throw new ErrorException("用户名未填写");
        }
        if (manager.getPassword() == null) {
            throw new ErrorException("密码未填写");
        }
```
# 实现方式
1. 可以使用过滤器/拦截器把空字符串设置成null(未尝试)
2. 注册一个SpringMVC `HandlerAdapter`来进行处理


使用SpringMVC时，所有的请求都是最先经过DispatcherServlet的，然后由DispatcherServlet选择合适的HandlerMapping和HandlerAdapter来处理请求，HandlerMapping的作用就是找到请求所对应的方法，而HandlerAdapter则来处理和请求相关的的各种事情。我们这里要讲的请求参数绑定也是HandlerAdapter来做的。





# 参数处理器

我们需要写一个自定义的请求参数处理器，然后把这个处理器放到HandlerAdapter中，这样我们的处理器就可以被拿来处理请求了。

```java
public class EmptyStringToNullModelAttributeMethodProcessor extends ServletModelAttributeMethodProcessor implements ApplicationContextAware {

    ApplicationContext applicationContext;

    public EmptyStringToNullModelAttributeMethodProcessor(boolean annotationNotRequired) {
        super(annotationNotRequired);
    }

    @Override
    protected void bindRequestParameters(WebDataBinder binder, NativeWebRequest request) {
        EmptyStringToNullRequestDataBinder toNullRequestDataBinderBinder = new EmptyStringToNullRequestDataBinder(binder.getTarget(), binder.getObjectName());
        RequestMappingHandlerAdapter requestMappingHandlerAdapter = applicationContext.getBean(RequestMappingHandlerAdapter.class);
        requestMappingHandlerAdapter.getWebBindingInitializer().initBinder(toNullRequestDataBinderBinder, request);
        toNullRequestDataBinderBinder.bind(request.getNativeRequest(ServletRequest.class));
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}

```


![enter image description here](https://images.morethink.cn/EmptyStringToNull.jpg)



# DataBinder

继承自`ExtendedServletRequestDataBinder`，主要用来自定义数据绑定处理

```java
public class EmptyStringToNullRequestDataBinder extends ExtendedServletRequestDataBinder {
    public EmptyStringToNullRequestDataBinder(Object target, String objectName) {
        super(target, objectName);
    }

    protected void addBindValues(MutablePropertyValues mpvs, ServletRequest request) {
        super.addBindValues(mpvs, request);
        for (PropertyValue propertyValue : mpvs.getPropertyValueList()) {
            if (propertyValue.getValue().equals(""))
                propertyValue.setConvertedValue(null);
        }
    }
}
```

# 注册器

> 话要从HandlerAdapter里系统自带的处理器说起。我这边系统默认带了24个处理器，其中有两个ServletModelAttributeMethodProcessor，也就是我们自定义处理器继承的系统处理器。SpringMVC处理请求参数是轮询每一个处理器，看是否支持，也就是supportsParameter方法， 如果返回true，就交给你出来，并不会问下面的处理器。这就导致了如果我们简单的把我们的自定义处理器加到HandlerAdapter的Resolver列中是不行的，需要加到第一个去。

```java
public class EmptyStringToNullProcessorRegistry implements ApplicationContextAware, BeanFactoryPostProcessor {

    private ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {

        RequestMappingHandlerAdapter requestMappingHandlerAdapter = applicationContext.getBean(RequestMappingHandlerAdapter.class);

        List<HandlerMethodArgumentResolver> resolvers = requestMappingHandlerAdapter.getArgumentResolvers();


        List<HandlerMethodArgumentResolver> newResolvers = new ArrayList<HandlerMethodArgumentResolver>();

        for (HandlerMethodArgumentResolver resolver : resolvers) {
            newResolvers.add(resolver);
        }
        EmptyStringToNullModelAttributeMethodProcessor processor = new EmptyStringToNullModelAttributeMethodProcessor(true);
        processor.setApplicationContext(applicationContext);
        newResolvers.add(0, processor);
        requestMappingHandlerAdapter.setArgumentResolvers(Collections.unmodifiableList(newResolvers));
    }
}
```

# XML配置

```xml
    <mvc:annotation-driven>
        <mvc:argument-resolvers>
            <bean class="studio.geek.databind.EmptyStringToNullModelAttributeMethodProcessor">
                <constructor-arg name="annotationNotRequired" value="true"/>
            </bean>
        </mvc:argument-resolvers>
    </mvc:annotation-driven>
```

**最后就可以完成将空字符串转为`null`**


参考文档

1. [ SpringMVC对象绑定时自定义名称对应关系](http://blog.csdn.net/zgzczzw/article/details/53912966)
2. [SpringMVC源码分析和一些常用最佳实践](http://neoremind.com/2016/02/springmvc%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B8%B8%E7%94%A8%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5/)
