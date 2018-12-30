---
title: Python 豆瓣顶帖
date: 2018-12-30 16:17:00
tags:
categories: Python
---

![](https://images.morethink.cn/278d71ed0f07f6d1d72a8afa76039cc2.png "🆙🆙")

由于在豆瓣发了个租房帖子，发现很快就被其他的帖子淹没，但是手动顶帖实在太累，😭，所以想通过自动顶帖的方式来解放双手！

<!-- more -->

# 评论请求分析

通过Chrome network 分析

![](https://images.morethink.cn/4220f35fed72284a44099dfcf27028d8.png "add_comment")

- 评论url是`https://www.douban.com/group/topic/129122199/add_comment`
- 需要带5个参数，其中 ck 是 cookie 里面的值，rv_comment 是 评论
- 返回302代表重定向

Python 模拟请求：

```python
# 豆瓣具体帖子
url = "https://www.douban.com/group/topic/129122199/"
# 豆瓣具体帖子回复的接口，格式是帖子链接+/add_comment
comment_url = url + "/add_comment"
cookie = 'cookie'
referer = url
agent = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181 Safari/537.36'
headers = {
    "Host": "www.douban.com",
    "Referer": referer,
    'User-Agent': agent,
    "Cookie": cookie
}
params = {
    "rv_comment": '🆙',
    "ck": re.findall("ck=(.*?);", headers["Cookie"])[-1],
    'start': '0',
    'submit_btn': '发送'
}
response = requests.post(comment_url, headers=headers, allow_redirects=False,
                         data=params, verify=False)
```

直接运行即可。

但是多运行几次就会发现，返回的状态码是200，而且没有顶帖成功。实际上是触发了豆瓣的防爬虫。

![](https://images.morethink.cn/96695d5876d98ca99abc79866020a284.png "触发了豆瓣验证码")

而且在我们顶帖的时候发送请求的时候还带有 captcha-solution 和 captcha-id 字段。

![](https://images.morethink.cn/8a57550867a8e02b3272d6043632f51b.png)

目前发现，每次评论就算相隔1分钟，只要满3次，就一定会弹出这个验证码进行验证。

# 验证码解析

遇到验证码我们就来破解验证码。

## tesserocr

识别图形验证码需要安装tesserocr这个库，下面介绍下tesserocr。

tesserocr是Python的一个OCR识别库，但其实是对tesseract做了一层Python Api的封装，核心还是tesseract，所以在安装tesserocr之前，需要先安装tesseract。`Tesseract`(/‘tesərækt/) 这个词的意思是”超立方体”，指的是几何学里的四维标准方体，又称”正八胞体”，是一款被广泛使用的开源 `OCR` 工具。


在Mac下，使用 brew 安装

```
brew install tesseract --all-languages
```

接下来再安装tesserocr即可：

```
brew install imagemagick
pip install tesserocr pillow
```

Python 代码如下：

```python
import tesserocr

from PIL import Image

if __name__ == '__main__':
    # 新建Image对象
    image = Image.open("/Users/liwenhao/Desktop/douban-captcha-example1.jpeg")
    # 调用tesserocr的image_to_text()方法，传入image对象完成识别
    result = tesserocr.image_to_text(image)
    print(result)
```

验证的图片如下：

![douban-captcha-example1](https://images.morethink.cn/douban-captcha-example1.jpeg "douban-captcha-example1")

结果无法识别。

换一张简单的图片试试：
![captcha-example1.jpg](http://images.morethink.cn/captcha-example1.jpg)

结果如下：
```
5594
```

看来 Tesseract 只能识别一些简单的验证码，不适合豆瓣验证码识别。

试试识别验证码平台。

## 百度OCR


**官方接入文档**: [文字识别-Python SDK接入文档](https://link.juejin.im?target=https%3A%2F%2Fcloud.baidu.com%2Fdoc%2FOCR%2FOCR-Python-SDK.html%23.E5.BF.AB.E9.80.9F.E5.85.A5.E9.97.A8)

- **重点：免费**  
- 通用识别（包括身份证、银行卡）500次/日，  
- 高精度则50次/日，  
- 驾驶证，行驶证，车票，营业执照，通用票据均为200次/日  

注意：
**支持2.7.+及3.+**

### 配置流程：

1. 先开通个百度的账号；  
2. 开通**文字识别服务**，打开后点击立即使用：https://cloud.baidu.com/product/ocr.html  
3. 点击步骤2，应该有个信息确认的，确认后，会进入到用户个人首页，向下滑动，直接点击文字识别:
![](https://user-gold-cdn.xitu.io/2018/6/11/163ecde99a94d224?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
4. 点击创建应用，输入一堆内容后，点击确认即可，然后点击我的应用，这里面的**API Key** 跟**Secret Key**需要使用到: ![](https://user-gold-cdn.xitu.io/2018/6/11/163ece17b329ea80?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
5. 点击右上角，用户中心，用户ID也需要用到:
![](https://user-gold-cdn.xitu.io/2018/6/11/163ece27d4bc3e76?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

6. 需要的信息准备好了，**pip** 安装一波
    ```Python
    pip install baidu-aip
    ```

### 测试一波

```python
import json

from aip import AipOcr

if __name__ == '__main__':
    APP_ID = ' '
    API_KEY = ' '
    SECRET_KEY = ' '

    client = AipOcr(APP_ID, API_KEY, SECRET_KEY)

    # 读取图片
    def get_file_content(file_path):
        with open(file_path, 'rb') as fp:
            return fp.read()


    image = get_file_content('/Users/liwenhao/Desktop/douban-captcha-example2.jpg')
    """ 调用通用文字识别(高精度), 图片参数为本地图片 """
    result = json.dumps(client.basicAccurate(image))
    print(result)
```
验证的图片如下：

![douban-captcha-example1](https://images.morethink.cn/douban-captcha-example1.jpeg "douban-captcha-example1")


结果走一波：

```
{"log_id": 3968431492157876638, "words_result_num": 1, "words_result": [{"words": " minute:"}]}
```

从结果可以看出识别出了这个验证码。
- `words_result_num` 是识别结果数
- `words_result` 是定位和识别结果数组
- `words` 是识别结果字符串

再来试试

![douban-captcha-example2](https://images.morethink.cn/douban-captcha-example2.jpg "douban-captcha-example2")

结果如下：
```
{"log_id": 5251449865676063710, "words_result_num": 0, "words_result": []}
```
没有识别出来，可以看到对于复杂一些的验证码还是会出现无法识别的情况，但是胜在免费。


## 超级鹰

对于无法识别的情况就需要打码平台了，业界比较出名的是 [超级鹰](https://link.juejin.im?target=http%3A%2F%2Fwww.chaojiying.com%2F) 。

超级鹰是按量级收费，量大便宜，标准价格:1元=1000题分，不同验证码类型，需要的题分不一样，详情可以到这里查询 http://www.chaojiying.com/price.html

python 代码如下：
```python
from hashlib import md5
import requests
import json


# 通过超级鹰识别验证码
def recognition_captcha(filename, code_type):
    im = open(filename, 'rb').read()
    params = {
        'user': '账号',
        'pass2': md5('密码'.encode('utf8')).hexdigest(),
        'softid': 'softid',
        'codetype': code_type
    }
    headers = {
        'Connection': 'Keep-Alive',
        'User-Agent': 'Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0)',
    }
    files = {'userfile': ('ccc.jpg', im)}
    resp = requests.post('http://upload.chaojiying.net/Upload/Processing.php', data=params, files=files,
                         headers=headers).json()
    return resp


# 调用代码
if __name__ == '__main__':
    print(json.dumps(recognition_captcha('/Users/liwenhao/Desktop/douban-captcha-example2.jpg', 1006)))
```

上传的验证码就是上面百度 OCR 未曾识别的验证码，如下：

![douban-captcha-example2](https://images.morethink.cn/douban-captcha-example2.jpg "douban-captcha-example2")
结果如下：
```
{"err_str": "OK", "err_no": 0, "md5": "0475b05654c376deb409bfef7eee75cd", "pic_id": "8054415552001300054", "pic_str": "yacvmd"}
```

发现 验证码 `yacvmd` 已出来。但是时间花了5s左右。后来测试发现对于豆瓣比较建的验证码花费的时间在1s内，因此从时间和准确性上面，最后还是采用了超级鹰打码平台。

# 失败微信通知

无论采用什么方式，都有可能出现失败的情况，我总不能采取 轮询 的方式，隔几个小时就去看看到底前面几次是否🆙成功，因此需要一个 异步通知 ，最开始想用 邮件，后来发现了 [Server酱](http://sc.ftqq.com/3.version) 这个神器，可以帮助我们发送微信通知，而且特别简单。

具体可以查看 [Server酱](http://sc.ftqq.com/3.version)。

# 完整代码
```python
import os

import requests
import urllib3
import re
from hashlib import md5
import random
from lxml import html
import logging

logging.basicConfig(level=logging.INFO, format='%(asctime)s.%(msecs)03d %(levelname)s: %(message)s',
                    datefmt='%Y-%m-%d %H:%M:%S')
urllib3.disable_warnings()


# 下载验证码图片
def download_captcha(captcha_url, agent):
    # findall返回的是一个列表
    captcha_name = re.findall("id=(.*?):", captcha_url)
    filename = "douban_%s.jpg" % (str(captcha_name[0]))
    logging.info("文件名为: " + filename)
    with open(filename, 'wb') as f:
        # 以二进制写入的模式在本地构建新文件
        header = {
            'User-Agent': agent,
            'Referer': captcha_url
        }
        f.write(requests.get(captcha_url, headers=header).content)
        logging.info("%s 下载完成" % filename)
    return filename


# 通过超级鹰识别验证码
def recognition_captcha(filename, code_type):
    im = open(filename, 'rb').read()
    params = {
        'user': '用户',
        'pass2': md5('密码'.encode('utf8')).hexdigest(),
        'softid': 'softid',
        'codetype': code_type
    }
    headers = {
        'Connection': 'Keep-Alive',
        'User-Agent': 'Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0)',
    }
    files = {'userfile': ('ccc.jpg', im)}
    resp = requests.post('http://upload.chaojiying.net/Upload/Processing.php', data=params, files=files,
                         headers=headers).json()
    # 错误处理
    if resp.get('err_no', 0) == 0:
        return resp.get('pic_str')


def result_verification(response):
    if response.status_code == 302:
        logging.info("豆瓣ding成功")
    else:
        logging.info(response.status_code)
        logging.info(response)
        url = "https://sc.ftqq.com/你的SCKEY.send?text=douban失败" + \
              str(random.randint(0, 1000))
        requests.post(url)
        logging.info("豆瓣ding失败，发送失败信息到微信")


# 豆瓣顶帖
def douban_ding():
    # 豆瓣具体帖子
    url = "https://www.douban.com/group/topic/129122199/"
    # 豆瓣具体帖子回复的接口，格式是帖子链接+/add_comment
    comment_url = url + "/add_comment"
    cookie = 'cookie'
    referer = url
    agent = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181 Safari/537.36'
    headers = {
        "Host": "www.douban.com",
        "Referer": referer,
        'User-Agent': agent,
        "Cookie": cookie
    }
    params = {
        "rv_comment": '🆙',
        "ck": re.findall("ck=(.*?);", headers["Cookie"])[-1],
        'start': '0',
        'submit_btn': '发送'
    }
    response = requests.get(url, headers=headers, verify=False).content.decode('utf-8')
    selector = html.fromstring(response)
    captcha_image = selector.xpath("//img[@id=\"captcha_image\"]/@src")
    if captcha_image:
        logging.info("发现验证码，下载验证码")
        captcha_id = selector.xpath("//input[@name=\"captcha-id\"]/@value")
        filename = download_captcha(captcha_image[0], agent)
        captcha_solution = recognition_captcha(filename, 1006)
        os.remove(filename)
        params['captcha-solution'] = captcha_solution
        params['captcha-id'] = captcha_id
    else:
        logging.info("没有验证码")
    response = requests.post(comment_url, headers=headers, allow_redirects=False,
                             data=params, verify=False)
    result_verification(response)


if __name__ == '__main__':
    douban_ding()
```
运行结果：
1. 第1次：
    ```
    2018-12-30 16:09:35.589 INFO: 没有验证码
    2018-12-30 16:09:36.436 INFO: 豆瓣ding成功
    ```
2. 第4次：
    ```
    2018-12-30 16:13:02.135 INFO: 发现验证码，下载验证码
    2018-12-30 16:13:02.135 INFO: 文件名为: douban_OJGsVa0hST4O2WhFA0VpMnR9.jpg
    2018-12-30 16:13:02.554 INFO: douban_OJGsVa0hST4O2WhFA0VpMnR9.jpg 下载完成
    2018-12-30 16:13:09.687 INFO: 豆瓣ding成功
    ```
效果图：

![](https://images.morethink.cn/b9eeaa536e309a201824eac764d11e33.png)


注：
1. 顶帖的时候控制好频率，不然容易被禁言。 ![](https://images.morethink.cn/douban-ban.jpg "豆瓣禁言")
