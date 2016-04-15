title: 学习使用redux
date: 2015-10-28 21:57:37
tags: [教程, react]
---

>在react火热的年代，flux作为fb提出的最适合react的数据模型，时下有非常多的实现。
而redux作为在众多的flux地实现中脱颖而出，及其精简的代码，却能带来实用的功能，正好自己的项目中要用，所以让我们来分析redux

为什么要写这个文档呢，因为我看官方文档各种看不懂啊，琢磨了半天都不理解，最后是去看了源码才看明白
因为他的一些概念没搞清楚的话，就不知道他的文档在说什么。为了不让更多的人掉坑里面，这里稍微解释一些概念。

学习redux需要知道redux的三个部分：

1. action
2. reducer
3. store

### action
redux中得action就是你自己定义的一个动作，什么是动作？你可以理解为用户的动作你做出的反应，最简单地例子就是当你进行分页的时候，
跳到特定的页数这个动作。我们可以通过类似如下的代码定义action：

```javascript
/*
 * action types
 */
 
export const ADD_TODO = 'ADD_TODO';
export const COMPLETE_TODO = 'COMPLETE_TODO';
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER';

/*
 * other constants
 */

export const VisibilityFilters = {
  SHOW_ALL: 'SHOW_ALL',
  SHOW_COMPLETED: 'SHOW_COMPLETED',
  SHOW_ACTIVE: 'SHOW_ACTIVE'
};

/*
 * action creators
 */

export function addTodo(text) {
  return { type: ADD_TODO, text };
}

export function completeTodo(index) {
  return { type: COMPLETE_TODO, index };
}

export function setVisibilityFilter(filter) {
  return { type: SET_VISIBILITY_FILTER, filter };
}
```
在这里定义action之后用来出发的，通过`dispatch`方法来触发动作，在这里action只是一些常亮的定义。
`dispatch`方法接收的参数是一个`object`，而且`object`必须包含一个`type`属性，告诉我们需要执行的操作。
而对象里面的包含的其他属性则可以在执行动作的时候用作其他用途。

`dispatch`方法是会在store连接组件的时候随着组件的props传递到各个组件的，所以组件内都是可以用的。

### reducer
这是在redux里面提出来的概念，具体啥含义请参考官网，因为我也解释不清楚╮(╯▽╰)╭

reducer在这里是核心，因为redux是只有一个store的，所以整个app的状态和数据都存储在一个store里面，
如果所有状态变化都在store里面进行逻辑操作，那么这个store肯定是无法维护的，所以在这里我们把状态的变化放到了reducer里面。
我们先来看一下如何定义一个reducer：

```javascript
import { combineReducers } from 'redux';
import { ADD_TODO, COMPLETE_TODO, SET_VISIBILITY_FILTER, VisibilityFilters } from './actions';
const { SHOW_ALL } = VisibilityFilters;

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return action.filter;
  default:
    return state;
  }
}

function todos(state = [], action) {
  switch (action.type) {
  case ADD_TODO:
    return [...state, {
      text: action.text,
      completed: false
    }];
  case COMPLETE_TODO:
    return [
      ...state.slice(0, action.index),
      Object.assign({}, state[action.index], {
        completed: true
      }),
      ...state.slice(action.index + 1)
    ];
  default:
    return state;
  }
}

const todoApp = combineReducers({
  visibilityFilter,
  todos
});

export default todoApp;
```
如你所见，reducer只是一个方法，在reducer里面根据传入的action里面的type进行不同的state地操作。
**在这里必须理解一点，在你调用`dispatch`方法的时候传入的action动作就是reducer里面接受的action**

在这里我们唯一用到redux的功能只有`combineReducers`方法，这个方法的作用是把不同的reducer合并到一起，
因为在创建store的时候我们只能传入一个reducer，但是我们不可能把所有逻辑操作写到一个reducer里面，所以这边提供了这个方法。

### store
store的作用即是整合所有的reducer，然后提供一些帮助方法，例如`dispatch`等方法让我们使用，
代码如下：

```
let store = createStore(reducer);
```
是的，就是这么简单。

### 如何跟react一起使用
请参考[文档](http://redux.js.org/docs/basics/UsageWithReact.html)
这边并不进行详细讲解，以为这不是这篇文章的重点，以后会单独在其他文章中进行讲解。

### 理解
如何理解redux的重点就在于，redux如何处理整个数据流的走向。
基本的思路如下：

```
component --dispatch(action)--> reducer --update(state)--> store --update(props)--> component
```
这就是整个数据的走向

**看到这里，你们肯定跟我有相同的想法：reducer到底是个什么东西！**

那么我们就来理解一下

我们看一下reducer的定义：
```javascript
function todos(state = [], action) {
  switch (action.type) {
  case ADD_TODO:
    return [...state, {
      text: action.text,
      completed: false
    }];
  case COMPLETE_TODO:
    return [
      ...state.slice(0, action.index),
      Object.assign({}, state[action.index], {
        completed: true
      }),
      ...state.slice(action.index + 1)
    ];
  default:
    return state;
  }
}
```
首先他接受两个参数，一个是state，一个是action。

action我们知道是在dispatch的时候传入的告诉我们进行什么操作的，那么state是什么？
state就是store里面存着的状态，即数据。我们可以看到`每个reducer都会返回state`，而这些state最终都会保存在store里面。

**每次触发一个action的时候，store调用reducer，同时传入本身保存着的state，reducer根据传入的state和action返回新的state，
store更新state，返回以props的方式传入组件，这就形成了整个数据流循环**

以上是redux的最基础使用，这也是redux的核心，然后后面还有一堆redux的扩展以及中间件进行学习，这仅仅是一个开始，以后还有更长的路要走^_^
