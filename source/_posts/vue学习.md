---
title: vue学习
date: 2025-04-09 10:00:00
tags:
  - js vue
---

# vue 数据流

vue 是单向数据流，其中 props 向下，emit 向上。但是可以通过 v-model 实现双向绑定，其本质上是 props 和 emit 的语法糖。

# vue 响应式数据方案

采用数据劫持+依赖追踪方式

数据劫持：
vue2 采用 Object.defineProperty 来拦截 getter 和 setter 操作，实现响应式
vue3 采用 proxy 拦截 getter，setter，has，ownKeys 等操作实现响应式

为什么改为 proxy：

- 因为 defineProperty 对于新增的对象属性无法响应。（需要 vue.set()或者 vue.delete()手动触发响应式更新）
- 初始化时性能没有 proxy 好，defineProperty 需要递归遍历对象的所有属性，并劫持 getter 和 setter，对深层嵌套对象性能较差。每次劫持需要创建一个 Dep 依赖收集器，内存开销大
- proxy 能监听数组，而 defineProperty 无法监听直接修改数组元素或者数组长度
- proxy 拦截的操作更多，有 13 种对象操作，包括 set，get，deleteProperty，has，ownKeys 等等

proxy 的局限：

- 兼容性差，proxy 是 ES6 特性，无法在 IE11 以下版本使用
- 单次拦截操作比 Object.defineProperty 更慢，但是整体性能更优越

# react 的响应式方案

采用不可变数据 + 虚拟 DOM 差异对比机制

状态驱动，通过 useState 等 Hook 管理状态。
当状态变化时，触发组件重新渲染，生成新的虚拟 DOM。
通过 diffing，找出最小化 DOM 操作，更新真实 DOM。

# 脏检查方案

早期双向绑定框架使用的方案，比如 angular1。
它将模版表达式和数据模型关联起来，形成一个监视器列表，定期检查其中的数据变化。如果查到数据发生变化（变脏），则执行更新操作。
只要检查到数据变脏，会一直更新，知道所有数据不再变化。（有最大次数限制，比如 10 次，防止无限循环）

优点：
兼容性好，支持 ie8，因为它不基于 ES6 的 proxy 和 Object.defineProperty
简单直接，因为不需要预先定义响应式数据，任何 JavaScript 对象均可被监视

缺点：
性能问题，每次检查都会全量检查监视器，数据量大会导致性能下降
不够精细，无论数据是否发生变化，每次都会全量检查，做不到精确定位变化点
依赖手动触发，异步操作中依赖手动触发，很容易导致界面不同步

# vue 中的虚拟 DOM

vue 借鉴了虚拟 DOM 的概念，在 vue3 中分离了核心模块和渲染模块，使得 vue 理论上拥有比 react 更好的跨平台支持。
