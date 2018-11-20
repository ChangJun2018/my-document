---
title: MongoDB(4.0) 在win下的安装
date: '2018-09-27 23:35:44'
tag: 
  - mongodb
meta:
  -
    name:  MongoDB(4.0) 在win下的安装
    content:  MongoDB(4.0) 在win下的安装
---

MongoDB(4.0) 在win下的安装
<!-- more -->

## 下载安装包
地址:[https://www.mongodb.com/download-center#community](https://www.mongodb.com/download-center#community)

## 安装过程
打开下载的安装包，一路安装下去，但是在这个前提之前有一个地方需要注意。

::: danger
在第三步的时候，将默认勾选install MongoDB Compass的勾去掉。
:::

```bash
# 第一步创建数据目录默认在C盘下
# 打开cmd输入以下命令
md data && cd data
md db && md log

# 以管理员方式打开命令终端执行一下命令
"C:\Program Files\MongoDB\Server\4.0\bin\mongod.exe" --dbpath="c:\data\db"

# 在win下执行命令启动服务和关闭服务
net start MongoDB
net stop MongoDB
```


