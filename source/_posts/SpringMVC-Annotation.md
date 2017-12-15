---
title: SpringMVC 参数绑定注解解析
date: 2017-12-12
tags:
categories: SpringMVC
---

本文介绍了用于参数绑定的相关注解。

绑定：将请求中的字段按照名字匹配的原则填入模型对象。

SpringMVC就跟Struts2一样，通过拦截器进行参数匹配。


代码在 https://github.com/morethink/MySpringMVC



# URI模板变量

这里指uri template中variable(路径变量)，不含queryString部分
<!-- more -->

## `@PathVariable`

当使用`@RequestMapping` URI template 样式映射时， 即 someUrl/{paramId}, 这时的paramId可通过 `@Pathvariable`注解绑定它传过来的值到方法的参数上。




示例代码：
```java
@RestController
@RequestMapping("/users")
public class UserAction {

    @GetMapping("/{id}")
    public Result getUser(@PathVariable int id) {
        return ResultUtil.successResult("123456");
    }

}
```

上面代码把URI template 中变量 ownerId的值和petId的值，绑定到方法的参数上。**若方法参数名称和需要绑定的uri template中变量名称不一致，需要在@PathVariable("name")指定uri template中的名称**。

# 请求头

## `@RequestHeader`

`@RequestHeader` 注解，可以把Request请求header部分的值绑定到方法的参数上。

示例代码：

这是一个Request 的header部分：

```
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding:gzip, deflate, br
Accept-Language:zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cache-Control:max-age=0
Connection:keep-alive
Host:localhost:8080
Upgrade-Insecure-Requests:1
User-Agent:Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.62 Safari/537.36
```

```java
@GetMapping("/getRequestHeader")
public Result getRequestHeader(@RequestHeader("Accept-Encoding") String encoding) {
    return ResultUtil.successResult(encoding);
}
```

上面的代码，把request header部分的 Accept-Encoding的值，绑定到参数encoding上。

## `@CookieValue `

可以把Request header中关于cookie的值绑定到方法的参数上。

例如有如下Cookie值：
`JSESSIONID=588DC770E582A3189B7E6210102EAE02`
参数绑定的代码：

```java
@RequestMapping("/getCookie")
public Result getCookie(@CookieValue("JSESSIONID") String cookie) {
    return ResultUtil.successResult(cookie);
}
```
即把JSESSIONID的值绑定到参数cookie上。

# 请求体

## `@RequestParam`

- 常用来处理简单类型的绑定，通过Request.getParameter() 获取的String可直接转换为简单类型的情况（ String--> 简单类型的转换操作由ConversionService配置的转换器来完成）；因为使用request.getParameter()方式获取参数，所以可以处理get 方式中queryString的值，也可以处理post方式中 body data的值；
- **用来处理Content-Type: 为 `application/x-www-form-urlencoded`编码的内容，提交方式GET、POST**；
- 该注解有两个属性： value、required； value用来指定要传入值的id名称，required用来指示参数是否必须绑定；


示例代码：

```java
@GetMapping("/tesRequestParam")
public Result tesRequestParam(@RequestParam("username") String username) {
    return ResultUtil.successResult(username);
}
```


## `@RequestBody`

该注解常用来处理Content-Type: 不是application/x-www-form-urlencoded编码的内容，例如application/json, application/xml等；

它是通过使用HandlerAdapter 配置的HttpMessageConverters来解析post data body，然后绑定到相应的bean上的。

因为配置有FormHttpMessageConverter，所以也可以用来处理 application/x-www-form-urlencoded的内容，处理完的结果放在一个MultiValueMap<String, String>里，这种情况在某些特殊需求下使用，详情查看FormHttpMessageConverter api;

示例代码：

```
@PostMapping("/tesRequestBody")
public Result tesRequestBody(@RequestBody User user) {

    return ResultUtil.successResult(user);
}
```

结果截图：

![](https://images.morethink.cn/f918dd11d4630c479a25548cca70acc2.png "Content-Type:application/json")

## `@RequestBody`通过list接收对象数组

在我们传递对象的时候，无论`Content-Type`是`x-www-form-urlencoded`还是`application/json`其实没有多大的关系，可是当我们需要传递对象数组的时候，表单编码就不行了，这时我们是可以采用json传递，然后后台使用`@RequestBody`注解，通过list接收来对象数组。

前端代码：

index.html
```javaScript
//打开页面时运行
$(document).ready(function () {
    var users = [];
    var user1 = {"username": "dd", "password": "123"};
    var user2 = {"username": "gg", "password": "123"};
    users.push(user1);
    users.push(user2);
    $.ajax({
        type: "POST",
        url: "users/saveUsers",
        timeout: 30000,
        dataType: "json",
        contentType: "application/json",
        data: JSON.stringify(users),
        success: function (data) {
            //将返回的数据展示成table
            showTable(data);
        },
        error: function () { //请求出错的处理
            $("#result").text("请求出错");
        }
    });
});
```

后台代码：

```java
@PostMapping("saveUsers")
public Result saveUsers(@RequestBody List<User> users) {
    return ResultUtil.successResult(users);
}
```

结果截图：

![](https://images.morethink.cn/2161af7bd2378ca110347dd73a4ecf94.png "表格展示数据")



# `@SessionAttribute`

该注解用来绑定HttpSession中的attribute对象的值，便于在方法中的参数里使用。该注解有value、types两个属性，可以通过名字和类型指定要使用的attribute 对象

示例代码：

```java
@PostMapping("/setSessionAttribute")
public Result setSessionAttribute(HttpSession session, String attribute) {
    session.setAttribute("attribute", attribute);
    return ResultUtil.SUCCESS_RESULT;
}

@GetMapping("/getSessionAttribute")
public Result getSessionAttribute(@SessionAttribute("attribute") String attribute) {
    return ResultUtil.successResult(attribute);
}
```

我们首先给session添加一个attribute，然后再取出这个attribute。

![](https://images.morethink.cn/03f2cfd9da78e5abd39254a7cc7c21d7.png "添加属性")

![](https://images.morethink.cn/6d17bbc4aeb10b48e4593ca5e82f3ca9.png "得到属性")

# `@ModelAttribute`

@ModelAttribute标注可被应用在方法或方法参数上。

## 方法使用@ModelAttribute标注

标注在方法上的`@ModelAttribute`说明方法是用于添加一个或多个属性到model上。这样的方法能接受与`@RequestMapping`标注相同的参数类型，只不过不能直接被映射到具体的请求上。

在同一个控制器中，标注了`@ModelAttribute`的方法实际上会在`@RequestMapping`方法之前被调用。

以下是示例：

```java
// Add one attribute
// The return value of the method is added to the model under the name "account"
// You can customize the name via @ModelAttribute("myAccount")

@ModelAttribute
public Account addAccount(@RequestParam String number) {
    return accountManager.findAccount(number);
}

// Add multiple attributes

@ModelAttribute
public void populateModel(@RequestParam String number, Model model) {
    model.addAttribute(accountManager.findAccount(number));
    // add more ...
}
```

**`@ModelAttribute`方法通常被用来填充一些公共需要的属性或数据**，比如一个下拉列表所预设的几种状态，或者宠物的几种类型，或者去取得一个HTML表单渲染所需要的命令对象，比如Account等。

@ModelAttribute标注方法有两种风格：
- 在第一种写法中，方法通过返回值的方式默认地将添加一个属性；
- 在第二种写法中，方法接收一个Model对象，然后可以向其中添加任意数量的属性。

可以在根据需要，在两种风格中选择合适的一种。

**一个控制器可以拥有多个`@ModelAttribute`方法。同个控制器内的所有这些方法，都会在`@RequestMapping`方法之前被调用。**

`@ModelAttribute`方法也可以定义在`@ControllerAdvice`标注的类中，并且这些`@ModelAttribute`可以同时对许多控制器生效。

属性名没有被显式指定的时候又当如何呢？在这种情况下，框架将根据属性的类型给予一个默认名称。举个例子，若方法返回一个Account类型的对象，则默认的属性名为"account"。可以通过设置@ModelAttribute标注的值来改变默认值。当向Model中直接添加属性时，请使用合适的重载方法addAttribute(..)-即带或不带属性名的方法。

@ModelAttribute标注也可以被用在`@RequestMapping`方法上。这种情况下，`@RequestMapping`方法的返回值将会被解释为model的一个属性，而非一个视图名，此时视图名将以视图命名约定来方式来确定。

## 方法参数使用`@ModelAttribute`标注

**标注在方法参数上的`@ModelAttribute`说明了该方法参数的值将由model中取得。如果model中找不到，那么该参数会先被实例化，然后被添加到model中。在model中存在以后，请求中所有名称匹配的参数都会填充到该参数中。**

这在Spring MVC中被称为数据绑定，一个非常有用的特性，我们不用每次都手动从表格数据中转换这些字段数据。

```java
@PostMapping
public Result saveUser(@ModelAttribute User user) {
    return ResultUtil.successResult(user);
}
```

以上面的代码为例，这个User类型的实例可能来自哪里呢？有几种可能:
- 它可能因为`@SessionAttributes`标注的使用已经存在于model中
- 它可能因为在同个控制器中使用了`@ModelAttribute`方法已经存在于model中，正如上一小节所叙述的
- 它可能是由URI模板变量和类型转换中取得的
- 它可能是调用了自身的默认构造器被实例化出来的


@ModelAttribute方法常用于从数据库中取一个属性值，该值可能通过`@SessionAttributes`标注在请求中间传递。在一些情况下，使用URI模板变量和类型转换的方式来取得一个属性是更方便的方式。




# 在不给定注解的情况下，参数是怎样绑定的？

通过分析`AnnotationMethodHandlerAdapter`和`RequestMappingHandlerAdapter`的源代码发现，方法的参数在不给定参数的情况下：
- 若要绑定的对象时简单类型：调用`@RequestParam`来处理的。
  这里的简单类型指Java的原始类型(boolean, int 等)、原始类型对象（Boolean, Int等）、String、Date等ConversionService里可以直接String转换成目标对象的类型。也就是说没有特别需求，不推荐使用`@RequestParam`。
- 若要绑定的对象时复杂类型：调用`@ModelAttribute`来处理的。也就是说如果不需要从model或者session中得到数据，@ModelAttribute可以不使用。


# `@RequestMapping`支持的方法参数

下面这些参数Spring在调用请求方法的时候会自动给它们赋值，所以当在请求方法中需要使用到这些对象的时候，可以直接在方法上给定一个方法参数的申明，然后在方法体里面直接用就可以了。

1. HttpServlet 对象，主要包括HttpServletRequest 、HttpServletResponse 和HttpSession 对象。 但是有一点需要注意的是在使用HttpSession 对象的时候，如果此时HttpSession 对象还没有建立起来的话就会有问题。
2. Spring 自己的WebRequest 对象。 使用该对象可以访问到存放在HttpServletRequest 和HttpSession 中的属性值。
3. InputStream 、OutputStream 、Reader 和Writer 。 InputStream 和Reader 是针对HttpServletRequest 而言的，可以从里面取数据；OutputStream 和Writer 是针对HttpServletResponse 而言的，可以往里面写数据。
4. 使用`@PathVariable` 、`@RequestParam` 、`@CookieValue` 和 `@RequestHeader` 标记的参数。
5. 使用`@ModelAttribute` 标记的参数。
6. java.util.Map 、Spring 封装的Model 和ModelMap 。 这些都可以用来封装模型数据，用来给视图做展示。
7. 实体类。 可以用来接收上传的参数。
8. Spring 封装的MultipartFile 。 用来接收上传文件的。
9. Spring 封装的Errors 和BindingResult 对象。 这两个对象参数必须紧接在需要验证的实体对象参数之后，它里面包含了实体对象的验证结果。

# 一个参数传多个值

在浏览器输入此URL`http://localhost:8080/admin/login.action?username=geek&password=geek&password=geek`

结果得到的对象为 ： `Manager{username='geek', password='geek,geek'}`




**参考文档**:
1. [@RequestParam @RequestBody @PathVariable 等参数绑定注解详解](http://blog.csdn.net/walkerjong/article/details/7946109)
2. [SpringMVC之Controller常用注解功能全解析](https://segmentfault.com/a/1190000005670764#articleHeader8)
3. [@ModelAttribute使用详解](http://wangwengcn.iteye.com/blog/1677024)
