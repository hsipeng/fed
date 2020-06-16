# Webpack
Webpack 核心

![157B7C0F-4C06-4E4B-9680-CC85FC7256BD](/var/folders/tl/8cg4zhtd4117s8c0fzp8bxt40000gn/T/net.shinyfrog.bear/BearTemp.AgkSvk/157B7C0F-4C06-4E4B-9680-CC85FC7256BD.png)


	* **Webpack** 就是一个大公司
	* **Compiler** 就像公司的董事会，只把握公司大方向的走向，不关心细节实现
	* **Compilation** 就像是 CEO，由董事会任命，主要操心整个公司运行，调度各个部门运作
	* **ModuleFactory** 就像各个部门了，从事打造各种产品细节
	* 最终输出的 bundle 就像是具体的产品

Webpack 可以认为是一种基于事件流的编程范例，内部的工作流程都是基于 **插件** 机制串接起来；
而将这些插件粘合起来的就是webpack自己写的基础类  [Tapable](https://github.com/webpack/tapable/blob/master/lib/Tapable.js)  是，plugin方法就是该类暴露出来的；
后面我们将看到核心的对象 Compiler、Compilation 等都是继承于该对象

1. Tapable
	* apply

更改上下文为当前 this,传入当前 **Tapable** 的上下文

![17C1A2EC-5664-4CE4-B522-32627D5DE88F](/var/folders/tl/8cg4zhtd4117s8c0fzp8bxt40000gn/T/net.shinyfrog.bear/BearTemp.EvA4Br/17C1A2EC-5664-4CE4-B522-32627D5DE88F.png)


2. Webpack 事件流
事件节点
	* entry-option：初始化options
	* run：开始编译
	* make：从entry开始递归的分析依赖，对每个依赖模块进行build
	* before-resolve - after-resolve： 对其中一个模块位置进行解析
	* build-module ：开始构建 (build) 这个module,这里将使用文件对应的loader加载
	* normal-module-loader：对用loader加载完成的module(是一段js代码)进行编译,用  [acorn](https://github.com/ternjs/acorn)  编译,生成ast抽象语法树。
	* program： 开始对ast进行遍历，当遇到require等一些调用表达式时，触发 call require 事件的handler执行，收集依赖，并。如：AMDRequireDependenciesBlockParserPlugin等
	* seal： 所有依赖build完成，下面将开始对chunk进行优化，比如合并,抽取公共模块,加hash
	* optimize-chunk-assets：压缩代码，插件 **UglifyJsPlugin** 就放在这个阶段
	* bootstrap： 生成启动代码
	* emit： 把各个chunk输出到结果文件