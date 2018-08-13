---
title: 前端框架 Vue.js
date: 2018-03-22 09:22:40
categories: FrontEnd
tags: [前端，vue, FrontEnd]
---

作为一个后端开发人员是需要了解一些前端框架，以便遇到前端问题的时候知道如何解决.

<!-- more -->

# 安装使用

## 独立版本

我们可以在 Vue.js 的官网上直接下载 vue.min.js 并用 `<script>` 标签引入

## 使用 `CDN` 方法

- BootCDN（国内） : https://cdn.bootcss.com/vue/2.2.2/vue.min.js
- unpkg：https://unpkg.com/vue/dist/vue.js, 会保持和 npm 发布的最新的版本一致。
- cdnjs : https://cdnjs.cloudflare.com/ajax/libs/vue/2.1.8/vue.min.js


## `NPM` 方法

```php
    npm install vue
```

## 命令行工具

```php
    # 全局安装 vue-cli
    $ cnpm install --global vue-cli
    # 创建一个基于 webpack 模板的新项目
    $ vue init webpack my-project
    # 这里需要进行一些配置，默认回车即可
```

进入项目，安装并运行：

```php
    $ cd my-project
    $ cnpm install
    $ cnpm run dev
```

> Listening at http://localhost:8080


# 目录结构

|目录/文件|说明|
|:----|:----|
|build|项目构建(webpack)相关代码|
|config|配置目录，包括端口号等。我们初学可以使用默认的。|
|node_modules|npm 加载的项目依赖模块|
|src|这里是我们要开发的目录，基本上要做的事情都在这个目录里。里面包含了几个目录及文件：|
|static|静态资源目录，如图片、字体等。|
|test|初始测试目录，可删除|
|.xxxx文件|这些是一些配置文件，包括语法配置，git配置等。|
|index.html|首页入口文件，你可以添加一些 meta 信息或统计代码啥的。|
|package.json|项目配置文件。|
|README.md|项目的说明文档，markdown 格式|


- src
   - + assets: 放置一些图片，如logo等。
   - + components: 目录里面放了一个组件文件，可以不用。
   - + App.vue: 项目入口文件，我们也可以直接将组件写这里，而不使用 components 目录。
   - + App.vue: 项目入口文件，我们也可以直接将组件写这里，而不使用 components 目录。



# 模板语法

## HTML

```php
    <div id="app">
        <div v-html="message"></div>
    </div>
        
    <script>
    new Vue({
      el: '#app',
      data: {
        message: '<h1>菜鸟教程</h1>'
      }
    })
    </script>
```


# 路由

## 安装

### 直接下载 / CDN

```php
    https://unpkg.com/vue-router/dist/vue-router.js
```

### NPM

```php
    cnpm install vue-router
```

## 简单实例

### HTML代码

```php
    <script src="https://unpkg.com/vue/dist/vue.js"></script>
    <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
     
    <div id="app">
      <h1>Hello App!</h1>
      <p>
        <!-- 使用 router-link 组件来导航. -->
        <!-- 通过传入 `to` 属性指定链接. -->
        <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
        <router-link to="/foo">Go to Foo</router-link>
        <router-link to="/bar">Go to Bar</router-link>
      </p>
      <!-- 路由出口 -->
      <!-- 路由匹配到的组件将渲染在这里 -->
      <router-view></router-view>
    </div>
```


### JavaScript 代码

```php
    // 0. 如果使用模块化机制编程，導入Vue和VueRouter，要调用 Vue.use(VueRouter)
 
    // 1. 定义（路由）组件。
    // 可以从其他文件 import 进来
    const Foo = { template: '<div>foo</div>' }
    const Bar = { template: '<div>bar</div>' }
     
    // 2. 定义路由
    // 每个路由应该映射一个组件。 其中"component" 可以是
    // 通过 Vue.extend() 创建的组件构造器，
    // 或者，只是一个组件配置对象。
    // 我们晚点再讨论嵌套路由。
    const routes = [
      { path: '/foo', component: Foo },
      { path: '/bar', component: Bar }
    ]
     
    // 3. 创建 router 实例，然后传 `routes` 配置
    // 你还可以传别的配置参数, 不过先这么简单着吧。
    const router = new VueRouter({
      routes // （缩写）相当于 routes: routes
    })
     
    // 4. 创建和挂载根实例。
    // 记得要通过 router 配置参数注入路由，
    // 从而让整个应用都有路由功能
    const app = new Vue({
      router
    }).$mount('#app')
     
    // 现在，应用已经启动了！
```

## NPM 路由实例

在 Github 上下载：https://github.com/chrisvfritz/vue-2.0-simple-routing-example

下载完后，解压该目录，重命名目录为 vue-demo，vu 并进入该目录，执行以下命令：

```php
    # 安装依赖，使用淘宝资源命令 cnpm
    cnpm install

    # 启动应用，地址为 localhost:8080
    cnpm run dev
```

如果你需要发布到正式环境可以执行以下命令：

```php
    cnpm run build
```


# 参考

- [Vue.js教程](http://www.runoob.com/vue2/vue-tutorial.html)










