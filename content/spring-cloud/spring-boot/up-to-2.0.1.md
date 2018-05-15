---
title: "Up to 2.0"
date: 2018-04-25T19:03:55+08:00
draft: true
---

将原项目spring-boot升级到2.0.1之后，许多相关版本也更新至最新版

druid-spring-boot-starter升至1.1.9,
报错
	Caused by: java.lang.ClassNotFoundException: com.alibaba.druid.filter.logging.Log4j2Filter
将druid升级至1.1.9
报错
java.lang.NoSuchMethodError: com.alibaba.druid.sql.ast.expr.SQLAggregateExpr.getOption()Lcom/alibaba/druid/sql/ast/expr/SQLAggregateExpr$Option;
	at com.dangdang.ddframe.rdb.sharding.parser.visitor.basic.mysql.MySQLSelectVisitor.visit(MySQLSelectVisitor.java:110)
	at com.alibaba.druid.sql.visitor.SQLASTOutputVisitor.printExpr(SQLASTOutputVisitor.java:886)
由于Sharding-JDBC1.4.1依赖的druid被替换造成不兼容，升级Sharding-JDBC版本至2.0.1时发现和项目已使用的分库分表代码不兼容的地方较多，遂升级成1.5.4.1。发现该版本不支持or条件，然后项目中大量使用了or故放弃。
因为Sharding-JDBC1.4.1依赖的druid版本为1.0.9，而druid-spring-boot-starter最低依赖版本为1.1.0故舍弃使用改为自定义dataSource配置.后来又使用druid-spring-boot-starter的最低版本暂无问题。

启动的时候发布广播事件时一直提示log4j找不到，但项目已经改为使用logback了，感觉很奇怪

