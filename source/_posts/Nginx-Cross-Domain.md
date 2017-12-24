---
title: 前端通过Nginx反向代理解决跨域问题
date: 2017-03-27
tags: [Nginx,跨域,反向代理]
categories: 服务器
---

在前面写的一篇文章[SpringMVC 跨域](https://www.morethink.cn/2017/03/09/SpringMVC-Cross-Domain/)，我们探讨了什么是跨域问题以及SpringMVC怎么解决跨域问题，解决方式主要有如下三种方式:
1. JSONP
2. CORS
3. WebSocket

可是这几种方式都是基于服务器配置的，即对于自己的网站是可以通过这几种方式解决的，可是现在遇到另一个需求(前面提到过，写扇贝插件，我们不能更改扇贝的服务器配置，也不能发短信叫他们给我配置一下)。

本文探讨了前端如何通过Nginx反向代理的方式解决跨域问题。

<!-- more -->

# 跨域

再次重申： **跨域是浏览器行为，不是服务器行为。**

实际上，请求已经到达服务器了，只不过在回来的时候被浏览器限制了。就像Python他可以进行抓取数据一样，不经过浏览器而发起请求是可以得到数据，想到通过Nginx的反向代理来解决跨域问题。

# 代理

所谓代理就是在我们和真实的服务器之间有一台代理服务器，我们所有的请求都是通过它来进行转接的。

## 正向代理

正向代理就是我们访问不了Google，但是我在国外有一台vps，它可以访问Google，我访问它，叫它访问Google后，把数据传给我。

**正向代理隐藏了真实的客户端**。

## 反向代理

> 大家都有过这样的经历，拨打10086客服电话，可能一个地区的10086客服有几个或者几十个，你永远都不需要关心在电话那头的是哪一个，叫什么，男的，还是女的，漂亮的还是帅气的，你都不关心，你关心的是你的问题能不能得到专业的解答，你只需要拨通了10086的总机号码，电话那头总会有人会回答你，只是有时慢有时快而已。那么这里的10086总机号码就是我们说的反向代理。客户不知道真正提供服务人的是谁。
>
>
> 反向代理隐藏了真实的服务端，当我们请求 www.baidu.com 的时候，就像拨打10086一样，背后可能有成千上万台服务器为我们服务，但具体是哪一台，你不知道，也不需要知道，你只需要知道反向代理服务器是谁就好了，www.baidu.com 就是我们的反向代理服务器，反向代理服务器会帮我们把请求转发到真实的服务器那里去。Nginx就是性能非常好的反向代理服务器，用来做负载均衡。


![](https://images.morethink.cn/reverse-proxy.jpg "反向代理")

**反向代理隐藏了真实的服务器**。

Nginx 就是一个很好的反向代理服务器，当然apache也可以实现此功能。

windows下Apache配置参考这篇文章：  [Windows Apache服务器配置](https://www.morethink.cn/2017/09/19/Windows-apache-Server/)

# Nginx

Nginx（发音同engine x）是一个 Web服务器，也可以用作反向代理，负载平衡器和 HTTP缓存。该软件由 Igor Sysoev 创建，并于2004年首次公开发布。同名公司成立于2011年，以提供支持。

我在[Windows下实现Nginx负载均衡](https://www.morethink.cn/2017/02/11/Nginx-Load-Balancing-under-Windows/)提到过Windows下Nginx命令使用。

## Nginx 反向代理模块 proxy_pass

`proxy_pass` 后面跟着一个 URL，用来将请求反向代理到 URL 参数指定的服务器上。例如我们上面例子中的 `proxy_pass https://api.shanbay.com`，则将匹配的请求反向代理到 https://api.shanbay.com。


通过在配置文件中增加`proxy_pass 你的服务器ip`,例如这里的扇贝服务器地址，就可以完成反向代理。


```
server {
        listen       80;
        server_name  localhost;
        ## 用户访问 localhost，则反向代理到https://api.shanbay.com
        location / {
            root   html;
            index  index.html index.htm;
            proxy_pass https://api.shanbay.com;
        }
}
```

# 配置html以文件方式打开

一般的情况下，我们的HTML文件时放置在Nginx服务器上面的，即通过输入 http://localhost/index.html ，但是在前端进行调试的时候，我们可能是通过 使用 file:///E:/nginx/html/index.html 来打开HTML。服务器打开不是特别方便。

而我们之所以要部署在服务器上，是想要使用浏览器自带的CORS头来解决跨域问题，如果不想把HTML放置在Nginx中，而想通过本地打开的方式来调试HTML，可以通过自己添加`Access-Control-Allow-Origin`等http头，但是我们的AJAX请求一定要加上`http://127.0.0.1/request`，而不能直接是 `/request`，于是将nginx.conf作如下配置：

```
location / {
    root   html;
    index  index.html index.htm;
    # 配置html以文件方式打开
    if ($request_method = 'POST') {
          add_header 'Access-Control-Allow-Origin' *;
          add_header 'Access-Control-Allow-Credentials' 'true';
          add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
          add_header 'Access-Control-Allow-Headers' 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
      }
    if ($request_method = 'GET') {
          add_header 'Access-Control-Allow-Origin' *;
          add_header 'Access-Control-Allow-Credentials' 'true';
          add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
          add_header 'Access-Control-Allow-Headers' 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
    }
    # 代理到8080端口
    proxy_pass        http://127.0.0.1:8080;

}
```


# 处理DELETE和PUT跨域请求

而现在我的后台是restful风格的接口，采用了delete和put方法，而上面的配置就无能为力了。

可以通过增加对非简单请求的判断来解决DELETE和PUT跨域请求。

非简单请求是那种对服务器有特殊要求的请求，比如请求方法是PUT或DELETE，或者Content-Type字段的类型是application/json。

非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。

服务器收到"预检"请求以后，检查了Origin、Access-Control-Request-Method和Access-Control-Request-Headers字段以后，确认允许跨源请求，就可以做出回应。

因此，为了使Nginx可以处理delete等非简单请求，Nginx需要作出相应的改变，更改配置如下

```
location / {
    # 完成浏览器的"预检"请求
		if ($request_method = 'OPTIONS') {
		add_header Access-Control-Allow-Origin *;
		add_header Access-Control-Allow-Credentials true;
		add_header Access-Control-Allow-Methods 'GET, POST, PUT, DELETE, OPTIONS';
		add_header 'Access-Control-Allow-Headers' 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
		return 204;
		}
    # 配置html在本地打开
    if ($request_method = 'POST') {
          add_header 'Access-Control-Allow-Origin' *;
          add_header 'Access-Control-Allow-Credentials' 'true';
          add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
          add_header 'Access-Control-Allow-Headers' 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
      }
    if ($request_method = 'GET') {
          add_header 'Access-Control-Allow-Origin' *;
          add_header 'Access-Control-Allow-Credentials' 'true';
          add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
          add_header 'Access-Control-Allow-Headers' 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
    }
    root   html;
    index  index.html index.htm;
    # 配置html在Nginx中打开
    location ~* \.(html|htm)$ {

		}
    # 代理到8080端口
    proxy_pass        http://127.0.0.1:8080;

}
```

我们还必须把我们的html代码放在Nginx中html文件夹内，即使用Nginx当做我们的前端服务器。

# URL重写

有时候我们仅仅只想将`/api`下的url反向代理到后端，可以通过在nginx.conf中配置url重写规则如下：

```
location / {
    root   html;
    index  index.html index.htm;
         location ^~ /api {
         rewrite ^/api/(.*)$ /$1 break;
         proxy_pass https://api.shanbay.com/;
         }
    }
```

这样的话，我们只用处理`/api`下的url。

在配置文件中我们通过`rewrite`将URL重写为真正要请求的`URL`，通过`proxy_pass`代理到真实的服务器IP或者域名。

# Cookie

如果Cookie的域名部分与当前页面的域名不匹配就无法写入。所以如果请求 www.a.com ，服务器 proxy_pass 到 www.b.com 域名，然后 www.b.com 输出 `domian=b.com` 的 Cookie，前端的页面依然停留在 www.a.com 上，于是浏览器就无法将 Cookie 写入。

可在nginx反向代理中设置：

```
location / {
    # 页面地址是a.com，但是要用b.com的cookie
    proxy_cookie_domain b.com a.com;  #注意别写错位置了 proxy_cookie_path / /;
    proxy_pass http://b.com;
}   
```

# 总结

Nginx解决跨域问题通过Nginx反向代理将对真实服务器的请求转移到本机服务器来避免浏览器的"同源策略限制"。

**参考文档**：
1. [Nginx反向代理配置（解决跨域问题）](http://www.jianshu.com/p/c872ffa500af)
2. 这两篇文章讲解了URL重写的规则和用法
    - [nginx配置url重写](https://xuexb.com/html/nginx-url-rewrite.html)
    - [nginx配置location总结及rewrite规则写法](http://seanlook.com/2015/05/17/nginx-location-rewrite/)
3. [Nginx配置文件详解](http://www.cnblogs.com/wxd0108/p/5217801.html)
