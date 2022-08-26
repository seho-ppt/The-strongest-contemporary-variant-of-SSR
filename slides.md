---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# persist drawings in exports and build
drawings:
  persist: false
---

# SSR当代最强变种 - Island架构


<div class="pt-12">
  <span class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
   纸贵 - 沈昊
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

# 什么是SSR,CSR?