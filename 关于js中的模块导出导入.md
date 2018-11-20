---
title: 关于js中的模块导出导入
date: '2018-09-29 23:35:44'
tag: 
  - javascript
meta:
  -
    name: Javascript
    content: export,export default,module.exports
  -
    name: Javascript
    content: 关于js中的模块导出导入
---

在写js时，我们经常会通过require导入一个模块，通过export ，export default ，module.exports 导出一个模块，在写代码的过程中想查阅一下其中的区别，并做一个记录。
<!-- more -->

## CommonJS
首先我们先来说一下CommonJS，node里面的模块遵守的就是该规范。
CommonJS定义的模块分别为：模块标识(module)、模块定义（exports）、模块引用（require）
<span data-type="color" style="color:rgb(34, 34, 34)">Node 应用由模块组成，采用 CommonJS 模块规范。</span>


## exports和module.exports
在运行一个node文件时，会给这个文件内部生成一个exports和module对象。而module又有一个exports属性。
exports = module.exports  = {}



![1042726218-597aee464dc71_articlex.png | center | 596x166](https://cdn.nlark.com/yuque/0/2018/png/166011/1538186002368-8e602752-9120-4129-b59d-3943b77de99f.png "")


他们都指向同一块内存区域

```javascript
# test.js
let a = 100
console.log(module.exports); //能打印出结果为：{}
console.log(exports); //能打印出结果为：{}

exports.a = 300;  // 修改module.exports的内容 他仅仅是辅助修改module.exports内存块内容

# test2.js
let b = require('./test.js')
console.log(b) // {a:300}
```

## export 和 export default
1. export与export default均可用于导出常量、函数、文件、模块等
2. 你可以在其它文件或模块中通过import+(常量 | 函数 | 文件 | 模块)名的方式，将其导入，以便能够对其进行使用
3. 在一个文件或模块中，export、import可以有多个，export default仅有一个
4. 通过export方式导出，在导入时要加{ }，export default则不需要

### export

```javascript
// 导出方式
export const str = "ChangJun";
export function getName(name) { 
  return name;
}

// 对应的导入方式
import { str, getName} from 'test';
```

### export default

```javascript
// 导出方式
const str = "ChangJun";
export default str;

// 对应的导入方式
import str from 'test';

// 使用export default命令，为模块指定默认输出，这样就不需要知道所要加载模块的变量名
// 一个文件内只能有一个export default
```

