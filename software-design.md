# SD — GoMovie Project

GoMovie 核心功能：

- 查询最近上映电影信息
- 电影购票
- 电影票转让

整体架构如下：

![](./sd-img/framework.png)

## 1. Client

GoMovie 项目的前端是一个电影订票 SPA。

项目代码组织如下：

```bash
.
├── build/                      # 配置webpack
│   └── ...
├── config/                     # 存放项目配置文件
│   ├── index.js                
│   └── ...
├── src/
│   ├── main.js                 # 应用入口
│   ├── url-config.js           # 存储了后端的地址
│   ├── App.vue                 # 应用根组件
│   ├── lib/                    # 一些通用的自定义函数库
│   ├── components/             # 页面中的可复用小部件
│   │   └── ...
│   ├── pages/                  # 主页面
│   │   └── ...
│   ├── store/                  # 引入vuex存储，作为全局状态树
│   │   ├── index.js
│   │   └── ...
│   └── assets/                 # 由webpack打包的静态资源
│       └── ...
├── static/                     # 不经过webpack打包的静态资源
├── test/
│   ├── e2e/                    # 配置端到端测试
│   └── unit/                   # 配置单元测试
├── .babelrc                    # babel设置
├── .postcssrc.js               # postcss设置
├── .eslintrc.js                # eslint设置
├── .editorconfig               # editor设置
├── .travis.yml                 # travis CI设置， 提供持续集成
├── index.html                  # 入口html
└── package.json                # 构建脚本以及依赖
```

### 1.1 技术选型

- 前端框架 Vue.js
  - 是一套构建用户界面的渐进式框架。与其他重量级框架不同的是，Vue 采用自底向上增量开发的设计。Vue 的核心库只关注视图层，因此易于上手。
  - MVVM 架构，模型、视图、视图模型三者的分离，减少各部件之间的耦合性，提高了开发效率。
- 前端组件库 ElementUI
  - 一套已经写好的 Vue 组件库。其中有很多通用性的组件，例如按钮、布局、输入框、表单等等，直接使用就不需要我们重新发明轮子，提高了开发的抽象级别和开发效率。
- 打包工具 Webpack
  - Webpack 是一个 Web 技术中的打包工具，可以整合各个文件的代码、图片、字体、样式表等等，将其压缩、丑化成一组文件，减少了这些资源在传输给用户时的时间损耗，也加强了对源代码的保密性。
- ES6 编译器 Babel 
  - 通过 Babel 工具，我们就可以编写 ES6 风格的 JavaScript，并且不用担心浏览器的支持问题，因为 Babel 可以将 ES6 编译成大多数浏览器都支持的 ES5 代码。
- 发送 ajax 请求 axios
  - 简单、轻量的 ajex 解决方案，让我们的前端应用可以很容易地向后端发送 ajex 请求。

### 1.2 架构设计

#### 1.2.1 整体架构

整个项目实现了 MVC 架构，而前端自身部分也实现了 MVVM 的架构，并且运行在与后端不同的服务器上，两者通过 REST API 进行信息交互，实现了前后端的完全分离。

![](./sd-img/framework2.png)

#### 1.2.2 Story Board

为了更直观地体现前端在用户角度的交互过程，我们绘制了整个系统的 Story Board 用来呈现前端中涉及到的所有页面，以及页面之间的跳转关系，如下图所示：

![](https://camo.githubusercontent.com/e383efc722f64b864a61ff36a98729e143e72e8b/687474703a2f2f6f6f386637786b7a702e626b742e636c6f7564646e2e636f6d2f3132333132333132333231332e706e67)

### 1.3 模块划分

我们根据 Vue.js 的单文件组件以及路由，对页面元素进行了具有层次结构的划分，具体分为 *页面* 、*组件*、和提供支持的 *lib 函数*。此外还单独划分出了单元测试和端到端测试的模块。模块划分与项目中代码的对应关系如下图所示：

![](./sd-img/hie.png)

#### 1.3.1 页面

做页面设计的时候，我们参考在前一阶段所绘制出的 Story Board 作为参考，划分出了以下这些页面模块：

- cinema-info 选择电影院页面
- confirm-order 确认订单与付款页面
- home 主页
- movie-info 显示电影详细信息
- orders 查看用户订单页面
- seat-info 选座页面
- trade-board 提供电影票转让服务

我们将这些 *页面* 每一个都单独做成了一个 vue 组件，放在 `(project_root)/src/pages/` 下：

![](./sd-img/pages.png)

在每一个 *页面* 中，我们定义其组织结构、样式和操作逻辑（分别在`<template>` `<script>` 和 `<style>` 块中），并且，每一个 *页面* 又可以调用第一级别的 *组件* ，从而达到代码复用的效果。而用户又通过前端提供的路由，在输入不同 URL 时将对应的 *页面* 反馈给用户浏览器上显示。路由配置如下所示：

```javascript
export default [{
  path: '/',
  name: 'home',
  component: require('@/pages/home')
}, {
  path: '/cinema-info',
  component: require('@/pages/cinema-info')
}, {
  path: '/movie-info',
  component: require('@/pages/movie-info')
}, {
  path: '/confirm-order',
  component: require('@/pages/confirm-order')
}, {
  path: '/orders',
  component: require('@/pages/orders')
}, {
  path: '/seat-info',
  component: require('@/pages/seat-info')
}, {
  path: '/trade-board',
  component: require('@/pages/trade-board')
}]
```

#### 1.3.2 组件

在 GoMovie 项目中我们将 *页面* 所用到的组件放在 `(project_root)/src/components/` 目录下，并且将只有一个页面用到的组件放在了以该页面命名的文件夹下。

![](./sd-img/components.png)

这里我们的很多组件其实只用到了一次，但是他们彼此之间以及与页面之间都是解耦的，只通过 props 和 event 的方式进行信息传递，每一个组件都可以被轻松扩展，并且可以被实现了相同接口的另一个组件所替代。这体现了高内聚低耦合的设计思路，提高了整个系统的可扩展和可维护性。

#### 1.3.3 lib 函数

项目中，自定义的通用的函数（与页面、组件等无关，比如日期、时间的计算等等），被统一放在了 `(project_root)/src/lib/` 下，为其他组件提供支持。

#### 1.3.4 测试模块

**单元测试：**主要关注组件的数据模型、方法的输入输出、组件间的数据传递情况和异步操作这四个方面。GoMovie 项目通过 karma 来进行单元测试。在 `(project_root)/test/unit/spec/` 目录下的不同文件中，进行断言编写。

**端到端测试：**端到端测试的方式通过测试模拟用户行为，打开浏览器自动化地做一些操作（如输入、点击等）看看程序是否能够达到预期等结果。Vue.js 可以使用 nightwatch 工具进行端到端测试。Nightwatch 是基于 NodeJS 的 e2e 测试框架，通过发送 HTTP 请求到 Selenium WebDriver 来控制浏览器进行测试

### 1.4 软件设计技术

#### 1.4.1 MVC / MV**

## 2. Server

