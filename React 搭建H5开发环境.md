---
title: React 搭建H5开发环境
date: '2018-10-09 23:35:44'
tag: 
  - react
meta:
  -
    name: react
    content: react,react h5
---

本篇将描述如何使用React来搭建H5的一个开发环境
<!-- more -->


# 📋 背景描述

本篇将描述如何使用React来搭建H5的一个开发环境，脚手架采用__create-react-app__，数据流框架采用__redux__，异步采用__redux-thunk__，使用__immutable__持久化数据结构，使用__swiper__构建H5滚动，使用__axios__做请求(使用async 和 await优化异步请求)，使用__react-loadabel 优化路由加载使用__sass来写css

# 💡 开发流程

1. 创建项目
2. 移动端端适配并配置sass
3. 构建swiper和animate
4. 配置react路由
5. 集成redux
6. 使用immutable和redux-immutable持久化数据结构
7. 使用babel-plugin-transform-async-to-genertor 优化异步时序
8. 使用react-loadable 优化路由加载优化首屏加载速度

# ⌚ 项目结构

如果你想跟着本篇敲，请先看一下项目结构，如果您只想观看一下，请跳过该步

```
├─public    # 不参与编译的静态资源
└─src       # 程序主入口
    │  Global.scss   # 全局sass
    │  index.js      # 项目入口文件
    │  Reset.scss    # 清除浏览器默认样式
    │  Router.js     # 路由配置文件
    │  
    ├─components   # 公共组件
    │  └─swiper   # swiper容器组件
    │          
    ├─pages
    │  ├─Answer   # swiper测试页面
    │  │      index.js
    │  │      index.scss
    │  │      
    │  └─test   # 测试页面
    │      │  
    │      └─store  # 页面下store
    │              
    ├─store   # 全局store设置
    │      
    └─utils   # 工具函数库

```

# 💭 开发过程

1. 创建项目

```javascript
// 使用create-react-app 创建项目
npx create-react-app my-app
// 运行项目
yarn start
```

1. 移动端适配

```javascript
// 首先将react配置暴露出来
yarn run eject 
// 安装以下依赖
npm install lib-flexible --save
npm install postcss-px2rem --save
// 在config/webpack.config.dev.js 中导入插件
const px2rem = require('postcss-px2rem')

 要修改的位置

{
test: /\.css$/,
use: [
  require.resolve('style-loader'),
  {
    loader: require.resolve('css-loader'),
    options: {
      importLoaders: 1,
    },
  },
  {
    loader: require.resolve('postcss-loader'),
    options: {
      ident: 'postcss',
      plugins: () => [
        require('postcss-flexbugs-fixes'),
        autoprefixer({
          browsers: [
            '>1%',
            'last 4 versions',
            'Firefox ESR',
            'not ie < 9', 
          ],
          flexbox: 'no-2009',
        }),
      ],
    },
  },
],
```

},

修改之后为

```javascript
{
    test: /\.(css|scss|sass)$/,
    use: [
      require.resolve('style-loader'),
      {
        loader: require.resolve('css-loader'),
        options: {
          importLoaders: 1,
        },
      },
      {
        loader: require.resolve('postcss-loader'),
        options: {
          ident: 'postcss',
          plugins: () => [
            require('postcss-flexbugs-fixes'),
            autoprefixer({
              browsers: [
                '>1%',
                'last 4 versions',
                'Firefox ESR',
                'not ie < 9', 
              ],
              flexbox: 'no-2009',
            }),
            px2rem({remUnit:75})
          ],
        },
      },
      {
        loader:require.resolve('sass-loader')
      }
    ],
  },
  
// 修改webpack.config.prod跟之前一样
// 删除/注释 public/index.html中的viewport代码
```

1. 构建swiper和animate

```javascript
// 安装swiper
yarn add swiper
// 在项目入口文件中引入animate.css
import './static/css/animate.min.css'
// src下新建components文件夹用于存放组件，新建swiper.js，导入我们的相关包
import React, { Component } from 'react'
import Swiper from 'swiper/dist/js/swiper.js'
import 'swiper/dist/css/swiper.min.css'
import {swiperAnimateCache,swiperAnimate} from 'swiper.animate.js'
// 新建一个swiper组件用于表示swiper容器组件，内部展示
class XSwiper extends Component {
  render() {
    const child = this.props.children
    return (
      <div className="swiper-container">
        <div className="swiper-wrapper">
          {
            React.Children.map(child, (ele, index)=> {
            return (
              <div className="swiper-slide">
                {ele}
              </div>
            )
          })
          }
        </div>
      </div>
    )
  }
  
  componentDidMount(){
    // 实例化swiper
    const mySwiper = new Swiper(".swiper-container", {
      direction: "vertical", 
      mousewheel: true, 
      grabCursor: true, 
      effect: "slide", 
      autoHeight: true,
      on:{
        init: function(){
          swiperAnimateCache(this); //隐藏动画元素 
          swiperAnimate(this); //初始化完成开始动画
        }, 
        slideChangeTransitionEnd: function(){ 
          swiperAnimate(this); //每个slide切换结束时也运行当前slide动画
        } 
      }  
    })
  }
}
export default XSwiper
// 在src/pages下新建用于滚动的swiper页面
import React, { Component,  } from 'react'
import XSwiper from '../../components/swiper'
import XTest from '../test'
class Answer extends Component {
  render() {
    return (
      <XSwiper> 
        <XTest />
        <XTest />
        <XTest />
      </XSwiper>
    )
  }
}
export default Answer
// 在pages中新建页面写入
import React, {Component} from 'react'
import './index.scss'
class XTest extends Component{
  constructor(props) {
    super(props);
    this.state = {
      flag: false
    }
  }
  render() {
    return (
        <div className="test">
          <h3 {...global.aniCreator('fadeIn')}>
            Hello React
          </h3>
        </div>
    )
  }
}
// 不要忘记加上swiper的css
.swiper-container {
  width: 100%;
  height: 100% ;
}
.swiper-slide {
  text-align: center;
  height: 100% !important;
}
// 此时运行yarn start 即可看到效果

```

1. 配置react路由

```javascript
// 安装react路由
yarn add react-router-dom
// 在项目根路径下新建Router.js
import React, {Component} from 'react';
import {Provider} from 'react-redux'
import {HashRouter, Switch, Route} from 'react-router-dom'
import Answer from './pages/Answer'
class Router extends Component {
  render() {
    return (
        <Provider store={store}>
          <HashRouter>
            <Switch>
              <Route path="/" exact component={Answer}></Route>
            </Switch>
          </HashRouter>
        </Provider>
    );
  }
}
export default Router;

```

1. 集成redux并配置redux-thunk

```javascript
// 安装redux 
yarn add redux react-redux redux-thunk

// 在pages/test/store/reducer.js写入
const defaultState = {
  name: 'React'
}

export default (state=defaultState, action={}) => {
  switch(action.type){
    case TEST:
      return state.set('test', '测试')
    default:
      return state
  }
}

// 在pages/test/store/reducer.js写入
import reducer from  './reducer'
import * as actionCreators from './actionCreator'
export { reducer, actionCreators }

// 在src/store/reducer.js下
import { combineReducers } from 'redux'
import { reducer as testReducer }  from '../pages/test/store'、

// 合并reducer
export default combineReducers({
  test: testReducer
})

// 在src/store/index.js下
import { createStore, applyMiddleware, compose } from 'redux';

const composeEnhancers =
  typeof window === 'object' &&
  window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ ?   
    window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({
    }) : compose;

const enhancer = composeEnhancers(
  applyMiddleware(),
);

const store = createStore(enhancer);

export default store

```

1. 使用immutable

```javascript
// 安装immutable
yarn add immutable redux-immutable 
// 修改 pages/test/store/reducer.js
import { fromJS } from 'immutable'

const defaultState = fromJS({
  name: 'React'
});

// 修改src/store/reducer.js
import { combineReducers } from 'redux-immutable'

// 获取test state即为
state.get('test').get('name')
// 或者
state.getIn('test'，'name')
```

1. 使用babel-plugin-async-to-generator 优化异步时序

```javascript
// 安装babel-plugin-async-to-generator
yarn add babel-plugin-async-to-generator
// 在package.json中添加babel插件
"babel": {
  "presets": [
    "react-app"
  ],
  "plugins": [
    "transform-async-to-generator"
  ]
}

// 在需要异步的aciton中
thunkTest = () => async (dispatch) => {
  let res = await axios.get('http://jsonplaceholder.typicode.com/posts');
  console.log(res);
  dispatch({ type: TEST, payload: {name: 2313213185648721} })
};

```

1. 使用react-loadable优化路由加载首屏加载速度

```javascript
// 安装react-loadabel
yarn add react-loadabel
// 在router.js 中
function Loding(){
  return <div>加载中</div>
}

let LodingAnswer = Loadable({
  loader: () => import('./pages/Answer'),
  loading: Loding
});

```

# 

# 🚩 开发问题

1. 问题1
2. 问题2

# 😀 结束语

由于之前常用vue做开发，学习react基础也已经很久了，但还没有运用到实战当中，我觉着最好的练习就是实践。因此就有了希望用react来做H5的这个想法，希望能从这个项目当中碰到问题，并解决问题，并了解一些react的东西。

我觉着要勇于尝试别的东西对于一名前端开发者来说是非常重要的东西。
