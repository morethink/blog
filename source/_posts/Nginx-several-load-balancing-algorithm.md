---
title: Nginx几种负载均衡算法及配置实例
date: 2018-01-05
tags: [负载均衡,Nginx]
categories: 服务器
---

本文装载自： https://yq.aliyun.com/articles/114683

Nginx负载均衡（工作在七层“应用层”）功能主要是通过upstream模块实现，Nginx负载均衡默认对后端服务器有健康检测的能力，仅限于端口检测，在后端服务器比较少的情况下负载均衡能力表现突出。

Nginx的几种负载均衡算法：
<!-- more -->
1. 轮询（默认）：每个请求按时间顺序逐一分配到不同的后端服务器，如果后端某台服务器宕机，则自动剔除故障机器，使用户访问不受影响。
2. weight：指定轮询权重，weight值越大，分配到的几率就越高，主要用于后端每台服务器性能不均衡的情况。
3. ip_hash：每个请求按访问IP的哈希结果分配，这样每个访客固定访问一个后端服务器，可以有效的解决动态网页存在的session共享问题。
4. fair（第三方）：更智能的一个负载均衡算法，此算法可以根据页面大小和加载时间长短智能地进行负载均衡，也就是根据后端服务器的响应时间来分配请求，响应时间短的优先分配。如果想要使用此调度算法，需要Nginx的upstream_fair模块。
5. url_hash（第三方）：按访问URL的哈希结果来分配请求，使每个URL定向到同一台后端服务器，可以进一步提高后端缓存服务器的效率。如果想要使用此调度算法，需要Nginx的hash软件包。

在upstream模块中，可以通过server命令指定后端服务器的IP地址和端口，同时还可以设置每台后端服务器在负载均衡调度中的状态，常用的状态有以下几种：
1. down：表示当前server暂时不参与负载均衡。
2. backup：预留的备份机，当其他所有非backup机器出现故障或者繁忙的时候，才会请求backup机器，这台机器的访问压力最轻。
3. max_fails：允许请求的失败次数，默认为1，配合fail_timeout一起使用
4. fail_timeout：经历max_fails次失败后，暂停服务的时间，默认为10s（某个server连接失败了max_fails次，则nginx会认为该server不工作了。同时，在接下来的 fail_timeout时间内，nginx不再将请求分发给失效的server。）


下面是一个负载均衡的配置示例，这里只列出http配置段，省略了其他部分配置：

```
http {
    upstream whsirserver {
    server 192.168.0.120:80 weight=5 max_fails=3 fail_timeout=20s;
    server 192.168.0.121:80 weight=1 max_fails=3 fail_timeout=20s;
    server 192.168.0.122:80 weight=3 max_fails=3 fail_timeout=20s;
    server 192.168.0.123:80 weight=4 max_fails=3 fail_timeout=20s;
    }
    server {
    listen 80;
    server_name blog.whsir.com;
    index index.html index.htm;
    root /data/www;
        location / {
        proxy_pass http://whsirserver;
        proxy_next_upstream http_500 http_502 error timeout invalid_header;
        }
    }
}
```


upstream负载均衡开始，通过upstream指定了一个负载均衡器的名称为whsirserver，这个名称可以自己定义，在后面proxy_pass直接调用即可。

proxy_next_upstream参数用来定义故障转移策略，当后端服务器节点返回500、502和执行超时等错误时，自动将请求转发到upstream负载均衡器中的另一台服务器，实现故障转移。
