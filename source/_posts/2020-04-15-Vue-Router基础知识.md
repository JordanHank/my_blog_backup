---
title: Vue-Router基础知识
date: 2020-04-15 21:17:03
categories: blog
toc: true
tags:
  - Vue
  - Vue-Router
  - 基础知识
---

使用vue 进行前端项目的时候，一般都是通过vue-cli 搭建的脚手架单页应用项目，这种页单应用项目，是通过组合组件来组成应用程序的，如何将组件(components) 映射到路由 (routes)中，进行方便快速的切换呢？官方推荐的就是——使用Vue Router!

<!--more-->

### 什么是Vue-Router？

生活中为了指明道路方向，一般都会在道路转弯处设置导航标示或者路牌。比如这样

![路牌图片](/assets/img/guide.jpg)

其实vue-router 也可以理解成为页面路由的导航，这个导航就是用来知名跳转到哪个页面用的。官方解释是这样的：Vue Router 是 [Vue.js](http://cn.vuejs.org/) 官方的路由管理器。它和 Vue.js 的核心深度集成，让构建单页面应用变得易如反掌。包含的功能有：

- 嵌套的路由/视图表
- 模块化的、基于组件的路由配置
- 路由参数、查询、通配符
- 基于 Vue.js 过渡系统的视图过渡效果
- 细粒度的导航控制
- 带有自动激活的 CSS class 的链接
- HTML5 历史模式或 hash 模式，在 IE9 中自动降级
- 自定义的滚动条行为

### 安装Vue-Router

页面使用可以通过[下载js 文件](https://unpkg.com/vue-router/dist/vue-router.js)或者通过CDN 引入到页面中， 在 Vue 后面加载 `vue-router`，它会自动安装的：

```vue
// CDN 引入 可以通过@ 指定 版本号 或者 Tag。
<script src="直指向在 NPM 发布的最新版本。你也可以像 "></script>

<script src="/path/to/vue.js"></script>
<script src="/path/to/vue-router.js"></script>
```

实际开发中更多的是通过`npm` 进行模块化引入到项目中的：

```vue
npm install vue-router
//如果使用淘宝镜像可以通过cnpm 命令进行安装
cnpm install vue-router

//必须要通过 Vue.use() 在项目入口 main.js 明确地安装路由
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)
```

### 基础使用

引入Vue-Router 之后如何使用呢？其实很简单，只需要满足三个要素：

+ 定义路由页面或组件对象并创建路由对象
+ 将路由对象挂在到Vue 跟实例上
+ 在页面通过`router-view` 作为路由渲染出口

1. Vue 应用的页面都是通过组件的形式构成的，所以在进行路由跳转之前一定需要将跳转的路由页面或者组件信息定义清楚：

```js
// 可以通过模版的方式定义组件页面
const Foo = {template: '<div>foo</div>'}
// 或者引入已经定义好的组件页面
import Bar from '../components/Bar.vue'

// 定义路由信息
const routes = [
  {path: '/foo', component: Foo},
  {path: '/bar', component: Bar}
]

// 创建router 实例，传入routes 配置
const router = new VueRouter({
  routes //(缩写) 相当于 routes: routes
})
```

2.路由实例需要挂载在Vue 跟实例上，这样才能保证整个应用页面都能获得`router` 对象，从而能动态获取路由参数或者进行动态路由跳转：

```js
// 这样挂载在app 实例下的路由已经完成了
const app = new Vue({
  router
}).$mount("#app")
```

3.将路由组件进行正确的挂载还不够，还需要有对应的渲染出口才能正确的使用路由进行页面渲染：

```html
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

#### routes

`routes` 是一组`route` 路由对象，所以要配置路由，需要知道路由对象的基本配置项：

+ `path` 必须指定，路由通过path 匹配路由页面
+ `name` 需要唯一，但是不是必须的，这个是作为路由的一个标示。
+ `alias` 路由别名，配置别名可以进行路由跳转，但是path 为 `/` 不生效。
+ `redirect` 路由重定向配置，会改变浏览器地址url。
+ `component` 路由页面组件。
+ `components` 命名视图，多个`router-view` 出口时需要对应配置。
+ `children` 嵌套路由配置。
+ `meta` 路由元信息，通常通过`beforeEach` 全局导航守卫用来做登陆验证。

### 路由跳转方式

进行路由跳转有下面两种方式：

+ 通过点击`router-link` 显然的标签进行路由跳转
+ 通过响应事件利用js 进行路由跳转

#### `router-link` 跳转：

`<router-link>` 默认会被渲染成一个 `<a>` 标签， 可以通过配置 `tag` 属性生成别的标签， 通过 `to` 属性指定目标地址，当目标路由被激活时，链接元素自动设置一个表示激活的 CSS 类名。


```vue
<router-link to="/foo">Go to Foo</router-link>
```

`<router-link>` 比起写死的 `<a href="...">` 会好一些，理由如下：

- 无论是 HTML5 history 模式还是 hash 模式，它的表现行为一致，所以，当你要切换路由模式，或者在 IE9 降级使用 hash 模式，无须作任何变动。
- 在 HTML5 history 模式下，`router-link` 会守卫点击事件，让浏览器不再重新加载页面。
- 当你在 HTML5 history 模式下使用 `base` 选项之后，所有的 `to` 属性都不需要写 (基路径) 了。

#### JS 跳转

通过js 进行路由跳转是实际开发中最常用的路由跳转方式，因为这样能方便的获取路由中的参数信息，便于进行动态跳转和进行业务处理。

```vue
<div class="tab" @click="toFoo"></div>

export default {
 methods: {
   toFoo() {
            const params = this.$route.query;
        this.$router.push('/foo')
   }
 }
}
```

### 动态传参

使用路由进行页面跳转时一般不仅仅只是简单的跳转某个页面，往往需要将一些参数数据传递给目标页面进行业务处理，这时候就需要进行动态传参数。动态传参一般有两种方式：

+ 通过路由路径进行传参
+ 通过`route` 对象进行参数传递

#### 路径传参

一般一些简单参数的传递可以直接通过路径进行传递，这是比较常见的业务场景，通过传递业务bh 给目标页面，目标页面获取的业务bh 后就可以直接通过接口获取页面所需要的数据了。

```vue
const User = {
template: '<div>User</div>'
}

const router = new VueRouter({
routes: [
  // 动态路径参数 以冒号开头
  { path: '/user/:id', component: User }
]
})
```

这样定义路由后，像`/user/foo` 和 `/user/bar` 这样的路由都会匹配到相同的`User` 组件页面。这种方式就是通过`:` 传递的参数，所以`/user` 后边的路径会被填充到`this.$route.params` 中，就像下面这样：

| 当前路径  | 匹配路径  | 参数$route.params |
| :-------: | :-------: | :---------------: |
| /user/foo | /user/:id |    {id: "foo"}    |
| /user/bar | /user/:id |    {id: "bar"}    |

这种传参方式其实和后台的Rest 接口定义传参方式类似，这种方式也支持分段传参数，类似这样：

```vue
const router = new VueRouter({
routes: [
      // 动态路径参数 以冒号开头
      { path: '/user/:id', component: User },
            { path: '/user/:id/post/:postid', component: Post}
]
})
```

这种分段传参数的方式同样是将参数设置在了`$route.params` 中，所以同样可以获取。

|      当前路径      |        匹配路径         |     参数$route.params     |
| :----------------: | :---------------------: | :-----------------------: |
| /user/foo/post/123 | /user/:id/post/:post_id | {id: "foo", post_id: 123} |



#### Route对象传参

`$route` 是注册在`Vue` 跟实例下的全局变量，表示当前的路由对象，可以从中获取路由对象中的所有信息，`route` 对象实例大概是这样的：

```json
$route {
    path: "/index",
    query: Object,
    params: Object,
    fullPath: "/index",
    name: "index",
    meta: Object
}
```

通常在需要进行页面跳转时，通过`this.$router` 注册在`Vue` 全局的方法来改变当前路由的数据，也就是`$route` 的信息，`$router` 常用的传参跳转路由方法包括：

```text
//push 到对应的路由
router.push(location, onComplete?, onAbort?)
router.push(location).then(onComplete).catch(onAbort)

//替换当前的路由地址 浏览器地址会修改
router.replace(location, onComplete?, onAbort?)
router.replace(location).then(onComplete).catch(onAbort)

//前进histoy 中的n 步
router.go(n)

//后退
router.back()

//前进
router.forward()
```

开发中常用的是通过`$routr.push(route)` 方法传递参数并进行路由跳转，因为是在当前路由表中直接加入了新的路由对象，所以跳转到目标页面后可以直接通过`$route` 拿到路由对象的所有信息，这样的好处就是，传递参数更加灵活不受限制。

```vue
<li v-for="menu in menus" @click="changeMenu(menu)"></li>

export default{
methods: {
    changeMenu(id) {
        //通过路径传参 这种方式和路径传参一样需要在路由配置中配置动态传参
        //this.$route.params.id 获取参数
        this.$route.push({path: '/user/${id}'})

        //通过路由name 属性跳转并传参（不能写path 不然无法获取参数）
        //this.$route.params.id 获取参数
        this.$route.push({
                name: "user",
                params: {
                    id: id
                }
        })

        //通过path 匹配路由 通过query 传递参数
        //这种情况会把query 参数显示在浏览器url中
        //this.$route.query.id 获取参数
        this.$router.push({
                path: '/user',
                query: {
                    id: id
                }
        })
    }
}
}
```

### 嵌套路由

`Vue` 项目中说是路由的页面，并不准确，其实路由的是页面组件，在`Vue` 项目中都是以组件的形式存在的，页面也不例外，一个个的组件构成了页面，一个个的页面又可以看成是新的组件，即然是这样那么进行路由的时候肯定就不能简单的只有一个路由出口，不然页面组件就无法达到复用的目的了，所以`vue-router` 通过嵌套路由解决了这个问题。

一般项目的入口处都会有一个根路由出口，类似这样：

```vue
<div id="app">
 <router-view></router-view>
</div>
```

这个地方保证了项目中的页面都能正常的渲染到浏览器窗口，但是组件页面中的路由页面有怎么定义呢？其实很简单，只需要在渲染的地方正确的放置路由子出口即可。一般的首页都是这样的：

```html
Index.vue 首页
+-------------------------------------------+
|Header.vue导航头组件                         |		
|                                           |
|+----------------------------------+	    |
||Content.vue路由子组件	            |       |
||                                  |       |
||                                  |	    |
|+----------------------------------+	    |
+-------------------------------------------+
```

这样的多个子页面组成的页面需要自己的`router-view` 作为渲染出口用来渲染子页面：

```vue
Index.vue
<div class="index">
    //导航头组件
    <Header></Header>
    //子路由出口
    <router-view></router-view>
</div>
```

嵌套页面的路由定义可以通过`children` 属性传入到路由页面的子路由配置，类似这样：

```js
const router = new VueRouter({
 routes: [
   {
     path: "/index",
     name: "index",
     component: Index,
     children: [
       {
         path: 'content',
         name: 'content'
         component: Content
       }
     ]
   }
 ]
})
---/index/content 匹配到content.vue
---/content 匹配到content.vue
```

> **注意：**  以 `/` 开头的嵌套路径会被当作根路径。 这让你充分的使用嵌套组件而无须设置嵌套的路径。

也就是说为了充分利用路由映射表，当子路由path 中带了`/` 那么也会被跟路径路由所匹配到，就想这样：

```js
const router = new VueRouter({
 routes: [
   {
     path: "/index",
     name: "index",
     component: Index,
     children: [
       {
         path: '/content',
         name: 'content'
         component: Content
       }
     ]
   }
 ]
})

// `/content` 路径也可以直接访问显示的是Content 页面
```

### 命名视图

对组件进行复用的时候，例如上面提到的Index.vue 页面，除了可以直接在页面中引用`Header` 组件，还可以通过命名路由的方式对组件页面进行组合使用，类似这样：

```vue
<div class="index">
    <router-view class="header"></router-view>
    <router-view class="notFound" name="notFount"></router-view>
    <router-view class="content" name="content"></router-view>
</div>
```

这种命名视图的方式组合组件形成页面的方式是通过`components` 属性完成的：

```json
{
 path: '/index',
 name: 'index',
 children: [
     {
         path: '/content',
         name: 'content',
         components: [{
             default: Header,
             notFound: Content
         }]
     },
     {
      path: '/not-found',
         name: 'notFound',
      components: [{
             default: Header,
          notFound: NotFound
         }]
  }]
}
```

这种方式在开发中暂时并没有遇到过，这里只是简单介绍下。

### 总结

其实`vue-router` 的基本使用还是比较简单容易上手的，只要搞清楚了路由的几种传参方式，以及嵌套路由的使用，在开发简单项目基本上是够用的，但实际开发项目中其实还要使用到许多进阶知识，例如路由元信息的使用，过度动效的自定义，路由懒加载实现等等，这些都需要继续学习并使用才能熟练掌握。