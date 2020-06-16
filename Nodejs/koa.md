# Koa
Koa 实例

app 实例、context、request、response

文档
*  [index API](https://github.com/demopark/koa-docs-Zh-CN/blob/master/api/index.md)  
*  [context API](https://github.com/demopark/koa-docs-Zh-CN/blob/master/api/context.md)  
*  [request API](https://github.com/demopark/koa-docs-Zh-CN/blob/master/api/request.md)  
*  [response API](https://github.com/demopark/koa-docs-Zh-CN/blob/master/api/response.md) 


## koa 主流程
```javascript
class Emitter {
  // node 内置模块
  constructor() {}
}
class Koa extends Emitter {
  constructor(options) {
    super();
    options = options || {};
    this.middleware = [];
    this.context = {
      method: "GET",
      url: "/url",
      body: undefined,
      set: function(key, val) {
        console.log("context.set", key, val);
      }
    };
  }
  use(fn) {
    this.middleware.push(fn);
    return this;
  }
  listen() {
    const fnMiddleware = compose(this.middleware);
    const ctx = this.context;
    const handleResponse = () => respond(ctx);
    const onerror = function() {
      console.log("onerror");
    };
    fnMiddleware(ctx)
      .then(handleResponse)
      .catch(onerror);
  }
}
function respond(ctx) {
  console.log("handleResponse");
  console.log("response.end", ctx.body);
}



```


## koa-compose (洋葱模型)

```javascript
function compose(middleware) {
  if (!Array.isArray(middleware))
    throw new TypeError(“Middleware stack must be an array!”);
  for (const fn of middleware) {
    if (typeof fn !== "function")
      throw new TypeError("Middleware must be composed of functions!");
  }

  // 传入对象 context 返回Promise
  return function(context, next) {
    // last called middleware #
    let index = -1;
    return dispatch(0);
    function dispatch(i) {
      if (i <= index)
        return Promise.reject(new Error("next() called multiple times"));
      index = i;
      let fn = middleware[i];
      if (i === middleware.length) fn = next;
      if (!fn) return Promise.resolve();
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err);
      }
    }
  };
}


```

类似结构

```javascript
// 这样就可能更好理解了。
// simpleKoaCompose
const [fn1, fn2, fn3] = this.middleware;
const fnMiddleware = function(context) {
  return Promise.resolve(
    fn1(context, function next() {
      return Promise.resolve(
        fn2(context, function next() {
          return Promise.resolve(
            fn3(context, function next() {
              return Promise.resolve();
            })
          );
        })
      );
    })
  );
};
fnMiddleware(ctx)
  .then(handleResponse)
  .catch(onerror);


```

## co 自执行generate
```javascript

function co(gen) {
  var ctx = this;
  var args = slice.call(arguments, 1);

  // we wrap everything in a promise to avoid promise chaining,
  // which leads to memory leak errors.
  // see https://github.com/tj/co/issues/180
  return new Promise(function(resolve, reject) {
    // 把参数传递给gen函数并执行
    if (typeof gen === "function") gen = gen.apply(ctx, args);
    // 如果不是函数 直接返回
    if (!gen || typeof gen.next !== "function") return resolve(gen);

    onFulfilled();

    /**
     * @param {Mixed} res
     * @return {Promise}
     * @api private
     */

    function onFulfilled(res) {
      var ret;
      try {
        ret = gen.next(res);
      } catch (e) {
        return reject(e);
      }
      next(ret);
    }

    /**
     * @param {Error} err
     * @return {Promise}
     * @api private
     */

    function onRejected(err) {
      var ret;
      try {
        ret = gen.throw(err);
      } catch (e) {
        return reject(e);
      }
      next(ret);
    }

    /**
     * Get the next value in the generator,
     * return a promise.
     *
     * @param {Object} ret
     * @return {Promise}
     * @api private
     */

    // 反复执行调用自己
    function next(ret) {
      // 检查当前是否为 Generator 函数的最后一步，如果是就返回
      if (ret.done) return resolve(ret.value);
      // 确保返回值是promise对象。
      var value = toPromise.call(ctx, ret.value);
      // 使用 then 方法，为返回值加上回调函数，然后通过 onFulfilled 函数再次调用 next 函数。
      if (value && isPromise(value)) return value.then(onFulfilled, onRejected);
      // 在参数不符合要求的情况下（参数非 Thunk 函数和 Promise 对象），将 Promise 对象的状态改为 rejected，从而终止执行。
      return onRejected(
        new TypeError(
          "You may only yield a function, promise, generator, array, or object, “ +
            ‘but the following object was passed: “’ +
            String(ret.value) +
            '"'
        )
      );
    }
  });
}



```

## koa 和 express 简单对比

### 基于 Promises 的控制流程
没有回调地狱。
通过 try/catch 更好的处理错误。
无需域。

### Koa 非常精简
不同于 Connect 和 Express, Koa 不含任何中间件.
不同于 Express, 不提供路由.
不同于 Express, 不提供许多便捷设施。 例如，发送文件.
Koa 更加模块化.

### Koa 对中间件的依赖较少
例如, 不使用 “body parsing” 中间件，而是使用 body 解析函数。

### Koa 抽象 node 的 request/response
减少攻击。
更好的用户体验。
恰当的流处理。

### Koa 路由（第三方库支持）
由于 Express 带有自己的路由，而 Koa 没有任何内置路由，但是有 koa-router 和 koa-route 第三方库可用。同样的, 就像我们在 Express 中有 helmet 保证安全, 对于 koa 我们有 koa-helmet 和一些列的第三方库可用。


## 参考
* [https://github.com/demopark/koa-docs-Zh-CN/blob/master/koa-vs-express.md](https://github.com/demopark/koa-docs-Zh-CN/blob/master/koa-vs-express.md)
* [学习 koa 源码的整体架构，浅析koa洋葱模型原理和co原理 - 掘金](https://juejin.im/post/5e69925cf265da571e262fe6#heading-16)