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

# 针对缺点和优点进行选型