---
title: react全家桶搭建多页面开发
date: '2018-10-24 16:40:44'
tag: 
  - react
meta:
  -
    name: react
    content: react,react,redux,webpack,babel
---

本篇将描述如何搭建React全家桶+多页面开发环境搭建
<!-- more -->


# 📋 前言
asdsdasd
关于react搭建全家桶、配置开发多页面、网上的例子有很多，我也是参考了很多，本篇笔记只是想通过自己手敲加深一下自己的记忆，通过这一次的这记录，对webpack 的一些基本配置有了一些了解，对前端环境搭建有了一些认识。

# 💡 目录

- 项目目录
- 安装webpack
- 安装react
- 配置loader
- 使用babel
- 使用htmlwebpackplugin
- 使用redux
- 使用webpack-dev-server
- 多入口页面配置
- 模块热替换
- 使用ESlint
- 使用react-router
- 使用redux-thunk
- 使用axios和async/await

# ⌚ 项目目录结构

![目录结构](https://images-cdn.shimo.im/3iXSl41zpaEjfmsD/123.png!thumbnail)

```javascript
dev // 调试版本打包输出
dist // 发布版本打包输出
src // 项目源码
webpack.config.build.js // webpack打包配置
webpack.config.dev.js // webpack调试配置
src/component // 公共组件
src/page/index  // 首页代码
src/page/other // 其它页面代码
.eslint   // eslint配置
.babelrc  // babel配置
```

# 💭 安装webpack

```javascript
npm install webpack --save
npm install webpack-cli --save
```

在webpack.config.dev.js 写入一个简单的webpack配置

```javascript
const path = require('path');
const srcRoot = './src';
module.exports = {
    mode: 'development' // 配置开发模式
    // 输入配置
    entry: [
      './app.js'
    ],,
​
    // 输出配置
    output: {
        path: path.resolve(__dirname, './dev'),
        filename: 'bundle.min.js'
    }
​
};
```

在package.json中添加以下命令:

```javascript
"scripts": {
    "dev": "./node_modules/.bin/webpack --config webpack.config.dev.js"
}
```

# 🚩 安装react

```javascript
npm install react react-dom --save
```

在src/page/index/index.js 中写入

```javascript
import ReactDom from 'react-dom';
import Main from './Main/Main.jsx';

ReactDom.render(<Main />, document.getElementById('root'));
```

在src/page/index/index.html 中写入

```javascript
<!DOCTYPE html>
<html>
<head>
    <title>index</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no">

</head>
<body>
<div id="root"></div>
</body>
</html>
```

在src/page/index/Main/Main.jsx中写入

```javascript
import React from 'react';

class Main extends React.Component {

    constructor(props) {
        super(props);

    }

    render() {
        return (<div>Main</div>);
    }
}

export default Main;

```

修改webpack配置入口文件

```javascript
entry: [
    path.resolve(srcRoot,'./page/index/index.js')
]
```

# 😀 配置loader

```javascript
npm install css-loader sass-loader style-loader file-loader --save
```

在webpack.config.dev.js中写入

```javascript
 module: {
  // 加载器配置
  rules: [
       { 
         test: /\.css$/, 
         use: ['style-loader', 'css-loader'], 
         include: path.resolve(srcRoot)
       },
       { 
         test: /\.scss$/, 
         use: ['style-loader', 'css-loader', 'sass-loader'], 
         include: path.resolve(srcRoot)
       }，
       // url-loader 处理静态文件
       // limit 表示超过多少就用base64来代替
       // name 设置图片的路径，名称是否使用hash
       { 
         test: /\.(png|jpg|jpeg)$/, 
         use: 'url-loader?limit=8192&name=images/[name].[hash].[ext]',
         include: path.resolve(srcRoot)    
       }
  ]
}
```

# 😀 使用babel

安装babel

```javascript
npm install babel-core babel-loader --save

// 安装babel 需要的编译的一些插件
npm install babel-preset-es2015 babel-preset-react babel-preset-stage-0 --save

// babel-polyfill 是对最新的一些js API 提供一个环境垫片，例如Generator、Promise全局对象
npm install --save babel-polyfill

// babel-polyfill 会将所有的新的语法加入进来，这时候就需要transform-runtime实现按需引入，引入我们需要的
npm install --save babel-plugin-transform-runtime
```

在webpack.config.dev.js module rules（加载器配置中）写入

```javascript
rules: [
       // 对js 和 jsx 进行编译
      { 
        test: /\.(js|jsx)$/, 
        use: [{loader:'babel-loader'}],
        include: path.resolve(srcRoot)
      }
]

```

在根目录下的.babelrc babel配置文件中写入

```javascript
{
    "presets": [
        "es2015",
        "react",
        "stage-0"
    ],
    "plugins": [
      ["transform-runtime", {
        "helpers": false,
        "polyfill": false,
        "regenerator": true,
        "moduleName": "babel-runtime"
      }]
  ]
}
```

presets 代表babel支持那种语法优先级**从下往上**
plugins babel解析的时候使用那些插件，优先级也是**从下往上**

# ❤️ 使用htmlwebpackplugin

安装html-webpack-plugin

```javascript
// html-webpack-plugin 是要将我们的html和我们打包后的js结合在一起来
npm install html-webpack-plugin --save
```

在webpack.config.dev.js 中写入

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');

plugins: [
    new HtmlWebpackPlugin({
        // filename 设置html输出的路径和文件名
        filename: path.resolve(devPath, 'index.html'),
        // template 设置以当前的那个html为模板
        template: path.resolve(srcRoot, './page/index/index.html'),
    })
]
```

# 🔥 使用redux

安装redux

```javascript
npm install redux react-redux --save
```

使用redux目录

```javascript
src/page/index/actions/actionType.js // 存放常量
src/page/index/actions/testAction.js // 测试action
src/page/index/reducers/main.js //对所有的reducer进行合并
src/page/index/reducers/testReducer.js //测试Reducer
```

在src/pages/index/index.js 中写入

```javascript
import ReactDom from 'react-dom';
import React from 'react';
import Main from './Main/Main.jsx';
import store from './store.js';
import { Provider } from 'react-redux';

ReactDom.render(
    <Provider store={store}>
        <Main />
    </Provider>
, document.getElementById('root'));

```

在src/page/index/store.js 中写入

```javascript
import { createStore } from 'redux';
import testReducer from './reducers/testReducer.js';
const store = createStore(todoReducer);

export default store;

```

在testReducer.js 中写入

```javascript
import { CHANGE_TEST} from '../actions/actionTypes.js';

const initialState = {
      testData:'chang'
};

const changeTest= (state, action) => {
  return { ...state, testData:action.obj.text }
}

const testReducer = (state = initialState, action) => {
  switch(action.type) {
    case CHANGE_TEST: return changeTest(state, action);
    default: return state;
  }
};
export default testReducer ;

```

在testAction.js 中写入

```javascript
import { CHANGE_TEST} from './actionTypes.js';

export const changeTest = (obj) => {
  return {
    type: CHANGE_TEST,
    obj: obj
  };
};

```

在Main.jsx 中写入

```javascript
import React from 'react';
import { connect } from 'react-redux';
import { changeTest } from '../actions/todoAction.js';

class Main extends React.Component {

    onClick(){
        let text = this.refs.input;
        this.props.dispatch(changeTest({
            text: text.value
        }))
    }
    render() {
        return (
            <div>
                <p>{this.props.testData}</p>
                <input ref="input" type="text"></input>
                <button onClick={()=>this.onClick()}>提交</button>
            </div>
        );
    }
}

export default connect(
    state => ({
        testData: state.testData
    })
)(Main);

```

使用redux-logger

```javascript
npm redux-logger install --save 
```

在src/page/index/store.js 中写入

```javascript
import { applyMiddleware, createStore } from 'redux';
import testReducer from './reducers/testReducer.js';
import logger from 'redux-logger'

const store = createStore(
  testReducer,
  applyMiddleware(logger)
)

export default store;
```

# 🍉 使用webpack-dev-server

安装webpack-dev-server

```javascript
// webpack-dev-server 是一个express 服务器
npm install webpack-dev-server --save
```

修改package.json 启动命令

```javascript
 "scripts": {
    "dev": "./node_modules/.bin/webpack-dev-server --config   webpack.config.dev.js",
}

```

配置webpack.config.dev.js

```javascript
devServer: {
    // 表示server文件的根目录
    "contentBase": devPath,
    // 开启GZIP
    "compress": true
}
```

# 🍋 多入口页面配置

我所理解的多入口页面配置就是将webpack的输入和输出都动态获取我们页面下的文件
在webpack.config.dev.js 中写入

```javascript
// 获取我们要打包生成的html
function gethtmlArrary(entryMap) {
  let htmlArrary = [];
  Object.keys(entryMap).forEach((key) => {
    let fullPathName = path.resolve(pageDir, key);
    let fileName = path.resolve(fullPathName, key + '.html');
    // 当前路径是否包含该html
    if (fs.existsSync(fileName)) {
      htmlArrary.push(new HtmlWebpackPlugin({
        filename: key + '.html',
        template: fileName,
        chunks: [key]
      }));
    }
  });
  return htmlArrary;
}

// 获取page下所有的入口文件
function getEntry() {
  let entryMap = {};
  fs.readdirSync(pageDir).forEach((pathname) => {
    let fullPathName = path.resolve(pageDir, pathname);
    // 判断当前路径的文件是一个文件还是文件夹
    let stat = fs.statSync(fullPathName);
    let fileName = path.resolve(fullPathName, mainFile);
    // 如果当前路径是文件夹并且包含index.js我们就把他放在入口对象里
    if (stat.isDirectory() && fs.existsSync(fileName)) {
      entryMap[pathname] = fileName;
    }
  });
  return entryMap
}

// 调用一下这两个方法
const entryMap = getEntry();
const htmlArrary = gethtmlArrary(entryMap);
```

修改webpack.config.dev.js

```javascript
module.exports={
  ...
  // 修改入口配置为获取的入口文件
  entry:entryMap
  plugins: [].concat(htmlArrary)
}
```

# 模块热替换

在webpack.config.dev.js 中写入

```javascript
devServer: {
    hot: true // 开启HMR（模块热替换）
}

plugins: [
    new webpack.NamedModulesPlugin(),
    new webpack.HotModuleReplacementPlugin(),
].concat(htmlArrary)

```

在react中使用

```javascript
npm install react-hot-loader --save

```

在src/page/index/Main/Container.jsx 中写入

```javascript
import React from 'react';
import Main from './Main.jsx';
import { hot } from 'react-hot-loader'

class Container extends React.Component {

    render() {
        return <Main />
    }
        
}
export default hot(module)(Container);


```

让redux也热更新
修改store.js

```javascript
if (module.hot) {
  module.hot.accept('./reducers/testReducer.js', () => {
     const nextRootReducer = require('./reducers/todoReducer.js').default;
      store.replaceReducer(nextRootReducer);
    });
}

```

修改index.js

```javascript
import ReactDom from 'react-dom';
import React from 'react';
import Container from './Main/Container.jsx';
import store from './store.js';

import { Provider } from 'react-redux';

ReactDom.render(
    <Provider store={store}>
        <Container />
    </Provider>
, document.getElementById('root'));

```

# 🍋 使用ESlint

安装eslint

```javascript
npm install eslint eslint-loader babel-eslint --save

// 安装eslint react配置
npm install eslint-plugin-react --save

```

修改webpack.config.dev.js 加载器配置

```javascript
{ 
  test: /\.(js|jsx)$/, 
  use: [
      {loader:'babel-loader'},
      {loader:'eslint-loader'}
    ] ,
  include: path.resolve(srcRoot)
}

```

在根目录下的.eslintrc中写入

```javascript
{
    "extends": ["eslint:recommended","plugin:react/recommended"],
    
    "parser": "babel-eslint",

    "globals": {
        "window": true,
        "document": true,
        "module": true,
        "require": true
    },
    "rules": {
        "react/prop-types" : "off",
        "no-console" : "off"
    }
}


```

**extends**    eslint有很多**默认的规则模版**从这里继承，此处添加了额安装的r**eact规则**
**parser       **eslint **解析器** 此处是babel-eslint  对babel解析器的包装，兼容eslint
**globals**     eslint 配置**全局变量不发生警告**
**rules         配置额外的规则  **

# 🎉 使用react-router

安装react

```javascript
npm install react-router-dom --save
// 如果使用redux 安装
npm install react-router-redux --save

```

修改index.js

```javascript
import React from 'react'
import ReactDom from 'react-dom'
import { Provider } from 'react-redux'
import Container from './Main/Container'
import { store, history } from './store.js'
import { ConnectedRouter } from 'react-router-redux'

ReactDom.render(
  <Provider store={store}>
    <ConnectedRouter history={history}>
      <Container />
    </ConnectedRouter>
  </Provider>,
  document.getElementById('root')
)


```

修改store.js

```javascript
import {createStore,applyMiddleware} from 'redux'
import mainReducer from './reducers/main';

import createHistory from 'history/createHashHistory'
import { routerMiddleware } from 'react-router-redux'

// 创建基于hash的history
const history = createHistory();

// 创建初始化tab
history.replace('home')

// 创建history的middleware
const historyMiddl = routerMiddleware(history);

const store = createStore(mainReducer, applyMiddleware(historyMiddl));


// 配置redux热更新
if (module.hot){
  module.hot.accept('./reducers/main',()=>{
    const nextRootReducer = require('./reducers/main');
    store.replaceReducer(nextRootReducer)
  })
}

module.exports = {
  store,
  history
}


```

react-router的三种方式
**BrowserRouter** 通过history/createBrowserHistory 引入，底层使用的**HTML5的pushState**
**HashRouter** 通过istory/createHashHistory，当路由切换时，动态修改**hash**，利用hashchange事件。
MemoryRouter 通过history/createMemoryHistory引入，将路由相关数据存入内存中，不涉及url更新，兼容性好。

使用react-router 例如在Main.jsx中

```javascript
import React, { Component } from 'react'
import { Route,withRouter } from 'react-router-dom'
import BottomBar from '../BottomBar/BottomBar.jsx'
import Home from '../Home/Home'
import Order from '../Other/Other.jsx'

class Main extends Component {
  constructor(props) {
    super(props)
  }

  render() {
    return (
      <div>
        <Route exact path="/home" component={Home} />
        <Route path="/other" component={Other} />
        <BottomBar />
      </div>
    )
  }
}

export default withRouter(Main)


```

# 📞 使用redux-thunk

安装redux-thunk

```javascript
npm install redux-thunk --save

```

修改store.js

```javascript
/**
 * @author: Chang Jun
 * @date: 2018/10/10
 * @Description:
 */

import {createStore,applyMiddleware} from 'redux'
import mainReducer from './reducers/main';
import thunk from 'redux-thunk';

import createHistory from 'history/createHashHistory'
import { routerMiddleware } from 'react-router-redux'

// 创建基于hash的history
const history = createHistory();

// 创建初始化tab
history.replace('home')

// 创建history的middleware
const historyMiddl = routerMiddleware(history);

const store = createStore(mainReducer, applyMiddleware(thunk,historyMiddl));


// 配置redux热更新
if (module.hot){
  module.hot.accept('./reducers/main',()=>{
    const nextRootReducer = require('./reducers/main');
    store.replaceReducer(nextRootReducer)
  })
}

module.exports = {
  store,
  history
}

```

# 📚 使用axios和async/await

安装axios

```javascript
npm install axios --save
// 安装babel插件
npm install babel-plugin-transform-async-to-generator --save

```

在.babelrc中写入

```javascript
 "plugins": [
     "transform-runtime",
       {
         "polyfill": false,
         "regenerator": true,
       }
  ]

```

# 📌 使用react-loadable

安装react-loadable

```javascript
npm install react-loadable --save

```

在路由组件中使用

```javascript
function Loading() {
  return <div>Loading...</div>;
}

let Main = Loadable({
  loader: () => import('./My'), 
  loading: Loading,
});

<Route path="/main" component={Main}></Route>


```








