---
title: Windows下Nginx实现负载均衡
date: 2017-02-11
tags: [反向代理,Nginx]
categories: 服务器
---

# Apache,Nginx

Apache和Nginx都属于属于 **静态页面服务器**，都有插件支持动态编程语言处理，但Nginx的IO模比Apache更适合跑代理。所以一般都作为**前端缓冲代理**(Nginx的反向代理功能)。
<!-- more -->

# Tomcat,Jetty
tomcat和Jetty都是Java Servlet容器，可以用来生成动态页面，主要用来跑Java的Web功能，当然也提供一个简单静态页面转换：
  - Jetty 是面向 Handler 的架构，就像 Spring 是面向 Bean 的架构，iBATIS 是面向 statement 一样，而 Tomcat 是以多级容器构建起来的，它们的架构设计必然都有一个“元神”，所有以这个“元神“构建的其它组件都是肉身
  - Jetty 可以很容易被扩展和裁剪，相比之下，Tomcat 要臃肿很多，Tomcat 的整体设计上很复杂


# 负载均衡

**tomcat的最大优势在于处理动态请求**，处理静态内容的能力不如Apache和Nginx，并且经过测试发现，tomcat在高并发的场景下，其接受的最大并发连接数是由限制的，连接数过多会导致tomcat处于"僵死"状态，因此，在这种情况下，**我们可以利用Nginx的高并发，低消耗的特点与tomcat一起使用**。因此，tomcat与Nginx、Apache结合使用共有如下几点原因：

1. Tomcat处理html的能力不如Apache和Nginx，tomcat处理静态内容的速度不如Apache和Nginx。
2. tomcat接受的最大并发数有限，接连接数过多，会导致tomcat处于"僵尸"状态，对后续的连接失去响应，需要结合Nginx一起使用。


通常情况下，tomcat与Nginx、Apache结合使用，Nginx、Apache既可以作为 **静态页面服务器**，也可以 **转发动态请求** 至tomcat服务器上。但在一个高性能的站点上，通常Nginx、Apache只提供代理的功能，也就是转发请求至tomcat服务器上，而对于静态内容的响应，则由前端 **负载均衡** 器来转发至专门的静态服务器上进行处理。其架构类似于如下图：

![tomcat-redis-Nginx](https://images.morethink.cn/morethink/images/tomcat-redis-nginx.png "负载均衡")

在这种架构中:
- Nginx作为前端代理时，如果是静态内容，如html、css等内容，则直接交给静态服务器处理；如果请求的图片等内容，则直接交给图片服务器处理。
- 如果请求的是动态内容，则交给tomcat服务器处理。

因此在这里，我们通过用Nginx作为代理服务器来转发来自前端的请求，

# 安装多个Tomcat


1. 修改Tomcat端口为:8081，8082(在同一台电脑上，避免端口冲突)


  |            | SHUTDOWN | HTTP/1.1 | redirectPort| AJP/1.3 |redirectPort|
  | ---        | ---     | ---      | ---          | ---     | ---        |
  | 默认        | 8005   | 8080     | 8443          | 8009    | 8443      |
  | Tomcat7.1  | 8101   | 8081     | -             | 8009    | -          |
  | Tomcat7.2  | 8102   | 8082     | -             | 8009    | -          |


2. 在两个Tomcat中context.xml中加入(与`redis`客户端交互)
```xml
     <Manager className="com.radiadesign.catalina.session.RedisSessionManager"
              host="localhost"
              port="6379"
              database="0"
              maxInactiveInterval="60" />
```
3. 两个Tomcat中加入jar包
  - commons-pool2-2.4.2.jar
  - jedis-2.7.3.jar
  - tomcat-redis-session-manager-1.2-tomcat-7-java-7.jar



# Redis

安装redis(Google，百度一下redis作为nosql，但是下载我的redis版本，在最后)，看一下bin下的RedisService.docx




# Windows下Nginx命令

1. 启动
直接点击Nginx目录下的Nginx.exe 或者 cmd运行start Nginx
2. 关闭
Nginx -s stop    或者    Nginx -s quit
stop表示立即停止Nginx,不保存相关信息
quit表示正常退出Nginx,并保存相关信息
3. 重启(因为改变了配置,需要重启)
Nginx -s reload
4. 关闭进程
tskill Nginx

# Nginx负载均衡配置

在Nginx.conf配置文件中加入注释部分

```
#  upstream  localhost   {
#     server   localhost:8081;
#     server   localhost:8082;
#  }
server {
    listen       80;
    server_name  localhost;
    location / {
        root   html;
        index  index.html index.htm;
  #     proxy_pass        http://localhost;
  #     proxy_set_header   Host             $host;
  #     proxy_set_header   X-Real-IP        $remote_addr;
  #     proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
     }
}
```

# 测试

1. 在两个Tomcat中的webapps下的ROOT下加入一个session.jsp

```
<%=session.getId()%> + Tomcat7.1
```

```
<%=session.getId()%> + Tomcat7.2
```

2. 依次运行redis，Tomcat，Nginx

![session-test1](https://images.morethink.cn/session-test1.jpg)

![session-test2](https://images.morethink.cn/session-test2.jpg)

**成功**
jar包，redis，Nginx，Tomcat ： http://pan.baidu.com/s/1c1NSPzU


# 总结

## Tomcat9配置

最开始，我采用的是Tomcat9，在我安装的时候总是有问题，很多东西已经改了，tomcat9安装来安装去，结果都是同一个Tomcat，有时候版本高就是有点坑。

**Tomcat9必须要配置 `CATALINA_HOME`**

问题 ：
1. 必须配置CATALINA_HOME变量，不然就闪退，但是如果配置就打开了自己的用的Tomcat，不是需要测试的Tomcat,结果是加的东西改变了他的属性


具体配置参考 ： [windows下安装多个tomcat服务](https://my.oschina.net/ihanfeng/blog/286062)

1. **安装服务**
在命令行中进入/Tomcat路径/bin/，执行“service.bat install”：
说明：
  1. 服务名和显示名称：service.bat中设置了默认的服务名称，不同版本分别命名为Tomcat4、Tomcat5、Tomcat6，如果需要自 定义服务名或服务的显示名称，可在service.bat中修改SERVICE_NAME或PR_DISPLAYNAME；
  2. 防火墙的影响：/bin/tomcat6.exe（或tomcat4.exe、tomcat5.exe）将被作为服务程序，如果有防火墙，需要设为允许作为服务。
2. **卸载服务**
在命令行中进入/Tomcat路径/bin/，执行“service.bat remove”：

## Jar包

jar包是一个很大的问题，有些jar包不支持(错误为仔细记录)，而且Tomcat-session jar的实现版本也不一样。

**不过正因为踩了坑，才更有趣，不是吗**？
