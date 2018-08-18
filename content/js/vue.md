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
##### 登录注册切换
将登录注册组件添加进主页，并添加一个action指定显示,点击相应按钮就设置不同的action值，然后确定显示登录还是注册

    <template>
      <div id="app" class="login-box">
        <Login v-if="action === 'login'"></Login>
        <Regis v-else></Regis>
        <div v-if="action === 'login'">
          <el-button type="primary">登录</el-button>
          <el-button type="primary" v-on:click="action='regis'">免费注册点这里</el-button>
        </div>
        <div v-else>
          <el-button type="primary">注册</el-button>
          <el-button type="primary" v-on:click="action='login'">已有账号，点击登录</el-button>
        </div>
      </div>
    </template>
    
    <script>
    import Login from './components/login.vue'
    import Regis from './components/regis.vue'
    
    export default {
      name: 'app',
      components: {
        Login,
        Regis
      },
      data: () => {
        return {
          action: 'login'
        }
      }
    }
    </script>
    
    <style>
    #app {
      font-family: 'Avenir', Helvetica, Arial, sans-serif;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
      text-align: center;
      color: #2c3e50;
      margin-top: 60px;
    }
    .login-box {
        margin: 0 auto;
        width: 290px;
        background: #ebf1f6;
        height: 360px;
        padding: 50px 35px 0px 35px;
    }
    </style>

在[v-if vs v-show](https://cn.vuejs.org/v2/guide/conditional.html#v-if-vs-v-show)进行v-if与v-show的说明。在本例中也只想进行登录注册的简单切换显示即可，所以改成v-show
即

    <Login v-show="action === 'login'"></Login>
    <Regis v-show="action === 'regis'"></Regis>
    <div v-show="action === 'login'">
      <el-button type="primary">登录</el-button>
      <el-button type="primary" v-on:click="action='regis'">免费注册点这里</el-button>
    </div>
    <div v-show="action === 'regis'">
      <el-button type="primary">注册</el-button>
      <el-button type="primary" v-on:click="action='login'">已有账号，点击登录</el-button>
    </div>
主页到此就能进行注册登录的切换了。但此时我想把登录注册按钮放进登录注册组件中，最开始也是这样放的但是提示没有action相关定义所以将其移动到App.vue中。在参考了[依赖注入](https://cn.vuejs.org/v2/guide/components-edge-cases.html#%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5)后发现在App.vue中增加provider后login.vue中能获取值，但是当改变值之后虽然也能使用 v-on:click="action='regis'"改变但是官方并不推荐这样做，并且提到了另一个工具[vuex](https://vuex.vuejs.org/zh/),显然登录注册组件需要共享action。在本例中state数据源对应action的值，view即登录注册两个2组件视图，actions即action的值不时页面需要进行不同的响应。但目前这样一个简单的单页应用不至于用它，所以打算改用[store 模式](https://cn.vuejs.org/v2/guide/state-management.html#%E7%AE%80%E5%8D%95%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86%E8%B5%B7%E6%AD%A5%E4%BD%BF%E7%94%A8),但正打算改用时又返回[依赖注入](https://cn.vuejs.org/v2/guide/components-edge-cases.html#%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5)所在页面中最先提到到的访问访问根实例，发现这个其实要简单许多。于是修改login.vue

    <template>
    <div>
        <div>登录</div>
        <div>
            <el-input placeholder="电话" prefix-icon="el-icon-phone" clearable> </el-input>
            <el-input type="password" placeholder="密码" clearable> </el-input>
        </div>
        <div>
            <el-button type="primary">登录</el-button>
            <el-button type="primary" v-on:click="this.$root.action='regis'">免费注册点这里</el-button>
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

之后删除App.vue中相关按钮发现页面没有任何按钮。在看文档下面有一个$parent，于是又将$root改为$parent，结果报错

    vue.runtime.esm.js?2b0e:587 [Vue warn]: Error in event handler for "click": "TypeError: Cannot read property '$parent' of null"
    ...

但看示例又是将$parent写入到function中于是又将其写入function中测试，最终通过

    <template>
    <div>
        <div>登录</div>
        <div>
            <el-input placeholder="电话" prefix-icon="el-icon-phone" clearable> </el-input>
            <el-input type="password" placeholder="密码" clearable> </el-input>
        </div>
        <div>
            <el-button type="primary">登录</el-button>
            <el-button type="primary" v-on:click="actionToRegis">免费注册点这里</el-button>
        </div>
    </div>
    </template>
    <script>
    export default {
      name: 'login',
      methods: {
          actionToRegis: function (e) {
              this.$parent.action = 'regis'
          }
      }
    }
    </script>
    <!-- Add "scoped" attribute to limit CSS to this component only -->
    <style scoped>
    </style>
regis.vue中也进行相应更改。由此也说明根组件并不是App.vue而是在main.js中new的组件
> 后来再看文档时发现[通过事件向父级组件发送消息](https://cn.vuejs.org/v2/guide/components.html#%E9%80%9A%E8%BF%87%E4%BA%8B%E4%BB%B6%E5%90%91%E7%88%B6%E7%BA%A7%E7%BB%84%E4%BB%B6%E5%8F%91%E9%80%81%E6%B6%88%E6%81%AF)更加合理即在login.vue中使用v-on:click="$emit('actionToRegis')"，在App.vue中加入 v-on:actionToRegis="action = 'regis'"，注册类似。所以多看官方文档很有必要，在实际做时才能真正体会到一些设计的作用
#### 后台交互
在之前项目中主要使用form提交与jquery封装的ajax提交，在vue中该如何把数据提交到后台呢?此时需要用到axios.

    npm install --save axios
由于demo项目与后台服务端口不一样，所以需要解决跨域问题。所以此时需要进行[代理配置](https://cli.vuejs.org/zh/config/#devserver-proxy)
增加vue.config.js文件(会自动加载)，加入devServe配置

    module.exports = {
        devServer : {
            proxy : 'http://localhost'
        }
    }
main.js加入一个测试请求

    import Vue from 'vue'
    import App from './App.vue'
    import './plugins/element.js'
    import axios from 'axios'
    
    Vue.config.productionTip = false
    
    new Vue({
      render: h => h(App)
    }).$mount('#app')
    
    axios.get('/user/login?name=1&password=2')
启动后台服务与前台项目，访问http://localhost:8080/时打开控制台查看network多了一个http://localhost:8080/user/login?name=1&password=2请求且正常返回表示代理配置成功。现在要注册组件中使用，这时就需要[添加实例属性](https://cn.vuejs.org/v2/cookbook/adding-instance-properties.html)
