---
title: "Vue"
date: 2018-08-17T09:54:47+08:00
draft: true
---

#### 前言
最初想使用vue+element UI搭建一个Demo项目以寻求能否解决已有项目的页面结构混乱、难以维护的痛点。在跟着vue、npm、webpack等官方文档学习了一些示例之后发现理解的还是很浅显，于是开始先搭建这个demo项目逐步加深理解.主要目标是

* 使用vue+element UI
* 实现登录、注册、系统主页布局、用户列表分页展示、增删改查用户、输入校验、日期搜索
* 前台项目与后台项目完全独立。后台使用spring boot、dubbo、mybatis
* 项目在前台编译好后能打包至后台相应目录

#### HelloWorld
##### 初始化项目
在之前的例子中使用的vue-cli是3.0以下版本，开始使用时element UI指出已为vue-cli3.0做好准备,于是参考[vue CLI 3文档](https://cli.vuejs.org/zh/guide/)构建HelloWorld项目。在使用过程中有一个体会就是vue的生态工具对开发者真的很友好。

    #初始化项目
    vue create hello-world
    #确认一些选项
    ...
    #增加element UI,过程也需进行选择，交互很友好
    vue add element
    #启动并确认页面是有一个element UI button
    npm run serve
##### 添加登录注册组件
初始化之后开始添加登录注册组件，由于主要是后端开发且才开始组件化前段编程，暂时不考虑组织结构，直接在已有的文件目录添加。
在components中添加注册组件regis.vue

    <template>
      <div>
          <div>注册</div>
          <div>
            <el-input
              placeholder="电话"
              prefix-icon="el-icon-phone"
              clearable>
            </el-input>
            <el-input
              type="password"
              placeholder="密码"
              clearable>
            </el-input>
            <el-input
              type="password"
              placeholder="重复密码"
              clearable>
            </el-input>
          </div>
      </div>
    </template>

    <script>
    export default {
      name: 'regis'
    }
    </script>
    
    <!-- Add "scoped" attribute to limit CSS to this component only -->
    <style scoped>
    
    </style>

登录组件login.vue

    <template>
      <div>
        <div>登录</div>
        <div>
          <el-input
            placeholder="电话"
            prefix-icon="el-icon-phone"
            clearable>
          </el-input>
          <el-input
            type="password"
            placeholder="密码"
            clearable>
          </el-input>
        </div>
      </div>
    </template>
    
    <script>
    export default {
      name: 'login'
    }
    </script>
    
    <!-- Add "scoped" attribute to limit CSS to this component only -->
    <style scoped>
    
    </style>
> 很明显，注册组件就比登录组件多个重复密码输入框，应该是可以复用的，这里以后再考虑