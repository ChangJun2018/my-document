---
title: dva数据流方案使用
date: '2018-09-14 23:35:44'
tag: 
  - react
meta:
  -
    name: dva
    content: react dva使用
  -
    name: dva
    content: dva react
---


dva是一个基于<span data-type="color" style="color:#FF4D4F">redux</span>和<span data-type="color" style="color:#FF4D4F">redux-saga</span>的数据流方案，同时内置了react-router。
<!-- more -->
## 技术背景


dva是一个基于<span data-type="color" style="color:#FF4D4F">redux</span>和<span data-type="color" style="color:#FF4D4F">redux-saga</span>的数据流方案，同时内置了react-router。

## 技术功能


1. 简化react中redux的繁琐配置
2. 合理组织model
3. dva-loading自动处理loding状态

## 使用步骤


#### 安装dav-cli

```bash
npm install dva-cli -g

dva  new dva-quickstart

cd dva-quickstart

npm start
```

#### 使用antd

安装antd  babel-plugin-import 用来按需加载

```bash
npm install antd babel-plugin-import --save
```

编辑 <span data-type="color" style="color:#FF4D4F">.webpackrc</span>，使 <span data-type="color" style="color:#FF4D4F">babel-plugin-import</span> 插件生效。

```bash
{
  "extraBabelPlugins": [
    ["import", { "libraryName": "antd", "libraryDirectory": "es", "style": "css" }]
  ]
}
```

### 概念（官网说明）
<span data-type="color" style="color:rgb(44, 62, 80)">数据的改变发生通常是通过用户交互行为或者浏览器行为（如路由跳转等）触发的，当此类行为会改变数据的时候可以通过 </span>`dispatch`<span data-type="color" style="color:rgb(44, 62, 80)"> </span>发起一个 action，如果是同步行为会直接通过 `Reducers`<span data-type="color" style="color:rgb(44, 62, 80)"> </span>改变 `State`<span data-type="color" style="color:rgb(44, 62, 80)"> </span>，如果是异步行为（副作用）会先触发 `Effects`<span data-type="color" style="color:rgb(44, 62, 80)"> </span>然后流向 `Reducers`<span data-type="color" style="color:rgb(44, 62, 80)"> </span>最终改变 `State`<span data-type="color" style="color:rgb(44, 62, 80)">，所以在 dva 中，数据流向非常清晰简明，并且思路基本跟开源社区保持一致（也是来自于开源社区）。</span>

## 快速使用
在src/routes中新建文件Hello.js,在其中写入

```javascript
import React from 'react';
import { Input } from 'antd';
import styles from './Hello.css'

export default function Hello() {
  return (
    <div className={styles.box}>
      <Input placeholder="Basic usage" />
      <p className={styles.text}>一个人</p>
    </div>
  )
}
```

这样我们就定义了ui组件，其中需要注意，当我们在<span data-type="color" style="color:#FF4D4F">routes</span>中写好了我们的组件后，我们需要去<span data-type="color" style="color:#FF4D4F">src/router.js</span> 下面定义一下我们的路由，然后启动项目访问相应的路径就能看到我们定义好的组件了。接下来我们定义<span data-type="color" style="color:#FF4D4F">model</span>,在<span data-type="color" style="color:#FF4D4F">src/models</span>创建<span data-type="color" style="color:#FF4D4F">hello.js</span>这个<span data-type="color" style="color:#FF4D4F">model</span>存放的就是<span data-type="color" style="color:#FF4D4F">hello.js</span>这个组件的状态。

```javascript
export default {
  namespace: 'hello',  
  state:{
    defaultValue:'HELLO CHANGJUN'
  } , 
  reducers: {
    'change'(state,{payload:value}){
      return {
        defaultValue: value
      }
    }
  },
}
```

其中要注意<span data-type="color" style="color:#FF4D4F">namespace</span>代表着在<span data-type="color" style="color:#FF4D4F">全局store上的key</span>，<span data-type="color" style="color:#FF4D4F">state</span>表示我们的<span data-type="color" style="color:#FF4D4F">状态</span>，<span data-type="color" style="color:#FF4D4F">reducers</span>就相当于<span data-type="color" style="color:#FF4D4F">redux</span>当中的<span data-type="color" style="color:#FF4D4F">reducer</span>，接受<span data-type="color" style="color:#FF4D4F">action</span>同步<span data-type="color" style="color:#FF4D4F">state</span>。这样我们就写好了我们的model，接下来我们要把她连接到我们的组件当中去，并写完我们的demo。回到的我们的组件当中写入

```javascript
import React from 'react';
import { connect } from 'dva';
import { Input } from 'antd';
import styles from './Hello.css'


function Hello({ dispatch, hello }) {
  function changeInputValue(e){
    dispatch({
      type:'hello/change',
      payload:e.target.value
    });
    console.log(e.target.value)
  }
  return (
    <div className={styles.box}>
      <Input placeholder="Basic usage" onChange={changeInputValue} value={hello.defaultValue} />
      <p className={styles.text}>{hello.defaultValue}</p>
    </div>
  )
}

function mapStateToProps({hello}) {
  return {hello}
}

export default connect(mapStateToProps)(Hello)

```

其中<span data-type="color" style="color:#FF4D4F">connect</span>是连接model和组件这个跟redux中的conect一样,利用一个函数将<span data-type="color" style="color:#FF4D4F">dispatch</span>和状态传递到<span data-type="color" style="color:#FF4D4F">props</span>中取，组件通过props即可拿到相应的状态和dispatch

## 使用问题


在定义好model时一定要记着去index.js去挂载一下。

```javascript
app.model(require('./models/hello').default);
```

还有Effect和Subscription订阅这两个api有空的时候补充一下。



