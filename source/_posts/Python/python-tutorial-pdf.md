---
title: 学以致用:Python爬取廖大Python教程制作pdf
date: 2019-01-10
tags:
categories: Python
---

![python-tutorial-pdf](https://images.morethink.cn/python-tutorial-pdf.jpeg "python-tutorial-pdf")

当我学了廖大的Python教程后，感觉总得做点什么，正好自己想随时查阅，于是就开始有了制作PDF这个想法。

想要把教程变成PDF有三步：
1. 先生成空html，爬取每一篇教程放进一个新生成的div，这样就生成了包含所有教程的html文件(`BeautifulSoup`)
2. 将html转换成pdf(`wkhtmltopdf`)
3. 由于廖大是写教程的，反爬做的比较好，在爬取的过程中还需要代理ip(`免费 or 付费`)

<!-- more -->

# BeautifulSoup

> [Beautiful Soup](http://www.crummy.com/software/BeautifulSoup/) 是一个可以从HTML或XML文件中提取数据的Python库.它能够通过你喜欢的转换器实现惯用的文档导航,查找,修改文档的方式.Beautiful Soup会帮你节省数小时甚至数天的工作时间.

## 安装

```shell
pip3 install BeautifulSoup4
```

## 开始使用

将一段文档传入 `BeautifulSoup` 的构造方法,就能得到一个文档的对象, 可以传入一段字符串或一个文件句柄.

如下所示：
```python
from bs4 import BeautifulSoup
soup = BeautifulSoup(open("index.html"))
soup = BeautifulSoup("<html>data</html>")
```

- 首先,文档被转换成Unicode,并且HTML的实例都被转换成Unicode编码.
- 然后,Beautiful Soup选择最合适的解析器来解析这段文档,如果手动指定解析器那么Beautiful Soup会选择指定的解析器来解析文档.

## 对象的种类
`Beautiful Soup` 将复杂 `HTML` 文档转换成一个复杂的树形结构,每个节点都是 `Python` 对象,所有对象可以归纳为 4 种: `Tag , NavigableString , BeautifulSoup , Comment .`

-   `Tag`：通俗点讲就是 `HTML` 中的一个个标签，类似 `div，p`。
-   `NavigableString`：获取标签内部的文字，如，`soup.p.string`。
-   `BeautifulSoup`：表示一个文档的全部内容。
-   `Comment：Comment` 对象是一个特殊类型的 `NavigableString` 对象，其输出的内容不包括注释符号.

## Tag

`Tag`就是`html`中的一个标签，用`BeautifulSoup`就能解析出来`Tag`的具体内容，具体的格式为`soup.name`,其中`name`是`html`下的标签，具体实例如下：

-   `print soup.title`输出`title`标签下的内容，包括此标签，这个将会输出
    ```html
    <title>The Dormouse's story</title>
    ```
-   `print soup.head`输出`head`标签下的内容
    ```html
    <head><title>The Dormouse's story</title></head>
    ```

**如果 Tag 对象要获取的标签有多个的话，它只会返回所以内容中第一个符合要求的标签**。

### Tag 属性

每个 `Tag` 有两个重要的属性 `name` 和 `attrs`：

-   `name`：对于`Tag`，它的`name`就是其本身，如`soup.p.name`就是`p`
-   `attrs`是一个字典类型的，对应的是属性-值，如`print soup.p.attrs`,输出的就是`{'class': ['title'], 'name': 'dromouse'}`,当然你也可以得到具体的值，如`print soup.p.attrs['class']`,输出的就是`[title]`是一个列表的类型，因为一个属性可能对应多个值,当然你也可以通过get方法得到属性的，如：`print soup.p.get('class')`。还可以直接使用`print soup.p['class']`


### get

`get`方法用于得到标签下的属性值，注意这是一个重要的方法，在许多场合都能用到，比如你要得到`<img src="#">`标签下的图像`url`,那么就可以用`soup.img.get('src')`,具体解析如下：

```python
# 得到第一个p标签下的src属性
print soup.p.get("class")   
```

### string

得到标签下的文本内容，只有在此标签下没有子标签，或者只有一个子标签的情况下才能返回其中的内容，否则返回的是`None`具体实例如下：

```python
# 在上面的一段文本中p标签没有子标签，因此能够正确返回文本的内容
print soup.p.string
# 这里得到的就是None,因为这里的html中有很多的子标签
print soup.html.string  
```

### `get_text()`

**可以获得一个标签中的所有文本内容，包括子孙节点的内容，这是最常用的方法**。



## 搜索文档树
BeautifulSoup 主要用来遍历子节点及子节点的属性，通过`Tag`取属性的方式只能获得当前文档中的第一个 tag，例如，`soup.p`。如果想要得到所有的`<p>` 标签,或是通过名字得到比一个 tag 更多的内容的时候,就需要用到 find_all()
```python
find_all(name, attrs, recursive, text, **kwargs )
```

find_all是用于搜索节点中所有符合过滤条件的节点。

**name参数**：是Tag的名字，如p,div,title
```py
# 1. 节点名
print(soup.find_all('p'))
# 2. 正则表达式
print(soup.find_all(re.compile('^p')))
# 3. 列表  
print(soup.find_all(['p', 'a']))
```

另外 attrs 参数可以也作为过滤条件来获取内容，而 limit 参数是限制返回的条数。



## CSS 选择器

以 CSS 语法为匹配标准找到 Tag。同样也是使用到一个函数，该函数为`select()`，返回类型是 list。它的具体用法如下：

```python
# 1. 通过 tag 标签查找
print(soup.select(head))
# 2. 通过 id 查找
print(soup.select('#link1'))
# 3. 通过 class 查找
print(soup.select('.sister'))
# 4. 通过属性查找
print(soup.select('p[name=dromouse]'))
# 5. 组合查找
print(soup.select("body p"))
```

# wkhtmltopdf

> - wkhtmltopdf主要用于HTML生成PDF。
> - pdfkit是基于wkhtmltopdf的python封装，支持URL，本地文件，文本内容到PDF的转换，其最终还是调用wkhtmltopdf命令。

## 安装

先安装wkhtmltopdf，再安装pdfkit。

1. https://wkhtmltopdf.org/downloads.html
2. pdfkit
    ```shell
    pip3 install pdfkit
    ```


## 转换url/file/string

```python
import pdfkit

pdfkit.from_url('http://google.com', 'out.pdf')
pdfkit.from_file('index.html', 'out.pdf')
pdfkit.from_string('Hello!', 'out.pdf')
```


## 转换url或者文件名列表

```python
pdfkit.from_url(['google.com', 'baidu.com'], 'out.pdf')
pdfkit.from_file(['file1.html', 'file2.html'], 'out.pdf')
```


## 转换打开文件

```python
with open('file.html') as f:
    pdfkit.from_file(f, 'out.pdf')
```


## 自定义设置


```python
options = {
    'page-size': 'Letter',
    'margin-top': '0.75in',
    'margin-right': '0.75in',
    'margin-bottom': '0.75in',
    'margin-left': '0.75in',
    'encoding': "UTF-8",
    'custom-header' : [
        ('Accept-Encoding', 'gzip')
    ]
    'cookie': [
        ('cookie-name1', 'cookie-value1'),
        ('cookie-name2', 'cookie-value2'),
    ],
    'no-outline': None,
    'outline-depth': 10,
}

pdfkit.from_url('http://google.com', 'out.pdf', options=options)
```

# 使用代理ip

爬取十几篇教程之后触发了这个错误：
![503](https://images.morethink.cn/a7fa858c35a52d0bd1ed89a611546ef1.png "503")

看来廖大的反爬虫做的很好，于是只好使用代理ip了，尝试了免费的[西刺免费代理](https://www.xicidaili.com/)后，最后选择了付费的 [阿布云](https://center.abuyun.com) ，感觉响应速度和稳定性还OK。

# 运行结果

运行过程截图：

![运行过程](https://images.morethink.cn/ad8217afdd1d38c665a1a0341ade499e.png "运行过程")

生成的效果图：
![效果图](https://images.morethink.cn/3b24e8815b5e9ac26184a3f1ab96bd1d.png "效果图")

代码如下：
```py
import time
import pdfkit
import requests
from bs4 import BeautifulSoup


# 使用 阿布云代理 
# 可以选择不使用或是其他代理
def get_soup(target_url):
    proxy_host = "http-dyn.abuyun.com"
    proxy_port = "9020"
    proxy_user = "你的用户"
    proxy_pass = "你的密码"
    proxy_meta = "http://%(user)s:%(pass)s@%(host)s:%(port)s" % {
        "host": proxy_host,
        "port": proxy_port,
        "user": proxy_user,
        "pass": proxy_pass,
    }

    proxies = {
        "http": proxy_meta,
        "https": proxy_meta,
    }
    headers = {'User-Agent':
                   'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36'}
    flag = True
    while flag:
        try:
            resp = requests.get(target_url, proxies=proxies, headers=headers)
            flag = False
        except Exception as e:
            print(e)
            time.sleep(0.4)

    soup = BeautifulSoup(resp.text, 'html.parser')
    return soup


def get_toc(url):
    soup = get_soup(url)
    toc = soup.select("#x-wiki-index a")
    print(toc[0]['href'])
    return toc


# ⬇️教程html
def download_html(url, depth):
    soup = get_soup(url)
    # 处理目录
    if int(depth) <= 1:
        depth = '1'
    elif int(depth) >= 2:
        depth = '2'
    title = soup.select(".x-content h4")[0]
    new_title = BeautifulSoup('<h' + depth + '>' + title.string + '</h' + depth + '>', 'html.parser')
    print(new_title)
    # 加载图片
    images = soup.find_all('img')
    for x in images:
        x['src'] = x['data-src']

    div_content = soup.find('div', class_='x-wiki-content')
    return new_title, div_content


def convert_pdf(template):
    html_file = "python-tutorial-pdf.html"
    with open(html_file, mode="w", encoding="utf8") as code:
        code.write(str(template))
    pdfkit.from_file(html_file, 'python-tutorial-pdf.pdf')


if __name__ == '__main__':
    # html 模板
    template = BeautifulSoup(
        '<!DOCTYPE html> <html lang="en"> <head> <meta charset="UTF-8"> <link rel="stylesheet" href="https://cdn.liaoxuefeng.com/cdn/static/themes/default/css/all.css?v=bc43d83"> <script src="https://cdn.liaoxuefeng.com/cdn/static/themes/default/js/all.js?v=bc43d83"></script> </head> <body> </body> </html>',
        'html.parser')
    # 教程目录
    toc = get_toc('https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000')
    for i, x in enumerate(toc):
        url = 'https://www.liaoxuefeng.com' + x['href']
        # ⬇️教程html
        content = download_html(url, x.parent['depth'])
        # 往template添加新的教程
        new_div = template.new_tag('div', id=i)
        template.body.insert(3 + i, new_div)
        new_div.insert(3, content[0])
        new_div.insert(3, content[1])
        time.sleep(0.4)
    convert_pdf(template)
```


**参考文档**：
1. [Beautiful Soup 文档](https://beautifulsoup.readthedocs.io/zh_CN/latest/)
2. [HTML 转 PDF 之 wkhtmltopdf 工具精讲](https://www.jianshu.com/p/4d65857ffe5e)
