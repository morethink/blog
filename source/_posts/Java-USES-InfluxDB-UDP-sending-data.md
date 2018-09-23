---
title: Java使用UDP发送数据到InfluxDB
date: 2018-09-23
tags:
    - InfluxDB
    - UDP
categories: 数据库
---

最近在做压测引擎相关的开发，需要将聚合数据发送到InfluxDB保存以便实时分析和控制QPS。

下面介绍对InfluxDB的使用。

# 什么是InfluxDB

InfluxDB是一款用Go语言编写的开源分布式时序、事件和指标数据库，无需外部依赖。该数据库现在主要用于存储涉及大量的时间戳数据，如DevOps监控数据，APP metrics, loT传感器数据和实时分析数据。

<!-- more -->

InfluxDB特征：

- 无结构(无模式)：可以是任意数量的列(tags)。
- 可以设置metric的保存时间。
- 支持与时间有关的相关函数(如min、max、sum、count、mean、median等)，方便统计。
- 支持存储策略：可以用于数据的删改(influxDB没有提供数据的删除与修改方法)。
- 支持连续查询：是数据库中自动定时启动的一组语句，和存储策略搭配可以降低InfluxDB的系统占用量。
- 原生的HTTP支持，内置HTTP API。
- 支持类似SQL语法。
- 支持设置数据在集群中的副本数。
- 支持定期采样数据，写入另外的measurement，方便分粒度存储数据。
- 自带web管理界面，方便使用(登入方式：http://< InfluxDB-IP >:8083)。
- 支持Grafana画图展示。

PS：有了InfluxDB+Grafana后，你就可以写一些简单的程序了，可以只负责写后端逻辑部分，数据都可以存入InfluxDB，然后通过Grafana展示出来。

# Mac安装InfluxDB

```shell
# 安装
brew install influxdb
# 启动
influxd -config /usr/local/etc/influxdb.conf
# 查看influxdb运行配置
influxd config
# 启动客户端
influx -precision rfc3339
```

# InfluxDB开启UDP配置

```shell
vim /usr/local/etc/influxdb.conf
```
开启udp配置，其他为默认值
```shell
[[udp]]
  enabled = true
```
udp配置含义：
```shell
[[udp]] – udp配置

    enabled：是否启用该模块，默认值：false。

    bind-address：绑定地址，默认值：”:8089″。

    database：数据库名称，默认值：”udp”。

    retention-policy：存储策略，无默认值。

    batch-size：默认值：5000。

    batch-pending：默认值：10。

    read-buffer：udp读取buffer的大小，0表示使用操作系统提供的值，如果超过操作系统的默认配置则会出错。 该配置的默认值：0。

    batch-timeout：超时时间，默认值：”1s”。

    precision：时间精度，无默认值。
```

# Java发送UDP数据报

我们知道InfluxDB是支持Http的，为什么我们还要采用UDP方式发送数据呢？

基于下列原因：
1. TCP数据传输慢，UDP数据传输快。
2. 网络带宽需求较小，而实时性要求高。
3. InfluxDB和服务器在同机房，发生数据丢包的可能性较小，即使真的发生丢包，对整个请求流量的收集影响也较小。


我们采用了worker线程调用`addMetric`方法将数据存储到缓存 `map` 中，send线程池来进行每个指定时间发送数据到Influxdb。

代码如下(也可参考`Jmeter`的`UdpMetricsSender`类)：
```java
@Slf4j
public class InfluxDBClient implements Runnable {
    private String measurement = "example";

    private final Object lock = new Object();

    private InetAddress hostAddress;

    private int udpPort;

    private volatile Map<String, List<Response>> metrics = new HashMap<>();

    private long time;

    private String transaction;

    public InfluxDBClient(String influxdbUrl, String transaction) {
        this.transaction = transaction;
        try {
            log.debug("Setting up with url:{}", influxdbUrl);
            String[] urlComponents = influxdbUrl.split(":");
            if (urlComponents.length == 2) {
                hostAddress = InetAddress.getByName(urlComponents[0]);
                udpPort = Integer.parseInt(urlComponents[1]);
            } else {
                throw new IllegalArgumentException("InfluxDBClient url '" + influxdbUrl + "' is wrong. The format shoule be <host/ip>:<port>");
            }
        } catch (Exception e) {
            throw new IllegalArgumentException("InfluxDBClient url '" + influxdbUrl + "' is wrong. The format shoule be <host/ip>:<port>", e);
        }
    }

    public void addMetric(Response response) {
        synchronized (lock) {
            if (metrics.containsKey(response.getLabel())) {
                metrics.get(response.getLabel()).add(response);
            } else {
                metrics.put(response.getLabel(), new ArrayList<>(Collections.singletonList(response)));
            }
        }
    }

    @Override
    public void run() {
        sendMetrics();
    }

    private void sendMetrics() {
        Map<String, List<Response>> tempMetrics;
        //复制数据到tempMetrics，清空原来metrics并初始化上次的大小
        synchronized (lock) {
            if (isEmpty(metrics)) {
                return;
            }
            time = System.currentTimeMillis();
            tempMetrics = metrics;
            metrics = new HashMap<>();
            for (Map.Entry<String, List<Response>> entry : tempMetrics.entrySet()) {
                metrics.put(entry.getKey(), new ArrayList<>(entry.getValue().size()));
            }
        }
        final Map<String, List<Response>> copyMetrics = tempMetrics;
        final List<MetricTuple> aggregateMetrics = aggregate(copyMetrics);
        StringBuilder sb = new StringBuilder(aggregateMetrics.size() * 200);
        //发送tempMetrics,生成一行数据，然后换行
        for (MetricTuple metric : aggregateMetrics) {
            sb.append(metric.getMeasurement()).append(metric.getTag()).append(" ")
                    .append(metric.getField()).append(" ").append(metric.getTimestamp() + "000000").append("\n");
        }
        //udp发送数据到Influxdb
        try (DatagramSocket ds = new DatagramSocket()) {
            byte[] buf = sb.toString().getBytes();
            DatagramPacket dp = new DatagramPacket(buf, buf.length, this.hostAddress, this.udpPort);
            ds.send(dp);
            log.debug("send {} to influxdb", sb.toString());
        } catch (SocketException e) {
            log.error("Cannot open udp port!", e);
        } catch (IOException e) {
            log.error("Error in transferring udp package", e);
        }
    }

    /**
     * 得到聚合数据
     *
     * @param metrics
     * @return
     */
    private List<MetricTuple> aggregate(Map<String, List<Response>> metrics) {

    }

    public boolean isEmpty(Map<String, List<Response>> map) {
        for (Map.Entry<String, List<Response>> entry : map.entrySet()) {
            if (!entry.getValue().isEmpty()) {
                return false;
            }
        }
        return true;
    }
}
```


**参考文档**：
1. [InfluxDB中文文档](https://jasper-zhang1.gitbooks.io/influxdb/content/)
2. [玩转时序数据库InfluxDB](http://www.ywnds.com/?p=10763)
