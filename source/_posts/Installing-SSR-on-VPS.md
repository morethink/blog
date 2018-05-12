---
title: 科学上网之在VPS上安装SSR
date: 2018-05-12
tags:
categories: 工具
---

本文对VPS安装SSR的过程进行总结，免得因为XX重新安装SSR时找不到安装方法。

# 安装SSR

简单的来说，如果你什么都不懂，那么你直接一路回车就可以了！
<!-- more -->
本脚本需要Linux root账户权限才能正常安装运行，所以 **如果不是 root账号，请先切换为root，如果是 root账号，那么请跳过**！

```
sudo su
```
输入上面代码回车后会提示你输入当前用户的密码，输入并回车后，没有报错就继续下面的步骤安装ShadowsocksR。

v2.0.0 版本以后的脚本，请先卸载旧脚本ShadowsocksR服务端，再重新安装！

```
wget -N --no-check-certificate https://softs.fun/Bash/ssr.sh && chmod +x ssr.sh && bash ssr.sh
```
备用下载地址（上面的链接无法下载，就用这个）：
```
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/ssr.sh && chmod +x ssr.sh && bash ssr.sh
```
下载运行后会提示你输入数字来选择要做什么。

运行脚本：

```
bash ssr.sh
```
输入对应的数字来执行相应的命令。
```

  请输入一个数字来选择菜单选项

 1. 安装 ShadowsocksR
 2. 更新 ShadowsocksR
 3. 卸载 ShadowsocksR
 4. 安装 libsodium(chacha20)
————————————
 5. 查看 账号信息
 6. 显示 连接信息
 7. 设置 用户配置
 8. 手动 修改配置
 9. 切换 端口模式
————————————
 10. 启动 ShadowsocksR
 11. 停止 ShadowsocksR
 12. 重启 ShadowsocksR
 13. 查看 ShadowsocksR 日志
————————————
 14. 其他功能
 15. 升级脚本

 当前状态: 已安装 并 已启动
 当前模式: 单端口

请输入数字(1-15)：
```

# 文件位置

- 安装目录：`/usr/local/shadowsocksr`
- 配置文件：`/etc/shadowsocksr/user-config.json`

# 设置为系统服务

ShadowsocksR 安装后，自动设置为 系统服务，所以支持使用服务来启动/停止等操作，同时支持开机启动。

- 启动 ShadowsocksR：`/etc/init.d/ssr start`
- 停止 ShadowsocksR：`/etc/init.d/ssr stop`
- 重启 ShadowsocksR：`/etc/init.d/ssr restart`
- 查看 ShadowsocksR状态：`/etc/init.d/ssr status`

ShadowsocksR 默认支持UDP转发，服务端无需任何设置。

# 定时重启

一些人可能需要定时重启ShadowsocksR服务端来保证稳定性等，所以这里用 crontab 定时。


```
# 输入这个命令可以查看当前配置的定时任务
crontab -l
# 如果提示命令不存在，下面是安装命令
# CentOS系统：
yum update
yum install -y crond
# Debian/Ubuntu系统：
apt-get update
apt-get install -y cron
```

安装 crontab 后，我们就能开始添加定时任务了：

```
crontab -l > "crontab.bak"
sed -i "/ssr restart/d" "crontab.bak"
echo -e "\n10 3 * * * /etc/init.d/ssr restart" >> "crontab.bak"
crontab "crontab.bak"
rm -r "crontab.bak"
```

下面是定时任务规则(代码前面的 * * * * * 分别对应：分钟 小时 日 月 星期)参考：

```
10 2 * * * /etc/init.d/ssr restart
# 这个代表 每天2点10分重启一次 ShadowsocksR

10 2 */2 * * /etc/init.d/ssr restart
# 这个代表 每隔2天的2点10分重启一次 ShadowsocksR

10 */4 * * * /etc/init.d/ssr restart
# 这个代表 每隔4小时的第10分重启一次 ShadowsocksR
```

# BBR和锐速

BBR和锐速都是用来提高翻墙速度的。

BBR是来自于Google的黑科技，目的是通过优化和控制TCP的拥塞，充分利用带宽并降低延迟，其目的就是要尽量跑满带宽，并且尽量不要有排队的情况。
BBR 这个特性其实是在 Linux 内核 4.9 才计划加入的。所以，要开启BBR，需要内核版本在Linux kernel 4.9以上，脚本会帮助我们安装。

在BBR之前，比较有名的就是国产的锐速了，不过，由于锐速是个国产的闭源软件，可能存在安全性问题，因此 **推荐使用安装BBR**。

- 启动脚本：
    ```
    bash ssr.sh
    ```
- 选择`14. 其他功能`
    ```
      1. 配置 BBR
      2. 配置 锐速(ServerSpeeder)
      3. 配置 LotServer(锐速母公司)
      注意： 锐速/LotServer/BBR 不支持 OpenVZ！
      注意： 锐速/LotServer/BBR 不能共存！
    ————————————
      4. 一键封禁 BT/PT/SPAM (iptables)
      5. 一键解封 BT/PT/SPAM (iptables)
      6. 切换 ShadowsocksR日志输出模式
      ——说明：SSR默认只输出错误日志，此项可切换为输出详细的访问日志
    ```
- 安装BBR或者锐速(推荐BBR)


# TCP优化
- 增加TCP连接数量
    ```
    vi /etc/security/limits.conf
    ```
- 添加两行：
    ```
    * soft nofile 51200
    * hard nofile 51200
    ```
- 设置ulimit：
    ```
    ulimit -n 51200
    ```

- 添加一些优化内容
    - 修改sysctl.conf
    ```
    vi  /etc/sysctl.conf
    ```
    - 插入代码：
    ```
    #TCP配置优化(不然你自己根本不知道你在干什么)
    fs.file-max = 51200
    #提高整个系统的文件限制
    net.core.rmem_max = 67108864
    net.core.wmem_max = 67108864
    net.core.netdev_max_backlog = 250000
    net.core.somaxconn = 4096
    net.ipv4.tcp_syncookies = 1
    net.ipv4.tcp_tw_reuse = 1
    net.ipv4.tcp_tw_recycle = 0
    net.ipv4.tcp_fin_timeout = 30
    net.ipv4.tcp_keepalive_time = 1200
    net.ipv4.ip_local_port_range = 10000 65000
    net.ipv4.tcp_max_syn_backlog = 8192
    net.ipv4.tcp_max_tw_buckets = 5000
    net.ipv4.tcp_fastopen = 3
    net.ipv4.tcp_mem = 25600 51200 102400
    net.ipv4.tcp_rmem = 4096 87380 67108864
    net.ipv4.tcp_wmem = 4096 65536 67108864
    net.ipv4.tcp_mtu_probing = 1
    net.ipv4.tcp_congestion_control = bbr
    #END OF LINE
    ```
    - 应用
    ```
    sysctl -p
    ```
    - 重启SSR
    ```
    /etc/init.d/ssr restart
    ```

**参考文档**：
1. [科学上网教程（一）——VPS上搭建SSR](https://jasper-1024.github.io/2016/06/26/VPS%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91%E6%95%99%E7%A8%8B%E7%B3%BB%E5%88%97/)
2. [『原创』CentOS/Debian/Ubuntu ShadowsocksR 单/多端口 一键管理脚本](https://doub.io/ss-jc42)
