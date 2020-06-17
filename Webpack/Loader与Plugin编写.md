# Webpack Loader 与 plugin
1. Loader 

	* 用法准则
		* * **简单易用**。
		* 使用**链式**传递。
		* **模块化**的输出。
		* 确保**无状态**。
		* 使用 **loader utilities**。
		* 记录 **loader 的依赖**。
		* 解析**模块依赖关系**。
		* 提取**通用代码**。
		* 避免**绝对路径**。
		* 使用 **peer dependencies**。

**src/loader.js**
```javascript

import { getOptions } from ‘loader-utils’;

export default function loader(source) {
  const options = getOptions(this);

  source = source.replace(/\[name\]/g, options.name);

  return `export default ${ JSON.stringify(source) }`;
}
```

**test/compiler.js**
```javascript
import path from ‘path’;
import webpack from ‘webpack’;
import memoryfs from ‘memory-fs’;

export default (fixture, options = {}) => {
  const compiler = webpack({
    context: __dirname,
    entry: `./${fixture}`,
    output: {
      path: path.resolve(__dirname),
      filename: 'bundle.js',
    },
    module: {
      rules: [{
        test: /\.txt$/,
        use: {
          loader: path.resolve(__dirname, '../src/loader.js'),
          options: {
            name: 'Alice'
          }
        }
      }]
    }
  });

  compiler.outputFileSystem = new memoryfs();

  return new Promise((resolve, reject) => {
    compiler.run((err, stats) => {
      if (err || stats.hasErrors()) reject(err);

      resolve(stats);
    });
  });
};
```

**test/loader.test.js**

```javascript
import compiler from ‘./compiler.js’;

test(‘Inserts name and outputs JavaScript’, async () => {
  const stats = await compiler(‘example.txt’);
  const output = stats.toJson().modules[0].source;

  expect(output).toBe(‘export default “Hey Alice!\\n”’);
});
```

**.babelrc**

```json
{
  “presets”: [[
    “@babel/preset-env”,
    {
      “targets”: {
        “node”: true
      }
    }
  ]]
}
```

pacakage.json

```json

{
  “name”: “go-ws-app”,
  “version”: “1.0.0”,
  “main”: “index.js”,
  “license”: “MIT”,
	“dependencies”: {
		“webpack”,
		“memory-fs”,
		“loader-utils”
	},
	“devDependecies”:{
		“jest”,
		“babel-jest”,
		“@babel/preset-env”
	}
}
```


2. Plugins

一个插件由以下构成
* 一个具名 JavaScript 函数。
* 在它的原型上定义 apply 方法。
* 指定一个触及到 webpack 本身的  [事件钩子](https://webpack.docschina.org/api/compiler-hooks/) 。
* 操作 webpack 内部的实例特定数据。
* 在实现功能后调用 webpack 提供的 callback。

```javascript
// 一个 JavaScript class
class MyExampleWebpackPlugin {
  // 将 `apply` 定义为其原型方法，此方法以 compiler 作为参数
  apply(compiler) {
    // 指定要附加到的事件钩子函数
    compiler.hooks.emit.tapAsync(
      ‘MyExampleWebpackPlugin’,
      (compilation, callback) => {
        console.log(‘This is an example plugin!');
        console.log('Here’s the `compilation` object which represents a single build of assets:', compilation);

        // 使用 webpack 提供的 plugin API 操作构建结果
        compilation.addModule(/* ... */);

        callback();
      }
    );
  }
}
```



### 钩子函数定义

* synchronous hooks(同步钩子)

  * SyncHook(同步钩子)
    * 通过 new SyncHook([params]) 定义。
    * 使用 tap 方法触及。
    * 使用 call(…params) 方法调用。

  * Bail Hooks(保释钩子)
    * 通过 SyncBailHook[params] 定义。
    * 使用 tap 方法触及。
    * 使用 call(…params) 方法调用。
    * 在这些 hooks 类型中，一个接一个地调用每个插件，并且 callback 会传入特定的
      args### 。如果任何插件返回任何非 undefined 值，则由 hook 返回该值，并且不再继续调用插件 callback。许多有用的事件，如
      optimizeChunks ,
      optimizeChunkModules 都是 SyncBailHooks 类型。
  * Waterfall Hooks(瀑布钩子)	
    * 通过 SyncWaterfallHook[params] 定义。
    * 使用 tap 方法触及。
    * 使用 call(...params) 方法调用。
    * 在这些 hooks 类型中，一个接一个地调用每个插件，并且会使用前一个插件的返回值，作为后一个插件的参数。必须考虑插件的执行顺序。 它必须接收来自先前执行插件的参数。第一个插件的值是
      init### 。因此，waterfall hooks 必须提供至少一个参数。这种插件模式用于 Tapable 实例，而这些实例与
      ModuleTemplate  ,
      ChunkTemplate  等 webpack 模板相互关联。





### asynchronous hooks(异步钩子)



* Async Series Hook(异步串行钩子)
  * 通过 AsyncSeriesHook[params] 定义。
  * 使用 tap/tapAsync/tapPromise 方法触及。
  * 使用 callAsync(...params) 方法调用。
  * 调用插件处理函数，传入所有参数，并使用签名
    (err?: Error) -> void### 调用回调函数。处理函数按照注册顺序进行调用。所有处理函数都被调用之后会调用
    callback 。 这种插件模式常用于
    emit ,
    run  等事件。
* Async waterfall(异步瀑布钩子)
  * 通过 AsyncWaterfallHook[params] 定义。
  * 使用 tap/tapAsync/tapPromise 方法触及。
  * 使用 callAsync(...params) 方法调用。
  * 调用插件处理函数，传入当前值作为参数，并使用签名
    (err?: Error) -> void 调用回调函数。在调用处理函数中的
    nextValue ，是下一个处理函数的当前值。第一个处理函数的当前值是
    init 。所有处理函数都被调用之后，会调用
    callback ，并且传入最后一个值。如果任何处理函数向
    err 方法传递一个值，则会调用 callback，并且将这个错误传入，然后不再调用处理函数。 这种插件模式常用于
    before-resolve ,
    after-resolve 等事件。
* Async Series Bail
  * 通过 AsyncSeriesBailHook[params] 定义。
  * 使用 tap/tapAsync/tapPromise 方法触及。
  * 使用 callAsync(...params) 方法调用。
  * someMethod() { // 调用一个 hook: this.hooks.compilation.call();
* Async Parallel
  * 通过 AsyncParallelHook[params] 定义。
  *  使用 tap/tapAsync/tapPromise 方法触及。
  * 使用 callAsync(...params) 方法调用。
* Async Series Bail
  * 通过 AsyncSeriesBailHook[params] 定义。
  * 使用 tap/tapAsync/tapPromise 方法触及。
  * 使用 callAsync(...params) 方法调用。

