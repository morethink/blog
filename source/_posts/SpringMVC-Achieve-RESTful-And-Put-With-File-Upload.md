---
title: SpringMVC实现PUT请求上传文件
date: 2017-02-08
tags: 
categories: SpringMVC
---

在JQuery中，我们可以进行REST ful中delete和put的请求，但是在java EE标准中，默认只有在POST请求的时候，servlet 才会通过getparameter()方法取得请求体中的相应的请求参数的数据。而PUT，delete请求的请求体中数据则默认不会被解析。

<!-- more -->


1. 关于delete请求：delete请求用来从服务器上删除资源。因此我们只需要把要删除的资源的ID上传给服务器，即使是批量删除的时候，也可以通过URL传参的方式将多个id传给servlet，因此，可以满足我们的需求，可以直接发送请求。
2. 关于put请求(指的是带有请求体)
    1. 没有文件时：SpringMVC提供了一个将post转换为put和delete的方法，通过在web.xml中注册一个HiddenHttpMethodFilter过滤器。
    2. 上传文件时：我们可以通过在web.xml中注册一个MultipartFilter，**一定要在HiddenHttpMethodFilter之前**。



# SpringMVC实现PUT,DELETE请求


```XML
 <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:dispatcher-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    <filter>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <servlet-name>dispatcher</servlet-name>
    </filter-mapping>
```
然后我们看源码：
```java
@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		String paramValue = request.getParameter(this.methodParam);
		if ("POST".equals(request.getMethod()) && StringUtils.hasLength(paramValue)) {
			String method = paramValue.toUpperCase(Locale.ENGLISH);
			HttpServletRequest wrapper = new HttpMethodRequestWrapper(request, method);
			filterChain.doFilter(wrapper, response);
		}
		else {
			filterChain.doFilter(request, response);
		}
	}
```
1. this.methodParam属性被默认初始化为"`_method`"，通过`request.getParameter(this.methodParam);`判断是put还是delete，
2. `"POST".equals(request.getMethod())`，而且必须要求是post方式提交的，
3. 然后它把request进行包装后传给下一个filter。
4. 因此，我们需要在提交的时候添加一个字段

```html
<form action="" id="formData" name="formData" method="post">
    <input type="text" name="username" id="username"/>
    <input type="hidden" name="_method" value="delete"/>
    <input type="submit" value="submit"/>
</form>
```


或者在`$.ajax`中
```JavaScript
    function login() {
        $.ajax({
            type: "post",//请求方式
            url: "",  //发送请求地址
            timeout: 30000,//超时时间：30秒
            data: {
                "username": $('#username').val(),
                "password": $("#password").val(),
                "_method": delete
            },
            dataType: "json",//设置返回数据的格式
            success: function (data) {
                console.log(data);
            },
            error: function () { //请求出错的处理

            }
        });
    }
```

然后我们就可以在后台`@RequestMapping(value = "", method = RequestMethod.PUT)`注解中标识我们的方法，最后就可以成功地获得数据。


# SpringMVC实现PUT请求上传文件

可是后来我又有遇到另外一个需求那就是修改的时候需要传送文件到put方法中，于是这种方法就不可行了，但是我在`HiddenHttpMethodFilter`源码中看到这样一句话
```
 * <p><b>NOTE: This filter needs to run after multipart processing in case of a multipart
 * POST request, due to its inherent need for checking a POST body parameter.</b>
 * So typically, put a Spring {@link org.springframework.web.multipart.support.MultipartFilter}
 * <i>before</i> this HiddenHttpMethodFilter in your {@code web.xml} filter chain.
```
和MultipartFilter源码中这样的注释
```
	/**
	 * Set the bean name of the MultipartResolver to fetch from Spring's
	 * root application context. Default is "filterMultipartResolver".
	 */
```
1. 也就是说我们可以通过在web.xml中注册一个MultipartFilter，**一定要在HiddenHttpMethodFilter之前**。
```XML
<filter>
        <filter-name>MultipartFilter</filter-name>
        <filter-class>org.springframework.web.multipart.support.MultipartFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>MultipartFilter</filter-name>
        <servlet-name>dispatcher</servlet-name>
    </filter-mapping>

    <filter>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>
```
2. 然后再在Spring的 root application context中添加如下代码
```xml
    <bean id="filterMultipartResolver"
          class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="maxUploadSize" value="209715200"/>
        <property name="defaultEncoding" value="UTF-8"/>
        <property name="resolveLazily" value="true"/>
    </bean>
```

3. FormData对象是html5的一个对象，目前的一些主流的浏览器都已经兼容。FormData对象是html5的一个对象，目前的一些主流的浏览器都已经兼容。
```JavaScript
        function test() {
            var form = new FormData(document.getElementById("tf"));
            form.append("_method", 'put');
            $.ajax({
                url: url,
                type: 'post',
                data: form,
                processData: false,
                contentType: false,
                success: function (data) {
                    window.clearInterval(timer);
                    console.log("over..");
                },
                error: function (e) {
                    alert("错误！！");
                    window.clearInterval(timer);
                }
            });
            get();//此处为上传文件的进度条
        }
```

```html
<form id="tf" method="post" name="formDada" enctype="multipart/form-data">
    <input type="file" name="file"/>
    <input type="text" name="id"/>
    <input type="text" name="name"/>
    <input type="button" value="提" onclick="test()"/>
</form>
```


**最后，就可以实现将文件上传提交给put方法。**
