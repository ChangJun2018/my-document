---
title: 在webstrom和vscode中配置ts
date: '2018-09-21 23:35:44'
tag: 
  - typescript
meta:
  -
    name: typescript
    content: typescript使用
  -
    name: typescript
    content: typescript的学习笔记
---

typescript作为前端开发的未来趋势，也是前端开发也应该掌握的。这里就不一一赘述其特性。我们来看一下如何在我们的代码编辑器中配置ts。个人比较常用webstrom和vscode，所以这里只描述这两种编辑器的配置。
<!-- more -->

## 背景

typescript作为前端开发的未来趋势，也是前端开发也应该掌握的。这里就不一一赘述其特性。我们来看一下如何在我们的代码编辑器中配置ts。个人比较常用webstrom和vscode，所以这里只描述这两种编辑器的配置。

> 本篇前提是你的电脑中已装有node 和typescript，如何安装请自行百度。

## vscode中配置ts

### 配置步骤

> 1. 新建项目文件夹testtypescript
> 2. cd testtypescript
> 3. 运行tsc --init 创建tscconfig.json文件
> 4. 修改outDir:"./js"
> 5. 点击终端-运行任务-监视tsconfig
> 6. 完成

### 代码演示

```javascript
# 1. 新建项目文件夹并cd到项目目录
  md testtypescript && cd testtypescript
# 2. 创建tsconfig文件
  tsc --init
# 3. 修改tsconfig文件其中的输出目录为根目录下的js文件夹
  "outDir": "./js"     
# 4. 点击vscode终端中的运行任务,监视tsconfig
# 5. 新建index.ts文件在其中输入
  const title:string="Hello,TypeScript"
# 6. 此时项目中会有一个js文件夹，至此配置完成
```

## webstrom中配置ts

### 配置步骤
> 1. 新建项目文件夹tscode
> 2. cd tscode
> 3. 运行tsc --init 创建tscconfig.json文件
> 4. 修改outDir:"./js"
> 5. 点击file-settings-Language-Typescript 勾选Recompile on changes
> 6. 完成

### 代码演示

```javascript
# 1. 新建项目文件夹并cd到项目目录
  md tscode && cd tscode
# 2. 创建tsconfig文件
  tsc --init
# 3. 修改tsconfig文件其中的输出目录为根目录下的js文件夹
  "outDir": "./js"     
# 4. 点击file-settings-Language-Typescript 勾选Recompile on changes
# 5. 新建index.ts文件在其中输入
  const title:string="Hello,TypeScript"
# 6. 此时项目中会有一个js文件夹，至此配置完成
```


## 最后
如果您觉着本篇文档不错，请您给一颗稻谷。
