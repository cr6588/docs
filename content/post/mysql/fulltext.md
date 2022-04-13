---
title: "Fulltext"
date: 2022-04-10T18:14:07+08:00
draft: true
---

# 全文索引
原本是想利用全文索引解决简单的查询优化，但使用了之后发现一些问题。
## 配置问题
全文索引都会涉及到一个最小分词长度长度的配置；
````
#在mysql查询配置
SHOW VARIABLES LIKE '%ft%';
#myism最小分词长度
ft_min_word_len
#innodb最小分词长度
innodb_ft_min_token_size

#查询中文分词引擎ngram的分词长度
SHOW VARIABLES LIKE '%ngram%';
ngram_token_size
````
如果都设置为2以及大于2，那么只有1个字符的模糊处理如何做？设为1，分词会使数据量太多?

## sql问题
默认的创建全文索引
````sql
ALTER TABLE `user` ADD FULLTEXT INDEX `username`(`username`);
````

含有中文的分词引擎需要使用ngram
````sql
ALTER TABLE `user` ADD FULLTEXT INDEX `name`(`name`) with parser ngram;
````

在常见情况下都是用户对一些字段输入一个搜索值进行联合查询，例如名字是多少，用户名是多少，反应sql上就会是
````sql
SELECT * from USER where match(name) against('xxx' in boolean mode) and match(username ) against('xxx' in boolean mode) limit 10;
````
不用自然语言模式(IN NATURAL LANGUAGE MODE)而是布尔模式（in boolean mode）

但是匹配的结果并不是预想的类似模糊匹配的效果
![x](/images/mysql/ft1.png)
![x](/images/mysql/ft2.png)

匹配的是只含有wx的结果,但是like模糊结果时是
![x](/images/mysql/ft3.png)
因此全文索引返回的到底是一个什么结果？
查看5.7的官方文档https://dev.mysql.com/doc/refman/5.7/en/fulltext-search.html
提到在自然语言模式会受到一些停用词影响，但在布尔模式中不会受影响，但其结果仍然与like相差甚远
![x](/images/mysql/ft4.png)