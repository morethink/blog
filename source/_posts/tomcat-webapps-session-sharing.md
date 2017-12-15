---
title: 单点登录之tomcat中session在两个webapp中实现共享
date: 2017-10-30
tags: [Redis,Tomcat,session]
categories: 服务器
---

现在遇到一个需求就是要求完成简单的单点登录，通过在一个tomcat实例中放置两个webapps应用ROOT应用和CEO应用来完成在ROOT应用登录后，在CEO可以直接使用，而未在ROOT应用登录时，不可以进去CEO应用。
实际上问题就是session如何在两个webapp中实现共享，通过上网搜索发现一个方法
<!-- more -->
# 方法1ServletContext
server.xml文件修改如下：
```xml
<Host name="localhost"  appBase="webapps"unpackWARs="true" autoDeploy="true">  
    //WebappA为项目名，crossContext="true"
    <Context path="/WebappA"  debug="9" reloadable="true" crossContext="true"/>
    <Context path="/WebappB"  debug="9" reloadable="true" crossContext="true"/>  
</Host>  
```
crossContext属性的意思是：如果设置为true，你可以通过ServletContext.getContext() 调用另外一个WEB应用程序，获得ServletContext 然后再调用其getAttribute() 得到你要的对象。
Java代码如下：
WebappA:
```java
HttpSession session = request.getSession();  
session.setAttribute("userId", "test");  
ServletContext ContextA =session .getServletContext();  
ContextA.setAttribute("session", session );  
```
WebappB:
```java
HttpSession sessionB = request.getSession();    
ServletContext ContextB = sessionB.getServletContext();    
ServletContext ContextA= ContextB.getContext("/WebappA");// 这里面传递的是 WebappA的虚拟路径  
HttpSession sessionA =(HttpSession)ContextA.getAttribute("session");  
System.out.println("userId: "+sessionA.getAttribute("userId"));  
```

初看这个方法，好像是完成我们的目标，可是在我实际应用时发现一个问题，就是当user1在登录前不可以进入CEO应用，在user1登录后才可以进入CEO应用，但是当user1退出之后，未登录的用户依然可以进入CEO应用。
后来仔细看了一下网上提供的方法，它只是在webappA的ServletContext存储了一个session值，然后传递给webAPPB，但是也仅仅只能传递一个session值，如果有两个用户的时候就会出现session覆盖。

于是探究其他解决方法。

# 方法2sessionCookiePath
在tomcat conf/context.html中有如下配置
- **sessionCookieName**
The name to be used for all session cookies created for this context. If set, this overrides any name set by the web application. If not set, the value specified by the web application, if any, will be used, or the name JSESSIONID if the web application does not explicitly set one.
- **sessionCookiePath**
The path to be used for all session cookies created for this context. If set, this overrides any path set by the web application. If not set, the value specified by the web application will be used, or the context path used if the web application does not explicitly set one. To configure all web application to use an empty path (this can be useful for portlet specification implementations) set this attribute to / in the global CATALINA_BASE/conf/context.xml file.
Note: Once one web application using sessionCookiePath="/" obtains a session, all subsequent sessions for any other web application in the same host also configured with sessionCookiePath="/" will always use the same session ID. This holds even if the session is invalidated and a new one created. This makes session fixation protection more difficult and requires custom, Tomcat specific code to change the session ID shared by the multiple applications.

也就是说我们可以通过`sessionCookiePath`属性使得一个tomcat实例下所有的webapps都共享一个session，通过sessionCookieName来指定sessionCookieName名字。

于是我就在tomcat conf/context.html中配置如下：
```xml
<Context sessionCookiePath="/" sessionCookieName="SESSIONID" >
</Context>
```
然后在进行测试，发现在ROOT应用和CEO应用中确实sessionCookie是一样的。可是当我在ROOT中进行`      session.setAttribute();`时，CEO应用不能从session中取得值，为null，也就是说，对CEO应用而言，ROOT应用在session所储存的值是不可见的。然后在CEO session中进行`session.setAttribute();`，ROOT应用总同样无法取得CEO存储在session中的数据，猜想可能是不同的webapps并不会共享相同的session内存，每一个webapps维护自己session HashTable，后来了解到 **session管理器是和context容器关联的，也就说每个web应用都会有一个session管理器**，所以CEO应用当然无法从ROOT应用存储的session中取值。

# 方法3redis session 共享

以前曾经了解过Nginx+tomcat+redis做负载均衡的内容，知道可以把session数据存储到redis中，然后tomcat再去redis取值。
而这次的tomcat中session在两个webapp中实现共享其实也可以通过这个方法进行处理。

在tomcat conf/context.html中配置如下：

```xml
<Context sessionCookiePath="/" sessionCookieName="SESSIONID" >
<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />
 	<Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
              host="172.22.4.16"
              port="6379"
              database="0"
              maxInactiveInterval="60" />
</Context>
```
然后即可实现tomcat中session在两个webapp中实现共享。

**jar包和windows redis安装包**
http://pan.baidu.com/s/1eSITNnc

# session过期策略

**tomcat-session怎么实现的过期策略？**
首先如果没有使用redis做session缓存，tomcat服务器在启动的时候初始化了一个守护线程,定期6*10秒去检查有没有Session过期.过期则清除。而使用的tomcat-session-redis做缓存，那么session过期之后就由redis进行删除，redis通过惰性删除和定期删除来删除过期的sessionID值。

当然sessionID还存在于客户端，那么客户端的sessionID清理过程是什么？
经过测试是当tomcat删除sessionID值之后，tomcat会重新生成一个sessionID值返回给客户端。
![](https://images.morethink.cn/4b5f8d1d886507e22022f753d970a0e1.png)

**总结：**
其实我这个功能就是单点登录，也就是说在A应用登录的情况下可以访问B应用，但是即使设置了`sessionCookiePath`，session的Attribute并没有共享，于是想到了先把session序列化到redis中，然后取出来判断，这样就可以实现单点登录。核心是session在这两个应用中必须是一样的，通过设置`sessionCookiePath`。


**注意：**
1. 当你使用自己的对象执行`session.setAttribute();`时，必须实现`Serializable`接口，不然无法进行序列化。
2. `maxInactiveInterval` 不起作用
tomcat日志描述：
警告: Manager.setMaxInactiveInterval() is deprecated and calls to this method are ignored. Session timeouts should be configured in web.xml or via Context.setSessionTimeout(int timeoutInMinutes).
信息: Will expire sessions after 120 seconds(默认是30分钟)
只能通过在ROOT下的web.xml或者全局的web.xml(CEO中的session-config无法生效)中配置才可生效。
    ```xml
    <!--设置session过期时间-->
    <session-config>
        <session-timeout>20</session-timeout>
    </session-config>
    ```
    配置成功后tomcat日志：
    信息: Will expire sessions after 120 seconds
3. 在context中配置host为静态ip 172.22.4.16报错如下:
![](https://images.morethink.cn/efe8ce59cb21d0131028c15e278c80f7.png)，Google发现原来redis和mysql一样都是已经默认绑定了localhost，只允许本机访问，于是更改`redis.windows-service.conf`如下
    ```conf
    bind 127.0.0.1 172.22.4.16
    ```
    之后就可以使用静态ip 172.22.4.16进行访问。
4. msi应用的安装，修复和卸载都是通过点击msi文件。
5. **还有一个共享session的方案是spring-session。** spring-session中通过自己生成session并且存储到redis中，还是需要设置`sessionCookiePath="/"`(在一个tomcat两个应用需要单点登录的情况)，其他session共享方案(在多个tomcat中实现单点登录可以参考 [spring session无法实现共享（多web应用）](http://www.jianshu.com/p/a4f49e73bce6))，但是spring-session不是服务器级别的，而是web 应用级别的，不受服务器如tomcat，jetty，jboss的限制。


**参考文档**
1. http://tomcat.apache.org/tomcat-7.0-doc/config/context.html
2. [搭建Tomcat集群&通过Redis缓存共享session的一种流行方案](https://segmentfault.com/a/1190000009591087)
3. [Tomcat的Session过期处理策略](http://blog.csdn.net/jiangguilong2000/article/details/51969250)
4. [Tomcat中session的管理机制](http://www.cnblogs.com/interdrp/p/4935614.html)
