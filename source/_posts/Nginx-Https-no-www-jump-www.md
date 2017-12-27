---
title: Nginx Https配置不带www跳转www
date: 2017-12-27
tags: [Nginx,Https]
categories: 服务器
---


把 morethink.cn和www.morethink.cn合并到一个server上去，使用301永久重定向。
然后将 https://morethink.cn 转到 https://www.morethink.cn 去。不过要在配置文件的 `server` https://www.morethink.cn
上配置default_server ssl;。
301永久重定向可以把搜索引擎的权重全部集中到 https://www.morethink.cn 上。

<!-- more -->

配置如下：

```
server {
    listen       80;
    server_name morethink.cn,www.morethink.cn;
    return 301 https://www.morethink.cn$request_uri;
}
server {
    listen 443;
    server_name morethink.cn;
    return 301 https://www.morethink.cn$request_uri;
}
server {
    listen 443 default_server ssl;
    server_name  www.morethink.cn;
    ssl on;
    ssl_certificate 1_www.morethink.cn_bundle.crt;
    ssl_certificate_key 2_www.morethink.cn.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    root         /var/www/hexo;
    include /etc/nginx/default.d/*.conf;
    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```


**参考文档**：
1. [腾讯云 Nginx Https 证书安装指引](https://cloud.tencent.com/document/product/400/4143)
2. [nginx配置http强制跳转https](https://docs.lvrui.io/2017/04/01/nginx%E9%85%8D%E7%BD%AEhttp%E5%BC%BA%E5%88%B6%E8%B7%B3%E8%BD%AChttps/)
