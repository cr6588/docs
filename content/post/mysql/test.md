---
title: "mysql性能测试"
date: 2021-02-07T11:08:39+08:00
draft: false
---
# mysql性能测试

参照https://cloud.tencent.com/developer/article/1536046

通过我们看各大厂商提供的指标，我们不难发现，主要是 4 个指标：

TPS ：Transactions Per Second ，即数据库每秒执行的事务数，以 commit 成功次数为准。
QPS ：Queries Per Second ，即数据库每秒执行的 SQL 数（含 insert、select、update、delete 等）。
RT ：Response Time ，响应时间。包括平均响应时间、最小响应时间、最大响应时间、每个响应时间的查询占比。比较需要重点关注的是，前 95-99% 的最大响应时间。因为它决定了大多数情况下的短板。
Concurrency Threads ：并发量，每秒可处理的查询请求的数量。
如果对基准测试不是很理解的胖友，可以看下 《详解 MySQL 基准测试和 sysbench 工具》 的第一部分基准测试简介。

总结来说，实际就是 2 个维度：

吞吐量
延迟

安装sysbench

添加数据库sbtest

准备测试数据
cd /usr/share/sysbench
 sysbench oltp_common.lua --time=300 --mysql-host=127.0.0.1 --mysql-port=31111 --mysql-user=root --mysql-password=1231111 --mysql-db=sbtest --table-size=1000000 --tables=10 --threads=32 --events=999999999   prepare
 
 测试
sysbench oltp_read_write.lua --time=300 --mysql-host=127.0.0.1 --mysql-port=31111 --mysql-user=root --mysql-password=1231111 --mysql-db=sbtest --table-size=1000000 --tables=10 --threads=32 --events=999999999  --report-interval=10  run
