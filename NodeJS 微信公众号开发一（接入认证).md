---
title: NodeJS 微信公众号开发一（接入认证)
date: '2018-11-10 10:35:44'
tag: 
  - wx
meta:
  -
    name: NodeJS 接入微信公众平台
    content: 接入微信公众平台
---

本篇记录一下通过NodeJS接入微信公众平台。
<!-- more -->


## 微信公众平台文档
[微信公众平台开发文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421135319)

## 实现过程

### 1. 启动一个web服务

```javascript
onst Koa = require('koa');
const config = require('./config/config');
const wechat = require('./wechat-lib/middleware');
const {reply} = require('./wechat/reply');
const app = new Koa();

app.use(wechat(config.wechat,reply));

app.listen(3006,()=>{
    console.log('服务器已经启动成功,端口号3006')
});
```

### 2. 定义一个中间件用于接入微信公众平台
```javascript
// 新建config.js写入
module.exports = {
    port:3006,
    wechat: {
        appID: '', 
        appSecret: '',
        token: ''
    }
};

// 新建middleware.js写入

const sha1 = require('sha1');
const getRawBody = require('raw-body');

module.exports = (config, reply) => {
    return async (ctx, next) => {
        // 接入微信服务器
        const {signature, timestamp, nonce, echostr} = ctx.query;
        const token = config.token;
        let str = [token, timestamp, nonce].sort().join('');
        const sha = sha1(str);
        if( sha == signature ){
            ctx.body=echostr
        }else{
            ctx.body='wrong'
        }
    }
};

```


## 项目参考文档
1. [Nodejs学习记录：用koa.js开发微信公众号](https://segmentfault.com/a/1190000012246932)
2. [koa2 + co-wechat快速搭建微信公众号对接服务](https://blog.csdn.net/zhaolandelong/article/details/79086077)


