---
title: koa学习笔记
date: '2018-09-22 23:35:44'
tag: 
  - koa
meta:
  -
    name: koa
    content: koa
  -
    name: koa
    content: koa的学习笔记
---

记录一下自己学习koa的学习笔记。
<!-- more -->



## 背景描述

koa 是由Express 原班人马打造的，致力于成为一个更小、更富有表现力、更健壮的
Web 框架。使用koa 编写web 应用，可以免除重复繁琐的回调函数嵌套， 并极大地提
升错误处理的效率。koa 不在内核方法中绑定任何中间件， 它仅仅提供了一个轻量优雅的
函数库，使得编写Web 应用变得得心应手。

## koa安装

```javascript
npm install koa --save 
```

## Koa基础知识
> 1. koa 路由
> 2. koa 中间件
> 3. koa ejs模板引擎使用
> 4. koa-bodyparser使用
> 5. koa-static静态资源中间件儿使用
> 6. koa art-template 模板引擎使用
> 7. koa cookie使用
> 8. koa session使用

## koa中的基本路由

### 路由基本使用

```javascript
# 安装路由
npm install koa-router --save

# 使用路由
const Koa = require('koa');
const router = require('koa-router')(); //注意：引入的方式
const app = new Koa();

router.get('/', function (ctx, next) {
    ctx.body="Hello ChangJun";
})

router.get('/blogs',(ctx,next)=>{
    ctx.body="博客页面"
});

app.use(router.routes()); 
app.use(router.allowedMethods());
app.listen(3000);
```

### 路由get传值

```javascript
const Koa = require('koa');
const router = require('koa-router')(); //注意：引入的方式
const app = new Koa();

router.get('/', function (ctx, next) {
    ctx.body="Hello ChangJun";
})

# 在koa2 中GET 传值通过request 接收，但是接收的方法有两种：query 和querystring。
router.get('/blogs',(ctx,next)=>{
    let request =ctx.request;
    let req_query = request.query;  // 格式化好的参数对象
    let req_querystring = request.querystring; //请求的字符串
    ctx.body="博客页面"
});

app.use(router.routes()); 
app.use(router.allowedMethods());
app.listen(3000);
```

### 动态路由

```javascript
router.get('/news/:aid',async (ctx)=>{
    console.log(ctx.params); //{ aid: '123' } //获取动态路由的数据
});
```

## koa中间件儿

如果get和post回调函数中，没有next参数，那么就匹配第一个路由，如果向下匹配的话，就需要next()
koa应用中主要有如下几种中间件
> 1. 应用级中间件
> 2. 路由级中间件
> 3. 错误处理中间件
> 4. 第三方中间件

### 应用级中间件
```javascript
const Koa = require('koa');
const Router = require('koa-router');
const app = new Koa();
const router = new Router();
app.use(async (ctx,next)=>{
    console.log(new Date());
    await next();
})
router.get('/', function (ctx, next) {
ctx.body="Hello koa";
})
app.use(router.routes()); 
app.use(router.allowedMethods()); 
app.listen(3000);
```

### 路由级中间件

```javascript
router.get('/', async(ctx, next)=>{
    console.log('1')
    next()
})
router.get('/', function (ctx) {
    ctx.body="Hello koa";
})
```

### 错误处理中间件

```javascript
app.use(async (ctx,next)=> {
    next();
    if(ctx.status==404){
        ctx.status = 404;
        ctx.body="这是一个404 页面"
    }
});
```

### 第三方中间件

```javascript
const static = require('koa-static');
const staticPath = './static';
app.use(static(
    path.join( __dirname, staticPath)
))
const bodyParser = require('koa-bodyparser');
app.use(bodyParser());
```

## koa-bodyparser使用

获取post提交的数据

### 安装
npm install koa-bodyparser --save

### 使用
```javascript
const Koa = require('koa');
const bodyParser = require('koa-bodyparser');
const app = new Koa();
app.use(bodyParser());
// 通过ctx.request.body获取post提交的数据
app.use(async ctx => {
    ctx.body = ctx.request.body;
}
```

## koa-static的使用
### 安装
npm install koa-static --save

### 使用
```javascript
const static = require('koa-static');
app.use(static(
    path.join( __dirname, 'public')
))
```


## art-template模板引擎的使用
### 安装
npm install --save art-template
npm install --save koa-art-template

### 使用
```javascript
const Koa = require('koa');
const render = require('koa-art-template');
const app = new Koa();

render(app, {
root: path.join(__dirname, 'view'),
extname: '.html',
debug: process.env.NODE_ENV !== 'production'
});

app.use(async function (ctx) {
    await ctx.render('index');
});
app.listen(8080);
```

## koa Cookie的使用
cookie保存在浏览器客户端，可以让我们用同一个浏览器访问同一个域名的时候共享数据
应用场景：
1. 保存用户信息
2. 浏览器历史纪录
3. 猜你喜欢的功能
4. 10天免登录
5. 多个页面之间的数据


```javascript
# 设置cookie的值
ctx.cookies.set(name, value, [options])
// options中包含
// maxAge 标示过期的毫秒数
// expires 过期的时间
// path cookie允许访问的页面


# koa中默认不允许设置中文cookie
1. new Buffer('hello, world!').toString('base64') 获取base64字符串存取cookie
2. new Buffer('aGVsbG8sIHdvcmxkIQ==', 'base64').toString() 从cookie中取出转回字符串
```

## koa Session的使用

### 安装
npm install koa-session --save

### 使用
```javascript
const session = require('koa-session');

// session中间件配置
const CONFIG = {
key: 'koa:sess', //cookie key (default is koa:sess)
maxAge: 86400000, // cookie 的过期时间maxAge in ms (default is 1 days)
overwrite: true, //是否可以overwrite (默认default true)
httpOnly: true, //cookie 是否只有服务器端可以访问httpOnly or not (default true)
signed: true, //签名默认true
rolling: false, //在每次请求时强行设置cookie，这将重置cookie 过期时间（默认：false）
renew: false, //(boolean) renew session when session is nearly expired,
};
app.use(session(CONFIG, app));

ctx.session.username = "Hello"; // 设置session
ctx.session.username // 获取session
```

## koa中路由模块化
> 1. 在目录下面新建一个文件夹routes
> 2. 在routes 里面配置对应的子页面
> 3. 比如在routes 新建index.js
> 4. 主应用中加载子路由模块

```javascript
// 子路由模块
const Router = require('koa-router');
let router = new Router();
router.get('/',async(ctx)=>{
    ctx.body="这是前台首页";
})
router.get('/news',async(ctx)=>{
    ctx.body ='这是前台新闻页面';
})
router.get('/user',async(ctx)=>{
    ctx.body ='这是用户页面';
})
module.exports=router;

// 主应用中

const Koa = require('koa');
const app = new Koa();
const Router = require('koa-router');

let index=require('./module/index.js');
let admin=require('./module/admin.js');

//装载所有子路由
let router = new Router();
router.use('/index,index.routes());
router.use('/',home.routes());

//加载路由中间件
app.use(router.routes());
app.use(router.allowedMethods());
app.listen(3000);
```
