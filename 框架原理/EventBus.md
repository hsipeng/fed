# Event Bus
## React/Vue不同组件之间是怎么通信的?
**Vue**
1. 父子组件用Props通信
2. 非父子组件用Event Bus通信
3. 如果项目够复杂,可能需要Vuex等全局状态管理库通信
4. $dispatch(已经废除)和$broadcast(已经废除)
**React**
1. 父子组件,父->子直接用Props,子->父用callback回调
2. 非父子组件,用发布订阅模式的Event模块
3. 项目复杂的话用Redux、Mobx等全局状态管理管库
4. 用新的 [Context Api](https://juejin.im/post/5a7b41605188257a6310fbec) 
我们大体上都会有以上回答,接下来很可能会问到如何实现Event(Bus),因为这个东西太重要了,几乎所有的模块通信都是基于类似的模式,包括安卓开发中的Event Bus,Node.js中的Event模块(Node中几乎所有的模块都依赖于Event,包括不限于http、stream、buffer、fs等).


## 代码实现
```javascript
class EventEmeitter {
  constructor() {
    this._events = this._events || new Map(); // 储存事件/回调键值对
    this._maxListeners = this._maxListeners || 10; // 设立监听上限
  }
}



// 触发名为type的事件
EventEmeitter.prototype.emit = function(type, ...args) {
  let handler;
  // 从储存事件键值对的this._events中获取对应事件回调函数
  handler = this._events.get(type);
  if (args.length > 0) {
    handler.apply(this, args);
  } else {
    handler.call(this);
  }
  return true;
};

// 监听名为type的事件
EventEmeitter.prototype.addListener = function(type, fn) {
  // 将type事件以及对应的fn函数放入this._events中储存
  if (!this._events.get(type)) {
    this._events.set(type, fn);
  }
};



EventEmeitter.prototype.removeListener = function(type, fn) {
  const handler = this._events.get(type); // 获取对应事件名称的函数清单

  // 如果是函数,说明只被监听了一次
  if (handler && typeof handler === 'function') {
    this._events.delete(type, fn);
  } else {
    let postion;
    // 如果handler是数组,说明被监听多次要找到对应的函数
    for (let i = 0; i < handler.length; i++) {
      if (handler[i] === fn) {
        postion = i;
      } else {
        postion = -1;
      }
    }
    // 如果找到匹配的函数,从数组中清除
    if (postion !== -1) {
      // 找到数组对应的位置,直接清除此回调
      handler.splice(postion, 1);
      // 如果清除后只有一个函数,那么取消数组,以函数形式保存
      if (handler.length === 1) {
        this._events.set(type, handler[0]);
      }
    } else {
      return this;
    }
  }
};

```

## 参考
* [面试官:既然React/Vue可以用Event Bus进行组件通信,你可以实现下吗? - 掘金](https://juejin.im/post/5ac2fb886fb9a028b86e328c)