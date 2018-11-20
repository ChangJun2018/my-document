---
title: Js原型、原型链、this、继承的笔记
date: '2018-11-05 23:35:44'
tag: 
  - javascript
meta:
  -
    name: Js继承
    content: Js原型、原型链、this、继承的知识点
---

重新阅读高程第六章，将继承这一块的知识点重新记录整理，相当于自己再一次的笔记。
<!-- more -->
# 🎈 前言
重新阅读高程第六章，将继承这一块的知识点重新记录整理，相当于自己再一次的笔记。这两天会不断翻阅以加深这一次的收获。前端路上不过遇到什么，翻过去就完了。我觉着没有什么比这更让人激动的了。哈哈。加油！

# 🎁 原型与原型链

* 所有的引用类型（数组、对象、函数）都具有对象特性，即可自由扩展属性（null）除外
* 所有的引用类型（数组、对象、函数）都有一个\_proto\_属性（隐式原型），属性值是一个普通的对象
* 所有的函数都有一个prototype（显式原型），属性值也是一个普通的对象
* 所有的引用类型（数组、对象、函数），\_proto\_(隐式原型)都指向它的构造函数的prototype（显式原型）属性值。
* 当试图得到一个对象的某个属性时，如果这个对象本身没有这个属性时，那么回去它的\_proto\_属性（隐式原型）即它的构造函数的prototype（显式原型）中去寻找
* 使用hasOwnProperty判断这个属性是否是对象本身的属性
* 使用instanceof 和  isPrototypeof 判断实例是否属于某种类型或者一个实例是否属于它的父类型

# 🌡️ new操作的时候发生了什么？

* 创建一个新对象
* 将构造函数的作用域赋给新对象（因此this就指向了这个新对象）
* 执行构造函数中的代码（为这个新对象添加属性）
* 返回新对象

# ⌚ 关于js执行上下文

Javascript可执行代码类型有```全局代码```、```函数代码```、```eval代码```,当js执行到一个函数的时候，它会进行准备工作，这个准备工作就叫做```执行上下文```,那我们写的函数有很多，就会有很多的执行上下文，那么js引擎和如何管理执行上下文的。

## 执行上下文栈

JavaScript 引擎创建了执行上下文栈（Execution context stack，ECS）来管理执行上下文

```javascript
// 假设ECStack是一个数组,js开始解释代码时最先遇到的就是全局代码
// 所以初始化的时候会先向执行上下文栈中压入一个全局执行上下文
ECStack = [
    globalContext // 全局执行上下文 程序结束时才会被清空
];


function fun1(){
  console.log('1')
}
function fun2(){
   fun1()
}
function fun3(){
   fun2()
}
fun3()

// js引擎是如何处理这段代码的
// 1. fun3()
ECStack.push(<fun3> functionContext)
// 2. fun2() 执行fun3的时候发现调用了fun2
ECStack.push(<fun2> functionContext)
// 3. fun1() 执行fun2的时候发现调用了fun1
ECStack.push(<fun1> functionContext)
// fun1执行完毕
ECStack.pop();
// fun2执行完毕
ECStack.pop();
// fun3执行完毕
ECStack.pop();

```

# ⌚ 关于js中this的问题

__this永远指向最后调用它的那个对象__

## 如何修改this
* 使用ES6箭头函数
> __箭头函数的 this 始终指向函数定义时的 this，而非执行时。__
* 在函数内部使用\_this = this
* 使用apply、call、bind
* new实例化一个对象


# 🎀 继承的实现方式

## 实现方式
* 原型链
* 借用构造函数
* 组合继承
* 原型式继承
* 寄生式继承
* 寄生组合式继承

## 原型链继承
<span data-type="color" style="color:#464b5d">利用原型让一个引用类型继承另一个引用类型的属性和方法</span>

```javascript
function Person() {
  // 定义父类型构造函数属性
  this.name = 'changjun'
}
// 父类型构造函数原型上定义方法
Person.prototype.getName = function() {
  console.log(this.name)
}

function Child() {}

// 将父类型实例赋值给子类型构造函数原型上
Child.prototype = new Person()

let child = new Child()
console.log(child.getName())
```

::: danger
- 引用类型的属性被所有实例共享
- 在创建子类型的实例时，不能向父类型构造函数传参
:::

```javascript
function Person() {
  this.names = ['changjun','changjun2']
}

function Child() {}

Child.prototype = new Person()
let child1 = new Child()
child1.names.push('changjun3')
console.log(child1.names) // ["changjun", "changjun2", "changjun3"]
let child2 = new Child()
console.log(child2.names) // ["changjun", "changjun2", "changjun3"]
```


## 借用构造函数继承
<span data-type="color" style="color:#464b5d">在子类型构造函数内部调用父类型构造函数</span>
::: tip
- 优点1 避免了引用类型的属性被所有实例共享
- 优点2 可以在类型构造函数中向父类型构造函数传参
:::

```javascript
function Person() {
  // 定义父类型构造函数属性
  this.names = ['changjun','changjun2']
}

function Child() {
  // 在子类型构造函数中调用父类型构造函数
  Person.call(this)
}

let child1 = new Child()
child1.names.push('changjun3')
console.log(child1.names) // ["changjun", "changjun2", "changjun3"]
let child2 = new Child()
console.log(child2.names) // ["changjun", "changjun2"]
```

::: danger
- 方法都在构造函数中定义,每次创建实例都会创建一遍方法
- 方法都在构造函数中定义，函数无法复用
:::

```javascript
function Person(name) {
  this.name = name
}

function Child(name) {
  // 在子类型构造函数中调用父类型构造函数并传参
  Person.call(this,name)
}

let child1 = new Child('changjun')
console.log(child1.name) // changjun
let child2 = new Child('changjun2')
console.log(child2.name) // changjun2
```

## 组合继承（伪经典继承）
<span data-type="color" style="color:#464b5d">将原型链和借用构造函数组合在一起，使用原型链实现对原型属性和方法的继承，通过借用构造函数来实现对实例属性的继承。</span>

```javascript
function Person(name){
  // 父类构造函数定义两个属性
  this.name = name;
  this.colors = ['red','blue','green'];
}

// 父类构造函数原型上定义一个方法
Person.prototype.getName = function (){
  console.log(this.name)
}

function Child(name,age){
  // 子类构造函数调用父类构造函数并传入参数
  // 继承
  Person.call(this,name)
  // 定义属性
  this.age = age
}

// 将父类型实例赋值给子类型构造函数原型
Child.prototype = new Person()

// 在新的子类型构造函数原型上定义方法
Child.prototype.alertAge = function(){
  console.log(this.age)
}

let ins = new Child('changjun',18)
ins.colors.push('black')
console.log(ins.colors) // ["red", "blue", "green", "black"]
ins.getName() // changjun
ins.alertAge() // 18

let ins2 = new Child('changjun2',20)
ins2.colors.push('black')
console.log(ins.colors) // ["red", "blue", "green", "black"]
ins2.getName() // changjun2
ins2.alertAge() // 20

```

## 原型式继承
<span data-type="color" style="color:#464b5d">借助已有的对象创建新的对象，将传入的对象作为创建的对象的原型</span>

::: danger
- 包含引用类型的属性值始终都会共享相应的值，这点跟原型链继承一样。
:::

```javascript
function createObj(o) {
  // 临时性的创建一个构造函数
  function F() {}
  // 将传入的对象作为构造函数的原型
  F.prototype = o
  // 返回这个临时类型的新实例
  return new F()
}

let person = {
  name: 'changjun',
  friends: ['changjun1', 'changjun2', 'changjun3']
}

let person1 = createObj(person)
let person2 = createObj(person)

// 此处需要注意当我们此时打印person1.name时
// 是person1是因为我们给person增加了一个属性并非是修改了原型上的name值
// 当我们注释掉这行此时再去打印person1.name 不用多说就是changjun了
person1.name = 'person1'
console.log(person1.name) // person1
console.log(person2.name) // changjun

person1.friends.push('changjun4')
console.log(person2.friends) //  ["changjun1", "changjun2", "changjun3", "changjun4"]
```

```javascript
// Object.create()方法规范了原型式继承，
// 接收两个参数，一个是用作新对象原型的对象（可选）一个为新对象定义额外属性的对象。
let person = {
  name: 'changjun',
  friends: ['changjun1', 'changjun2', 'changjun3']
}

let person1 = Object.create(person)
let person2 = Object.create(person)

// Object.create()方法的第二个参数与Object.defineProperties()方法的第二个参数格式一样
// 每个属性都是自己的描述符定义的
// 以这种方式指定的任何属性都会覆盖原型对象上的同名属性
let person3 = Object.create(person,{
  name:{
    value:"ZhangSan"
  }
})

person1.name = 'person1'
console.log(person1.name) // person1
console.log(person2.name) // changjun

person1.friends.push('changjun4')
console.log(person2.friends) //  ["changjun1", "changjun2", "changjun3", "changjun4"]

console.log(person3.name) //ZhangSan  覆盖掉原型上对象的属性
```

## 寄生式继承
<span data-type="color" style="color:#464b5d">创建一个仅用于封装继承过程的函数，该函数在内部以某种形式来做增强对象，最后返回对象。</span>

::: danger
- 跟借用构造函数模式一样，每次创建对象都会创建一遍方法。
:::

```javascript
function createObj(o){
  let clone = Object.create(o) // 调用函数创建一个新的对象
  clone.getName = function (){ // 增强这个对象 添加一个新方法
    console.log('hello')
  }
  return clone;  // 返回这个对象
}

let person = {
  name:'changjun',
  friends: ['changjun1', 'changjun2', 'changjun3']
}

let person1 = createObj(person)
person1.getName() // hello
```

## 寄生组合式继承
<span data-type="color" style="color:#464b5d">组合继承存在的问题是,无论在什么情况下都会调用两次父类型构造函数,一次在创建子类型原型的时候,另一次是在子类型构造函数内部。</span>

::: tip
- 只调用一次父类型构造函数
- 避免了在子类型原型上穿件不必要的多余的属性
- 原型链不变，能够正常使用instanceof
:::


```javascript
function Parent(name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
}

Parent.prototype.getName = function() {
  console.log(this.name)
}

function Child(name,age){
  Parent.call(this,name)  // 第二次调用父类新,在新对象上创建实例属性
  this.age = age
}

Child.prototype = new Parent() // 第一次调用父类型 获取父类型的属性到原型中

Child.prototype.sayName = function(){
  console.log(this.name)
}
let child1 = new Child('changjun',18)
console.log(child1) // 有两组实例属性一组在实例上,一组在原型中
```

```javascript
/**
 * 寄生组合继承
 * @param {*} child 子类型构造函数
 * @param {*} parent 父类型构造函数
 * 
 */
function inheritPrototype(child,parent){
  // 1. 创建父类型原型的副本
  let prototype = Object(parent.prototype)
  // 2. 为创建的副本添加constructor属性
  prototype.constructor = child;
  // 3. 将新创建的对象即副本重新赋值给子类型的原型
  child.prototype = prototype
}

function Parent(name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
}

Parent.prototype.getName = function() {
  console.log(this.name)
}

function Child(name,age){
  Parent.call(this,name)
  this.age = age;
}

inheritPrototype(Child,Parent);
Child.prototype.sayName = function (){
  console.log(this.name)
}

let child1 = new Child('changjun',18)
console.log(child1);

```




