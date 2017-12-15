
---
title: windows Apache服务器配置
date: 2017-09-19
tags: 代理
categories: 服务器
---



# Apache

64位可以而32位不可以

## 安装Apache服务
**注意：**
- 如果没有自己设置Apache服务名，后面都可不跟-n "服务名"，即采用默认的服务名称。
- 必须用管理员提示符打开，直接用shift+F10打开命令行是不行的。

<!-- more -->


**命令：**

1. 将apache注册为服务
    - httpd -k install
      将Apache注册为windows服务，可以指定的服务名为"apache"。
    - httpd -k install -n "服务名"　
      将Apache注册为windows服务，自己指定一个服务名字。
    - httpd -k install -n "服务名" -f "conf\my.conf"
      将Apache注册为windows服务，自己指定一个服务名字，并且使用特定配置文件。
2. 卸载Apache服务
    - httpd.exe -k uninstall -n "服务名"　
      移除Apache服务，-n 后面跟自己取得Apache服务器名字
3. 启动Apache服务
      - httpd.exe -k start -n "服务名"　
4. 停止Apache服务
    - httpd.exe -k stop -n "服务名"　
    - httpd.exe -k shutdown -n "服务名" 　
5. 重启Apache服务
      - httpd.exe -k restart -n "服务名"　


想要正确启动Apache 服务，还需要在httpd.conf中配置`Define SRVROOT "E:\Apache24"`为本地Apache位置。

## Apache反向代理配置

1. 需要开启apache代理的拓展，将httpd.conf中下列注释取消
```
LoadModule access_compat_module modules/mod_access_compat.so
LoadModule proxy modules/proxy.so
LoadModule proxy_connect modules/proxy_connect.so
LoadModule proxy_http modules/proxy_http.so
LoadModule proxy_html modules/proxy_html.so
```

2. 添加配置
```
<VirtualHost *:80>
    ServerName localhost
    <Proxy *>
        Order deny,allow
        Allow from all
    </Proxy>
ProxyPass / http://127.0.0.1:8080/
ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

即可反向代理成功到localhost:8080




**参考文档：**
1. [windows下apache最新下载、安装配置](http://meiling.blog.51cto.com/6220221/1786922)
2. [Apache服务器的下载与安装](http://www.cnblogs.com/yerenyuan/p/5460336.html)
3. [Apache24（window）](http://www.jianshu.com/p/203185c61838)
