---
title: 微信小程序上传base64图片
date: '2018-09-28 10:35:44'
tag: 
  - wx
meta:
  -
    name: 微信小程序上传base64图片
    content: 微信小程序上传base64图片
---

在小程序中可以通过wx.uploadFile()上传文件到服务器，但在此处不是通过这种方式，后端接口需要上传base64图片。
<!-- more -->

## 问题描述
在小程序中可以通过wx.uploadFile()上传文件到服务器，但在此处不是通过这种方式，后端接口需要上传base64图片。


## 我的思路

---

那既然要上传base64图片，无非就是拿到图片之后将图片转化成base64形式，那么在印象中小程序是没有现成的api可调用，那么我们一步步来，首先通过chooseImage()拿到我们要上传的图片。

```javascript
export function chooseImage (e) {
  return new Promise((resolve, reject) => {
    wx.chooseImage({
      sizeType: ['compressed'], // 可以指定是压缩图
      sourceType: ['album', 'camera'],
      count: 1,
      success (res) {
        resolve(res.tempFilePaths)
      },
      fail (e) {
        showModal('提示', '请再试一下')
      }
    })
  })
}
```

通过这个方法我们拿到我们将要上传的图片，然后我们来看一下接下来怎么搞。
## 解决办法

---

我们chooseimg之后会获取一个图片地址，接下来，我们调用wx.getFileStystemManager().readFile()关于这一接口，传入我们chooseimg获取到的图片地址，可以参考一下文档，我们可以直接通过该接口将我们获取到的图片转化成base64格式。
[https://developers.weixin.qq.com/miniprogram/dev/api/file/FileSystemManager.readFile.html](https://developers.weixin.qq.com/miniprogram/dev/api/file/FileSystemManager.readFile.html)

```javascript
export function getImgBase64 (imgurl) {
  return new Promise((resolve, reject) => {
    wx.getFileSystemManager().readFile({
      filePath: imgurl,
      encoding: 'base64',
      success: res => {
        resolve('data:image/jpg;base64,' + res.data)
      }
    })
  })
}
```

由此我们就可以获取到我们转好的base64格式图片，接下来按照相应参数上传接口即可。

## 更好补充

---

如果你有更好的解决方法，请您补充在这里。如果您解决了这个问题，请给颗稻谷。

