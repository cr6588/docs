---
title: "Sql"
date: 2019-09-26T15:51:48+08:00
draft: true
---

mysql update where 不能用子查询
eg:
````sql
UPDATE pub_small_tool_50845485713523 a
SET a.newest = 1
WHERE
    a.type = 1
AND (
    a.shopId,
    a.itemId,
    a.buyerAccount,
    a.expireTime
) IN (
    SELECT
        shopId,
        itemId,
        buyerAccount,
        MAX(expireTime) expireTime
    FROM
        pub_small_tool_50845485713523
    WHERE
        type = 1
    GROUP BY
        shopId,
        itemId,
        buyerAccount
)
````
改成右连接
````sql
UPDATE pub_small_tool_50845485713523 a
RIGHT JOIN (
    SELECT
        shopId,
        itemId,
        buyerAccount,
        MAX(expireTime) expireTime
    FROM
        pub_small_tool_50845485713523
    WHERE
        type = 1
    GROUP BY
        shopId,
        itemId,
        buyerAccount
) A ON (
    A.shopId = a.shopId
    AND A.itemId = a.itemId
    AND A.buyerAccount = a.buyerAccount
    AND A.expireTime = a.expireTime
)
SET a.newest = 1
WHERE
    a.type = 1;
````
MySQL Explain详解
概要描述：
id:选择标识符
select_type:表示查询的类型。
table:输出结果集的表
partitions:匹配的分区
type:表示表的连接类型
possible_keys:表示查询时，可能使用的索引
key:表示实际使用的索引
key_len:索引字段的长度
ref:列与索引的比较
rows:扫描出的行数(估算的行数)
filtered:按表条件过滤的行百分比
Extra:执行情况的描述和说明