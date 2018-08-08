---
title: "Webpack"
date: 2018-08-08T10:38:04+08:00
draft: true
---

#### 概念[^1]

> At its core, webpack is a static module bundler for modern JavaScript applications. When webpack processes your application, it internally builds a dependency graph which maps every module your project needs and generates one or more bundles.

#### 作用

在[getting-started](https://webpack.js.org/guides/getting-started/)说得很清楚
传统的html与静态文件结构类似
##### project

      webpack-demo
      |- package.json
    + |- index.html
    + |- /src
    +   |- index.js

##### src/index.js

    function component() {
      let element = document.createElement('div');
    
      // Lodash, currently included via a script, is required for this line to work
      element.innerHTML = _.join(['Hello', 'webpack'], ' ');
    
      return element;
    }
    
    document.body.appendChild(component());

##### index.html

    <!doctype html>
    <html>
      <head>
        <title>Getting Started</title>
        <script src="https://unpkg.com/lodash@4.16.6"></script>
      </head>
      <body>
        <script src="./src/index.js"></script>
      </body>
    </html>

##### 带来的问题
> * It is not immediately apparent that the script depends on an external library.
> * If a dependency is missing, or included in the wrong order, the application will not function properly.
> * If a dependency is included but not used, the browser will be forced to download unnecessary code.

即

* 脚本中并未明确依赖外部库。（在index.js中找不到对<span>https://unpkg.com/lodash@4.16.6<span>的依赖）
* 如果依赖想缺失或引用顺序错误页面不能正常运行。（在index.html中缺失<span>https://unpkg.com/lodash@4.16.6<span>或者其在index.js之后，index.html会错误）
* 如果引入的依赖项不使用，浏览器会强制下载不必要的依赖

#### 参考文档
[^1]: [webpack官网Concepts](https://webpack.js.org/concepts/)

