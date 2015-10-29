title: redux深入学习之中间件机制
date: 2015-10-29 11:41:20
tags: react | flux | redux
---

>上一篇文章讲解了redux如何使用，本篇文章将进一步深入，从redux的源码入手，深入学习redux的中间件机制。
在这里我们会以一个`redux-thunk`中间件为例，逐步分解redux的中间机制如何操作，如何执行。
闲话不多说，上代码。

### 如何加载中间件
```javascript
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from './reducers/index';

// create a store that has redux-thunk middleware enabled
const createStoreWithMiddleware = applyMiddleware(
  thunk
)(createStore);

const store = createStoreWithMiddleware(rootReducer);
```
这里需要用到`redux`中提供的一个工具方法，叫做`applyMiddleware`,向该方法传入你想要使用的中间件，完了之后再传入`createStore`方法，
最终形成新的创建store的方法。

这显然是一个**装饰器模式**，通过不同的中间件对`createStore`方法进行修饰，最后形成新的`createStore`方法，那么创建的store就具有这些中间件的特性，
非常出色的设计，惊喜不仅在这，看了之后的代码你就更不得不佩服作者的代码设计能力。

瞬间觉得别人都是码神，而我就是码农有木有/(ㄒoㄒ)/~~

### 中间件加载机制的实现
先来看`applyMiddleware`方法的实现
```javascript
import compose from './compose';

/**
 * Creates a store enhancer that applies middleware to the dispatch method
 * of the Redux store. This is handy for a variety of tasks, such as expressing
 * asynchronous actions in a concise manner, or logging every action payload.
 *
 * See `redux-thunk` package as an example of the Redux middleware.
 *
 * Because middleware is potentially asynchronous, this should be the first
 * store enhancer in the composition chain.
 *
 * Note that each middleware will be given the `dispatch` and `getState` functions
 * as named arguments.
 *
 * @param {...Function} middlewares The middleware chain to be applied.
 * @returns {Function} A store enhancer applying the middleware.
 */
export default function applyMiddleware(...middlewares) {
  return (next) => (reducer, initialState) => {
    var store = next(reducer, initialState);
    var dispatch = store.dispatch;
    var chain = [];

    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    };
    chain = middlewares.map(middleware => middleware(middlewareAPI));
    dispatch = compose(...chain)(store.dispatch);

    return {
      ...store,
      dispatch
    };
  };
}
```
这就是redux里面这个方法的源码，其中还一半是注释有木有。。。本来以为肯定有百来行代码的
当然这里不得不说es6的特性提供了非常多的帮助，所以为了省力吧es6玩透还是灰常有必要的（更别说为了装X了(*^__^*) ）

从这里开始代码就有点绕了，我们逐行分析
```javascript
return (next) => (reducer, initialState) => {...}
```
整个`applyMiddleware`方法就是返回了一个方法，根据`applyMiddleware`方法的使用，我们可以知道`next`就是`createStore`方法，
因为最终我们要返回的是一个装饰过的`createStore`方法，那么接收的参数肯定是不会变，所以最终我们调用`createStoreWithMiddleware`方法其实就是调用
```javascript
function (reducer, initialState) {
	var store = next(reducer, initialState); // next即为最初的createStore方法
	// ...以下省略
}
```

```javascript
var store = next(reducer, initialState);
var dispatch = store.dispatch;
var chain = [];
```
这里没什么好讲的，首先创建了一个store，这个store就是最原始的通过`createStore`创建的store，后两行只是变量赋值

```javascript
var middlewareAPI = {
  getState: store.getState,
  dispatch: (action) => dispatch(action)
};
chain = middlewares.map(middleware => middleware(middlewareAPI));
dispatch = compose(...chain)(store.dispatch);
```
这里是关键，必须详细进行讲解。

首先，这边声明了一个`middlewareAPI`对象，这个对象包含两个方法：

1. getState：store中的getState方法的引用
2. dispatch：对本身的dispatch方法进行一次封装

然后
```javascript
chain = middlewares.map(middleware => middleware(middlewareAPI));
```
我们来仔细看看这行代码，首先我们对所有的中间件进行一个map，map结果就是调用中间件方法，将`middlewareAPI`作为参数传入,
这里我们拿`redux-thunk`中间件举例，来看看一个中间件是长什么样子的，传入的参数又是用来干嘛的。
```javascript
export default function thunkMiddleware({ dispatch, getState }) {
  return next => action =>
    typeof action === 'function' ?
      action(dispatch, getState) :
      next(action);
}
```
`redux-thunk`的功能是让action支持异步，让我们可以在action中跟服务器进行交互等操作，而他的实现。。。(⊙﹏⊙)b是的，又是这么几行代码。

我们回顾之前的代码，在map所有中间件的时候我们调用了`thunkMiddleware`方法，传入两个方法`dispatch`和`getState`，然后返回了一个方法，
我们大致抽象一下，应该如下：
```javascript
function (next) {
  
  return function (action) {
    typeof action === 'function' ?
      action(dispatch, getState) :
      next(action)
  }
  
}
```

于是我们接下去分析`applyMiddleware`里面的代码，
```javascript
chain = middlewares.map(middleware => middleware(middlewareAPI));
```
现在我们知道chain是一个数组，每一项是调用每个中间件之后的返回函数

```javascript
dispatch = compose(...chain)(store.dispatch);
```
compose是redux里面的一个帮助函数，代码如下：
```javascript
export default function compose(...funcs) {
  return arg => funcs.reduceRight((composed, f) => f(composed), arg);
}
```
~~~~(>_<)~~~~我已经不想再吐槽什么了，

我们看到这边先调用了`compose`函数，传入了结构后的`chain`数组，然后`compose`函数返回的也是一个函数：
```javascript
function (arg) {
  return funcs.reduceRight((composed, f) => f(composed), arg);
  // funcs就是中间件数组
}
```
然后我们把`store.dispatch`函数作为`arg`传入这个结果，这里reduceRight可以参考[这里](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/ReduceRight)
。那么这边得到的结果是什么呢？
```javascript
// 假设中间件数组是[A, B, C]
// 那么结果就是A(B(C(store.dispatch)))
```

再次结合`redux-thunk`来看，我们假设只有一个中间件，那么最终的`dispatch`方法就是
```javascript
function (action) {
  typeof action === 'function' ?
    action(dispatch, getState) :
    next(action)
}
// 这里的next方法，就是真正的store.dispatch方法
// 这里的dispatch是(action) => store.dispatch(action)
```
我们再结合`redux-thunk`的使用方法来分析一下，
```javascript
function incrementAsync() {
  return dispatch => {
    setTimeout(() => {
      // Yay! Can invoke sync or async actions with `dispatch`
      dispatch(increment());
    }, 1000);
  };
}
```
这是使用`redux-thunk`时可以定义的异步action，我们触发action的时候调用的是
```javascript
dispatch(incrementAsync())
```
`incrementAsync`返回的是
```javascript
function (dispatch) {
  setTimeout(() => {
    // Yay! Can invoke sync or async actions with `dispatch`
    dispatch(increment());
  }, 1000);
}
```
这个时候我们回想经过中间件加工的`dispatch`方法：
```javascript
function (action) {
  typeof action === 'function' ?
    action(dispatch, getState) :
    next(action)
}
// 这里的next方法，就是真正的store.dispatch方法
// 这里的dispatch是(action) => store.dispatch(action)
```
action是一个函数，所以`action === 'function' ?`成立，那么就执行action， 并把中间件接收到的dispatch方法（`(action) => store.dispatch(action)`）方法作为参数传入，
在异步方法执行完之后再次触发真正的action。如果action不是异步的，那么久直接返回一个对象，这个时候`action === 'function' ?`不成立，就直接调用`next`，
也就是原始的`store.dispatch`方法。

我们再接着想，如果我们有许多个中间件，那么没一个中间件的`next`就是下一个中间件直到最后一个中间件调用`store.dispatch`为止。

以上的代码非常绕，建议去专研一下源码。这么精简的代码包含了非常多的函数式编程的思想，也用到了装饰器模式的原理，不得不说：

>**太烧脑啦/(ㄒoㄒ)/~~**