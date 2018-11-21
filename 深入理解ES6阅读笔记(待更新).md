---
title: 深入理解ES6阅读笔记(待更新)
date: '2018-11-20 23:35:44'
tag: 
  - javascript
meta:
  -
    name: javascript
    content: ES6使用
---


最近阅读深入《深入理解ES6》这本书，将其中自己之前没见到的新概念新东西，见到的没理解的理解记录一下。
<!-- more -->

* ### <a name="xovhul"></a>**关于临时死区TDZ（Temporal Dead Zone）**
     首先来看一下TDZ是什么吗，TDZ用来描述let和const的不提升效果。关于TDZ的有两处一处是**let和const**，另一个是**默认参数的TDZ**。
     首先先来看一段代码

```javascript
if(setcolor){
  console.log(typeof value)   // 报错
  let color = 'red'
}
```

根据这个我们来理解一下TDZ，在js中当我们用var定义一个变量时，js引擎在扫描代码时发现var时，会将变量提升到作用域顶部，但是当我们用let或者const定义时，例如在代码中看到的那样，js引擎会将其放在TDZ当中，此时访问在TDZ当中的变量会导致js引擎抛出错误。

那来看一下下面这段代码，当我们将type of 放在声明变量的代码块外执行呢？

```javascript
console.log(typeof value) // undefined
if(setcolor){
  let color = 'red'
}
```

我们可以看到是undefined，这是为什么呢？原因是因为typeof 是在声明变量的代码块外执行的，此时变量还不存在在TDZ当中，因此当我们访问变量就是未定义。

默认参数也有TDZ，在说明这个之前先说明一下，函数参数有自己的作用域和临时死区，其与函数体的作用域是相互独立的，这也就是说参数的默认值不可以访问函数体内声明的变量。

接下来我们来看一段代码
```javascript
function add(first=last,last){
    return first+last
}
add(1,1) // 2
add(undefined,1)  // 报错 
```

我们来看一下执行过程是怎样的
```javascript
当我们调用add(1,1)时
let first = 1，
let last = 1
当我们调用add(undefined,1)时
let first = last
let last = 1
```

这个时候我们就可以很明显的看到当我们定义first的时候变量last还在TDZ当中，要记住当js引擎访问在TDZ当中的变量时，会抛出错误。这就是默认参数的TDZ了。

js的世界真是奇妙，听到TDZ这个概念还是在社区当中，继续努力探索，加油！
