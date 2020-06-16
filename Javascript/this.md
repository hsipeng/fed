## this

## this 解释



![函数this](https://user-gold-cdn.xitu.io/2020/4/21/1719d51548402b0f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## this 的绑定



- his的绑定在创建执行上下文时确定
- 大多数情况函数调用的方式决定this的值，this在执行时无法赋值
- this的值为当前执行的环境对象，非严格下总是指向一个对象，严格下可以是任意值
- 全局环境下this始终指向window，严格模式下函数的调用没有明确调用对象的情况下，函数内部this指向undefined，非严格下指向window
- 箭头函数的this永远指向创建当前词法环境时的this
- 作为构造函数时，函数中的this指向实例对象
- this的绑定只受最靠近调用它的成员的引用
- 执行上下文在被执行的时候才会创建，创建执行上下文时才会绑定this，所以this的指向永远是在执行时确定

```
function foo(){
  console.dir(this) // window ,严格下undefined
}
foo()
-----------------------------------------------
function foo(){
  console.dir(this) //非严格Number对象，严格模式 5
}
foo.call(5)
```



* 普通函数中的this

* 箭头函数中的this

  我们可以把箭头函数看成透明，其上下文中的this就是它的this

* 如何改变函数中的this

最简单的方法通过apply、call、bind来给函数绑定this

- apply方法中第一个参数为被调用的函数中的this指向，传入你想要绑定的this值即可，第二个参数为被调用函数的参数集合，通常是个数组
- call与apply方法基本一致，区别在于传入参数形式不同，call传入的参数为可变参数列表，参数按逐个传入
- bind方法与以上不同的是不会直接调用函数，只是先绑定函数的this，到要使用的时候调用即可，此方法返回一个绑定this与参数之后的新函数，其传入参数形式同call
- 通过变量保留指定this来达到固定this



### 如何正确判断this的指向？

如果用一句话说明 this 的指向，那么即是: 谁调用它，this 就指向谁。

但是仅通过这句话，我们很多时候并不能准确判断 this 的指向。因此我们需要借助一些规则去帮助自己：

this 的指向可以按照以下顺序判断:

#### 全局环境中的 this

浏览器环境：无论是否在严格模式下，在全局执行环境中（在任何函数体外部）this 都指向全局对象 `window`;

node 环境：无论是否在严格模式下，在全局执行环境中（在任何函数体外部），this 都是空对象 `{}`;

#### 是否是 `new` 绑定

如果是 `new` 绑定，并且构造函数中没有返回 function 或者是 object，那么 this 指向这个新对象。如下:

> 构造函数返回值不是 function 或 object。`new Super()` 返回的是 this 对象。

```
function Super(age) {
    this.age = age;
}

let instance = new Super('26');
console.log(instance.age); //26
```

> 构造函数返回值是 function 或 object，`new Super()`是返回的是Super种返回的对象。

```
function Super(age) {
    this.age = age;
    let obj = {a: '2'};
    return obj;
}

let instance = new Super('hello'); 
console.log(instance);//{ a: '2' }
console.log(instance.age); //undefined
```

#### 函数是否通过 call,apply 调用，或者使用了 bind 绑定，如果是，那么this绑定的就是指定的对象【归结为显式绑定】。

```
function info(){
    console.log(this.age);
}
var person = {
    age: 20,
    info
}
var age = 28;
var info = person.info;
info.call(person);   //20
info.apply(person);  //20
info.bind(person)(); //20
```

这里同样需要注意一种**特殊**情况，如果 call,apply 或者 bind 传入的第一个参数值是 `undefined` 或者 `null`，严格模式下 this 的值为传入的值 null /undefined。非严格模式下，实际应用的默认绑定规则，this 指向全局对象(node环境为global，浏览器环境为window)

```
function info(){
    //node环境中:非严格模式 global，严格模式为null
    //浏览器环境中:非严格模式 window，严格模式为null
    console.log(this);
    console.log(this.age);
}
var person = {
    age: 20,
    info
}
var age = 28;
var info = person.info;
//严格模式抛出错误；
//非严格模式，node下输出undefined（因为全局的age不会挂在 global 上）
//非严格模式。浏览器环境下输出 28（因为全局的age会挂在 window 上）
info.call(null);
```

#### 隐式绑定，函数的调用是在某个对象上触发的，即调用位置上存在上下文对象。典型的隐式调用为: `xxx.fn()`

```
function info(){
    console.log(this.age);
}
var person = {
    age: 20,
    info
}
var age = 28;
person.info(); //20;执行的是隐式绑定
```

#### 默认绑定，在不能应用其它绑定规则时使用的默认规则，通常是独立函数调用。

非严格模式： node环境，执行全局对象 global，浏览器环境，执行全局对象 window。

严格模式：执行 undefined

```
function info(){
    console.log(this.age);
}
var age = 28;
//严格模式；抛错
//非严格模式，node下输出 undefined（因为全局的age不会挂在 global 上）
//非严格模式。浏览器环境下输出 28（因为全局的age会挂在 window 上）
//严格模式抛出，因为 this 此时是 undefined
info(); 
```

#### 箭头函数的情况：

箭头函数没有自己的this，继承外层上下文绑定的this。

```
let obj = {
    age: 20,
    info: function() {
        return () => {
            console.log(this.age); //this继承的是外层上下文绑定的this
        }
    }
}

let person = {age: 28};
let info = obj.info();
info(); //20

let info2 = obj.info.call(person);
info2(); //28
```

