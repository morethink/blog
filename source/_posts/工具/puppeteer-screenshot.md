---
title: puppeteer截图
date: 2019-01-09
tags:
categories: 工具
---

![puppeteer](https://images.morethink.cn/puppeteer.jpg "puppeteer")


puppeteer是谷歌官方出品的一个通过 [DevTools](https://chromedevtools.github.io/devtools-protocol/) 协议控制 headless Chrome 的Node库。可以通过Puppeteer的提供的api直接控制Chrome模拟大部分用户操作来进行UI Test或者作为爬虫访问页面来收集数据。

# 安装

直接运行安装命令：
```shell
npm install puppeteer
```

如果出现无法安装的问题，可以使用淘宝镜像。

<!-- more -->

# puppeteer实现滑动截图
在我 puppeteer 使用截全屏的过程中发现有些图片无法截取到，而实际上是因为有些图片是懒加载的，如果你没有滑动到图片的位置，那么这个图片是不会加载。

现在我的方式是采用模拟浏览器滚动条滑动的方式滑动底部来使图片加载出来。

代码如下：

```javascript
const puppeteer = require('puppeteer');

(async () => {
    const browser = await puppeteer.launch({
        headless: false
    });
    const page = await browser.newPage();
    await page.goto('https://www.cnblogs.com/morethink/p/6525216.html');
    await page.setViewport({
        width: 1200,
        height: 800
    });

    await autoScroll(page);

    await page.screenshot({
        path: '1.png',
        fullPage: true
    });

    await browser.close();
})();


function autoScroll(page) {
    return page.evaluate(() => {
        return new Promise((resolve, reject) => {
            var totalHeight = 0;
            var distance = 100;
            var timer = setInterval(() => {
                var scrollHeight = document.body.scrollHeight;
                window.scrollBy(0, distance);
                totalHeight += distance;
                if (totalHeight >= scrollHeight) {
                    clearInterval(timer);
                    resolve();
                }
            }, 100);
        })
    });
}
```


# puppeteer 实现 html element 截图

在某些情况下我们只想要针对html的某个位置进行截图而不是针对页面截全屏。

puppeteer提供了ElementHandle.screenshot 方法，该方法参数和page.screenshot 一样。而ElementHandle 对象是页面内的Dom对象。可以帮助我对 html element进行截图。这样的话你想截取页面的哪部分就截取页面的哪部分。

代码如下：
```javascript
const puppeteer = require('puppeteer');

(async () => {
    const browser = await puppeteer.launch({
        headless: false
    });
    const page = await browser.newPage();
    await page.goto('https://www.cnblogs.com/morethink/p/6525216.html');
    await page.setViewport({
        width: 1200,
        height: 800
    });
    //获取页面Dom对象
    let body = await page.$('#cnblogs_post_body');
    //调用页面内Dom对象的 screenshot 方法进行截图
    await body.screenshot({
        path: '2.png'
    });
    await browser.close();
})();
```

**参考文档**：
1. https://github.com/GoogleChrome/puppeteer/blob/v1.11.0/docs/api.md#elementhandlescreenshotoptions
