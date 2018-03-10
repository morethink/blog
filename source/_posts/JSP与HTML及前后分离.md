---
title: JSP与HTML及前后分离
date: 2016-10-21
tags:
    - JSP
    - HTML
categories: Web前端
---

# JSP是什么

首先要知道JSP的本质其实是个Servlet，index.jsp在访问的时候首先会自动将该页面翻译生一个index_jsp.java文件，即Servlet代码。

打开这个类你会发现这个类继承了类`org.apache.jasper.runtime.HttpJspBase.SUN`在JSP API中定义了一个接口`HttpJspBase`,这个接口继承了`JspPage`接口，而JspPage接口又继承了Servlet接口，因此WEB容器必须实现这些接口。`org.apache.jasper.runtime.HttpJspBase`就是Tomcat对JSP API中HttpJspBase接口的实现。因此JSP页面在本质上就是Servlet程序，而Servlet程序要被WEB容器调用执行，必须在WEB.XML中注册映射，对于JSP，这些则由WEB容器自动完成。

<!-- more -->

# HTML是什么

参考维基百科的定义：

> 超文本标记语言（英语：HyperText Markup Language，简称：HTML）是一种用于创建网页的标准标记语言。HTML是一种基础技术，常与CSS、JavaScript一起被众多网站用于设计令人赏心悦目的网页、网页应用程序以及移动应用程序的用户界面[1]。网页浏览器可以读取HTML文件，并将其渲染成可视化网页。HTML描述了一个网站的结构语义随着线索的呈现，使之成为一种标记语言而非编程语言。
>
> HTML元素是构建网站的基石。HTML允许嵌入图像与对象，并且可以用于创建交互式表单，它被用来结构化信息——例如标题、段落和列表等等，也可用来在一定程度上描述文档的外观和语义。HTML的语言形式为尖括号包围的HTML元素（如<html>），浏览器使用HTML标签和脚本来诠释网页内容，但不会将它们显示在页面上。
>
> HTML可以嵌入如JavaScript的脚本语言，它们会影响HTML网页的行为。网页浏览器也可以引用层叠样式表（CSS）来定义文本和其它元素的外观与布局。维护HTML和CSS标准的组织万维网联盟（W3C）鼓励人们使用CSS替代一些用于表现的HTML元素[2]。

# JSP VS HTML

**JSP与Html相比，谁更适合做页面**？

java的web开发里我们一般使用jsp来编写页面，当然也可以使用先进点的模板引擎开发页面例如velocity，freemark等，不管我们页面使用的是jsp还是模板引擎，这些类似html的文件其实并不是真正的html，例如jsp本质其实是个servlet也就是一个java程序，所以它们的本质是服务端语言和html的一个整合技术，在实际运行中web容器会根据服务端的返回数据将jsp或模板引擎解析成浏览器能解析的html，然后传输这个html到浏览器进行解析。

现在比较提倡前后分离，前后端通过API来相互连接，后端给出接口，前端通过AJAX访问接口，拿到数据，动态更新。

# 前后分离

**前后端分离是趋势，但是短时间内不可能取代不分离的**。

因为前后分离导致：数据和表现分离，只需要静态的html和动态的接口（例如jsp），数据在浏览器端实现动态加载。因此存在问题，例如SEO，搜索引擎难以识别等，我们可以参考淘宝前端的设计，在 java 接口和 html 输出之前用 NodeJS 代理一层，暂时能解决 SEO 的问题。

理想情况是，先出文档（前后端都认可），然后后端、前端都按照文档来，一切以接口规定的为准。 重点在于接口！靠 API 来分离前后端，解决前后端大团队、多版本、复杂功能协作的问题。定义好了接口，前端就可以用不用等后端，直接用模拟的数据格式，方便地进行前端测试了。

前端可以通过Nginx来解决跨域问题。
具体可以查看 [前端通过Nginx反向代理解决跨域问题](https://www.morethink.cn/Nginx-Cross-Domain/)

说重点，API 相比前后端混写、模板引擎之类的东西的好处：
- 方便设计、开发、测试（前端不再需要依赖后端，后端也不需要依赖前端，就可以各干各的，独立测试代码）
- 方便记录和统计功能使用（后端相同功能的入口位置统一，不同功能的位置也可以合理有序地组织）
- 方便修改和版本控制等（后端可以提供多版本的 API，不需要修改已有代码，不影响已有 API 的功能）
- **最重点的是**：
你的Team要是分工不明确、人少、功能简单直接、代码修改不多，就完全不需要分离，就酱。
- **最明显的**：
前端代码不用被后端粘贴来粘贴去了，后端的相同代码，也不需要各种位置粘贴来粘贴去了。
- **隐藏的好处**：
到时候出了问题，照着 API 设计文档一对比，就知道是前端用的不对，还是后端写的不对，分分钟找到背锅侠。
- 可以一套接口，浏览器，Android通用，极大减少开发量。


# RESTFul API

我们知道前后分离之后是靠API统一前后端的开发，**而RESTFul API目前是前后端分离的最佳实践**。

它有如下好处：
- 轻量，直接通过http，不需要额外协议，post/get/put/delete操作
- 面向资源，一目了然，具有自解释性
- 数据描述简单，一般通过json或者xml做数据通信

下面参考阮一峰的博客解释一下RESTful架构。

要理解RESTful架构，最好的方法就是去理解Representational State Transfer这个词组到底是什么意思，它的每一个词代表了什么涵义。如果你把这个名称搞懂了，也就不难体会REST是一种什么样的设计。

## 资源（Resources）

REST的名称"表现层状态转化"中，省略了主语。"表现层"其实指的是"资源"（Resources）的"表现层"。

所谓"资源"，就是网络上的一个实体，或者说是网络上的一个具体信息。它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的实在。你可以用一个URI（统一资源定位符）指向它，每种资源对应一个特定的URI。要获取这个资源，访问它的URI就可以，因此URI就成了每一个资源的地址或独一无二的识别符。

所谓"上网"，就是与互联网上一系列的"资源"互动，调用它的URI。

## 表现层（Representation）

"资源"是一种信息实体，它可以有多种外在表现形式。我们把"资源"具体呈现出来的形式，叫做它的"表现层"（Representation）。

比如，文本可以用txt格式表现，也可以用HTML格式、XML格式、JSON格式表现，甚至可以采用二进制格式；图片可以用JPG格式表现，也可以用PNG格式表现。

URI只代表资源的实体，不代表它的形式。严格地说，有些网址最后的".html"后缀名是不必要的，因为这个后缀名表示格式，属于"表现层"范畴，而URI应该只代表"资源"的位置。它的具体表现形式，应该在HTTP请求的头信息中用Accept和Content-Type字段指定，这两个字段才是对"表现层"的描述。

## 状态转化（State Transfer）

访问一个网站，就代表了客户端和服务器的一个互动过程。在这个过程中，势必涉及到数据和状态的变化。

互联网通信协议HTTP协议，是一个无状态协议。这意味着，所有的状态都保存在服务器端。因此，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生"状态转化"（State Transfer）。而这种转化是建立在表现层之上的，所以就是"表现层状态转化"。

客户端用到的手段，只能是HTTP协议。具体来说，就是HTTP协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE。它们分别对应四种基本操作：GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT用来更新资源，DELETE用来删除资源。

## 总结RESTful架构

综合上面的解释，我们总结一下什么是RESTful架构：
1. **每一个URI代表一种资源**；
2. **客户端和服务器之间，传递这种资源的某种表现层**；
3. **客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"**。

# GraphQL

GraphQL是一种API查询语言，是由Facebook创建并最终开源的，可以认为是REST的一种替代品。来自AXA Banque的API架构师Lauret给出了一些可对二者进行比较的切入点：

- GraphQL能够通过一次查询得到所有需要的数据，从而减少网络跳转的次数。
- GraphQL采用所见即所得模型，这样客户端代码不易出错。
- RESTful HTTP通过使用状态码和HTTP verb，提高了结果的一致性和可预测性。
- 借助超媒体，在用户使用API时可以“发现”资源间的关系，这简化了RESTful用户的具体实现。
- HTTP实现了缓存机制而GraphQL还没有实现。
- GraphQL给用户提供了schema，这很有用，但是需要注意的是接口描述并非API文档。

**Lauret认为GraphQL的主要优势是其使用的所见即所得(WYSIWYG)模型**。也就是说，查询结果的结构是查询结构本身的精确映射，这样的话，用户在解排（unmarshal）响应的时候不容易出错。

有兴趣的朋友可以看看这两篇解释GraphQL的文章：
- [GraphQL和REST对比时需要注意些什么](http://www.infoq.com/cn/news/2017/07/graphql-vs-rest#)
- [安息吧 REST API，GraphQL 长存](http://www.zcfy.cc/article/rest-apis-are-rest-in-peace-apis-long-live-graphql-3935.html)

**我觉得GraphQL更多的是作为REST的一种替代品**。

**参考文档**：
1. [JSP本质](http://blog.csdn.net/sdyy321/article/details/5838717)
2. [JSP/Servlet 工作原理](http://www.blogjava.net/fancydeepin/archive/2013/09/30/404571.html)
3. [HTML - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/HTML)
4. [关于大型网站技术演进的思考（九）--网站静态化处理--总述（1）](http://www.cnblogs.com/sharpxiajun/p/4282789.html)
5. [这样是不是就叫 前后端分离开发了？](https://segmentfault.com/q/1010000011514355)
6. http://2014.jsconf.cn/slides/herman-taobaoweb/index.html#/
7. [理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)
