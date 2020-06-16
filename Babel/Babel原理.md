# Babel
把 JavaScript 中 es2015/2016/2017/2046 的新语法转化为 es5，让低端运行环境(如浏览器和 node )能够认识并执行


### 使用方法
总共存在三种方式：
使用单体文件 (standalone script)
命令行 (cli)
构建工具的插件 (webpack 的 babel-loader, rollup 的 rollup-plugin-babel)。


### 运行方式和插件
Babel 总共分为三个阶段：解析，转换，生成。
Babel 本身不具有任何转化功能，它把转化的功能都分解到一个个 plugin 里面。因此当我们不配置任何插件时，经过 babel 的代码和输入是相同的。
插件总共分为两种：
* 当我们添加 **语法插件** 之后，在解析这一步就使得 babel 能够解析更多的语法。(顺带一提，babel 内部使用的解析类库叫做 babylon，并非 babel 自行开发)

* 当我们添加 **转译插件** 之后，在转换这一步把源码转换并输出。这也是我们使用 babel 最本质的需求。
比起语法插件，转译插件其实更好理解，比如箭头函数 (a) => a 就会转化为 function (a) {return a}。完成这个工作的插件叫做 babel-plugin-transform-es2015-arrow-functions。


![https://pic3.zhimg.com/80/v2-7b71cd51932cf5d5f5a2491412c1191e_1440w.jpg](https://pic3.zhimg.com/80/v2-7b71cd51932cf5d5f5a2491412c1191e_1440w.jpg)

## 插件编写

```javascript
var babel = require(‘babel-core’);
var t = require(‘babel-types’);
const visitor = {
  BinaryExpression(path) {
    const node = path.node;
    let result;
    // 判断表达式两边，是否都是数字
    if (t.isNumericLiteral(node.left) && t.isNumericLiteral(node.right)) {
      // 根据不同的操作符作运算
      switch (node.operator) {
        case “+”:
          result = node.left.value + node.right.value;
          break
        case “-“:
          result = node.left.value - node.right.value;
          break;
        case “*”:
          result =  node.left.value * node.right.value;
          break;
        case “/“:
          result =  node.left.value / node.right.value;
          break;
        case “**”:
          let i = node.right.value;
          while (--i) {
            result = result || node.left.value;
            result =  result * node.left.value;
          }
          break;
        default:
      }
    }
    // 如果上面的运算有结果的话
    if (result !== undefined) {
      // 把表达式节点替换成number字面量
      path.replaceWith(t.numericLiteral(result));
      let parentPath = path.parentPath;
      // 向上遍历父级节点
      parentPath && visitor.BinaryExpression.call(this, parentPath);
    }
  }
};
module.exports = function (babel) {
  return {
    visitor
  };
}

const babel = require(“babel-core”);
const result = babel.transform(“const result = 1 + 2;”,{
  plugins:[
    require(“./index”)
  ]
});
console.log(result.code); // const result = 3;

//const result = 100 + 10 - 50 >>> const result = 60;
//const result = (100 / 2) + 50 >>> const result = 100;
//const result = (((100 / 2) + 50 * 2) / 50) ** 2 >>> const result = 9;


```

## 参考
* [Babel 插件手册](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md#builders)
* [一口（很长的）气了解 babel - 知乎](https://zhuanlan.zhihu.com/p/43249121)
* [GitHub - axetroy/babel-plugin-pre-calculate-number: pre calculate number expression](https://github.com/axetroy/babel-plugin-pre-calculate-number)