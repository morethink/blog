---
title: 常见web攻击总结
date: 2018-04-07
tags:
categories: Web安全
---

搞Web开发离不开安全这个话题，确保网站或者网页应用的安全性，是每个开发人员都应该了解的事。本篇主要简单介绍在Web领域几种常见的攻击手段及Java Web中的预防方式。

- [XSS](#XSS)
- [SQL注入](#SQL注入)
- [DDOS](#DDOS)
- [CSRF](#CSRF)

<!-- more -->

项目地址： https://github.com/morethink/web-security

# XSS

## 什么是XSS

XSS攻击：跨站脚本攻击(Cross-Site Scripting)，为了不和层叠样式表(Cascading Style Sheets, CSS)的缩写混淆，故将跨站脚本攻击缩写为XSS。XSS是一种常见的web安全漏洞，它允许攻击者将恶意代码植入到提供给其它用户使用的页面中。不同于大多数攻击(一般只涉及攻击者和受害者)，XSS涉及到三方，即攻击者、客户端与Web应用。XSS的攻击目标是为了盗取存储在客户端的cookie或者其他网站用于识别客户端身份的敏感信息。一旦获取到合法用户的信息后，攻击者甚至可以假冒合法用户与网站进行交互。

XSS通常可以分为两大类：
1. 存储型XSS，主要出现在让用户输入数据，供其他浏览此页的用户进行查看的地方，包括留言、评论、博客日志和各类表单等。应用程序从数据库中查询数据，在页面中显示出来，攻击者在相关页面输入恶意的脚本数据后，用户浏览此类页面时就可能受到攻击。这个流程简单可以描述为：`恶意用户的Html输入Web程序->进入数据库->Web程序->用户浏览器`。
2. 反射型XSS，主要做法是将脚本代码加入URL地址的请求参数里，请求参数进入程序后在页面直接输出，用户点击类似的恶意链接就可能受到攻击。


比如说我写了一个网站，然后攻击者在上面发布了一个文章，内容是这样的 `<script>alert(document.cookie)</script>`,如果我没有对他的内容进行处理，直接存储到数据库，那么下一次当其他用户访问他的这篇文章的时候，服务器从数据库读取后然后响应给客户端，浏览器执行了这段脚本，就会将cookie展现出来，这就是典型的存储型XSS。

如图：
![](https://images.morethink.cn/a1c6ebf6de227e086d0289f34d8c5f76.png)


## 如何预防XSS

答案很简单，坚决不要相信用户的任何输入，并过滤掉输入中的所有特殊字符。这样就能消灭绝大部分的XSS攻击。

目前防御XSS主要有如下几种方式：
1. 过滤特殊字符
避免XSS的方法之一主要是将用户所提供的内容进行过滤(如上面的`script`标签)。
2. 使用HTTP头指定类型
`w.Header().Set("Content-Type","text/javascript")`
这样就可以让浏览器解析javascript代码，而不会是html输出。

# SQL注入

## 什么是SQL注入

攻击者成功的向服务器提交恶意的SQL查询代码，程序在接收后错误的将攻击者的输入作为查询语句的一部分执行，导致原始的查询逻辑被改变，额外的执行了攻击者精心构造的恶意代码。

举例：`' OR '1'='1`

这是最常见的 SQL注入攻击，当我们输如用户名 admin ，然后密码输如`' OR '1'=1='1`的时候，我们在查询用户名和密码是否正确的时候，本来要执行的是`SELECT * FROM user WHERE username='' and password=''`,经过参数拼接后，会执行 SQL语句 `SELECT * FROM user WHERE username='' and password='' OR '1'='1'`，这个时候1=1是成立，自然就跳过验证了。
如下图所示：

![](https://images.morethink.cn/69855b1538333659f26afc281feb4e30.png)


但是如果再严重一点，密码输如的是`';DROP TABLE user;--`，那么 SQL命令为` SELECT *  FROM user WHERE username='admin' and password='';drop table user;--'` 这个时候我们就直接把这个表给删除了。

## 如何预防SQL注入

- 在Java中，我们可以使用预编译语句(PreparedStatement)，这样的话即使我们使用 SQL语句伪造成参数，到了服务端的时候，这个伪造 SQL语句的参数也只是简单的字符，并不能起到攻击的作用。
- 对进入数据库的特殊字符（`'"\尖括号&*`;等）进行转义处理，或编码转换。
- 在应用发布之前建议使用专业的SQL注入检测工具进行检测，以及时修补被发现的SQL注入漏洞。网上有很多这方面的开源工具，例如sqlmap、SQLninja等。
- 避免网站打印出SQL错误信息，比如类型错误、字段不匹配等，把代码里的SQL语句暴露出来，以防止攻击者利用这些错误信息进行SQL注入。

在上图展示中，使用了Java JDBC中的`PreparedStatement`预编译预防SQL注入，可以看到将所有输入都作为了字符串，避免执行恶意SQL。

# DDOS
## 什么是DDOS
DDOS：分布式拒绝服务攻击（Distributed Denial of Service），简单说就是发送大量请求是使服务器瘫痪。DDos攻击是在DOS攻击基础上的，可以通俗理解，dos是单挑，而ddos是群殴，因为现代技术的发展，dos攻击的杀伤力降低，所以出现了DDOS，攻击者借助公共网络，将大数量的计算机设备联合起来，向一个或多个目标进行攻击。

在技术角度上，DDoS攻击可以针对网络通讯协议的各层，手段大致有：TCP类的SYN Flood、ACK Flood，UDP类的Fraggle、Trinoo，DNS Query Flood，ICMP Flood，Slowloris类等等。一般会根据攻击目标的情况，针对性的把技术手法混合，以达到最低的成本最难防御的目的，并且可以进行合理的节奏控制，以及隐藏保护攻击资源。

下面介绍一下TCP协议中的SYN攻击。

## SYN攻击
在三次握手过程中，服务器发送 `SYN-ACK` 之后，收到客户端的 `ACK` 之前的 TCP 连接称为半连接(half-open connect)。此时服务器处于 `SYN_RCVD` 状态。当收到 ACK 后，服务器才能转入 `ESTABLISHED` 状态.

`SYN `攻击指的是，攻击客户端在短时间内伪造大量不存在的IP地址，向服务器不断地发送`SYN`包，服务器回复确认包，并等待客户的确认。由于源地址是不存在的，服务器需要不断的重发直至超时，这些伪造的`SYN`包将长时间占用未连接队列，正常的`SYN`请求被丢弃，导致目标系统运行缓慢，严重者会引起网络堵塞甚至系统瘫痪。

## 如何预防DDOS
阿里巴巴的安全团队在实战中发现，DDoS 防御产品的核心是检测技术和清洗技术。检测技术就是检测网站是否正在遭受 DDoS 攻击，而清洗技术就是清洗掉异常流量。而检测技术的核心在于对业务深刻的理解，才能快速精确判断出是否真的发生了 DDoS 攻击。清洗技术对检测来讲，不同的业务场景下要求的粒度不一样。

# CSRF
## 什么是CSRF

CSRF（Cross-site request forgery），中文名称：跨站请求伪造，也被称为：one click attack/session riding，缩写为：CSRF/XSRF。

你这可以这么理解CSRF攻击：攻击者盗用了你的身份，以你的名义发送恶意请求。CSRF能够做的事情包括：以你名义发送邮件，发消息，盗取你的账号，甚至于购买商品，虚拟货币转账......造成的问题包括：个人隐私泄露以及财产安全。

## CSRF的原理

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

## 模拟CSRF攻击

### 登录A网站

用户名和密码都是admin。

`http://localhost:8081/login.html`:
![](https://images.morethink.cn/e298f8ef08869557b8fb60034f06bb80.png)

### 你有权限删除1号帖子

`http://localhost:8081/deletePost.html`:
![](https://images.morethink.cn/897d358f2677d053bb9555ff69d112ac.png)


### 登录有CSRF攻击A网站的B网站

`http://localhost:8082/deletePost.html`:

![](https://images.morethink.cn/csrf-attack.gif)

明显看到B网站是8082端口，A网站是8081端口，但是B网站的删除2号帖子功能依然实现。

## 如何预防CSRF攻击

简单来说，CSRF 就是网站 A 对用户建立信任关系后，在网站 B 上利用这种信任关系，跨站点向网站 A 发起一些伪造的用户操作请求，以达到攻击的目的。

而之所以可以完成攻击是因为B向A发起攻击的时候会把A网站的cookie带给A网站，也就是说cookie已经不安全了。

### 通过Synchronizer Tokens

Synchronizer Tokens： 在表单里隐藏一个随机变化的 csrf_token csrf_token 提交到后台进行验证，如果验证通过则可以继续执行操作。这种情况有效的主要原因是网站 B 拿不到网站 A 表单里的 csrf_token

这种方式的使用条件是PHP和JSP等。因为cookie已经不安全了，因此把csrf_token值存储在session中，然后每次表单提交时都从session取出来放到form表单的隐藏域中，这样B网站不可以得到这个存储到session中的值。

下面是JSP的：
```html
<input type="hidden" name="random_form" value=<%=random%>></input>
```

但是我现在的情况是html，不是JSP，并不能动态的从session中取出csrf_token值。只能采用加密的方式了。

### Hash加密cookie中csrf_token值

这可能是最简单的解决方案了，因为攻击者不能获得第三方的Cookie(理论上)，所以表单中的数据也就构造失败了。

我采用的hash加密方法是JS实现Java的HashCode方法，得到hash值，这个比较简单。也可以采用其他的hash算法。

前端向后台传递hash之后的csrf_token值和cookie中的csrf_token值，后台拿到cookie中的csrf_token值后得到hashCode值然后与前端传过来的值进行比较，一样则通过。

#### 你有权限删除3号帖子
`http://localhost:8081/deletePost.html`

![](https://images.morethink.cn/2ac5eab98780646c6c36dcdc98fa50c7.png)

#### B网站的他已经没有权限了

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


# 总结

上面一共提到了4种攻击方式，分别是XSS攻击（关键是脚本，利用恶意脚本发起攻击），SQL注入（关键是通过用SQL语句伪造参数发出攻击），DDOS攻击（关键是发出大量请求，最后令服务器崩溃），CSRF攻击（关键是借助本地cookie进行认证，伪造发送请求）。


**参考文档**：
1. [XSS实战：我是如何拿下你的百度账号](https://zhuanlan.zhihu.com/p/24249045)
2. [总结几种常见web攻击手段及其防御方式](http://www.cnblogs.com/-new/p/7135814.html)
3. [浅谈CSRF攻击方式](https://www.cnblogs.com/hyddd/archive/2009/04/09/1432744.html)
4. [jQueue 动态设置form表单的action属性的值和方法](http://blog.csdn.net/zzhongcy/article/details/20133883)
5. [javascript的String到int(32位)的hash算法](https://www.thinksaas.cn/group/topic/304242/)
