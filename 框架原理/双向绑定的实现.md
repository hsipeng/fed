# Proxy比defineproperty优劣对比?

**双向绑定**其实已经是一个老掉牙的问题了,只要涉及到MVVM框架就不得不谈的知识点,但它毕竟是Vue的三要素之一.
**Vue三要素**:
* 响应式: 例如如何监听数据变化,其中的实现方法就是我们提到的双向绑定
* 模板引擎: 如何解析模板
* 渲染: Vue如何将监听到的数据变化和解析后的HTML进行渲染


## 基于数据劫持双向绑定的实现思路
**数据劫持**是双向绑定各种方案中比较流行的一种,最著名的实现就是Vue。
基于数据劫持的双向绑定离不开Proxy与Object.defineProperty等方法对对象/对象属性的”劫持”,我们要实现一个完整的双向绑定需要以下几个要点。
1 利用Proxy或Object.defineProperty生成的Observer针对对象/对象的属性进行”劫持”,在属性发生变化后通知订阅者
2 解析器Compile解析模板中的Directive(指令)，收集指令所依赖的方法和数据,等待数据变化然后进行渲染
3 Watcher属于Observer和Compile桥梁,它将接收到的Observer产生的数据变化,并根据Compile提供的指令进行视图渲染,使得数据变化促使视图变化


## Proxy实现的双向绑定的特点
* Proxy可以直接监听对象而非属性
* Proxy可以直接监听数组的变化
* Proxy有多达13种拦截方法,不限于apply、ownKeys、deleteProperty、has等等是Object.defineProperty不具备的。
Proxy返回的是一个新对象,我们可以只操作新的对象达到目的,而Object.defineProperty只能遍历对象属性直接修改。
Proxy作为新标准将受到浏览器厂商重点持续的性能优化，也就是传说中的新标准的性能红利。
当然,Proxy的劣势就是兼容性问题,而且无法用polyfill磨平,因此Vue的作者才声明需要等到下个大版本(3.0)才能用Proxy重写。


## 参考
* [https://www.cxymsg.com/guide/devsProxy.html#_3-proxy实现的双向绑定的特点](https://www.cxymsg.com/guide/devsProxy.html#￼_3-proxy实现的双向绑定的特点)