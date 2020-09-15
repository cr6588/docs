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
#### 本地安装jar以及源码

mvn install:install-file -Dfile=jar包的位置 -DgroupId=上面的groupId -DartifactId=上面的artifactId -Dversion=上面的version -Dpackaging=jar

mvn install:install-file -Dfile=jar包源码的位置 -DgroupId=上面的groupId -DartifactId=上面的artifactId -Dversion=上面的version -Dpackaging=jar -Dclassifier=sources