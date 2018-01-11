---
title: 查询在一张表不在另外一张表的记录及效率探究
date: 2017-11-16
tags: [MySQL]
categories: 数据库
---

在我做项目的时候遇到一个需求，要将存在于表ta而不存在于表tb中的数据查询出来。

记录使用的方法和探讨效率。

<!-- more -->

# 数据准备

创建表ta，并且使用存储过程插入13000条数据，在我的机器上运行时间: 346.719s。如果觉得插入的速度比较慢,可以直接导入我建好的表，百度云地址 http://pan.baidu.com/s/1dFtovg1 ，里面已经有数据了，直接导入sql执行即可，这样比用存储过程要快很多。

```sql
DROP TABLE IF EXISTS ta;

CREATE TABLE `ta` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=13000 DEFAULT CHARSET=utf8;

DROP PROCEDURE IF EXISTS ta_insert;

DELIMITER $$
CREATE PROCEDURE ta_insert()
 MODIFIES SQL DATA
 BEGIN
 SET @i=1;
 SET @max=13000;
 WHILE @i<@max DO
 INSERT INTO `ta` VALUES ();
 SET @i = @i + 1;
END WHILE;
end $$

CALL ta_insert();
```

创建表tb，并且使用存储过程插入10000条数据，在我的机器上运行时间:  224.102s。

```sql
DROP TABLE IF EXISTS tb;

CREATE TABLE `tb` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=8000 DEFAULT CHARSET=utf8;

DROP PROCEDURE IF EXISTS tb_insert;

DELIMITER $$
CREATE PROCEDURE tb_insert()
 MODIFIES SQL DATA
 BEGIN
 SET @i=1;
 SET @max=8000;
 WHILE @i<@max DO
 INSERT INTO `tb` VALUES ();
 SET @i = @i + 1;
END WHILE;
end $$

CALL tb_insert();
```

# 子查询

使用`NOT IN`，ta表中的每一个id值都要去与tb表中的id匹配，匹配到就停止，也就是说，存在于ta表而不存在于tb表的id值需要与所有tb表中的id值进行匹配。

执行子查询时，MYSQL需要创建临时表，查询完毕后再删除这些临时表，所以，子查询的速度会受到一定的影响，这里多了一个创建和销毁临时表的过程。

平均时间为 0.04s。
```sql
SELECT ta.id FROM ta WHERE ta.id IN (SELECT id FROM tb)
```

# 左连接

使用 `LEFT JOIN`，ta表左连接tb表，而存在于ta表不存在于tb表中的字段为`NULL`，于是我们可以通过判断`WHERE tb.id IS NULL`来找到存在于ta表而不存在于tb表的id值。

平均时间是 0.06s。
```sql
SELECT ta.id FROM ta LEFT JOIN tb ON ta.id = tb.id WHERE tb.id IS NULL
```
# 效率之谜

## 版本问题？
按理来说，连接应该比子查询要快，但是在我进行试验的时候发现却不是这样的，子查询居然还比连接要快。

![](https://images.morethink.cn/20ef641be3a9a9aa899e5e373e05eef9.png "疑惑")

搜索了解到
> 对于类似NOT IN这样的子查询，也能受益于subquery materialize，将子查询的结果集cache到临时表里，使用hashindex来进行检索；物化的子查询可以看到select_type字段为SUBQUERY，而在MySQL5.5里为DEPENDENT SUBQUERY

可能是版本原因，我用的是mysql5.7，可能做了优化。

于是使用mysql5.5再次测试。

发现子查询和左连接的查询时间都在0.12s附近，还是不能说明连接比子查询高效。进一步猜测，我的数据组织格式是否出现了问题，于是使用上面百度云盘连接中的mm_member表和mm_log表(这是 [mysql（4）—— 表连接查询与where后使用子查询的性能分析。](https://www.cnblogs.com/cdf-opensource-007/p/6540521.html) 提供的数据) 。

## 猜测验证

**子查询**

```sql
SELECT mm_member.id FROM mm_member WHERE mm_member.id NOT IN (SELECT DISTINCT mm_log.member_id FROM mm_log)
```

**左连接**

```sql
SELECT mm_member.id FROM mm_member LEFT JOIN (SELECT DISTINCT mm_log.member_id FROM mm_log ) AS mm
ON mm.member_id = mm_member.id WHERE mm.member_id IS NULL
```

**mysql5.7**

子查询为1.1s左右，左连接为1.55s，子查询依然速度较快。

**mysql5.5**

子查询为48s左右，左连接为1.4s，将近34倍的差距，由此印证上面引用的那部分，**mysql5.7确实已经多子查询做了优化，使其达到了逼近左连接的效率**。

**那为什么我自己所建立的表无法体现版本的这种性能差别？**

猜测应该是数据类型的原因，可能int类型的查询效率已经都优化好了。

# 网传最高效

不太清楚其中的原理，并且在我的测试中性能跟连接差不多。

```sql
SELECT id FROM ta WHERE (SELECT COUNT(1) AS num FROM tb WHERE ta.id = tb.id) = 0
```

> 而且网上流传的版本([(数据库篇) SQL查询~ 存在一个表而不在另一个表中的数据](http://blog.csdn.net/windren06/article/details/8188136))为
>
> select * from B where (select count(1) as num from A where A.ID = B.ID) = 0
> 应该是
> select * from A where (select count(1) as num from  B where A.ID = B.ID) = 0
> 大表在前，小表在后。


**注意**：
1. 存储过程循环插入比普通方式插入数据慢很多倍。
2. 索引可以有效提高搜索效率。
3. 不是所有的子查询都比连接慢的。


**参考文档**：
1. [MySQL 5.6的优化器改进](http://mysqllover.com/?p=919)
2. [mysql（4）—— 表连接查询与where后使用子查询的性能分析。](https://www.cnblogs.com/cdf-opensource-007/p/6540521.html)
