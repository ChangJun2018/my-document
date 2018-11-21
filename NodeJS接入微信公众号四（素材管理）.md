---
title: NodeJS接入微信公众号四（素材管理）
date: '2018-11-11 10:35:44'
tag: 
  - wx
meta:
  -
    name: NodeJS 接入微信公众平台
    content: 接入微信公众平台
---

上一篇记录了如何获取accessToken和通过mongoose存储到mongoodb中去。本篇主要记录一下微信公众号素材管理。
<!-- more -->

[素材管理](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1444738726)

在这之前记录一下，如果遇到错误依然去官方文档的错误码当中去看，或者留心文档当中的描述，例如可能是http和https区别等等。

接下在来wechat-lib/index.js 中将要请求的地址定义好
```javascript
const api = {
  accessToken: base + 'token?grant_type=client_credential', // 获取票据accessToken
  temporary: {
    upload: base + 'media/upload?', // 上传临时素材
    fetch: base + 'media/get?' // 获取临时素材
  },
  permanent: {
    upload: base + 'material/add_material?', // 新增其他类型永久素材
    uploadNews: base + 'material/add_news?', // 新增永久图文素材
    uploadNewsPic: base + 'media/uploadimg?', // 上传图片返回一个url
    fetch: base + 'material/get_material?', // 获取永久素材
    del: base + 'material/del_material?', // 删除永久素材
    update: base + 'material/update_news?', // 修改永久图文素材
    count: base + 'material/get_materialcount?', // 获取素材数量
    batch: base + 'material/batchget_material?' // 获取素材列表
  }
}
```

地址文档当中都有，接下来其实做的就很简单了，根据文档将请求数据做好，进行请求就好了。这一块不赘述，看代码就好。

```javascript
 /**
   * 上传素材
   * @param {*} token 票据token
   * @param {*} type 素材类型
   * @param {*} material 素材文件路径
   * @param {*} permanent 临时或者永久 默认临时
   */
  uploadMaterial(token, type, material, permanent = false) {
    let form = {} // 存储表单数据

    // 获取上传地址
    let url = api.temporary.upload // 默认是临时素材上传地址

    // 判断是否是上传永久还是临时素材
    if (permanent) {
      url = api.permanent.upload
      form = Object.assign(form, permanent) // form是个obj，继承外面传入的新对象
    }

    // 上传图文消息的图片素材
    if (type === 'pic') {
      url = api.permanent.uploadNewsPic
    }

    // 图文和非图文的素材提交表单切换
    if (type === 'news') {
      url = api.permanent.uploadNews
      form = material
    } else {
      // 创建文件流（文件路径）挂载到form下面的media属性中
      form.media = fs.createReadStream(material)
    }

    // 生成新的上传地址
    let uploadUrl = `${url}access_token=${token}`

    // 根据素材永久性填充token
    if (!permanent) {
      uploadUrl += `&type=${type}`
    } else {
      if (type != 'news') {
        form.access_token = token
      }
    }

    const options = {
      method: 'POST',
      url: uploadUrl,
      json: true
    }

    // 图文和非图文在request提交主题判断
    if (type === 'news') {
      options.body = form
    } else {
      options.formData = form
    }

    return options
  }

  /**
   * 获取素材本身
   * @param {*} token 票据access_token
   * @param {*} mediaId 媒体文件ID
   * @param {*} type 类型
   * @param {*} permanent 临时or永久
   */
  fetchMaterial(token, mediaId, type, permanent) {
    let form = {}
    let fetchUrl = api.temporary.fetch // 默认获取临时素材

    if (permanent) {
      fetchUrl = api.permanent.fetch
    }
    let url = fetchUrl + 'access_token=' + token
    let options = { method: 'POST', url }

    if (permanent) {
      form.media_id = mediaId // 要获取的素材的media_id
      form.access_token = token // 调用接口凭证
      options.body = form
    } else {
      if (type === 'video') {
        url = url.replace('https:', 'http:') // 获取临时视频文件素材时不支持https下载，调用该接口需http协议
      }

      url += '&media_id=' + mediaId
    }

    return options
  }

  /**
   * 删除素材
   * @param {*} token 票据accessToken
   * @param {*} mediaId 媒体文件ID
   */
  deleteMaterial(token, mediaId) {
    const form = {
      media_id: mediaId
    }
    const url = `${api.permanent.del}access_token=${token}&media_id=${mediaId}`

    return { method: 'POST', url, body: form }
  }

  /**
   * 更新素材
   * @param {*} token 票据accessToken
   * @param {*} mediaId 媒体文件ID
   * @param {*} news 修改的图文素材
   */
  updateMaterial(token, mediaId, news) {
    let form = {
      media_id: mediaId
    }
    form = Object.assign(form, news)

    const url = `${
      api.permanent.update
    }access_token=${token}&media_id=${mediaId}`

    return { method: 'POST', url, body: form }
  }

  /**
   * 获取素材总数
   * @param {*} token // 接口调用凭证
   */
  countMaterial(token) {
    const url = `${api.permanent.count}access_token=${token}`
    return { method: 'POST', url }
  }

  /**
   * 获取素材列表
   * @param {*} token 票据token
   * @param {*} options 参数
   */
  batchMaterial(token, options) {
    options.type = options.type || 'image' // 素材的类型
    options.offset = options.offset || 0 // 从全部素材的该偏移位置开始返回，0表示从第一个素材 返回
    options.count = options.count || 10 // 返回素材的数量，取值在1到20之间

    const url = `${api.permanent.batch}access_token=${token}`

    return { method: 'POST', url, body: options }
  }
```

接下来在wechat/reply.js 调用一下我们的回复

```javascript
else if (content === '4') {
      // 上传零食素材
      let data = await client.handle(
        'uploadMaterial', //上传临时素材方法
        'image', // 图片
        resolve(__dirname, '../2.jpg') // 文件路径
      )
      reply = {
        type: 'image',
        mediaId: data.media_id // 媒体文件上传后，获取标识
      }
    } else if (content === '5') {
      let data = await client.handle(
        'uploadMaterial',
        'video',
        resolve(__dirname, '../1.mp4')
      )
      reply = {
        type: 'video',
        title: '高艳琪',
        description: '我爱你',
        mediaId: data.media_id
      }
    } else if (content === '6') {
      let data = await client.handle(
        'uploadMaterial',
        'video',
        resolve(__dirname, '../6.mp4'),
        {
          type: 'video',
          description: '{"title": "高艳琪呀", "introduction": "我爱你我爱你"}'
        }
      )

      reply = {
        type: 'video',
        title: '高艳琪我爱你 2',
        description: '高艳琪我爱你',
        mediaId: data.media_id
      }
    } else if (content === '7') {
      let data = await client.handle(
        'uploadMaterial',
        'image',
        resolve(__dirname, '../2.jpg'),
        {
          type: 'image'
        }
      )

      reply = {
        type: 'image',
        mediaId: data.media_id
      }
    } else if (content === '8') {
      let data = await client.handle(
        'uploadMaterial',
        'image',
        resolve(__dirname, '../2.jpg'),
        {
          type: 'image'
        }
      )
      let data2 = await client.handle(
        'uploadMaterial',
        'pic',
        resolve(__dirname, '../2.jpg'),
        {
          type: 'image'
        }
      )
      console.log(data2)

      let media = {
        articles: [
          {
            title: '这是服务端上传的图文 1',
            thumb_media_id: data.media_id,
            author: 'Scott',
            digest: '没有摘要',
            show_cover_pic: 1,
            content: '点击去常峻的个人网站',
            content_source_url: 'https://blog.52chinaweb.com/'
          },
          {
            title: '这是服务端上传的图文 2',
            thumb_media_id: data.media_id,
            author: 'Scott',
            digest: '没有摘要',
            show_cover_pic: 1,
            content: '点击去常峻的个人网站',
            content_source_url: 'http://github.com/'
          }
        ]
      }

      let uploadData = await client.handle('uploadMaterial', 'news', media, {})

      let newMedia = {
        media_id: uploadData.media_id,
        index: 0,
        articles: {
          title: '这是服务端上传的图文 1',
          thumb_media_id: data.media_id,
          author: 'Scott',
          digest: '没有摘要',
          show_cover_pic: 1,
          content: '点击去常峻的个人网站',
          content_source_url: 'https://blog.52chinaweb.com/'
        }
      }

      console.log(uploadData)

      let mediaData = await client.handle(
        'updateMaterial',
        uploadData.media_id,
        newMedia
      )

      console.log(mediaData)

      let newsData = await client.handle(
        'fetchMaterial',
        uploadData.media_id,
        'news',
        true
      )
      let items = newsData.news_item
      let news = []

      items.forEach(item => {
        news.push({
          title: item.title,
          description: item.description,
          picUrl: data2.url,
          url: item.url
        })
      })

      reply = news
    } else if (content === '9') {
      let counts = await client.handle('countMaterial')
      console.log(JSON.stringify(counts))

      let res = await Promise.all([
        client.handle('batchMaterial', {
          type: 'image',
          offset: 0,
          count: 10
        }),
        client.handle('batchMaterial', {
          type: 'video',
          offset: 0,
          count: 10
        }),
        client.handle('batchMaterial', {
          type: 'voice',
          offset: 0,
          count: 10
        }),
        client.handle('batchMaterial', {
          type: 'news',
          offset: 0,
          count: 10
        })
      ])

      console.log(res)

      reply = `
        image: ${res[0].total_count}
        video: ${res[1].total_count}
        voice: ${res[2].total_count}
        news: ${res[3].total_count}
      `
    }
```





