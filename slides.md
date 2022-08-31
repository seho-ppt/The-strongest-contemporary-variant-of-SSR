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
- 根据前端全栈框架演变的不同渲染方法 (框架的不同也可能造就渲染模式的差异, 因为其核心理念不同)

---

# SSG

> SSG指的就是在服务端间歇性生成静态页面的渲染方式

比如在资讯网站,用户看到的文章内容都是一样的,那么就可以把需要多次渲染的文章页面,只用渲染生成一次缓存到服务器中,当下一次请求就可以直接返回已经生成好的html给客户端; 这就意味着用户看到的内容不会是最新的, 有可能服务器会在1分钟/1小时才会生成/缓存一次.

静态生成的网站, 你可以非常方便的通过CI部署在CDN / Vercel / Cloudflare上

<div class="flex justify-start items-center">
  <logos-cloudflare class="text-8xl"/>
  <simple-icons:vercel class="text-6xl ml-8"></simple-icons:vercel>
  <logos-maxcdn class="text-8xl ml-8"></logos-maxcdn>
</div>

---

# Nuxt3 - 混合模式 + 边缘侧渲染

在这一章中主要介绍偏Nuxt3的新通用渲染模式, 在传统的全栈框架, 我们可以在整个程序中选择单独一种作为渲染方案, 而现在你可以根据路由使用多种渲染方案:

- a路由构建生成静态页面
- b路由使用服务端渲染
- c路由使用客户端渲染