![](/post-img/nuxt/img01.png)

## 为什么要用Nuxt？

* SEO

我们需要搜索引擎更多地抓取到我们的内容，更详细地认识到我们的网页结构，而不是仅对首页或特定静态页进行索引。

题外话：第三方的解决方案

可以建立HTTP一个中间层，在判断到访问来源是蜘蛛时，输出已缓存好的html数据，此数据若不存在，则调用第三方服务对html进行缓存，往复进行。

另一方法是自行构建蜘蛛渲染逻辑，当识别UA为搜索引擎时，拿服务端已准备好的模板和数据进行渲染输出html数据，反之，则输出SPA应用代码。

弊端：需要针对蜘蛛编写一套独立的渲染模板，因为大部分情况下SPA的代码是没法直接在服务端使用的，搜索引擎若检测到蜘蛛抓取数据和真实访问数据不一致，会做降权惩罚，也就意味着渲染模板还必须和SPA预期输出一模一样。

* 用户体验

当用户网络比较慢、或者带宽比较差的时候，通过减少页面请求数量，来保证尽快看到基本的内容。但是避免强制用户下载整个单页面应用，也远没有下载个单独的预先渲染过的HTML文件性能高。


## Nuxt是什么？

* pages：各页面组件，用于生成对应路由，支持嵌套，支持动态路由
* components：各组件，用于你自己管理公共组件或非公共组件
* layouts：宿主布局页面模板组件，用于你可以把不同的页面指定使用不同的布局
* assets：用于webpack编译的各类资源，通常是一些小的资源，如代替雪碧图之类的图片等东西
* middleware：中间件，首屏渲染和路由跳转前均执行对应中间件（登录态校验等），可以返回promise或直接next（很实用！）
* plugins：插件，SPA中用的各类第三方组件和一些node模块都可以在这引入，甚至可以引入自己编写的第三方库
* store：内置了vuex，可以直接返回数据模块或返回一个自建vuex根对象，具体要翻文档

## 路由（Routing）

常用的有这几种：Basic Dynamic Nested

Nuxt封装了路由的生成，你不需要额外编写路由，pages文件夹下的结构，会自动生成对应的路由。

从源码可以看到，路由的生成主要是在lib/build.js里面处理的，大致步骤如下：

1. 通过glob获取pages下所有文件，得到数组
2. 过滤掉pages和.vue等不相关的字符串
3. 对带有_符号进行处理
4. 处理动态路由和嵌套路由
5. 对子路由排序

## 异步数据（Async Data）

支持Promise async/await callback

在组件结构中，其属于宿主layout下的子组件，不属于页面组件，无法使用页面组件中的fetch方法，官方的解释是子组件无法使用阻塞异步请求，即：子组件得到的异步数据无法用于服务端渲染，这对于程序是合理的，避免异常阻塞，简化业务模型。

如果需要这些异步数据增强站内内链SEO，我们可以巧妙地使用内置vuex中的nuxtServerInit这个API，这个API实在nuxt程序实例化之后第一次执行的方法，其内部返回一个promise，我们可以在这里完成我们站内的所有子组件异步请求，随后将数据映射至对应子组件即可。

另外的方法是在mounted 方法去调用异步数据。

注：在这个data方法里面，我们获取不到 this指针，因为data方法在组件初始化之前就已经被调用了。


## 指令（Commands）

![](/post-img/nuxt/img02.png)

这里有一篇文章讲的很不错 [深入NUXT，看看一条命令行的背后到底发生了什么](https://segmentfault.com/a/1190000008114613)

下面主要来看下nuxt generate这个指令，大致步骤如下：

1. 遍历路由
2. 调用renderRoute函数，得到html字符串
3. 处理压缩
4. 写入文件

## 踩到的坑

Nuxt版本：0.9.6

没有一个CDN配置入口
build的主css和js使用hash后，引用的时候直接把[hash]当成字符串，如下所示：
把Nuxt当做中间件使用的时候，没有了req.session，用户数据只能从cookie获取，但是在服务端渲染时还没有window
每个组件加上name，对路由跳转有好处

## NUXT 能为我们做什么

就是我们无需为了路由划分而烦恼，你只需要按照对应的文件夹层级创建 .vue 文件就行
无需考虑数据传输问题，nuxt 会在模板输出之前异步请求数据（需要引入 axios 库），而且对 vuex 有进一步的封装
内置了 webpack，省去了配置 webpack 的步骤，nuxt 会根据配置打包对应的文件

## SSR on Vue

[Vue 基于 NUXT 的 SSR](https://segmentfault.com/a/1190000007933349)

## SSR on React

React 提供了两个方法 renderToString 和 renderToStaticMarkup 用来将组件（Virtual DOM）输出成 HTML 字符串，这是 React 服务器端渲染的基础，它移除了服务器端对于浏览器环境的依赖，所以让服务器端渲染变成了一件有吸引力的事情。

[React 的服务器端渲染](https://link.zhihu.com/?target=https%3A//hulufei.gitbooks.io/react-tutorial/content/server-rendering.html)

## SUMMARY

使用Nuxt框架的时候还是0.9.6的版本，框架本身还有很多地方需要优化，网上的文档资源基本没有，所以也只能踩一个坑补一个坑，大部分时间都是看在看源码，改源码。

在这个过程中也发现自身对于原生JS还是没吃透，一直在跟着业务走，需求又很赶，很多时候都是拿来主义，追求能解决问题，而很少花时间去深究问题的本质，这给我们一定的警示。

Nuxt目前来讲还没看到大型项目在使用，感觉这个框架也只是比较适用于轻量级的，偏展示型的网站。

以上~
