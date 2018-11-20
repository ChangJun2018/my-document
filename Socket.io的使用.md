---
title: Socket.io的使用
date: '2018-10-02 23:35:44'
tag: 
  - nodejs
meta:
  -
    name: Socket.io的使用
    content: Socket.io的使用
---

<span data-type="color" style="color:rgb(47, 47, 47)"><span data-type="background" style="background-color:rgb(255, 255, 255)">socket.io是一个实现实时web通信的JavaScript库。它包含两部分，在浏览器上运行的客户端库和在nodejs上运行的服务器端库</span></span>
<!-- more -->


## 原生nodejs使用socket.io

1. 搭建一个服务
2. 安装socket.io
3. 创建一个socket对象
4. 服务端和客户端通信

### 首先搭建一个最基础的http server 服务

```javascript
const http = require('http');
const fs = require('fs');

const app = http.createServer(function(req,res){
    fs.readFile('app.html',function(err,data){
        res.writeHead(200,{"Content-Type":"text/html;charset='utf-8'"});
        res.end(data)
    })
})

app.listen(3000, function() {
  console.log('listen on 8001')
})
```

### 安装socket.io

```bash
npm install socket.io --save
```

### 创建socket.io连接
```javascript
const io = require('socket.io')(app);
io.on('connection',function(socket){
    console.log('服务器建立连接了');
})
```
此时在浏览器当中访问[http://127.0.0.1:3000/socket.io/socket.io](http://127.0.0.1:3000/socket.io/socket.io.js).js 就可以访问到一个js文件了
```javascript
# 在app.html中引入js文件
<script src="http://localhost:3000/socket.io/socket.io.js"></script>
<script>
    var socket = io('http://localhost:3000/');  /*和服务器建立连接*/
</script>
```

### 服务端和客户端通信
```javascript
//客户端给服务器发送数据
socket.emit('to-server',{
    client:'我是客户端的数据'
})
//监听服务器的广播
socket.on('to-client',function(data){
    console.log(data)
})

// -------------------
// 服务端接收客户端发送的数据
socket.on('to-server',function(data){
    console.log(data)
}）
socket.emit('to-client','我是服务器的数据')
```

其中需要注意的是socket.emit是__谁发送给我的我发送给谁__。
io.emit表示给当前连接的__所有客户端发送__

## express中使用socket.io
```javascript
var app = require('express')();
var http = require('http').Server(app);
var io = require('socket.io')(http);

app.get('/', function(req, res) {
  res.sendFile(__dirname + '/app.html')
})

io.on('connection',function(socket){
    // 服务端接收客户端发送的数据
    socket.on('to-server',function(data){
        console.log(data)
    }）
    // 服务端给客户端发送数据
    socket.emit('to-client','我是服务器的数据')
})

```

客户端同原生nodejs连接socket.io

::: tip
注意：可以通过__socket.join(id)__实现指定加入分组，通过__socket.to(id).emit()__给指定分组内的用户发送消息
:::




### koa中使用socket.io
```javascript
const Koa=require('koa');
const router = require('koa-router')();
const views = require('koa-views');
const url=require('url');
// 引入koa-socket
const IO = require( 'koa-socket' )
// 创建socket.io对象
const io = new IO();
const app=new Koa();

io.attach( app );

app.use(views('views',{
    extension:'ejs'  /*应用ejs模板引擎*/
}))

router.get('/',async (ctx)=>{
   let title="你好ejs";
   await ctx.render('index',{
        title:title
    });
})

app.use(router.routes());   /*启动路由*/
app.use(router.allowedMethods());

app._io.on( 'connection', socket => {
    // 服务端接收客户端发送的数据
    socket.on('to-server',function(data){
        console.log(data)
    }）
    // 服务端给客户端发送数据
    socket.emit('to-client','我是服务器的数据')
})

```

