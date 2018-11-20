---
title: loadash常用方法总结
date: '2018-10-11 09:05:13'
tag: 
  - javascript
meta:
  -
    name: loadash笔记
    content: loadash语法,总结

---

[Lodash](https://lodash.com/) 是一款非常知名的 JavaScript 工具库，能够让开发者十分便捷地操纵数组和对象
<!-- more -->

# 😊 项目描述
[Lodash](https://lodash.com/) 是一款非常知名的 JavaScript 工具库，能够让开发者十分便捷地操纵数组和对象。ES6出来之后有很多方法可以代替掉loadash，但是这对我这个从来没有用过lodash的人来说，并不影响我去学习去使用loadash，我觉着好的库好的方法都是值得去学习的，至少对我来说我觉着loadash可以优化我们的代码。再者webpack-plugin-loadash也可以支持lodash按需加载。
# ⏰ 常用API一览
1. _reject()（根据条件去除某元素）
2. _pick() （根据第二个参数的key的数组，筛选第一个参数中的值并返回）
3. _clone() （浅拷贝）
4. _cloneDeep()（深拷贝）
5. _keys() （返回object所有的key）
6. _find （查找数据）
7. _keyBy  （以某个属性为键，将数组转为对象）
8. _filter （根据条件过滤出符合条件的元素,返回新数组）
9. _map （从集合中挑出一个key，将其值作为数组返回）
10. _max/_min/_sum （数组中最大值、最小值、数组求和）
11. _pad/_padStart/_padEnd （在字符串两端、开头、末尾补齐字符）
12. _debounce （防抖函数）
13. _times （迭代函数n次）
14. _uniqueld （生成元素中唯一的id）
15. _startsWith/_startsEnd （判断字符串是否以xx开头/结尾）
16. _truncate （截取指定长度后加...）
17. _omit （返回移除部分属性的副本,例如提交表单时，不需要那些字段）
18. _pick （与omit相反,提取部分属性，提交表单时，只提交那些字段）
19. _random（生成随机数）
20. _union （合并两数组并去重）
21. _uniq （去重）
22. _orderBy （排序）
# 🔋 常用API详细描述
## _reject
```javascript
let foo = [
    {id: 0, name: "aaa", age: 33},
    {id: 1, name: "bbb", age: 25}
]
let bar = _.reject(foo, ['id', 0])

//bar = [{id: 1, name: "bbb", age: 25}]

```
## _pick
```javascript
var foo = {id: 0, name: "aaa", age: 33}
var bar = _.pick(foo, ['name', 'age'])
//bar = {name: "aaa", age: 33}
```
## _clone
```javascript
var objects = [{ 'a': 1 }, { 'b': 2 }];

var shallow = _.clone(objects);
console.log(shallow[0] === objects[0]);
// => true
```
## _cloneDeep
```javascript
var original = { foo: "bar" };
var copy = _.cloneDeep(original);
copy.foo = "new value";
// copy -> { foo: "new value" } Yeah!
// original -> { foo: "bar" } Yeah!
```
## _keys
```javascript
var foo = {id: 0, name: "aaa", age: 33}
var bar = _.keys(foo)
//bar = ['id', 'name', 'age']
```
## _find
```javascript
var foo = [
    {id: 0, name: "aaa", age: 33},
    {id: 1, name: "bbb", age: 25}
]
var bar = _.find(foo, ['id', 0])
//bar = {id: 0, name: "aaa", age: 33}

```
## _keyBy
```javascript
var foo = var foo = [
    {id: 0, name: "aaa", age: 33},
    {id: 1, name: "bbb", age: 25}
]
var bar = _.keyBy(foo, 'name')
//bar = {
//    aaa: {id: 0, name: "aaa", age: 33},
//    bbb: {id: 1, name: "bbb", age: 25}
//}
```
## _filter
```javascript
var foo = var foo = [
    {id: 0, name: "aaa", age: 33},
    {id: 1, name: "bbb", age: 25}
]
var bar = _.filter(foo, ['name', "aaa"])
//bar = {
//    aaa: {id: 0, name: "aaa", age: 33}
//}
```
## _map
```javascript
var foo = var foo = [
    {id: 0, name: "aaa", age: 33},
    {id: 1, name: "bbb", age: 25}
]
var bar = _.map(foo, 'name')
//bar = ["aaa", "bbb"]
```
## _max/_min/_sum
```javascript
var foo = [1, 2, 3, 4]
var bar = _.max(foo)
//bar = 4
bar = _.min(foo)
//bar = 1
bar = _.sum(foo)
//bar = 10

```
## _pad/_padStart/_padEnd
```javascript
var foo = "helloworld"
var bar = _.pad(foo, 14, '-')
//bar = --helloworld--
bar = _.padStart(foo, 14, '-')
//bar = ----helloworld
bar = _.padEnd(foo, 14, '-')
//bar = helloworld----
```
## _debounce
在触发防抖函数的事件后面加个延迟时间，如果在该延迟时间内没有继续触发事件，那么执行防抖函数。设想一个场景，电梯门打开的时候，如果在2s内陆续有人按电梯按钮，那么每次电梯都会再重新等待2s，直至2s中没人按按钮再触发关门操作。

```javascript
function validateEmail() {
    // Validate email here and show error message if not valid
}

var emailInput = document.getElementById("email-field");
emailInput.addEventListener("keyup", _.debounce(validateEmail, 500));

```
## _times
返回迭代函数n次
```javascript
_.times(10, (i)=>console.log(i))
```
## _startsWith/_startsEnd
```javascript
_.startsWith('abc', 'a');
// => true
```
## _truncate
```javascript
_.truncate('hi-diddly-ho there, neighborino', {
  'length': 24,
  'separator': ' '
});
```
## _omit
```javascript
var object = { 'a': 1, 'b': '2', 'c': 3 };

_.omit(object, ['a', 'c']);
// => { 'b': '2' }
```
## _pick
```javascript
var object = { 'a': 1, 'b': '2', 'c': 3 };

_.pick(object, ['a', 'c']);
// => { 'a': 1, 'c': 3 }
```
## _random
```javascript
_.random(0, 5);
```
## _union
```javascript
_.union([2, 1], [4, 2], [1, 2]);
// => [2, 1, 4]
```
## _uniq
```javascript
_.uniq([2, 1, 2]);
// => [2, 1]
```
## _orderBy
```javascript
var users = [
  { 'user': 'fred',   'age': 48 },
  { 'user': 'barney', 'age': 34 },
  { 'user': 'fred',   'age': 42 },
  { 'user': 'barney', 'age': 36 }
];

// 以 `user` 升序排序 再 以 `age` 降序排序。
_.orderBy(users, ['user', 'age'], ['asc', 'desc']);
// => 结果: [['barney', 36], ['barney', 34], ['fred', 48], ['fred', 42]]

```
# 🐱 相关文档
1. [为什么不用ES6完全代替loadash](https://blog.csdn.net/zjerryj/article/details/76864598)
2. [常用的loadash方法](http://www.huzerui.com/blog/2017/01/22/lodash-useful-function/)
3. [loadash常用api](https://segmentfault.com/a/1190000008738183)

