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

#### Code Splitting
这一章节初步介绍了代码分离的三种方式
* Entry Points：在webpack.config.js中手动配置分开后的js
* Prevent Duplication:使用splitChunks将原本2个js都含有lodash的js分成三个js,将lodash单独分开成一个js,其余2个js不含有lodash并在其后引入
* Dynamic Imports:在真正需要的时候才开始import lodash.或者在支持promisepromise时使用 async functions.[^2]

Entry Points与Prevent Duplication很明显需要结合使用降低网络开销。
#### Lazy Loading

使用一个例子更加体现了再需要时才进行加载。打开控制台可以发现点击按钮式请求js,并出现输出。由于后续使用Vue,可以查看[Vue参考](https://alexjoverm.github.io/2017/07/16/Lazy-load-in-Vue-using-Webpack-s-code-splitting/)
#### Caching
在实际开发中当更改了js之后。希望js的名称改变以确保浏览器在下次时不会使用缓存。
此时可以在webpack.config.js中output配置contenthash来改变js名称

    output: {
        filename: '[name].[contenthash].js',
        path: path.resolve(__dirname, 'dist')
    }

但是一些我们没有更改的js,我们希望每次编译后不需要它更改文件名，这时就需要

    optimization: {
       runtimeChunk: 'single'
    }

> 在当前版本(4.16.5)时发现并没有此问题

对于更加不会更改的第三方库单独提取到另一个模块

    optimization: {
     runtimeChunk: 'single',
     splitChunks: {
       cacheGroups: {
         vendor: {
           test: /[\\/]node_modules[\\/]/,
           name: 'vendors',
           chunks: 'all'
         }
       }
     }
    }
此时npm run build

    Hash: 07c99b8741a34af74c15
    Version: webpack 4.16.5
    Time: 671ms
    Built at: 2018-08-16 14:04:57
                              Asset       Size  Chunks             Chunk Names
       main.08bc5d76a8323612728c.js  260 bytes       0  [emitted]  main
    vendors.20937c3d68263d1973c2.js   69.5 KiB       1  [emitted]  vendors
    runtime.01308d7cc7f8f5e719e6.js   1.42 KiB       2  [emitted]  runtime
                         index.html  353 bytes          [emitted]
    Entrypoint main = runtime.01308d7cc7f8f5e719e6.js vendors.20937c3d68263d1973c2.js main.08bc5d76a8323612728c.js
    [1] ./src/index.js 292 bytes {0} [built]
    [2] (webpack)/buildin/global.js 489 bytes {1} [built]
    [3] (webpack)/buildin/module.js 497 bytes {1} [built]

增加print.js后

    Hash: 1c6be846d72884f7fe06
    Version: webpack 4.16.5
    Time: 435ms
    Built at: 2018-08-16 14:06:54
                              Asset       Size  Chunks             Chunk Names
       main.37299f4ed831bda7ef44.js  327 bytes       0  [emitted]  main
    vendors.f15f56021940d8b8eb38.js   69.5 KiB       1  [emitted]  vendors
    runtime.01308d7cc7f8f5e719e6.js   1.42 KiB       2  [emitted]  runtime
                         index.html  353 bytes          [emitted]
    Entrypoint main = runtime.01308d7cc7f8f5e719e6.js vendors.f15f56021940d8b8eb38.js main.37299f4ed831bda7ef44.js
    [1] (webpack)/buildin/global.js 489 bytes {1} [built]
    [2] (webpack)/buildin/module.js 497 bytes {1} [built]
    [3] ./src/index.js + 1 modules 453 bytes {0} [built]
        | ./src/index.js 385 bytes [built]
        | ./src/print.js 63 bytes [built]
        + 1 hidden module

可以看出vendors其实是有变化的，但vendors是第三方库，我们其实是不想它进行变化的，所以增加一个插件

    plugins: [
      new CleanWebpackPlugin(['dist']),
      new HtmlWebpackPlugin({
        title: 'Caching'
      }),
      new webpack.HashedModuleIdsPlugin()
    ]
npm run build不能成功，猜测是在4.16.5版本中被移除或者是被替代未知？，在npm搜索中发现一个hashedmoduleidsplugin,npm i hashedmoduleidsplugin并改名HashedModuleIdsPlugin与hashedmoduleidsplugin均不能成功。原因未知


#### 参考文档
[^1]: [webpack官网Concepts](https://webpack.js.org/concepts/)
[^2]: [async function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function)