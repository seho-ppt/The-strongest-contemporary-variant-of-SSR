---
# try also 'default' to start simple
theme: apple-basic
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# apply any windi css classes to the current slide
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# persist drawings in exports and build
layout: intro
---

# SSR当代最强变种 - Island架构

前端娱乐圈最新花活

<div class="absolute bottom-10">
  <span class="font-700">
    纸贵-沈昊
  </span>
</div>


<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# 目录

- 什么是SSR,CSR?
- 现代前端开发困局
- 如何理解同构架构
- SSR变种盘点
- SPA & MPA
- 什么是Island(孤岛)架构
- 为什么要使用Island
- 实现Island架构的全栈框架(Astro, Qwik)
- 孤岛组件化 / 孤岛细粒度 的双维度对抗
- Code

---

# 什么是SSR

- SSR (Server Side Rendering) 指的就是在服务端组装网页内容的渲染技术, 例如JSP,PHP或者传统多页HTML应用, 对于客户端而言可以避免多次的HTTP请求, 更利于SEO/首屏渲染速度提升.

<!-- 显示网站web -->
<div>
  <!-- <logos:nuxt-icon class="text-8xl mt-20"/> -->

  <br/>

开发时:

```html
<body>
    我是类服务端模板引擎生成的html页面, 我通常是这样渲染数据的:
    ${data.year}-${data.month}-${data.day}
</body>
```

<br/>
<br/>

服务端渲染后, 客户端拿到的html网页:
```html
<body>
    我是类服务端模板引擎生成的html页面, 我通常是这样渲染数据的:
    2022-1-1
</body>
```
</div>


---

# 爬虫优先

为了更好理解SSR渲染模式对于SEO有什么好处, 我们可以尝试了解一波爬虫(百度叫蜘蛛, 谷歌叫爬虫); 爬虫会在互联网上抓取一部分网页, 抽取其中的url放在一个待抓取的队列, 然后通过dns解析拿到一批网站的ip, 再通过网站下载器下载网站; 那么其中“抓取部分”, 就是重中之重的部分;

> 抓取的策略也很简单, 就是最重要的网页;

- SSR组装的工作是在服务端, 它确保了内容在客户端立即呈现, 不需要等待资源加载完成, 因此它呈现结果是异常高效的, 所以爬虫就非常喜欢这样的页面.

(查看网页源代码)

```html
<div class="search-hots-fline" data-spm-ab="fline">
  <a href="http://s.taobao.com/search?spm=1.7274553.1997520241-2.2.TpEKPQ&amp;q=耳机&amp;refpid=420463_1006&amp;source=tbsy&amp;style=grid&amp;tab=all&amp;pvid=d0f2ec2810bcec0d5a16d5283ce59f67">耳机</a>
  <a href="http://s.taobao.com/search?spm=1.7274553.1997520241-2.2.TpEKPQ&amp;q=女包&amp;refpid=420464_1006&amp;source=tbsy&amp;style=grid&amp;tab=all&amp;pvid=d0f2ec2810bcec0d5a16d5283ce59f68">时尚女包</a>
  <a href="http://s.taobao.com/search?spm=1.7274553.1997520241-2.2.TpEKPQ&amp;q=沙发&amp;refpid=420465_1006&amp;source=tbsy&amp;style=grid&amp;tab=all&amp;pvid=d0f2ec2810bcec0d5a16d5283ce59f68">沙发</a>
</div>
```

高权重的电商网站/官网, 需要用搜索流量引流的主要业务, SEO ER都会非常注重爬虫效率

---

# 什么是CSR

- CSR (Client Side Rendering) 相对于SSR, 是完全相反的渲染技术, 它把在服务端的组装工作放到了客户端, 在初次请求的时候仅仅会返回一个引导性HTML, 剩余的页面交互和渲染通常会通过JS在一个页面中切换视图;

<br/>

> 在我们熟悉的VUE,REACT这一类的视图层框架中, 就是默认沿用CSR渲染的设计

CSR只有一个空html文档, 通常会引用一个或多个js文件, 这就意味着爬虫需要等待js文件加载完成, 才可以显示网页, 比如Vue (Vite)的模板代码就是这样:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>

```


---

# 强交互性

> SSR虽然对于高权重的电商网站/官网, 是非常有用的, 但是对于低权重的网站, 它只是一个简单的静态页面, 并不能满足用户的需求, 因此它不是一个好的选择; 当我们用户需要在网站上做强交互时, 比如频繁跳转页面等, 那么使用SSR就会造成过多的网络往返 (或者服务器压力).

vue router.ts
```ts
import { createRouter, createWebHashHistory } from 'vue-router'

const router = createRouter({
  history: createWebHashHistory(),
  routes: [
    //...
  ],
})
```

<v-click>

通常的单页应用路由是用hash实现, 它是一个锚点, vue会根据#后的url去切换不同的页面, 这个切换的过程全程是js完成的, 因此不需要额外的服务器压力, 只需要等待js文件加载完成, 就可以显示网页了.


```bash
https://localhost:8080/#/home
```
</v-click>

---

# 针对缺点和优点进行选型

> SSR的主要优点

- SSR效果很好
- 首屏渲染性能更好

<br/>

> CSR的主要优点

- 强交互场景下节省网络往返
- 没有服务端压力

---

# 鱼和熊掌可以兼得-同构架构

> 一般的通用渲染方案: 同构架构即SSR + CSR

在首屏下, 通过Nodejs渲染页面, 在非首屏下, 通过浏览器渲染页面. 这样既保证了SEO的需求和首屏可交互性, 也保证了在强交互场景下用户所需要的用户体验, 也减少了服务器压力.

```ts
asyncData({ store, fetch, route }) {
  // 首次加载页面, 会经过nodejs请求api
  return fetch('/api/data').then(res => res.json())
}
```

Nuxt.js / Next.js 都是这么做的, 在同构架构下, 我们的js代码需要同时运行在浏览器和服务器上, 所以我们需要框架来尽可能的磨平客户端和服务端的api差异, 并且提供一个独立的模块, 来保存同构代码.

> 那么Nuxt这一类的全栈框架是如何做到静态页面使其仍然可以交互呢?

---

# 喝水 / 脱水 / 注水

<div class="flex justify-between items-center mt-10">
  <div class="item mt-5" style="width: 20%;">
    <img src="https://static.yinzhuoei.com/typecho/2021/11/10/35243711893279/1.png" alt="">
    <div class="text-center mt-5">喝水</div>
    <div class="text-xs text-center mt-2">将数据流补充到页面上下文中 (服务端) </div>
  </div>
  <div class="item mt-5" style="width: 20%;">
    <img src="https://static.yinzhuoei.com/typecho/2021/11/10/355283501440982/fpic6008.jpeg" alt="">
    <div class="text-center mt-5">脱水</div>
    <div class="text-xs text-center mt-2">客户端需要直出HTML, 需要把数据嵌入到静态的HTML</div>
  </div>
  <div class="item mt-5" style="width: 20%;">
    <img src="https://static.yinzhuoei.com/typecho/2021/11/10/35243711893279/1.png" alt="">
    <div class="text-center mt-5">注水</div>
    <div class="text-xs text-center mt-2">客户端执行/下载静态页面中的JS, 使其页面重新变得可交互</div>
  </div>
</div>

---

# 在Vite中实现注水功能

> 在Vite中给我们提供了一部分的SSR功能, 虽然是实验性的, 但是非常有助于我们理解注水实现

- 实现asyncData函数

```ts
export const render = async (url) => {
  const { app, router } = createApp()
    router.push(url)
    await router.isReady()
    let data = {}
    // 命中路由组件，且执行asyncData这个函数
    if (router.currentRoute.value.matched[0].components.default.asyncData) {
      const asyncFunc = router.currentRoute.value.matched[0].components.default.asyncData
      data = asyncFunc.call()
    }
    const html = await renderToString(app)
    return { html, data }
}
```


---

# 脱水序列化 (Vite Server)

- 我们需要把上一个部分拿到的data和html字符串在服务端进行脱水, 原理就是把data合并到html中

```ts {5|6-7|8-10|all}
app.use('*', async (req, res) => {
  try {
    const url = req.originalUrl
    let template = readFileSync(resolve('index.html'), 'utf-8')
    template = await vite.transformIndexHtml(url, template)
    const { render } = await vite.ssrLoadModule('./src/entry-server.js')
    const { html: appHtml, data } = await render(url)
    // 拼接标签，把data序列化插入到文档中
    const html = template.replace(`<!--ssr-outlet-->`, `${appHtml}<script>window.__data__=${JSON.stringify(data)}</script>`)
    res.status(200).set({ 'Content-Type': 'text/html' }).end(html)
  } catch (error) {}
})
```

- 同理, 我们的Vuex数据也是这样持久化data的, 当在客户端时, 只需要将各种类型的data, 重新注入到实例中, 我们的Vue组件就可以正常交互了

---

# 注水

- 将已经序列化的data和初始化的空data进行合并

```ts {8-14|all}
router.isReady().then(() => {
  const component = router.currentRoute.value.matched[0].components.default
  let _data = {}
  // 判断是否是函数
  if (typeof component.data === 'function') {
    _data = component.data.call()
  }
  // 判断是否有脱水的data
  if (window.__data__) {
    _data = {
      ..._data,
      ...window.__data__
    }
  }
  component.data = () => _data
  app.mount('#app')
})
```

---

# SSR变种
> 随着前端业务发展, SSR和CSR已经不能满足需求, 所以衍生出了很多变种方案, 但是核心理念都会围绕以下几个点:

- 服务器运维成本降低
- 页面更新频率 (是否特别依赖及时更新, 比如资讯网站需要频繁更新)
- 偏向服务端侧渲染 (Pre-rendering) 或者在客户端渲染上做文章
- 根据前端全栈框架演变的不同渲染方法 (框架的不同也可能造就渲染模式的差异, 因为其核心理念不同)
- 根据部署平台的特点, 比如Vercel, 也有独有的渲染方法ISR (静态增量渲染生成)

---

# SSG (Pre-rendering)

> SSG指的就是在服务端间歇性生成静态页面的渲染方式

比如在资讯网站,用户看到的文章内容都是一样的,那么就可以把需要多次渲染的文章页面,只用渲染生成一次缓存到服务器中,当下一次请求就可以直接返回已经生成好的html给客户端; 这就意味着用户看到的内容不会是最新的, 有可能服务器会在1分钟/1小时才会生成/缓存一次.

静态生成的网站, 你可以非常方便的通过CI部署在CDN / Vercel / Cloudflare上

<div class="flex justify-start items-center">
  <logos-cloudflare class="text-8xl"/>
  <simple-icons:vercel class="text-6xl ml-8"></simple-icons:vercel>
  <logos-maxcdn class="text-8xl ml-8"></logos-maxcdn>
</div>

---

# Nuxt3 - 混合模式

在这一章中主要介绍偏Nuxt3的新通用渲染模式, 在传统的全栈框架, 我们可以在整个程序中选择单独一种作为渲染方案, 而现在你可以根据路由使用多种渲染方案:

- a路由构建生成静态页面
- b路由使用服务端渲染
- c路由使用客户端渲染

Nuxt得益于最新的JS服务器引擎 (Nitro), 实现了路由缓存以及边缘侧渲染

```ts
export default defineNuxtConfig({
  routes: {
    '/': { prerender: true }, // Once per build (via builder)
    '/blog/*': { static: true }, // Once on-demand per build (via lambda)
    '/stats/*': { swr: '10 min' }, // Once on-demand each 10 minutes (via lambda)
    '/admin/*': { ssr: false }, // Client-Side rendered
    '/react/*': { redirect: '/vue' }, // Redirect Rules
  }
})
```

---


# SPA & MPA

> 前端网页程序中分为单页应用(SPA)和多页应用(MPA), 我也相信前端同学们都已经非常了解它们了 (老八股文题目了😄)

- 多页应用程序是由多个html文档组成的网站, 当用户访问新的导航时, 浏览器会有一个新请求到达服务器, 传统的MPA框架有很多, 比如ruby on rails, 还有php laravel等等.

<div class="flex justify-end mt-5">
  <logos-php class="text-5xl" />
  <logos-rails class="text-5xl ml-4" />
</div>

- 单页应用指的是单个js应用程序组成的网站, 只有一个页面, 在浏览器中加载js->操作视图->然后呈现html; 但是spa也可能在服务器上生成html, 比如我们熟知的nuxtjs, nextjs, remix, vue/react等等

<div class="flex justify-end mt-5">
  <logos-vue class="text-4xl" />
  <logos-react class="text-4xl ml-4" />
</div>



---

# Island(孤岛)架构

> 你有没有想过不管是CSR/SSR及其变种都已经非常成熟, 为什么前端圈子还要再继续演变新的架构体系

现代网站过于依赖JS, 不管我们使用的是什么框架, 网页(CSR)都需要加载过多的JS文件, 在传统的SSR架构下, 服务端需要给客户端发送巨多的JS代码(混合在HTML中)使页面重新注水; 而且大多数情况下, 网站动态的内容虽然只有一小部分但是却用了臃肿的框架/库去额外渲染, 这样就造成了大量的资源浪费, 甚至会影响到用户体验.

<img width="100" height="100" src="data:image/jpeg;base64,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"/>

所以孤岛架构的目标就是CSR的优秀交互体验以及SSR的SEO性能, 用最自然的描述语言(HTML)去编写现代高性能网站. 除了上述这些, 它还拥有几个特点:

- 组件化
- 动静分离, 静态内容无需经过处理, 动态内容自我注水(此体系所基于的技术就是部分/选择注水)

---

# 孤岛一词的含义

<img width="400" height="200" class="m-auto" src="https://res.cloudinary.com/wedding-website/image/upload/v1596766231/islands-architecture-1.png"/>

<br/>

> 网站可分为动态交互内容和静态内容, 孤岛架构网站的动态内容每一块都是单独开发和维护的, 不会限制你的技术栈(即UI层渲染, 你可以使用Vue/React/Preact); 在常见的网站中, 比如官网/电商/博客网站, 在静态不变的内容下, 通常都会有交互式组件, 比如聊天对话框, 可变的链接跳转, 那么这些交互功能即可以理解为一个又一个“孤岛”, 孤岛和孤岛之间互不干扰.

---

# 动态孤岛组件

<img class="m-auto" width="300" height="200" src="https://miro.medium.com/max/700/1*mObEGoYvsOarH4qo0Sa8ww.png"/>

<br/>

> 我们需要再次梳理一下, 孤岛架构中的动态孤岛组件主要解决了什么问题, 或许你在前面的章节中已经了解过它们的优点, 那么我们这一部分将对优点进行剖析.

- 每一个交互式孤岛组件是完全隔离的, 意味着不会影响彼此的渲染, 静态内容亦是如此.
- 不会限制你的技术栈, 即UI层渲染
- 异步交互, 在初次渲染时不会有JS加载

---
