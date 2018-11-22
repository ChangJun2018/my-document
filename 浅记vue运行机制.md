---
title: 浅记vue运行过程
date: '2018-11-21 09:05:13'
tag: 
  - vue
meta:
  -
    name: vue运行过程
    content: 浅记vue运行过程

---

阅读了掘金小册《剖析vuejs内部运行机制》感觉对vue 的运行流程有了一个认识，之后再去阅读vue源码应该会有帮助，对vue的运行流程记录一下。
<!-- more -->



![vue运行机制.jpg | center | 826x574](https://cdn.nlark.com/yuque/0/2018/jpeg/166011/1542767681684-e1fe56e1-950a-4a8b-a022-2180996c3d40.jpeg "")


1. 实例化```Vue```之后会调用```init```函数，负责初始化生命周期、事件、```props```、```methods```、```datga```、```computed```、```watch```等
2. 通过```object.defineProperty```设置```setter```与```getter```实现响应式与依赖收集。
3. 接下来进行编译首先是```parse```利用正则将```template```模板当中的指令、```class```、```style```等数据解析成AST（抽象语法树）
4. 其次```optimize```时标记静态节点，主要是为了让后面```diff```算法进行对比时跳过该节点。
5. ```generate```将抽象语法树转化成```render function```字符串，转化成```VNode```（虚拟DOM）
6. 每一个组件实例都有一个```watcher```实例对象，组件渲染时，会将属性记录为依赖，当依赖项的```setter```调用时，通知```watcher```计算，更新组件（update），即```setter -> Watcher -> update```
7. 当数据变化时，执行```render function```就会得到一个新的```VNode```节点，此时就会进入```patch```去进行新旧节点的比较（diff算法）从而计算出他们的差异，只需要修改这些有差异的地方。

