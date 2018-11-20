---
title: create-react-app 配置less,antd按需加载
date: '2018-09-14 23:44:44'
tag: 
  - react
meta:
  -
    name: create-react-app 配置less,antd按需加载
    content: create-react-app 配置less,antd按需加载
  -
    name: create-react-app
    content: 配置less
---

在react脚手架中配置antd,less按需加载
<!-- more -->

## <a name="wxa4cg"></a>安装

```bash
npm install react-router-dom axios less less-loader antd babel-plugin-import --save
```


## <a name="4lxrao"></a>暴露配置
    
```bash
// 暴露react配置
npm run eject
```

## <a name="xtnvzy"></a>开始配置
    在config/webpack.config.dev.js中写入配置

## 写入位置

```javascript
/// 写在这里
/// 写在这里
/// 写在这里
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
          // Necessary for external CSS imports to work
          // https://github.com/facebookincubator/create-react-app/issues/2677
          ident: 'postcss',
          plugins: () => [
            require('postcss-flexbugs-fixes'),
            autoprefixer({
              browsers: [
                '>1%',
                'last 4 versions',
                'Firefox ESR',
                'not ie < 9', // React doesn't support IE8 anyway
              ],
              flexbox: 'no-2009',
            }),
          ],
        },
      },
    ],
  }
```

## 写入内容

```javascript
{
        loader: require.resolve('postcss-loader'),
        options: {
          // Necessary for external CSS imports to work
          // https://github.com/facebookincubator/create-react-app/issues/2677
          ident: 'postcss',
          plugins: () => [
            require('postcss-flexbugs-fixes'),
            autoprefixer({
              browsers: [
                '>1%',
                'last 4 versions',
                'Firefox ESR',
                'not ie < 9', // React doesn't support IE8 anyway
              ],
              flexbox: 'no-2009',
            }),
          ],
        },
      },
      {
        loader:require.resolve('less-loader'),、
        // 这里是antd定制主题
        options: {
          modules:false,
          modifyVars: {
            '@primary-color': '#f9c700',
          }
        },
      }
    ],
 },
```

## antd按需加载样式
### 写入位置

```javascript
// Process JS with Babel.
{
test: /\.(js|jsx|mjs)$/,
include: paths.appSrc,
loader: require.resolve('babel-loader'),
options: {
  //写在这里
  //写在这里
  //写在这里
  // This is a feature of `babel-loader` for webpack (not Babel itself).
  // It enables caching results in ./node_modules/.cache/babel-loader/
  // directory for faster rebuilds.
  cacheDirectory: true,
},
},
```
### 写入之后

```javascript
 // Process JS with Babel.
{
test: /\.(js|jsx|mjs)$/,
include: paths.appSrc,
loader: require.resolve('babel-loader'),
options: {
  plugins: [
    ['import', [{ libraryName: "antd", style: true }]],
  ],
  // This is a feature of `babel-loader` for webpack (not Babel itself).
  // It enables caching results in ./node_modules/.cache/babel-loader/
  // directory for faster rebuilds.
  cacheDirectory: true,
},
},
```

::: danger
备注:按照上面配置时，如果运行项目发现是less报错,请安装 less 2.7以上版本
npm install less@^2.7.3 --save
:::

