---
title: IDEA远程调试
date: 2021-06-06
tags: [IDEA]
categories: Java
---

在工作中经常会因为测试环境代码不及预期或者产生bug需要查看哪个步骤出现了问题，这个时候IDEA 远程debug就派上用场了。只要本地有运行的源代码就可以进行调试。

![debug)](https://kinsta.com/wp-content/uploads/2020/04/wordpress-debug.png)

<!-- more -->

## 查看调试端口


1. 登陆远程服务器。
2. 执行shell命令 ` ps -ef|grep java|grep address`，查看对应的debug端口号。
3. 如果已存在则可以直接在IDEA中设置远程调试。
4. 不存在可以在Java程序启动时 加上 调试参数，如下：
```java
 java -server -Xms512m -Xmx512m -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5555 -Djava.ext.dirs=. ${main_class}
```
其中 `address` 为调试端口号。

## 如何启动调试

1. 添加 `Remote` 配置：选择对于源码项目，填写需要debug的 `IP` 和 `address`，如图：



![](https://images.morethink.cn/a1647e77708587ecd46a4a929a1d87c8.png)

2. 点击启动调试

![](https://images.morethink.cn/c010637fc481b7a4e3e3b88d4b19bf26.png)



## 调试参数设置



右键断点可以看到`Suspend`和`Condition`两个关键字：

![dd](https://images.morethink.cn/78eac97d87a8cd0175121830099bcc85.png)

### `Suspend`-不阻塞其他线程

Suspend是断点暂停的生效范围。

- Suspend：未勾选，程序运行到断点处并不会阻塞，而会继续执行后面的逻辑。

- Suspend：勾选(是默认值)，代表程序运行到断点处会阻塞。
  - All(默认值)：代表断点会阻塞所有线程。所以才会出现 `Stop The World` 的可怕情况。
  - Thread：勾选，**代表断点只会阻塞当前线程。避免远程debug时多人同时访问应用导致的冲突问题**。因此，在多线程调试时，若你希望不阻塞程序，最好选择 Thread 当前线程阻塞策略，这样就不会影响到其他线程的工作。

### `Condition`-不阻塞其他请求

通过设置上面的 `Suspend`参数为 `Thread` 可以不停止整个程序(`Stop The World`)，但是被调试代码存在被其他人调用的情况，这样就会干扰我们的调试，有没有更加细粒度的调试？

![](https://images.morethink.cn/b720f9a71364c0131661c68c50baaff1.png)

有，答案就是 `condition ` ，如上图，通过 设置 `content="1"` 这样只有满足条件的请求会被断点。





### EVAL-查看&修改值

1. 当我们在调试的时候，如果查看某一个值，可以通过计算表达式，按快捷键 `ALT + F8` 就可以打开 `Evaluate Expression` ，如图：

![](https://images.morethink.cn/4c07ce20314df54c51c73309f84fa63d.png)

2. 修改一个值，同样可以通过 `EVAL表达式`，如图：

![](https://images.morethink.cn/eb3368fc0f28b69a6868ac84de1a626f.png)



参考文档：

1. [idea 高级调试技巧](https://www.cnblogs.com/yjmyzz/p/idea-advanced-debug-tips.html)
