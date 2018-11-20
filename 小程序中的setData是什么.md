---
title: 微信小程序中的setDate
date: '2018-11-19 10:35:44'
tag: 
  - wx
meta:
  -
    name: 微信小程序中的setDate
    content: 微信小程序中的setDate
---

小程序中setDate是什么？今天被问到了这个问题，说实话写小程序也写的挺久的了，忽然一愣，还真的不太清楚小程序setDate，为什么要setDate，其设计思路是什么。索性查阅资料学习记录一下。
<!-- more -->

小程序中setDate是什么？今天被问到了这个问题，说实话写小程序也写的挺久的了，忽然一愣，还真的不太清楚小程序setDate，为什么要setDate，其设计思路是什么。索性查阅资料学习记录一下。

## 小程序的底层实现

小程序的底层是如何实现的呢？小程序的**视图层**是采用**WebView**作为**渲染载体，逻辑层**是采用独立的**JavascriptCore**作为**运行环境。**这里在说一下**WebView**和**JavascriptCore**是什么吧，之前在了解[Hybird](https://blog.52chinaweb.com/frontend/hybridnote.html)时，有写过一篇文档。

**WebView**是手机中内置了一款高性能**webkit内核浏览器**,在SDK 中封装的一个组件。 没有提供地址栏和导航栏，**WebView**只是单纯的展示一个网页界面。

<span data-type="color" style="color:rgb(47, 47, 47)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><strong>JavaScriptCore</strong></span></span><span data-type="color" style="color:rgb(47, 47, 47)"><span data-type="background" style="background-color:rgb(255, 255, 255)">是</span></span><span data-type="color" style="color:rgb(47, 47, 47)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><strong>webkit</strong></span></span><span data-type="color" style="color:rgb(47, 47, 47)"><span data-type="background" style="background-color:rgb(255, 255, 255)">的一个重要组成部分，主要是对JS进行解析和提供执行环境（不需要通过浏览器）</span></span>

## 小程序的底层设计

小程序底层设计是**双线程**，如果了解Hybird开发方式的话就可以知道小程序的设计理念也是如此，采用接近web的开发方式，但是也保有了Hybird的优点**体验流畅**，可**快速迭代**更新



![xiaochengxu.png | center | 826x613](https://cdn.nlark.com/yuque/0/2018/png/166011/1542609664662-541e4645-d575-44ea-b562-7813d15189e0.png "")

如图所示，小程序的双线程就是**渲染层**与**逻辑层。**

渲染层，通过WebView线程去渲染界面相关的内容，一个小程序存在多个页面，所以渲染层存在多个webview线程。

逻辑层，创建一个单独的线程去运行JavaScript，执行小程序相关的业务逻辑代码。

## 小程序（双线程）的通信方式
既然小程序是双线程的，那么我们如何通过逻辑层改变数据，应用到视图层呢，两者之间是怎么通信的呢。

逻辑层和渲染层的通信会经由**微信客户端（Native）**做中转，逻辑层发送网络请求也经由 Native 转发 

当前，视图层和逻辑层的数据传输，实际上通过两边提供的 evaluateJavascript 所实现。**即用户传输的数据，需要将其转换为字符串形式传递，（因此要避免setData过长）同时把转换后的数据内容拼接成一份 JS 脚本，再通过执行 JS 脚本的形式传递到两边独立环境。**

而 evaluateJavascript 的执行会受很多方面的影响，数据到达视图层并不是实时的。所以我们的setData函数将数据从逻辑层发送到视图层，是异步的。

## 小程序的渲染机制
* 通过模版数据绑定和虚拟 DOM 机制，小程序提供了带有数据绑定语法的 DSL 给到开发者，用来在渲染层描述界面的结构。
* 小程序在逻辑层提供了设置页面数据的 api（setData）
* 逻辑层需要更改界面时，只要把修改后的 data 通过 setData 传到渲染层。
* 渲染层会根据前面提到的渲染机制重新生成 VD（虚拟 DOM）树，并更新到对应的 DOM 树上，引起界面变化。






