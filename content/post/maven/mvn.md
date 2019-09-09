---
title: "Mvn"
date: 2019-08-05T11:42:46+08:00
draft: true
---

#### 多核编译
````shell
mvn -T 1C clean install -Dmaven.compile.fork=true
-T,--threads <arg>                     Thread count, for instance 2.0C
                                        where C is core multiplied
````