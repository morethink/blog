---
title: Redis和Memcached区别
date: 2017-12-02
tags: [Redis,Memcached]
categories: 数据库
---

本文参考 [Redis与Memcached的区别](http://blog.51cto.com/gnucto/998509)。

如果简单地比较Redis与Memcached的区别，大多数都会得到以下观点：

1. Redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
2. Redis支持数据的备份，即master-slave模式的数据备份。
3. Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。

抛开这些，可以深入到Redis内部构造去观察更加本质的区别，理解Redis的设计。

<!-- more -->

# 网络IO模型

## Memcached网络IO模型

Memcached是 **多线程非阻塞IO复用** 的网络模型，分为监听主线程和worker子线程，监听线程监听网络连接，接受请求后，将连接描述字pipe传递给worker线程，进行读写IO, 网络层使用libevent封装的事件库，多线程模型可以发挥多核作用，但是引入了cache coherency和锁的问题，比如，Memcached最常用的stats 命令，实际Memcached所有操作都要对这个全局变量加锁，进行计数等工作，带来了性能损耗。

![](https://images.morethink.cn/0dcb499bd022cea2069865875decc2b7.png "Memcached网络IO模型")

## Redis网络IO模型

Redis使用 **单线程非阻塞IO复用** 模型，自己封装了一个简单的AeEvent事件处理框架，主要实现了epoll、kqueue和select，对于单纯只有IO操作来说，单线程可以将速度优势发挥到最大，但是 **Redis也提供了一些简单的计算功能，比如排序、聚合等，对于这些操作，单线程模型实际会严重影响整体吞吐量，CPU计算过程中，整个IO调度都是被阻塞住的**。

# 内存管理方面

- Memcached使用预分配的内存池的方式，使用slab和大小不同的chunk来管理内存，Item根据大小选择合适的chunk存储，内存池的方式可以省去申请/释放内存的开销，并且能减小内存碎片产生，但这种方式也会带来一定程度上的空间浪费，并且在内存仍然有很大空间时，新的数据也可能会被剔除，原因可以参考Timyang的文章： http://timyang.net/data/Memcached-lru-evictions/
- Redis使用现场申请内存的方式来存储数据，并且很少使用free-list等方式来优化内存分配，会在一定程度上存在内存碎片，Redis跟据存储命令参数，会把带过期时间的数据单独存放在一起，并把它们称为临时数据，非临时数据是永远不会被剔除的，即便物理内存不够，导致swap也不会剔除任何非临时数据(但会尝试剔除部分临时数据)，这点上 **Redis更适合作为存储而不是cache**。


# 数据一致性问题

- Memcached提供了cas命令，可以保证多个并发访问操作同一份数据的一致性问题。
- Redis没有提供cas命令，并不能保证这点，不过Redis提供了事务的功能，可以保证一串命令的原子性，中间不会被任何操作打断。

# 集群管理的不同

Memcached是全内存的数据缓冲系统，Redis虽然支持数据的持久化，但是全内存毕竟才是其高性能的本质。作为基于内存的存储系统来说，机器物理内存的大小就是系统能够容纳的最大数据量。如果需要处理的数据量超过了单台机器的物理内存大小，就需要构建分布式集群来扩展存储能力。

## 分布式Memcached

**Memcached本身并不支持分布式**，因此只能在客户端通过像一致性哈希这样的分布式算法来实现Memcached的分布式存储。下图给出了Memcached的分布式存储实现架构。当客户端向Memcached集群发送数据之前，首先会通过内置的分布式算法计算出该条数据的目标节点，然后数据会直接发送到该节点上存储。但客户端查询数据时，同样要计算出查询数据所在的节点，然后直接向该节点发送查询请求以获取数据。

![](https://images.morethink.cn/b4dc233573a34d272d52a270ff01d3f8.png "分布式Memcached")

## Redis Cluster
相较于Memcached只能采用客户端实现分布式存储，Redis更偏向于在服务器端构建分布式存储。最新版本的Redis已经支持了分布式存储功能。Redis Cluster是一个实现了分布式且允许单点故障的Redis高级版本，它没有中心节点，具有线性可伸缩的功能。下图给出Redis Cluster的分布式存储架构，其中节点与节点之间通过二进制协议进行通信，节点与客户端之间通过ascii协议进行通信。在数据的放置策略上，Redis Cluster将整个key的数值域分成4096个哈希槽，每个节点上可以存储一个或多个哈希槽，也就是说当前Redis Cluster支持的最大节点数就是4096。Redis Cluster使用的分布式算法也很简单：`crc16( key ) %HASH_SLOTS_NUMBER`。

![](https://images.morethink.cn/ecb156696a22333a19ea5647b7eaefcc.png "Redis-Cluster")

为了保证单点故障下的数据可用性，Redis Cluster引入了Master节点和Slave节点。在Redis Cluster中，每个Master节点都会有对应的两个用于冗余的Slave节点。这样在整个集群中，任意两个节点的宕机都不会导致数据的不可用。当Master节点退出后，集群会自动选择一个Slave节点成为新的Master节点。

![](https://images.morethink.cn/c7b06587ee2b47925a540cc1c10ca44f.png "Redis-Cluster-2")

# 存储方式及其它方面

- Memcached基本只支持简单的key-value存储，不支持枚举，不支持持久化和复制等功能
- Redis除key/value之外，还支持list,set,sorted set,hash等众多数据结构，提供了KEYS进行枚举操作，但不能在线上使用，如果需要枚举线上数据，Redis提供了工具可以直接扫描其dump文件，枚举出所有数据，Redis还同时提供了持久化和复制等功能。

**根据以上比较不难看出，当我们不希望数据被踢出，或者需要除key/value之外的更多数据类型时，或者需要落地功能时，使用Redis比使用Memcached更合适**。

# 单线程的Redis为什么这么高效

##  单线程模型

Redis客户端对服务端的每次调用都经历了发送命令，执行命令，返回结果三个过程。其中执行命令阶段，由于Redis是单线程来处理命令的，所有每一条到达服务端的命令不会立刻执行，所有的命令都会进入一个队列中，然后逐个被执行。并且多个客户端发送的命令的执行顺序是不确定的。但是可以确定的是不会有两条命令被同时执行，不会产生并发问题，这就是Redis的单线程基本模型。

## 单线程模型每秒万级别处理能力的原因

1. 纯内存访问。数据存放在内存中，内存的响应时间大约是100纳秒，这是Redis每秒万亿级别访问的重要基础。
2. 非阻塞I/O，Redis采用epoll做为I/O多路复用技术的实现，再加上Redis自身的事件处理模型将epoll中的连接，读写，关闭都转换为了时间，不在I/O上浪费过多的时间。
3. 单线程避免了线程切换和竞态产生的消耗。
4. Redis采用单线程模型，每条命令执行如果占用大量时间，会造成其他线程阻塞，对于Redis这种高性能服务是致命的，所以Redis是面向高速执行的数据库。


**总结**：
　　1. Redis使用最佳方式是全部数据in-memory。
　　2. Redis更多场景是作为Memcached的替代者来使用。
　　3. 当需要除key/value之外的更多数据类型支持时，使用Redis更合适。
　　4. 当存储的数据不能被剔除时，使用Redis更合适。
　　5. 需要分布式部署时，使用Redis更合适。


**参考文档**：
1. [Redis与Memcached的区别](http://blog.51cto.com/gnucto/998509)
2. [Redis和Memcached的区别](https://www.biaodianfu.com/redis-vs-memcached.html?spm=5176.100239.blogcont238409.17.w1SLGA)
