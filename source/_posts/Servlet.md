---
title: Servlet 浅析
date: 2016-12-11
tags: Servlet
categories: Java
---


在我们学习Servlet之前，有必要了解一下Web容器的工作模式
1. 我们所有的请求其实都是先到达了web容器，然后才分发给已经注册好的Servlet
2. 请求由Servlet的service方法调用`doGet()`和`doPost()`进行反应。

<!-- more -->

# JSP

JSP的本质是一个Servlet类，他弥补了Servlet不好做界面的劣势。

# Servlet 工作原理解析


# url匹配和url-pattern

**url的匹配**
当一个请求发送到servlet容器的时候，容器先会将请求的url减去当前应用上下文的路径作为servlet的映射url，比如我访问的是http://localhost/test/aaa.html，我的应用上下文是test，容器会将http://localhost/test去掉，剩下的/aaa.html部分拿来做servlet的映射匹配。这个映射匹配过程是有顺序的，而且当有一个servlet匹配成功以后，就不会去理会剩下的servlet了（filter不同，后文会提到）。其匹配规则和顺序如下：
1. 精确路径匹配。例子：比如servletA 的url-pattern为 /test，servletB的url-pattern为 /* ，这个时候，如果我访问的url为http://localhost/test ，这个时候容器就会先进行精确路径匹配，发现/test正好被servletA精确匹配，那么就去调用servletA，也不会去理会其他的servlet了。
2. 最长路径匹配。例子：servletA的url-pattern为/test/*，而servletB的url-pattern为/test/a/*，此时访问http://localhost/test/a时，容器会选择路径最长的servlet来匹配，也就是这里的servletB。
3. 扩展匹配，如果url最后一段包含扩展，容器将会根据扩展选择合适的servlet。例子：servletA的url-pattern：*.action
4. 如果前面三条规则都没有找到一个servlet，容器会根据url选择对应的请求资源。如果应用定义了一个default servlet，则容器会将请求丢给default servlet（什么是default servlet？后面会讲）。
根据这个规则表，就能很清楚的知道servlet的匹配过程，所以定义servlet的时候也要考虑url-pattern的写法，以免出错。

**对于filter，不会像servlet那样只匹配一个servlet，因为filter的集合是一个链，所以只会有处理的顺序不同，而不会出现只选择一个filter。**Filter的处理顺序和filter-mapping在web.xml中定义的顺序相同。

**url-pattern详解**
在web.xml文件中，以下语法用于定义映射：
1. 以”/’开头和以”/*”结尾的是用来做路径映射的。
2. 以前缀”*.”开头的是用来做扩展映射的。
3. “/” 是用来定义default servlet映射的。
4. 剩下的都是用来定义详细映射的。比如： /aa/bb/cc.action

所以，为什么定义”/*.action”这样一个看起来很正常的匹配会错？因为这个匹配即属于路径映射，也属于扩展映射，导致容器无法判断

# 生命周期

![Alt text](./1476700589878.png)

# Servlet装载的三种情况：

1. 自动装载：某些Servlet如果需要在Servlet容器启动时就加载，需要在web.xml下它的`<Servlet>`标签里中，添加优先级代码：
```
<Servlet>
<load-on-startup>1<load-on-startup>
</Servlet>
```
数字越小表示该servlet的优先级越高，会先于其他自动装载的优先级较低的先装载,**在SpringMVC中有这个配置**
``` xml
   <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>*.form</url-pattern>
    </servlet-mapping>
```
2. Servlet容器启动后，客户首次向某个Servlet发送请求时，Tomcat容器会加载它
3. 当Servlet类文件被更新后，也会重新自动加载
Servlet是长期驻留在内存里的。某个Servlet一旦被加载，就会长期存在于服务器的内存里，直到服务器关闭
Servlet被装载后，Servlet容器通过反射创建一个Servlet实例并且调用Servlet的init()方法进行初始化。在Servlet的整个生命周期内，init()方法只被调用一次


# Servlet得到JSP对象

![Alt text](./1476789507228.png)


# Servlet 3.0

为什么出现了Servlet3.0，一切的一切都是为了效率的提高，恰如，框架的出现。
[Servlet 3.0 新特性详解](https://www.ibm.com/developerworks/cn/java/j-lo-servlet30/)
