---
title: NodeJs 微信公众号开发二（回复策略）
date: '2018-11-10 12:35:44'
tag: 
  - wx
meta:
  -
    name: NodeJS 接入微信公众平台
    content: 接入微信公众平台
---

本篇记录一下自定义微信公众号的回复策略
<!-- more -->

上篇实现了一个最简单的NodeJS接入微信公众平台，本篇我们首先将上一篇我们的认证中间件完善一下，同时我们来写一下我们的微信公众号回复策略。还是之前那样只是为了给自己理顺一下思路和过程。让自己加深一下记忆。

[微信公众平台测试号申请](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login&token=&lang=zh_CN)

## 接入流程
* 对请求参数<span data-type="color" style="color:#1D39C4">token</span>、时间戳、随机数进行排序之后与加密串进行比较
* 进行<span data-type="color" style="color:#1D39C4">POST</span>和<span data-type="color" style="color:#1D39C4">GET</span>区分<span data-type="color" style="color:#1D39C4">GET</span>表示认证，<span data-type="color" style="color:#1D39C4">POST</span>请求推送消息
* 通过<span data-type="color" style="color:#1D39C4">raw-body</span> 获取微信发送的原始数据
* 对微信发送的<span data-type="color" style="color:#1D39C4">xm</span>l进行解析
* 将解析后的<span data-type="color" style="color:#1D39C4">xml</span>对象转化成<span data-type="color" style="color:#1D39C4">json</span>对象
* 调用我们的回复策略
* 根据微信发送的消息和我们的回复策略生成我们的xml模版
* 返回给微信服务器

## 实现流程

在wechat-lib/middleware.js 中写入

```javascript
const sha1 = require('sha1')  // 加密函数库
const getRawBody = require('raw-body') // 获取原始数据库

const util = require('./util') // 自定义的工具函数库

/**
 * @desc 微信服务器认证的中间件
 * config 微信公众号相关配置
 * reply 回复策略
 */
module.exports = (config, reply) => {
  /**
   * @desc 加载认证中间件儿
   * ctx是Koa的应用上下文
   * next就是串联中间件儿的钩子函数
   */
  return async (ctx, next) => {
    // 接入微信服务器
    // 加密串、时间戳、随机数、随机字符串
    const { signature, timestamp, nonce, echostr } = ctx.query

    const token = config.token

    // 对token、时间戳、随机数进行排序
    let str = [token, timestamp, nonce].sort().join('')

    // 对排序之后的字符串进行加密
    const sha = sha1(str)

    // 进行get和post请求区分，如果是get就是认证
    // 如果是post即是推送消息的
    if (ctx.method === 'GET') {
      if (sha === signature) {
        ctx.body = echostr
      } else {
        ctx.body = 'Failed'
      }
    } else if (ctx.method === 'POST') {
      if (sha !== signature) {
        return (ctx.body = 'Failed')
      }

      // 获取微信post请求的原始数据
      const data = await getRawBody(ctx.req, {
        length: ctx.length, //请求格式
        limit: '1mb', // 请求数据大小
        encoding: ctx.charset // 编码格式
      })

      // 对微信post的xml进行解析成对象
      const content = await util.parseXML(data)

      // 将解析后的xml对象转成一个json对象
      const message = util.formatMessage(content.xml)

      // 将微信发送的消息挂载到上下文中的weixin属性上
      ctx.weixin = message

      // 调用我们定义好的回复策略，传入上下文
      await reply.apply(ctx, [ctx, next])

      // 获取我们回复微信的对象
      const replyBody = ctx.body
      // 获取维信发送的用户信息
      const msg = ctx.weixin

      // 生成回复微信的xml模板
      const xml = util.tpl(replyBody, msg)
      
      ctx.status = 200
      ctx.type = 'application/xml'
      ctx.body = xml
    }
  }
}

```

截止到目前为止需要写的工具函数有
* 解析xml到对象-（xml2js）
* 将解析后的xml对象转化成json对象
* 生成xml模版（需要我们自定义回复模版）

在wechat-lib/util.js 中写入
```javascript
const xml2js = require('xml2js') // 解析xml的模块
const template = require('./tpl') // 微信自定义回复模板
/**
 * 解析xml到对象
 */
exports.parseXML = xml => {
  return new Promise((resolve, reject) => {
    xml2js.parseString(xml, { trim: true }, (err, content) => {
      if (err) reject(err)
      else resolve(content)
    })
  })
}

// 把解析的xml转成一个json对象
const formatMessage = result => {
  let message = {}

  if (typeof result === 'object') {
    const keys = Object.keys(result)

    for (let i = 0; i < keys.length; i++) {
      let item = result[keys[i]]
      let key = keys[i]
      if (!(item instanceof Array) || item.length === 0) {
        continue
      }
      if (item.length === 1) {
        let val = item[0]
        if (typeof val === 'object') {
          message[key] = formatMessage(val)
        } else {
          message[key] = (val || '').trim()
        }
      } else {
        message[key] = []
        for (let j = 0; j < item.length; j++) {
          message[key].push(formatMessage(item[j]))
        }
      }
    }
  }
  return message
}


/**
 * @desc 生成回复微信的xml模版
 * content 内容
 * message 消息
 */
exports.tpl = (content, message) => {

  // 默认是文本
  let type = 'text'

  // 图文消息
  if (Array.isArray(content)) {
    type = 'news'
  }

  // 空消息
  if (!content) content = 'Empty News'

  // 非图文非文本
  if (content && content.type) {
    type = content.type
  }

  let info = Object.assign(
    {},
    {
      content: content, // 内容
      msgType: type, // 消息类型
      createTime: new Date().getTime(), //当前时间
      toUserName: message.FromUserName, // 开发者微信号
      fromUserName: message.ToUserName // 发送方帐号
    }
  )
    
  return template(info) //jiang 
}
```

接下来我们自定义一下我们的回复模版，在wechat-lib/tpl.js 中写入

<span data-type="color" style="color:#F5222D">此处一定注意微信开发者文档中的回复xml模版拷贝过来之后一定要去处理一下其中的空格。</span>

```javascript
const ejs = require('ejs'); // ejs模板引擎


/**
 * 定义微信自定义回复
 * 采用ejs解析
 */

const tpl = `
<xml>
  <ToUserName><![CDATA[<%= toUserName %>]]></ToUserName>
  <FromUserName><![CDATA[<%= fromUserName %>]]></FromUserName>
  <CreateTime><%= createTime %></CreateTime>
  <MsgType><![CDATA[<%= msgType %>]]></MsgType>
  <% if (msgType === 'text') { %>
    <Content><![CDATA[<%- content %>]]></Content> 
  <% } else if (msgType === 'image') { %>
    <Image>
    <MediaId><![CDATA[<%= content.mediaId %>]]></MediaId>
    </Image>
  <% } else if (msgType === 'voice') { %>
    <Voice>
      <MediaId><![CDATA[<%= content.mediaId %>]]></MediaId>
    </Voice>
  <% } else if (msgType === 'video') { %>
    <Video>
      <MediaId><![CDATA[<%= content.mediaId %>]]></MediaId>
      <Title><![CDATA[<%= content.title %>]]></Title>
      <Description><![CDATA[<%= content.description %>]]></Description>
    </Video>
  <% } else if (msgType === 'music') { %>
    <Music>
      <Title><![CDATA[<%= content.title %>]]></Title>
      <Description><![CDATA[<%= content.description %>]]></Description>
      <MusicUrl><![CDATA[<%= content.musicUrl %>]]></MusicUrl>
      <HQMusicUrl><![CDATA[<%= content.hqMusicUrl %>]]></HQMusicUrl>
      <ThumbMediaId><![CDATA[<%= content.thumbMediaId %>]]></ThumbMediaId>
    </Music>
  <% } else if (msgType === 'news') { %>
    <ArticleCount><![CDATA[<%= content.length %>]]></ArticleCount>
    <Articles>
      <% content.forEach(function(item) { %>
        <item>
          <Title><![CDATA[<%= item.title %>]]></Title>
          <Description><![CDATA[<%= item.description %>]]></Description>
          <PicUrl><![CDATA[<%= item.picUrl %>]]></PicUrl>
          <Url><![CDATA[<%= item.url %>]]></Url>
        </item>
      <% }) %>
    </Articles>
  <% } %>
</xml>
`;

const compiled = ejs.compile(tpl);

module.exports = compiled;
```

接下来我们定义一下我们的回复策略

在wechat/reply.js 中写入

```javascript
const { resolve } = require('path')   // 用来转化路径

exports.reply = async (ctx, next) => {
  const message = ctx.weixin // 获取微信发送的消息

  // 消息类型判断
  if (message.MsgType === 'text') {
    // 获取到内容
    let content = message.Content
    // 默认的回复文案
    let reply = 'Oh,你说的 ' + content + '太复杂了,无法解析'

    if (content === '1') {
      reply = '常峻第一大爱好,高艳琪'
    } else if (content === '2') {
      reply = '常峻第二大爱好,代码'
    } else if (content === '3') {
      reply = '常峻第三大爱好,烫头'
    } else if (content === '常峻') {
      reply = '常峻有三大爱好'
    } 

    // 将要回复的数据挂载到上下文中的body属性中
    ctx.body = reply
  }

  await next()
}

```

将配置项进行抽离新建config/config.js 中写入
```javascript
module.exports = {
  port: 3008, // 端口号
  db: 'mongodb://localhost:27017/d', // mongodb地址
  wechat: {
    appID: '', // 公众号id
    appSecret: '', // 公众号密码
    token: '' // token
  }
};
```

在app.js中写入
```javascript
const Koa = require('koa');
const config = require('./config/config');
const wechat = require('./wechat-lib/middleware');
const {reply} = require('./wechat/reply');

// 生成Koa实例
const app = new Koa();
// 配置回复中间件儿
app.use(wechat(config.wechat, reply));
app.listen(3008, () => {
  console.log('服务器已经启动成功,端口号3008')
});
```




