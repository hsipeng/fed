# HTML

## doctype的作用是什么？

DOCTYPE是html5标准网页声明，且必须声明在HTML文档的第一行。来告知浏览器的解析器用什么文档标准解析这个文档，不同的渲染模式会影响到浏览器对于 CSS 代码甚至 JavaScript 脚本的解析

文档解析类型有：

- BackCompat：怪异模式，浏览器使用自己的怪异模式解析渲染页面。（如果没有声明DOCTYPE，默认就是这个模式）
- CSS1Compat：标准模式，浏览器使用W3C的标准解析渲染页面。

> IE8还有一种介乎于上述两者之间的近乎标准的模式，但是基本淘汰了。



## 这三种模式的区别是什么？

- 标准模式(standards mode)：页面按照 HTML 与 CSS 的定义渲染
- 怪异模式(quirks mode)模式： 会模拟更旧的浏览器的行为
- 近乎标准(almost standards)模式： 会实施了一种表单元格尺寸的怪异行为（与IE7之前的单元格布局方式一致），除此之外符合标准定义



## HTML5与HTML4的不同之处

- 文件类型声明（<!DOCTYPE>）仅有一型：<!DOCTYPE HTML>。
- 新的解析顺序：不再基于SGML。
- 新的元素：section, video, progress, nav, meter, time, aside, canvas, command, datalist, details, embed, figcaption, figure, footer, header, hgroup, keygen, mark, output, rp, rt, ruby, source, summary, wbr。
- input元素的新类型：date, email, url等等。
- 新的属性：ping（用于a与area）, charset（用于meta）, async（用于script）。
- 全域属性：id, tabindex, repeat。
- 新的全域属性：contenteditable, contextmenu, draggable, dropzone, hidden, spellcheck。
- 移除元素：acronym, applet, basefont, big, center, dir, font, frame, frameset, isindex, noframes, strike, tt



## script标签中defer和async的区别？

- defer：浏览器指示脚本在文档被解析后执行，script被异步加载后并不会立刻执行，而是等待文档被解析完毕后执行。
- async：同样是异步加载脚本，区别是脚本加载完毕后立即执行，这导致async属性下的脚本是乱序的，对于script有先后依赖关系的情况，并不适用。 ![2019-06-13-07-13-42](https://xiaomuzhu-image.oss-cn-beijing.aliyuncs.com/c84fdc0e47268832fa8914ab4d125002.png)

> 蓝色线代表网络读取，红色线代表执行时间，这俩都是针对脚本的；绿色线代表 HTML 解析



## 前端储存的方式

cookies、localstorage、sessionstorage、Web SQL、IndexedDB



## 这些方式的区别是什么？

- cookies： 在HTML5标准前本地储存的主要方式，优点是兼容性好，请求头自带cookie方便，缺点是大小只有4k，自动请求头加入cookie浪费流量，每个domain限制20个cookie，使用起来麻烦需要自行封装
- localStorage：HTML5加入的以键值对(Key-Value)为标准的方式，优点是操作方便，永久性储存（除非手动删除），大小为5M，兼容IE8+
- sessionStorage：与localStorage基本类似，区别是sessionStorage当页面关闭后会被清理，而且与cookie、localStorage不同，他不能在所有同源窗口中共享，是会话级别的储存方式
- Web SQL：2010年被W3C废弃的本地数据库数据存储方案，但是主流浏览器（火狐除外）都已经有了相关的实现，web sql类似于SQLite，是真正意义上的关系型数据库，用sql进行操作，当我们用JavaScript时要进行转换，较为繁琐。
- IndexedDB： 是被正式纳入HTML5标准的数据库储存方案，它是NoSQL数据库，用键值对进行储存，可以进行快速读取操作，非常适合web场景，同时用JavaScript进行操作会非常方便。





