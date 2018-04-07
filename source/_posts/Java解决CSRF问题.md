---
title: Java解决CSRF问题
date: 2018-01-04
tags: CSRF
categories: Web安全
---

项目地址： https://github.com/morethink/web-security-csrf

# CSRF是什么？

CSRF（Cross-site request forgery），中文名称：跨站请求伪造，也被称为：one click attack/session riding，缩写为：CSRF/XSRF。

# CSRF可以做什么？

你这可以这么理解CSRF攻击：攻击者盗用了你的身份，以你的名义发送恶意请求。CSRF能够做的事情包括：以你名义发送邮件，发消息，盗取你的账号，甚至于购买商品，虚拟货币转账......造成的问题包括：个人隐私泄露以及财产安全。

<!-- more -->

# CSRF的原理

下图简单阐述了CSRF攻击的思
![](https://images.morethink.cn/138ad4f05b47533bf46904dc165167cc.png)


从上图可以看出，要完成一次CSRF攻击，受害者必须依次完成两个步骤：
1. 登录受信任网站A，并在本地生成Cookie。
2. 在不登出A的情况下，访问危险网站B。

看到这里，你也许会说：“如果我不满足以上两个条件中的一个，我就不会受到CSRF的攻击”。是的，确实如此，但你不能保证以下情况不会发生：
1. 你不能保证你登录了一个网站后，不再打开一个tab页面并访问另外的网站。
2. 你不能保证你关闭浏览器了后，你本地的Cookie立刻过期，你上次的会话已经结束。（事实上，关闭浏览器不能结束一个会话，但大多数人都会错误的认为关闭浏览器就等于退出登录/结束会话了......）
3. 上图中所谓的攻击网站，可能是一个存在其他漏洞的可信任的经常被人访问的网站。

下面讲一讲java解决CSRF攻击的方式。

# 模拟CSRF攻击

## 登录A网站

用户名和密码都是admin。

`http://localhost:8081/login.html`:
![](https://images.morethink.cn/e298f8ef08869557b8fb60034f06bb80.png)

## 你有权限删除1号帖子

`http://localhost:8081/deletePost.html`:
![](https://images.morethink.cn/897d358f2677d053bb9555ff69d112ac.png)


## 登录有CSRF攻击A网站的B网站

`http://localhost:8082/deletePost.html`:

![](https://images.morethink.cn/csrf-attack.gif)

明显看到B网站是8082端口，A网站是8081端口，但是B网站的删除2号帖子功能依然实现。

# 预防CSRF攻击

简单来说，CSRF 就是网站 A 对用户建立信任关系后，在网站 B 上利用这种信任关系，跨站点向网站 A 发起一些伪造的用户操作请求，以达到攻击的目的。

而之所以可以完成攻击是因为B向A发起攻击的时候会把A网站的cookie带给A网站，也就是说cookie已经不安全了。

## 通过Synchronizer Tokens

Synchronizer Tokens： 在表单里隐藏一个随机变化的 csrf_token csrf_token 提交到后台进行验证，如果验证通过则可以继续执行操作。这种情况有效的主要原因是网站 B 拿不到网站 A 表单里的 csrf_token

这种方式的使用条件是PHP和JSP等。因为cookie已经不安全了，因此把csrf_token值存储在session中，然后每次表单提交时都从session取出来放到form表单的隐藏域中，这样B网站不可以得到这个存储到session中的值。

下面是JSP的：
```html
<input type="hidden" name="random_form" value=<%=random%>></input>
```

但是我现在的情况是html，不是JSP，并不能动态的从session中取出csrf_token值。只能采用加密的方式了。

## Hash加密cookie中csrf_token值

这可能是最简单的解决方案了，因为攻击者不能获得第三方的Cookie(理论上)，所以表单中的数据也就构造失败了。

我采用的hash加密方法是JS实现Java的HashCode方法，得到hash值，这个比较简单。也可以采用其他的hash算法。

前端向后台传递hash之后的csrf_token值和cookie中的csrf_token值，后台拿到cookie中的csrf_token值后得到hashCode值然后与前端传过来的值进行比较，一样则通过。

### 你有权限删除3号帖子
`http://localhost:8081/deletePost.html`

![](https://images.morethink.cn/2ac5eab98780646c6c36dcdc98fa50c7.png)

### B网站的他已经没有权限了

我们通过UserFilter.java给攻击者返回的是403错误，表示服务器理解用户客户端的请求但拒绝处理。

`http://localhost:8082/deletePost.html`:
![](https://images.morethink.cn/csrf-attack-fail-failure.gif)

攻击者不能删除4号帖子。

前端代码：

deletePost.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>deletePost</title>
    <script type="text/javascript" src="js/jquery.min.js"></script>
    <script type="text/javascript">
        function deletePost() {
            var url = '/post/' + document.getElementById("postId").value;
            var csrf_token = document.cookie.replace(/(?:(?:^|.*;\s*)csrf_token\s*\=\s*([^;]*).*$)|^.*$/, "$1");
            console.log('csrf_token=' + csrf_token);
            $.ajax({
                type: "post",//请求方式
                url: url,  //发送请求地址
                timeout: 30000,//超时时间：30秒
                data: {
                    "_method": "delete",
                    "csrf_token": hash(csrf_token) // 对csrf_token进行hash加密
                },
                dataType: "json",//设置返回数据的格式
                success: function (result) {
                    if (result.message == "success") {
//                    window.location.href = "index.html";
                        $("#result").text("删除成功");
                    } else {
                        $("#result").text("删除失败");
                    }
                },
                error: function () { //请求出错的处理
                    $("#result").text("请求出错");
                }
            });
        }

        // javascript的String到int(32位)的hash算法
        function hash(str) {
            var hash = 0;
            if (str.length == 0) return hash;
            for (i = 0; i < str.length; i++) {
                char = str.charCodeAt(i);
                hash = ((hash << 5) - hash) + char;
                hash = hash & hash; // Convert to 32bit integer
            }
            return hash;
        }
    </script>
</head>
<body>
<h3>删除帖子</h3>
帖子编号 ： <input type="text" id="postId"/>
<button onclick="deletePost();">deletePost</button>

<br/>
<br/>
<br/>
<div>
    <p id="result"></p>
</div>
</body>
</html>
```

后台代码：

UserInterceptor.java

```java
package cn.morethink.interceptor;

import cn.morethink.util.JsonUtil;
import cn.morethink.util.Result;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.PrintWriter;

/**
 * @author 李文浩
 * @date 2018/1/4
 */
public class UserInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String method = request.getMethod();
        System.out.println(method);
        if (method.equalsIgnoreCase("POST") || method.equalsIgnoreCase("DELETE")
                || method.equalsIgnoreCase("PUT")) {
            String csrf_token = request.getParameter("csrf_token");
            System.out.println(csrf_token + "1222222222222222222222222222222222222222222222");
            Cookie[] cookies = request.getCookies();
            if (cookies != null && cookies.length > 0 && csrf_token != null) {
                for (Cookie cookie : cookies) {
                    if (cookie.getName().equals("csrf_token")) {
                        if (Integer.valueOf(csrf_token) == cookie.getValue().hashCode()) {
                            return true;
                        }
                    }
                }
            }
        }
        Result result = new Result("403", "你还想攻击我??????????", "");
        PrintWriter out = response.getWriter();
        out.write(JsonUtil.toJson(result));
        out.close();
        return false;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}


```

**注意**：
1. cookie必须要设置PATH才可以生效，否则在下一次请求的时候无法带给服务器。
2. Spring Boot 出现启动找不到主类的问题时可以mvn clean一下。
3. Filter设置response.sendError(403)在Spring Boot没有效果。


**参考文档**：
1. [浅谈CSRF攻击方式](https://www.cnblogs.com/hyddd/archive/2009/04/09/1432744.html)
2. [jQueue 动态设置form表单的action属性的值和方法](http://blog.csdn.net/zzhongcy/article/details/20133883)
3. [javascript的String到int(32位)的hash算法](https://www.thinksaas.cn/group/topic/304242/)
