# CSS



## 盒模型
页面渲染时，dom 元素所采用的 **布局模型**。可通过`box-sizing`进行设置。根据计算宽高的区域可分为：

- `content-box` (W3C 标准盒模型)
- `border-box` (IE 盒模型)
- `padding-box`





## BFC


**块级格式化上下文**，是一个独立的渲染区域，让处于 BFC 内部的元素与外部的元素相互隔离，使内外元素的定位不会相互影响。

> IE下为 Layout，可通过 zoom:1 触发

- 触发条件:
  - 根元素
  - `position: absolute/fixed`
  - `display: inline-block / table`
  - `float` 元素
  - `ovevflow` !== `visible`
- 规则:
  - 属于同一个 BFC 的两个相邻 Box 垂直排列
  - 属于同一个 BFC 的两个相邻 Box 的 margin 会发生重叠
  - BFC 中子元素的 margin box 的左边， 与包含块 (BFC) border box的左边相接触 (子元素 absolute 除外)
  - BFC 的区域不会与 float 的元素区域重叠
  - 计算 BFC 的高度时，浮动子元素也参与计算
  - 文字层不会被浮动层覆盖，环绕于周围
- 应用:
  - 阻止`margin`重叠
  - 可以包含浮动元素 —— 清除内部浮动(清除浮动的原理是两个`div`都位于同一个 BFC 区域之中)
  - 自适应两栏布局
  - 可以阻止元素被浮动元素覆盖

### 什么是BFC？BFC的布局规则是什么？如何创建BFC？

Box 是 CSS 布局的对象和基本单位，页面是由若干个Box组成的。

元素的类型 和 `display` 属性，决定了这个 Box 的类型。不同类型的 Box 会参与不同的 Formatting Context。

> Formatting Context

Formatting Context 是页面的一块渲染区域，并且有一套渲染规则，决定了其子元素将如何定位，以及和其它元素的关系和相互作用。

Formatting Context 有 BFC (Block formatting context)，IFC (Inline formatting context)，FFC (Flex formatting context) 和 GFC (Grid formatting context)。FFC 和 GFC 为 CC3 中新增。

> [BFC布局规则](https://www.w3.org/TR/2011/REC-CSS2-20110607/visuren.html#block-formatting)

- BFC内，盒子依次垂直排列。
- BFC内，两个盒子的垂直距离由 `margin` 属性决定。属于同一个BFC的两个相邻Box的margin会发生重叠【符合合并原则的margin合并后是使用大的margin】
- BFC内，每个盒子的左外边缘接触内部盒子的左边缘（对于从右到左的格式，右边缘接触）。即使在存在浮动的情况下也是如此。除非创建新的BFC。
- BFC的区域不会与float box重叠。
- BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。
- 计算BFC的高度时，浮动元素也参与计算。

> 如何创建BFC

- 根元素
- 浮动元素（float 属性不为 none）
- position 为 absolute 或 fixed
- overflow 不为 visible 的块元素
- display 为 inline-block, table-cell, table-caption

> BFC 的应用

1. 防止 margin 重叠 (同一个BFC内的两个两个相邻Box的 `margin` 会发生重叠，触发生成两个BFC，即不会重叠)
2. 清除内部浮动 (创建一个新的 BFC，因为根据 BFC 的规则，计算 BFC 的高度时，浮动元素也参与计算)
3. 自适应多栏布局 (BFC的区域不会与float box重叠。因此，可以触发生成一个新的BFC)





## 层叠上下文
元素提升为一个比较特殊的图层，在三维空间中 **(z轴)** 高出普通元素一等。

- 触发条件
  - 根层叠上下文(`html`)
  - `position`
  - css3属性
    - `flex`
    - `transform`
    - `opacity`
    - `filter`
    - `will-change`
    - `-webkit-overflow-scrolling`
- 层叠等级：层叠上下文在z轴上的排序
  - 在同一层叠上下文中，层叠等级才有意义
  - `z-index`的优先级最高



![img](https://user-gold-cdn.xitu.io/2019/2/14/168e9d9f3a1d368b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





## 居中布局 水平，水平垂直
水平居中

- 行内元素: `text-align: center`
- 块级元素: `margin: 0 auto`
- `absolute + transform`
- `flex + justify-content: center`

垂直居中

- `line-height: height`
- `absolute + transform`
- `flex + align-items: center`
- `table`

水平垂直居中

- `absolute + transform`
- `flex + justify-content + align-items`



## 选择器优先级
- `!important` > 行内样式 > `#id` > `.class` > `tag` > * > 继承 > 默认
- 选择器 **从右往左** 解析
- 无论多少个class 也不能覆盖id



## 去除浮动


- 通过增加尾元素清除浮动
  - `:after /   : clear: both`
- 创建父级 BFC
- 父级设置高度



## link @import


`link`功能较多，可以定义 RSS，定义 Rel 等作用，而`@import`只能用于加载 css

当解析到`link`时，页面会同步加载所引的 css，而`@import`所引用的 css 会等到页面加载完才被加载

`@import`需要 IE5 以上才能使用

`link`可以使用 js 动态引入，`@import`不行



## em/rem/px 区别


- px：绝对单位，页面按精确像素展示。
- em：相对单位，基准点为父节点字体的大小，如果自身定义了font-size按自身来计算（浏览器默认字体是16px），整个页面内1em不是一个固定的值。
- rem：相对单位，可理解为”root em”, 相对根节点html的字体大小来计算，CSS3新加属性，chrome/firefox/IE9+支持



## css 预处理器
CSS预处理器的原理: 是将类 CSS 语言通过 **Webpack 编译** 转成浏览器可读的真正 CSS。在这层编译之上，便可以赋予 CSS 更多更强大的功能，常用功能:

- 嵌套
- 变量
- 循环语句
- 条件语句
- 自动前缀
- 单位转换
- mixin复用



## 伪类/伪元素


### 是什么？

伪类（pseudo-class） 是一个以冒号(:)作为前缀，被添加到一个选择器末尾的关键字，当你希望样式在特定状态下才被呈现到指定的元素时，你可以往元素的选择器后面加上对应的伪类。

伪元素用于创建一些不在文档树中的元素，并为其添加样式。比如说，我们可以通过::before来在一个元素前增加一些文本，并为这些文本添加样式。虽然用户可以看到这些文本，但是这些文本实际上不在文档树中。

### ## 区别

其实上文已经表达清楚两者区别了，伪类是通过在元素选择器上加入伪类改变元素状态，而伪元素通过对元素的操作进行对元素的改变。

我们通过`p::before`对这段文本添加了额外的元素，通过 `p:first-child`改变了文本的样式。



## flex 的理解

`flex`属性是`flex-grow`, `flex-shrink` 和 `flex-basis`的简写，默认值为`0 1 auto`。后两个属性可选。

> ```css
> .item {
>   flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
> }
> ```

该属性有两个快捷值：`auto` (`1 1 auto`) 和 none (`0 0 auto`)。

建议优先使用这个属性，而不是单独写三个分离的属性，因为浏览器会推算相关值。



`flex-grow`属性定义项目的放大比例，默认为`0`，即如果存在剩余空间，也不放大。

> ```css
> .item {
>   flex-grow: <number>; /* default 0 */
> }
> ```



`flex-shrink`属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。

> ```css
> .item {
>   flex-shrink: <number>; /* default 1 */
> }
> ```



`flex-basis`属性定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为`auto`，即项目的本来大小。

> ```css
> .item {
>   flex-basis: <length> | auto; /* default auto */
> }
> ```

它可以设为跟`width`或`height`属性一样的值（比如350px），则项目将占据固定空间。