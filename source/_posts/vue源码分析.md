---
title: vue源码分析
date: 2021-04-07 12:00:00
tags:
  - js
  - vue
---

# 调试环境搭建

首先 clone vue 项目：https://github.com/vuejs/vue.git

然后在源码目录中找到 package.json，在 scripts 下，修改 dev 为：`"dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev --sourcemap"`，为 dev 下编译的代码设置 sourcemap，用来断点调试源码。

执行`npm run dev`，这样会在 dist 目录下编译出一个`vue.js`文件，同时会编译出一个对应的 sourcemap 文件。我们即可用该`vue.js`文件调试。（需要注意的是，vue 项目使用 rollup 打包，可能需要先全局安装 rollup：`cnpm i rollup -g `)

然后在根目录下 examples 文件夹下新建一个 index.html，并引用我们新编译的 vue.js 文件

```html
<html>
  <header>
    <script src="../dist/vue.js"></script>
  </header>
  <body>
    <div id="app"></div>
    <script>
      new Vue({
        el: "#app",
        template: "<div>Hello World</div>",
      });
    </script>
  </body>
</html>
```

现在即可在该 index.html 文件中调试代码

# 构建主流程分析

# 核心过程分析

## 变化检测

## 虚拟 dom

## 模板编译

## 生命周期

## 参考资料

[理解 vue 实现原理，实现一个简单的 Vue 框架](https://blog.csdn.net/pur_e/article/details/53066275)

[剖析 Vue 实现原理 - 如何实现双向绑定 mvvm]("https://github.com/DMQ/mvvm")

[Vue.js 源码（1）：Hello World 的背后](https://segmentfault.com/a/1190000006866881)
