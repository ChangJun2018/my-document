---
title: NodeJS接入微信公众号三（获取accessToken）
date: '2018-11-10 13:35:44'
tag: 
  - wx
meta:
  -
    name: NodeJS 接入微信公众平台
    content: 接入微信公众平台
---

access_token是公众号的全局唯一接口调用凭据,本篇记录一下获取access_token
<!-- more -->

[获取access_token](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140183)

上篇记录了nodejs认证和回复策略，本篇记录一下获取accessToken，并且将其存储到mongodb数据库中。


<span data-type="color" style="color:rgb(51, 51, 51)"><span data-type="background" style="background-color:rgb(255, 255, 255)">access_token是公众号的全局唯一接口调用凭据，公众号调用各接口时都需使用access_token。开发者需要进行妥善保存。access_token的存储至少要保留512个字符空间。access_token的有效期目前为2个小时，需定时刷新，重复获取将导致上次获取的access_token失效。</span></span>

## 实现流程
* 检验数据库中的token是否过期
* 过期刷新当前token
* 将token存入数据库中

但是当我们在处理token问题时，应该先将我们的mongodb准备好。

## mongodb相关流程
* 定义表结构Schema（name、token、expires\_in、meta）
* 定义存储是前置钩子方法用于判断更新token还是存储
* 定义静态方法 获取数据苦中的token和存储token到数据库中
* 初始化所有的Schema
* 数据库连接，监听断开连接、异常、成功

在根目录下新建app/database/schema/token.js 写入

```javascript
/**
 * @author: Chang Jun
 * @date: 2018/11/10
 * @Description: Token的数据模型
 */

// Schema 定义MongoDB中集合Collection里文档document的结构 每个schema会映射到mongodb中的一个collection
// Model  model是由schema生成的模型，可以对数据库的操作。
//Entity：由Model创建的实体，可操作数据库。
const mongoose = require('mongoose')
const Schema = mongoose.Schema

// 定义表结构
const TokenSchema = new Schema({
  name: String, //accessToken
  token: String, // 内容
  expires_in: Number, // 过期时间
  meta: {
    createdAt: {
      type: Date, // 创建时间
      default: Date.now()
    },
    updatedAt: {
      type: Date, // 更新时间
      default: Date.now()
    }
  }
})

// 前后钩子即pre()和post()方法，又称为中间件，是在执行某些操作时可以执行的函数。中间件在schema上指定，类似于静态方法或实例方法等
// 前后钩子方法即中间件儿
// 在每次进行存储的时候判断token是新增的还是更新
TokenSchema.pre('save', function(next) {
  // 如果该条数据是新增的创建时间就等于更新时间
  if (this.isNew) {
    this.meta.createdAt = this.meta.updatedAt = Date.now()
  } else {
    this.meta.updatedAt = Date.now()
  }

  next()
})

// tokenschema的静态方法
TokenSchema.statics = {
  // 查询数据库中的token
  async getAccessToken() {
    const token = await this.findOne({
      name: 'access_token'
    })

    if (token && token.token) {
      token.access_token = token.token
    }

    return token
  },

  // 将token存储在数据库中
  async saveAccessToken(data) {
    let token = await this.findOne({
      name: 'access_token'
    })

    // 如果token存在更新token相关信息
    if (token) {
      token.token = data.access_token
      token.expires_in = data.expires_in
    } else {
      token = new Token({
        name: 'access_token',
        token: data.access_token,
        expires_in: data.expires_in
      })
    }
    await token.save()
    return data
  }
}

const Token = mongoose.model('Token', TokenSchema)

```

在app/database/init.js 中写入
```javascript
/**
 * @author: Chang Jun
 * @date: 2018/11/10
 * @Description: 初始化MongoDB
 */
const mongoose = require('mongoose')
const { resolve } = require('path') 
const glob = require('glob')  // 匹配文件库

// 将mongoose默认promise修改成默认的Promise
mongoose.Promise = global.Promise

// 初始化所有的Schema
exports.initSchemas = () => {
  glob.sync(resolve(__dirname, './schema', '**/*.js')).forEach(require)
}

// 连接数据库
exports.connect = db => {
  let maxConnectTimes = 0 // 最大连接时长

  return new Promise(resolve => {
    // 开发环境下将调试模式开启
    if (process.env.NODE_ENV !== 'production') {
      mongoose.set('debug', true)
    }
    mongoose.connect(
      db,
      { useNewUrlParser: true }  // 避免警告
    )

    // 如果断开连接之后尝试重新连接
    mongoose.connection.on('disconnect', () => {
      maxConnectTimes++
      if (maxConnectTimes < 5) {
        mongoose.connect(db)
      } else {
        throw new Error('数据库挂了呢')
      }
    })

    // 连接发生错误
    mongoose.connection.on('error', err => {
      maxConnectTimes++
      if (maxConnectTimes < 5) {
        mongoose.connect(db)
      } else {
        throw new Error(err)
      }
    })

    // 连接成功
    mongoose.connection.on('open', () => {
      resolve()
      console.log('MongoDB 连接成功')
    })
  })
}
```

此时修改我们的app.js为
```javascript
const Koa = require('koa');
const config = require('./config/config');
const wechat = require('./wechat-lib/middleware');
const {reply} = require('./wechat/reply');
const {initSchemas, connect} = require('./app/database/init');

(async () => {
  // 连接数据库
  await connect(config.db);
  // 初始化文档模型
  initSchemas();
  // 生成Koa实例
  const app = new Koa();
  // 配置回复中间件儿
  app.use(wechat(config.wechat, reply));
  app.listen(3008, () => {
    console.log('服务器已经启动成功,端口号3008')
  });
})();
```

接下来我们来做token的相关处理工作，在wechat-lib/index.js 中写入

```javascript
/*
 * 获取token
 *
 * */
const fs = require('fs')
const request = require('request-promise') // 将request请求promise化

const base = 'https://api.weixin.qq.com/cgi-bin/' // 请求微信api基础地址

const api = {
  accessToken: base + 'token?grant_type=client_credential', // 获取票据accessToken
}

module.exports = class Wechat {
  constructor(opts) {
    this.opts = Object.assign({}, opts)
    this.appID = opts.appID
    this.appSecret = opts.appSecret
    this.getAccessToken = opts.getAccessToken
    this.saveAccessToken = opts.saveAccessToken
    this.fetchAccessToken()
  }

  /**
   * 对request请求的封装
   * @param {*} options 请求参数
   */
  async request(options) {
    options = Object.assign({}, options, { json: true })
    try {
      const res = await request(options)
      return res
    } catch (err) {
      console.log(err)
    }
  }

  /* 获取token
   *  1. 检验数据库中的token是否过期
   *  2. 过期则刷新当前token
   *  3. token入库
   * */
  async fetchAccessToken() {
    // 获取数据库中的token
    let data = await this.getAccessToken()

    // 检查token是否过期
    if (!this.isValidToken(data)) {
      data = await this.updateAccessToken()
    }
    // 将token进行保存
    await this.saveAccessToken(data)
    return data
  }

  /**
   * 获取token的方法
   */
  async updateAccessToken() {
    const url = `${api.accessToken}&appid=${this.appID}&secret=${
      this.appSecret
    }`
    const data = await this.request({ url })
    const now = new Date().getTime()
    // token失效时间设置小于两小时
    const expiresIn = now + (data.expires_in - 20) * 1000
    data.expires_in = expiresIn
    return data
  }

  /**
   * 检查token是否过期
   * 过期时间与当前时间比较
   * @param {*} data
   */
  isValidToken(data) {
    if (!data || !data.expires_in) {
      return false
    }
    const expiresIn = data.expires_in // 获取过期时间
    const now = new Date().getTime()
    // 如果现在的时间小于过期时间
    if (now < expiresIn) {
      return true
    } else {
      return false
    }
  }
}

```




