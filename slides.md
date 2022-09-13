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
- Island和微前端的区别
- 为什么要使用Island
- 实现Island架构的全栈框架(Astro, Qwik, Fresh)
- 孤岛组件化 / 孤岛细粒度 的双维度对抗

---

# 什么是SSR

- SSR (Server Side Rendering) 指的就是在服务端组装网页内容的渲染技术, 例如JSP,PHP应用, 对于客户端而言可以避免多次的HTTP请求, 更利于SEO/首屏渲染速度提升.

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

- SEO效果很好
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

<!--
asyncData(){
 return {
  hello: "world"
 }
}
-->

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

# Vite中实现asyncData

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

# Vite脱水序列化

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

静态生成的网站, 你可以非常方便的通过CI部署在Vercel / Cloudflare上

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

<br/>

> 我们简单回顾了SPA & MPA的概念, 这将有助我们理解接下来的Island架构和其背后代表的全栈框架
---


# 现代Web网站开发困局

极大部分的现代网站都需要JS, 不仅可以完成复杂的需求以及交互体验, 但是随着时间的推移, 脚本的数量将会成指数型增长. 那么很显然JS脚本过多并不是一个好事情:

- 网络带宽, 大量的JS代码被发送到客户端, 不仅对服务器是一个挑战, 在客户端低网速下, 等待也是很昂贵的
- 更久的启动时间, 客户端需要大量JS激活组件, 使其变得可交互

<br/>

而且JS是单线程的, 无法利用多核CPU解决根本问题.

> 解决问题的方法也很简单, 不要JS不就得了!

<br/>

<img width="100" height="100" src="data:image/jpeg;base64,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"/>

<br/>

---

# Island(孤岛)架构

现代网站过于依赖JS, 不管我们使用的是什么框架, 网页(CSR)都需要加载过多的JS文件, 在传统的SSR架构下, 服务端需要给客户端发送巨多的JS代码(混合在HTML中)使页面重新注水; 而且大多数情况下, 网站动态的内容虽然只有一小部分但是却用了臃肿的框架/库去额外渲染, 造成了其他组件的阻塞

<img width="100" height="100" src="data:image/jpeg;base64,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"/>

所以孤岛架构的目标就是CSR的优秀交互体验以及SSR的SEO性能, 用最自然的描述语言(HTML)去编写现代高性能网站. 除了上述这些, 它还拥有几个特点:

- 组件化
- 动静分离, 静态内容无需经过处理, 动态内容自我注水(此体系所基于的技术就是部分/选择注水)

---

# 孤岛一词的含义

<img width="400" height="200" class="m-auto" src="https://res.cloudinary.com/wedding-website/image/upload/v1596766231/Islands-architecture-1.png"/>

<br/>

> 网站可分为动态交互内容和静态内容, 孤岛架构网站的动态内容每一块都是单独开发和维护的, 不会限制你的技术栈(即UI层渲染, 你可以使用Vue/React/Preact); 在常见的网站中, 比如官网/电商/博客网站, 在静态不变的内容下, 通常都会有交互式组件, 比如聊天对话框, 可变的链接跳转, 那么这些交互功能即可以理解为一个又一个“孤岛”, 孤岛和孤岛之间互不干扰.

---

# 动态孤岛组件

<img class="m-auto" width="300" height="200" src="https://miro.medium.com/max/700/1*mObEGoYvsOarH4qo0Sa8ww.png"/>

<br/>

> 我们需要再次梳理一下, 孤岛架构中的动态孤岛组件主要解决了什么问题, 或许你在前面的章节中已经了解过它们的优点, 那么我们这一部分将对优点进行剖析.

- 每一个交互式孤岛组件是完全隔离的, 意味着不会影响彼此的渲染, 静态内容亦是如此.
- 不会限制你的技术栈, 即UI层渲染
- 异步交互, 在初次渲染时不会有JS

我们了解到了动态孤岛组件的特征之后, 就可以来了解一下其实现细节
---

# 动态孤岛组件实现细节

> 当我们实现一个孤岛动态组件时, 在服务端会将js剥离:

```tsx
import { useState } from "preact/hooks";

const Card = () => {
  const [count, setCount] = useState(0);
  const add = () => {
    setCount(count + 1);
  };
  return (
    <div>
      <div>{count}</div>
      <button onclick={add}>ADD</button>
    </div>
  );
};

```

```html

<Island url="***/card" renderer-url="nodemodules/preact.js">
  <div>
    <div>0</div>
    <button>ADD</button>
  </div>
</Island>

```

---


# 动态孤岛组件渲染 & 激活

> 渲染的时机取决于Island相关框架的设计, 但是默认情况下会在服务端渲染为静态html, 那么动态组件重新激活/注水的时机有可能是立即的, 也有可能是异步的.

如果在业务场景下, 你希望你的购买按钮能够立即显示, 那么此时页面显示时就会让组件重新注水

```html
<button>立即购买</button>
```

那么还有一些场景下, 组件并不在可视区域, 如果此时立即注水, 那么就会造成性能浪费

```html
<footer>
  <Concat></Concat>
</footer>
```

<br/>

> 所以Island体系下的框架, 会给动态组件提供多种激活策略, 比如: 可视区域激活, 延迟激活, 立即激活, 媒体查询激活, 甚至是手动激活.

---

# 微前端? 还是渐进式增强?

- 微前端?

我们现在对Island架构有了一个初步的认识, 你也有可能和我一样在初次学习时和微前端概念有所混淆, 因为它们都提倡把应用程序分解为独立单元(进行部署), 但是微前端的每个单元并不完全使用html实现.

- 渐进式增强?

众所周知Vue是一门渐进式框架, 大家也都相当熟悉渐进式的意思, 它与Island一样, 有着渐进式概念; Island在SSR的基础上加入了类微前端概念提供了独立的组件单元, 使其交互变得更自由, 性能更贴近原始HTML. 在传统渐进式增强中, 我们通常会在客户端查询元素并且在元素上初始化一个动态函数, 而在Island中, 动态组件将经过服务端, 由服务端生成一个专属此组件的js文件, 让组件自给自足.
---

# Island框架盘点 - Astro

> 实现Island架构的优秀框架数量并不多, 每一个框架都有各自的优点, 在此次分享中我会着重介绍这三款框架; 尤其将Astro作为我们的主角, 介绍它的设计思路, 以及它的实现原理.

<br/>


- 不限制你的UI层渲染框架, 你可以自由选择solid.js, svelte, react, preact, vue等等, 也具有.astro的单组件提供静态渲染以及囊括多组件的能力(在astro中你可以同时编写多个框架组件, 并且可以一起工作)

- 提供了优秀的动态组件激活时机, 比如可见激活, 立即激活, 延迟激活, 媒体查询激活, 手动激活等等

- 优秀的部署能力, 不仅可以优雅降级到SSR还可以部署到Vercel,Cloudflare等边缘平台


<br/>

> 开始演示代码
---

# Astro代码演示

<iframe style="width: 50vw;height: 40vw;" src="https://stackblitz.com/edit/github-hfsxr7?embed=1&file=src/pages/index.astro"></iframe>

---

# Island框架盘点 - Qwik

> 有一个很有趣的事情是, 在Astro官网中比较了多个框架的功能和性能, 它唯独没有与Qwik进行比较;

<br/>


- 称之为最细节的Island框架并不夸大, 它拥有颗粒度最细的部分注水功能, 在其余大部分框架都在以组件为单位做激活工作时, Qwik可以把激活放到一个函数单位上
- 基于jsx语法进行开发, 并且和其他框架一样, 可以很好的和ts,tailwindcss等现代工具结合
- 在应用启动时上做了很多优化, 它尽可能的延迟执行和下载脚本, Qwik只需要1kb的运行时代码就可以让整个应用程序动态组件进行激活, 使其交互

<br/>

> 开始演示代码

---

# Qwik代码演示


<iframe style="width: 50vw;height: 40vw;" src="https://stackblitz.com/edit/qwik-vite-todoapp-a6ndeo?embed=1&file=index.html"></iframe>


---

# Island框架盘点 - Fresh

> Fresh是deno核心作者开源的一套“下一代web框架”, 它基于deno且仅使用Preact作为UI层渲染

- 基于deno, 有安全性, 以及高性能的运行时(jsx & ts)
- 因为运行时是deno缘故, fresh是没有构建流程的, 而且island组件都是异步即时加载(JIT)
- 同样由于deno, 它对部署平台非常苛刻, 如果你选择在边缘部署, 就只能选择官方的Deno Deploy
- fresh的核心就是路由系统 + 模板引擎, 所以页面都是在服务端即使渲染的

个人不推荐使用fresh部署你的网站, 因为从以下几个角度出发

- 基于deno的生态, 需要一定的学习成本和踩坑成本
- 它仅支持preact作为你的UI渲染
- 孤岛组件在客户端激活策略单一

---


# 你应该选择细度更高的孤岛吗?

> 你已经了解了 3 个孤岛框架的特点, 我们将着重分析Astro以及Qwik的优势, 它们在激活组件策略上有很大的不同;

Astro:

```html
<Card client:visible/>
```

Qwik:

```html
<input
  onInput$={(event) => {
    const input = event.target as HTMLInputElement;
    state.name = input.value;
  }}
></input>
```

<br/>

> 如果你的网站中需要针对动态孤岛组件进行非常细度定制化, 比如在“联系我们”组件中, 输入邮箱的操作是低频的, 其余组件是静态组件, 我们就可以使用Qwik框架将性能损耗压到最小; 但是反之你的网站有大量的交互组件, 而且你希望有多种激活策略(且可以用多种UI框架去构建), 那么Astro就是你的最佳选择;

---

# 国内外前端技术走向

> 不知道你有没有注意到, 从APP的潮流出现, 瓜分了大量的web流量之后, 国内外前端技术就走了不同的道路, 国外前端技术往往更注重用户体验, 国内前端技术却更注重业务需要和技术快速变现;

- 以前的时代, 我们都是用类jquery/jsp/php去完成前端, 出现了很多由服务端渲染的页面, 到此国内外研究的技术是一样的.

- app的出现将web流量抢占, 按理说我们应该提升web体验去留住用户; 但是我们却发明了APP中打开APP(小程序), 而且每一家平台的小程序标准都不一样, 所以又有了一类的跨端小程序框架去帮助我们完成需求.

- 而国外开发者为了留住web用户进而推进了PWA规范, 从nuxt, next, remix这一类服务端/同构渲染框架开始, 又演变了如今的island架构, 他们真正的目标是为了提升用户体验, 将业务最主要的功能(比如购买按钮)最先展示.

> 用户最先看到核心业务, 能给企业带来什么?


---

# island架构带来了什么?

页面访问速度很重要么? 对于商业网站来说的确是的, 缓慢的访问速度可能会损失数百万美元, 在web.dev中有一份例子:

- 对于Mobify，每 低100 毫秒的转化速度提高 1%,主页加载速度每降低 100 毫秒,基于会话的转化率就会增加 1.11%, 平均年收入增长近 380,000 美元.

- 当 AutoAnything 将页面加载时间缩短一半时,销售额提高了50%，销售额提高了12%到13%。

- Pinterest的注册速度提高了40%，从15%的注册时间缩短了40%，搜索引擎流量和注册次数增加了15%。

- BBC发现，他们的网站每加载一秒钟，就会额外失去10%的用户。

> 所以, 请重视你的互联网产品页面加载速度以及TTI时间
