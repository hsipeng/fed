# redux 与 react-redux

## createStore 实现

```javascript
function createStore(reducer, state = null){
  let listeners = [];
  let getState = ()=>state;
  let subscribe = (listener) => listeners.push(listener)
  let dispatch = (action){
    state = reducer(state, action);
    listeners.forEach(fn => fn())
  }
  return {
    getState,
    subscribe,
    dispatch
  }
}
```



## redux 中间件
在dispatch前后，执行一些代码，达到增强dispatch的效果，有点类似装饰器的原理。

## compose
把一个函数数组，按照顺序，从数组最后向前按照顺序执行，并且，把前一个执行的函数返回值，作为下一个执行函数的入参

```javascript
export default function compose(...funcs) {
  if (funcs.length === 1) {
    return funcs[0]
  }
  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```


## applyMiddleware
```javascript
export default function applyMiddleware(…middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    const store = createStore(reducer, preloadedState, enhancer)
    let dispatch = store.dispatch
    let chain = []

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```

## 参考
* [redux 简单实现](https://www.cxymsg.com/guide/redux.html#简单实现redux)
* [redux源码分析之五：applyMiddleware - 有赞美业前端团队 - SegmentFault 思否](https://segmentfault.com/a/1190000016296209)
* [react-redux一点就透，我这么笨都懂了！ - 掘金](https://juejin.im/post/5af00705f265da0ba60fb844)
