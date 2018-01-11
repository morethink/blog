---
title: Win10下通过IIS调试ASP程序遇到的问题和解决方案
date: 2018-01-06
tags: [IIS,ASP]
categories: 服务器
---

最近维护了以前别人的写的一个ASP的系统，记录一下调试过程中的问题和解决方案。


# 环境篇

# 万维网发布服务(W3SVC)已经停止

**问题**：
万维网发布服务(W3SVC)已经停止。除非万维网发布服务(W3SVC)正在运行,否则无法启动网站。

<!-- more -->

![](https://images.morethink.cn/d479322a25d8c93f9bf481695af50858.png)

**解决方法**：

需要先启动整个应用。

![](https://images.morethink.cn/31b0292ddbc24480229d1fbce590bb8e.png)
# IIS服务

控制面板>>程序和功能>>启动或关闭Windows功能>>IIS服务
![](https://images.morethink.cn/729c9c06256b9029cadf7e00cbbfc667.png)

但是这样仅仅是开启了IIS服务，会出现Http500错误，不能运行ASP程序，因为IIS服务器默认并没有帮我们配置ASP或者ASP.NET环境，需要自己手动配置(在此过程中，我启动过多次电脑)。

# 配置ASP环境

ASP配置如下：
![](https://images.morethink.cn/a1339b7725b29585940ccd745fcd7512.png)

如果需要ASP.NET，需要如下配置：

![](https://images.morethink.cn/8e827a63559409b25937857d136e236f.png)

# IIS7中出现An error occurred on the server when processing the URL错误

**错误描述**：
An error occurred on the server when processing the URL. Please contact the system administrator.If you are the system administrator please click here to find out more about this error.

1. 打开控制面板→管理工具→Internet 信息服务(IIS)管理器→双击“ASP”图标
![](https://images.morethink.cn/4cc9001c24f9705e91f4257074f8673c.png)
2. 在左边的窗口中找到你的网站，然后在右边的窗口中展开“调试属性”，把“将错误发送到浏览器”设为True即可
![](https://images.morethink.cn/4a629fd49cf2edefc97cc61a7f1d0d4f.png)

此时你再运行ASP程序时就会看到具体的错误了，然后再根据错误提示进行相应的修改即可。

# 代码篇

# ADODB.Connection 错误 '800a0e7a'

**具体错误**：
ADODB.Connection 错误 '800a0e7a'
未找到提供程序。该程序可能未正确安装。

**原因**：

因为系统是64位的win10，所以会出现这个问题。

**解决办法**：
找到IIS应用程序池，“设置应用程序池默认属性”->“常规”->”启用 32 位应用程序”，设置为 True。

![](https://images.morethink.cn/bb814d72e30f1a99899cc7e919fa774e.png)
height="100%" width="100%"

style="width:757px; height:455px;"
这样问题就解决了。


# ADODB.Recordset 错误 '800a0cc1'

**描述**：
ADODB.Recordset 错误 '800a0cc1'
在对应所需名称或序数的集合中，未找到项目。

**解决**：
一般是字段写错了或者，你的数据库没有这个字段。


# iframe自适应

JS代码：
```javascript
//iframe高度自适应
function IFrameReSize(iframename) {
    var pTar = document.getElementByIdx_x_x(iframename);
    if (pTar) { //ff
        if (pTar.contentDocument && pTar.contentDocument.body.offsetHeight) {
            pTar.height = pTar.contentDocument.body.offsetHeight;
        } //ie
        else if (pTar.Document && pTar.Document.body.scrollHeight) {

            pTar.height = pTar.Document.body.scrollHeight;
        }
    }
}
//iframe宽度自适应
function IFrameReSizeWidth(iframename) {
    var pTar = document.getElementByIdx_x_x(iframename);
    if (pTar) { //ff
        if (pTar.contentDocument && pTar.contentDocument.body.offsetWidth) {
            pTar.width = pTar.contentDocument.body.offsetWidth;
        } //ie
        else if (pTar.Document && pTar.Document.body.scrollWidth) {
            pTar.width = pTar.Document.body.scrollWidth;
        }
    }
}
```
Iframe框配置：
```html
<iframe src="Main.htm" scrolling="no" frameborder="0" height="100%"
id="mainFrame" width="100%" onload='IFrameReSize("mainFrame");IFrameReSizeWidth("mainFrame");'>
</iframe>
```

# ACCESS分页
```sql
select * from news where nid between
(SELECT min(nid) from
(select top 4 nid from newsdata order by nid desc))
and
(SELECT min(nid) from
(select top 1 nid from newsdata order by nid desc))
order by nid desc
```

利用top和min函数分别找出分页的起始ID和结束ID，如果要按照升序排列，就要用top和max来找出起始ID和结束ID，之后在使用between语句直接选取。注意三个地方的排序方式必须一致，查询条件也必须一致。

**参考文档**：
1. [简单又高效的Access分页语句](http://www.ljf.cn/archives/2281)
