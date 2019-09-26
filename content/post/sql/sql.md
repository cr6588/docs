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