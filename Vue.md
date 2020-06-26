# Vue 渐进式JavaScript框架

## Vue的特点

- 不断完善浏览器特性的前端框架。
- 用于构建用户界面的渐进式框架。
- Vue的核心库只关心视图层，即只关注界面布局。
- JQuery是JavaScript类库，支持DOM，通信等。与Vue关注点不同。
- 易于上手，便于与第三方库（如：vue-router，axios(通信框架)，vuex(状态管理框架)等）整合。
  - vue-router 根据后端返回的结果集，由前端完成页面跳转。
  - axios 前端跟后端的通信框架
  - vuex状态管理框架
- http是无状态的，vuex管理状态。

## 前端三要素

+ HTML（结构）：决定网页的结构和内容
+ CSS（表现）：设定网页的表现样式
+ JavaScript（行为）：用于控制网页的行为。CSS支持面向对象编程，SASS，LESS

## 行为层（JavaScript）

### 原生JS

前端都使用原生JavaScript，不使用JQuery。

浏览器全支持ES5，但现在基本都有ES6，通过webpack打包，将ES6转化为ES5，这样浏览器就都支持了。

### TypeScript

由微软主导开发，是JS的一个超集，支持面向对象等特性，需要编译成JS后，才能被浏览器正确执行。

前端是原生JS和TypeScript二选一。

### JavaScript框架

- JQuery，优化DOM操作，但DOM操作太频繁，影响前端性能。

- Angular 由Google收购的前端框架，由Java程序员开发，对后台程序员友好，对前端程序员不太友好。引入模块化开发理念。

  迭代版本不合理。

- React Facebook出品，在内存中虚拟DOM操作，有效提升前端宣言效率，缺点是使用复杂，需要额外学习一门JSX语言。

- Vue 渐进式就是逐步实现新特性的意思，如实现模块化开发、路由、状态管理等特性。其特性是综合了Angular（模块化）和

  React（虚拟DOM）的有点。并使用原生JS开发。

- Axios 前端通信框架

  Java通信使用HttpClient，Feign的底层是HttpClient，HttpClient的底层是UrlConnection，即Java的Net包下的类库。

### UI框架

+ Ant-Design 阿里巴巴出品，基于React的UI框架
+ Element-UI 基于Vue的UI框架
+ BootStrap Twitter推出的一个用于前端开发的开源工具包
+ AmazeUI 又叫“妹子UI”，一款HTML5跨屏前端框架

### JavaScript构建工具

+ Babel   JS编译工具，主要用于浏览器不支持的ES新特性，比如用于编译TypeScript.

+ WebPack  模块打包器，主要用于打包、压缩、合并及按序加载。

## 三端统一

###  混合开发（Hybrid App）

 一套代码三端（PC、Android、iOS）统一，并能调用到设备底层硬件（如GPS、摄像头等），打包方式主要有一下两种

+ 云打包 HBuildX
+ 本地打包 Cordova（前身是PhoneGap）

### 后端技术

方便前端人员开发后端应用，出现了NodeJS。NodeJS框架及项目管理工具如下：

+ Express  NodeJS框架
+ Koa Express简化版
+ NPM  项目综合管理工具，类似于Maven
+ YARN  NPM的替代方案，类似于Maven和Gradle的关系。

## 前后端分离的演变史

### 后端为主的MVC时代

为了降低开发的复杂度，以后端为出发点

MVVM（异步通信为主）：Model、View、ViewModel

ViewModel是处在中间，跟Model和View双向绑定，当View或Model发生变化的时候，ViewModel能够监控到，并将变化反应给对方。开发者不需要操作DOM来改变View内容，比如使用JQuery操作DOM。

- Model：模型层，在这里表示 JavaScript 对象
- View：视图层，在这里表示 DOM（HTML 操作的元素）
- ViewModel：连接视图和数据的中间件，**Vue.js 就是 MVVM 中的 ViewModel 层的实现者** 

ViewMode封装出来的数据模型包括状态和行为两部分。

View的两大核心要素

1. 数据驱动：通过观察者感知View或Model的变化

2. 组件化：页面上每个独立的可交互的区域视为一个组件，通过多个组件拼装完成页面开发。

Vue.js 框架就是一个MVVM模式的实现者，MVVM模式的核心是ViewModel层，ViewMode就是一个观察者。

### Axios

Axios是一个开源的可以用在浏览器和NodeJS的异步通信框架。主要作用是实现AJAX异步通信。

其功能特点如下：

+ 从浏览器中创建、xMLHttpRequests

+ 支持Promise API，实现链式编程。

+ 自动转换JSON数据

```javascript
      <script type="text/javascript">
          var vm = new Vue({
              el: '#app',
              data(){  //函数对象，需要有return返回一个对象。
                  return{
                      info:{
                          name:'',
                          url:''
                      }
                  }
              }
          });
      </script>
```

  ### 组件

组件是可复用的Vue实例，是一组可以重复使用的模板。
