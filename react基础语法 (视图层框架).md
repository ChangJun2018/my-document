---
title: react基础语法 (视图层框架)
date: '2018-07-20 09:05:13'
tag: 
  - react
meta:
  -
    name: react笔记
    content: react语法,学习

---

学习react的学习笔记,方便自己经常查阅加深印象。
<!-- more -->

# react基础语法 (视图层框架)

1. 一个react组件初始化的样子

```javascript
import React, { Component } from 'react';

class test extends Component {
    render() {
        return (
            <div>
                
            </div>
        );
    }
}

export default test;
```

1. this.state用于存储组件内状态的值

```javascript
this.state={
            inputValue:'',
            list:[]
        }
```

1. this.setState用于改变组件状态值

```javascript
this.setState({
   inputValue:'', 
   list:[]
})
```

- 最新的react中提倡修改this.setState应该

```javascript
this.setState(()=>{
    return {
       inputValue:'', 
       list:[] 
    }
})
```

返回一个对象，可以`()`表示返回一个对象

```javascript
this.setState(()=>({
    inputValue:'', 
    list:[]  
}))
```

1. 样式的导入与应用,在组件内导入样式即可，最好将样式放在所导入的所有组件最后

```javascript
// 在jsx中添加样式名需className
<input className='input'/>
```

1. 在修改组件状态值的时候可以有一个参数prevState表示之前的状态值

```javascript
this.setState((prevState)=>({
            list:[...prevState.list,prevState.inputValue],
            inputValue:''
}))
```

1. 父组件通过属性传值，子组件通过this.props接受父组件穿过的值

```javascript
//  父组件通过属性传值
<TodoItem 
key={index}
content={item} 
index={index}
deleteItem={this.handleDeleteClick}
/>
// 子组件通过this.props接收传值
this.props.index
```

1. 组件内的方法,向子组件传事件会存在this指向问题

```javascript
<TodoItem 
key={index}
content={item} 
index={index}
deleteItem={this.handleDeleteClick.bind(this)}
/>
// 让代码看着舒服一点的话一般在constructor中将this指向修改好
constructor(props){
        super(props);
        this.handleInputValue = this.handleInputValue.bind(this)
        this.handleBtnClick = this.handleBtnClick.bind(this)
        this.handleDeleteClick = this.handleDeleteClick.bind(this)
}
```

1. 子组件通过propTypes来校验父组件传过来的值的类型

```javascript
// 要在组件内先引入 import PropTypes form 'prop-types'
TodoItem.propTypes = {
    // PropTypes.arrayOf(PropTypes.number,PropTypes.string) 既可以是number也可以是string
    // PropTypes.oneOfType([PropTypes.number,PropTypes.string]) 要么是number要么是stringn
    // 要注意oneOfType里面是一个数组
    content:PropTypes.string.isRequired,
    index:PropTypes.number,
    deleteItem:PropTypes.func
}
 
// 通过defaultProps定义默认值
TodoItem.defaultProps = {
    test : 'Hello ChangJun'
}
```

1. 通过ref找dom

```javascript
// 在标签上绑定属性ref 
ref={(input)=>{ this.input = input }}
// 通过this.input可直接获取到dom
const value = this.input.value
```

# react的生命周期函数

> 生命周期函数指在某一个时刻组件会自动执行的函数

```javascript
  // 当组件即将被挂在到页面的时刻自动执行
  componentWillMount(){
    console.log('componentWillMount')
  }
  // 组件被挂在到页面之后自动执行
  componentDidMount(){
    console.log('componentDidMount')
  }
  // 组件被更新之前,它会自动被执行
  shouldComponentUpdate(){
    console.log('shouldComponentUpdate')
    return true
  }
  // 组件被更新之前,自动执行
  // 在shouldComponentUpdate之后
  // 如果shouldComponentUpdate返回false,它就不会执行
  componentWillUpdate(){
    console.log('componentWillUpdate')
  }
  // 组件更新完成之后,自动执行
  componentDidUpdate(){
    console.log('componentDidUpdate')
  }
  // 子组件要从父组件接受参数
  // 父组件的render函数被重新执行了,那么componentWillReceiveProps就会被执行
  // 如果这个组件第一次存在于父组件中,不会执行
  // 如果这个组件之前存在于父组件中,才会被执行
  componentWillReceiveProps(){
    console.log('child componentWillReceiveProps')
  }
  // 当这个组件从页面中被剔除的时候,会被执行
  componentWillUnmount(){
    console.log('child componentWillUnmount')
  }
```

# react中的动画

```
//通过在标签上动态添加css属性然后在css中利用transition或者animation做动画
<div className={this.state.show?'show':'hide'}>hello</div>
```

1. `npm install react-transition-group --save`
2. `import { CSSTransition} from 'react-transition-group'`在组件中引用

```javascript
// 在要加动画组件之外加上<CSSTransition></CSSTransition>组件
<CSSTransition
in={this.state.show} //根据状态
timeout={1000} //动画时长
unmountOnExit //结束时移除dom
classNames='fade'  // 绑定动画前缀
onEntered={(el)=>{el.style.color='blue'}} //入场动画结束的钩子
appear={true} //第一次加载时有动画
>
  <div>Hello</div>
</CSSTransition>

// CSSTransition会在相应组件上加上样式
.fade-enter /* 动画进入 */
.fade-enter-active /* 入场动画结束之前 */
.fade-enter-done /* 入场动画结束 */
.fade-exit /* 出场动画开始 */
.fade-exit-active /* 出场动画结束之前 */
.fade-exit-done /* 出场动画结束 */

// 如果有多个组件动画
import { TransitionGroup} from 'react-transition-group'
//在每个组件上报上CSSTransition,在需要多个组件动画的外层套上
//<TransitionGroup></TransitionGroup>
```

# react的其它

1. state、render、props的关系

> 当state或者props发生改变时,render函数就会被执行一次,当父组件的render函数被执行时,子组件的render函数也会被执行

1. react dom

```javascript
  1. 生成state数据 
  2. jsx模版 
  3. 数据+模版生成虚拟DOM(虚拟DOM就是一个js对象,用来描述真实的DOM) 
  4. 用虚拟的DOM生成真实的DOM,显示
  5. state发生变化
  6. 数据+模版生成虚拟DOM
  7. 比较原始虚拟DOM和新的DOM的区别
  8. 直接操作DOM
```

1. react中的组件

- 普通组件 (一个普通的react组件)
- ui组件 (仅负责ui,不负责代码的业务逻辑,使用到的函数、状态从容器组件中传过来)
- 容器组件 (render函数中返回一个ui组件,负责代码的业务逻辑)
- 无状态组件 (当一个普通组件中只有render函数时,可以将其写为无状态组件,返回一个函数。无状态组件要比普通的组件性能好很多，因为无状态组件中只返回一个函数，没有生命周期函数。)

# Redux Reducer+Flux (数据层框架)

- 概念理解

1. React Components (组件) `借书的用户`
2. Action Creators `说的我要借什么书的这句话`
3. Store (存储数据的公共区域) `管理员`
4. Reducers `记录本`

- 安装 npm install redux --save
- 使用

1. 创建store文件夹下面的index.js 对应store
2. 在创建store之前应先创建reducer.js
3. reducer返回一个函数(store,action)
4. 在index.js中引入reducer并将reducer给到createStore index.js

```javascript
import { createStore } from 'redux'
import reducer from './reducer'
const store = createStore(reducer)
export default store;
//reducer.js
const defaultState = {
  inputValue:'',
  list:[]
}jav
export default (state = defaultState, action)=>{
  return state
} 
```

- 在组件中使用的流程

1. 在组件中绑定事件
2. 定义action(告诉store你要干什么)
3. store.dispatch(action) store派发action到reducer
4. reducer接受到action根据action.type来进行相应的操作
5. reducer可以接收state,但绝不能修改state
6. 在reducer中深拷贝一份原来的state进行修改并返回新的state
7. 在组件constructor中store.subscribe(func)订阅一个store.
8. 当store里面的数据发生改变的时候会自动执行func函数
9. func中可以this.setState(store.getState())同步store到组件中

```javascript
// jsx
// 组件中
<Input onChange={this.changeInputValue}/>
changeInputValue(e){
    const action={
      type:'change_input_value',
      value:e.target.value
    }
    store.dispatch(action)
}
// reducer.js中
if(action.type === 'change_input_value'){
    const newState = JSON.parse(JSON.stringify(state))
    newState.inputValue = action.value
    return newState
} 
```

- redux-devtools(中间件) redux调试工具,要使用需要在创建store时，调用createStore方法除了传入reducer还应加上`window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()`
- redux-thunk(中间件) 一句话总结redux-thunk可以使action返回的不仅仅是一个对象,可以返回一个函数 使用了redux-thunk的store/index.js

```javascript
import { createStore compose , applyMiddleWare } from 'redux'
import thunk from 'redux-thunk' 
import reducer from './reducer'

# redux调试工具
const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;

# 创建store
# 使用redux-thunk借助redux中的applyMiddleware
const store = createStore(reducer,composeEnhancers(
  applyMiddleware(thunk)
));
# 使用redux-thunk就可以在actioncreater中返回一个函数可以用来异步加载数据,返回的函数中的参数是dispatch
export const getList = ()=>{
    return (dispatch)=>{
        #可以做axios的请求
    }
}
```

- redux-saga redux-saga异步的工作方式,通过Generator函数来异步加载数据

- immutable

> 使用immutable管理store,immutable会将state对象转化成immutable对象,immutable对象的set方法,会结合之前的immutable对象的值返回一个新的对象。 在reducer.js中

```javascript
import { fromJS } from 'immutable'

# 将state转成immutable对象
const defaultState = fromJS({
    show:false
})

state.set('show',true)
state.get('show')
```