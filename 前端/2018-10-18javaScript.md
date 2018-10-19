---
title: 2018-10-18javaScript 
tags: js,数据类型,函数,面向对象,新特性
grammar_cjkRuby: true
---
## javascript
JavaScript一种直译式脚本语言，是一种动态类型、弱类型、基于原型的语言，内置支持类型。它的解释器被称为JavaScript引擎，为浏览器的一部分，广泛用于客户端的脚本语言，最早是在[HTML](https://baike.baidu.com/item/HTML)（标准通用标记语言下的一个应用）网页上使用，用来给HTML网页增加动态功能。

**名字由来：**在1995年时，由Netscape公司的Brendan Eich，在网景导航者浏览器上首次设计实现而成。因为Netscape与Sun合作，Netscape管理层希望它外观看起来像Java，因此取名为JavaScript。但实际上它的语法风格与Self及Scheme较为接近。

**JS组成：**
- ECMAScript，描述了该语言的语法和基本对象。
- 文档对象模型（DOM），描述处理网页内容的方法和接口。
- 浏览器对象模型（BOM），描述与浏览器进行交互的方法和接口。
****
### 1\. CSS和JS在网页中的放置顺序是怎样的？

- css样式放在页面的head中。（主要是放在底部会出现白屏或无样式内容闪烁。**如果使用 @import 标签,即使 CSS 放入 link, 并且放在头部,也可能出现白屏**）

- js文件放置在body的结尾处。（对于图片和CSS, 在加载时会并发加载(如一个域名下同时加载两个文件). 但在加载 JavaScript 时,会禁用并发,并且阻止其他内容的下载. **所以把 JavaScript 放入页面顶部也会导致 白屏现象**）
- js所依赖的库(比如jquery)放置在js的前面。
*****
### 2\. 解释白屏和FOUC
浏览器的白屏与无样式内容闪烁（Flash of unstyled content）,是由于浏览器加载与显示页面方式不同造成的：
IE 会出现白屏现象，这是因为其等待页面组件包括样式表全部加载完成后才呈现整个页面。若样式表放在页面底部，将会出现白屏。——样式表在页面中位置并不影响页面中组件的下载时间，但是会影响页面的呈现。 IE 为什么会这样做？因为样式表仍在加载，构建呈现树就是一种浪费，因为在所有样式表加载并解释完毕之前无需要绘制任何东西，否则，在其准备好之前显示的内容会遇到FOUC问题。
这样IE 就避免了FOUC问题，也就自然的会出现白屏现象。在IE中，将样式表放在文档底部公导致白屏问题的情形有以下几种：
1. 在新窗口中打开时
2. 重新加载时
3. 作为主页打开时
FireFox 当把样式表放在页面底部时，会遇到FOUC问题，因为FF为了用户体验，对所有都是对页面中的组件逐步呈现。
ps：引入样式表时，有两种方式：link和@import

```
      link:     <link rel = "stylesheet" href = "styles1.css">
      @import: 
                <style type = "text/css">
                    @import url("styles1.css");
                    @import url("styles2.css");
                    ...  ...
                </style>
```

一个style块可以包含多个@import规则，但@import规则必须放在所有其他规则之前。使用@import规则需要注意的是：即便把@import规则放在文档的HEAD标签中，可能导致页面组件下载时的无序性，进而导致白屏（对于IE）和FOUC(对于FF)问题的产生。
为了很好的避免白屏和FOUC问题，请遵循以下规则：**使用LINK标签将样式表放在文档的HEAD中**
****
### 3\. async和defer的作用是什么？有什么区别
都是用来异步加载script 标签的
没有 defer 或 async，浏览器会立即加载并执行指定的脚本，“立即”指的是在渲染该 script 标签之下的文档元素之前，也就是说不等待后续载入的文档元素，读到就加载并执行。

```
<script async src="script.js"></script>
```

有 async，加载和渲染后续文档元素的过程将和 script.js 的加载与执行并行进行（异步）。用在具有 src属性的脚本上。

```
<script defer src="script.js"></script>
```

有 defer，加载后续文档元素的过程将和 script.js 的加载并行进行（异步），但 script.js 的执行要在所有元素解析完成之后，DOMContentLoaded 事件触发之前完成。用在具有 src属性的脚本上。
除此之外还有如下区别：
- async是html5的属性，存在兼容性的问题，IE10以下的浏览器无法使用。

- async的加载和执行与文档读取同步执行，defer加载与async类似，但是执行要等到元素解析完成以后执行，且DOMContentLoaded 事件触发要等到其执行完成后开始。
详细可以参考MDN [https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/script11](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/script)
*****
### 4\. 简述网页的渲染机制
*   解析 HTML 标签, 构建 DOM 树
*   解析 CSS 标签, 构建 CSSOM 树
*   把 DOM 和 CSSOM 组合成 渲染树 (render tree)
*   在渲染树的基础上进行布局, 计算每个节点的几何结构
*   把每个节点绘制到屏幕上 (painting)
    [中文版参考链接12](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction?hl=cn)
	
	
	### JavaScript 定义了几种数据类型? 哪些是原始类型?哪些是复杂类型?原始类型和复杂类型的区别是什么?
JavaScript 语言的每一个值，都属于某一种数据类型。JavaScript 的数据类型，共有六种。（ES6 又新增了第七种 Symbol 类型的值）
1. 数值（number）：整数和小数（比如1和3.14）。
2. 字符串（string）：字符组成的文本（比如”Hello World”）。
3. 布尔值（boolean）：true（真）和false（假）两个特定值。
4. undefined：表示“未定义”或不存在，即由于目前没有定义，所以此处暂时没有任何值。
5. null：表示无值，即此处的值就是“无”的状态。
6. 对象（object）：各种值组成的集合。

通常，数值、字符串、布尔值这三种类型，合称为原始类型（primitive type）的值，即它们是最基本的数据类型，不能再细分了。
对象则称为合成类型（complex type）的值，因为一个对象往往是多个原始类型的值的合成，可以看作是一个存放各种值的容器。至于undefined和null，一般将它们看成两个特殊值。
对象是最复杂的数据类型，又可以分成三个子类型。
1. 狭义的对象（object）
2. 数组（array）
3. 函数（function）

[参考MDN4](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures)
[参考阮一峰 js教程4](http://javascript.ruanyifeng.com/grammar/types.html)
*****
### typeof和instanceof的作用和区别?
###### typeof操作符
返回一个字符串,指示未经计算的操作数的类型。
```
// Numbers
typeof 37 === 'number';
typeof 3.14 === 'number';
typeof Math.LN2 === 'number';
typeof Infinity === 'number';
typeof NaN === 'number'; // 尽管NaN是"Not-A-Number"的缩写
typeof Number(1) === 'number'; // 但不要使用这种形式!

// Strings
typeof "" === 'string';
typeof "bla" === 'string';
typeof (typeof 1) === 'string'; // typeof总是返回一个字符串
typeof String("abc") === 'string'; // 但不要使用这种形式!

// Booleans
typeof true === 'boolean';
typeof false === 'boolean';
typeof Boolean(true) === 'boolean'; // 但不要使用这种形式!

// Symbols
typeof Symbol() === 'symbol';
typeof Symbol('foo') === 'symbol';
typeof Symbol.iterator === 'symbol';

// Undefined
typeof undefined === 'undefined';
typeof declaredButUndefinedVariable === 'undefined';
typeof undeclaredVariable === 'undefined'; 

// Objects
typeof {a:1} === 'object';

// 使用Array.isArray 或者 Object.prototype.toString.call
// 区分数组,普通对象
typeof [1, 2, 4] === 'object';

typeof new Date() === 'object';

// 下面的容易令人迷惑，不要使用！
typeof new Boolean(true) === 'object';
typeof new Number(1) ==== 'object';
typeof new String("abc") === 'object';

// 函数
typeof function(){} === 'function';
typeof Math.sin === 'function';
```
###### instanceof 运算符
用来测试一个对象在其原型链中是否存在一个构造函数的 prototype 属性。用来检测 constructor.prototype 是否存在于参数 object 的原型链上。
typeof对数组（array）和对象（object）返回的字符串都是"object"。所以可以通过instanceof操作符区别数组与对象。
```
var a = [];
var b = {};
a instanceof Array; // true
b instanceof Array; // false
```

### 如何把非数值转化为数值?

[http://javascript.ruanyifeng.com/grammar/number.html#toc1011](http://javascript.ruanyifeng.com/grammar/number.html#toc10)
######Number
可以应用于任何数据类型
Number()转化规则：
- 对于boolean值，true和false分别返回1和0
- 对于null，返回0
- 对于undefined，返回NaN
- 对于数值，只是简单的传入和返回
- 对于字符串。有几种情况
- 字符串只包含数字，则将其转换为十进制数值。对于像“011”，0会被忽略返回11.
- 如果字符串包含有效的十六进制格式的数值，先将其转换为十进制数
- 如果字符串为空，返回0
- Number方法的参数是对象时，将返回NaN，除非是包含单个数值的数组。
- 除以上，都返回NaN
- 对于对象。调用对象的valueof()方法，然后按照前面的规则转换返回的值。
###### parseInt
- 用于将字符串转为整数
- 如果字符串头部有空格，空格会被自动去除。

- 如果parseInt的参数不是字符串，则会先转为字符串再转换。
- 字符串转为整数的时候，是一个个字符依次转换，如果遇到不能转为数字的字符，就不再进行下去，返回已经转好的部分。
- 如果字符串的第一个字符不能转化为数字（后面跟着数字的正负号除外），返回NaN。
- 所以，parseInt的返回值只有两种可能，要么是一个十进制整数，要么是NaN。
- 如果字符串以0x或0X开头，parseInt会将其按照十六进制数解析。
- 对于那些会自动转为科学计数法的数字，parseInt会将科学计数法的表示方法视为字符串，因此导致一些奇怪的结果。
###### parseFloat
- 用于将一个字符串转为浮点数。

- 如果字符串符合科学计数法，则会进行相应的转换。
- 如果字符串包含不能转为浮点数的字符，则不再进行往后转换，返回已经转好的部分。
*****
### toString
- toString()括号里面填上8、16、32等数据可将数值转换成对应的其他进制。。得到的是一个字符串。若存在该进制没有的数值，则忽略我们输入的进制数。 不能用于null，undefined。这种情况下应该使用String()。
### void(0)
这个运算符能向期望一个表达式的值是undefined的地方插入，会产生副作用的表达式。
void 运算符通常只用于获取 undefined 的原始值，一般使用 void(0)（等同于 void 0）。在上述情况中，也可以使用全局变量undefined 来代替（假定其仍是默认值）。
###### void(0)与undefined的区别
1. undefined可以在局部作用域中被覆写。
2. void运算返回之都是undefined。

[MDN void(0)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/void)
*****
#### NaN
NaN，即非数值（Not a Number）是一个特殊的数值（typeof NaN的结果是"number")。这个数值用于表示一个本来要返回的操作数未返回数据的情况。主要出现在将字符串解析成数字出错的场合。举个例子，在其他编程语言中，任何数值除以非数值都会导致错误从而停止代码执行，但在JavaScript中，会返回NaN，并不影响其他代码的运行。
特别之处：
1. 任何NaN的值进行操作都会返回一个NaN。
2. NaN与任何值都不相等即NaN === NaN也是false。

JavaScript定义了isNaN()函数。这个函数接受一个参数，并确定该参数是否“不是数值”。它在接受一个参数后，若参数不是数值，函数会尝试将参数转为数值。任何不能被转换为数值的值都会使函数返回true。
******
### Infinity
Infinity表示“无穷”，用来表示两种场景。一种是一个正的数值太大，或一个负的数值太小，无法表示；另一种是非0数值除以0，得到Infinity。
- Infinity有正负之分，Infinity表示正的无穷，-Infinity表示负的无穷。
```
Infinity === -Infinity // false
1 / -0 // -Infinity
-1 / -0 // Infinity
```
上面代码中，非零正数除以-0，会得到-Infinity，负数除以-0，会得到Infinity。
- 由于数值正向溢出（overflow）、负向溢出（underflow）和被0除，JavaScript 都不报错，而是返回Infinity，所以单纯的数学运算几乎没有可能抛出错误。
- Infinity大于一切数值（除了NaN），-Infinity小于一切数值（除了NaN）。
*****
### ==和===有什么区别
==表示近似相等，当等号两边类型不同，在比较时会进行类型转换。
===表示严格相等，当等号两边类型不同，不会进行类型转换，而是会直接返回 false。
*****
### break和continue
两个都可以用于结束循环，区别在于执行break时会彻底退出循环，彻底终止。
而执行continue时，会停止本次循环，执行下次循环。
*****
### null和undefined
- null表示空值，即该处的值现在为空。调用函数时，某个参数未设置任何值，这时就可以传入null，表示该参数为空。比如，某个函数接受引擎抛出的错误作为参数，如果运行过程中未出错，那么这个参数就会传入null，表示未发生错误。
- undefined表示“未定义”。典型场景：
1. 变量声明了，但没有赋值
2. 调用函数时，应该提供的参数没有提供，该参数等于 undefined
3. 函数没有返回值时，默认返回 undefined
*****
### 运算符优先级
运算符的优先级《JavaScript权威指南》中有个表阐述的很好（我去掉了位运算部分），其中R/L代表结合性是右结合还是左结合，num->num表示操作符期望的数据类型和计算结果类型，lval指左值

|运算符	|操作	|结合性	|类型|
|-----|------|------|-------|
|++	|自增	|R	|lval->num|
|--	|自减|	R	|lval->num|
|-	|求反|	R	|num->num|
|+（一个操作数）	|转换为数字	|R	|num->num|
|~	|按位求反	R|	int->int|
|!	|逻辑非|	R	|bool->bool|
|delete	|删除属性	R|	lval->bool|
|typeof	|检测数据类型	|R	|any->str|
|void|	返回undefined	|R	|any->undefined|
|*、/、%	|乘、除、求余	|L|	num,num->num|
|+、-|	加、减	|L	|num,num->num|
|+	|字符串拼接|	L	|str,str->str|
|<、<=、>、>=	|数字大小或字母表顺序|	L|	num/str,num/str->bool|
|instanceof	|对象类型	|L	|obj,function->bool|
|in	|测试属性是否存在|	L	|str,obj->bool|
|==	|判断相等|	L	|any,any->bool|
|！=|	判断不等|	L	|any,any->bool|
|===	|判断恒等|	L	|any,any->bool|
|!==	|判断非恒等|	L	|any,any->bool|
|&&|	逻辑与	|L	|any,any->any|
| \ \	|逻辑或|	L	|any,any->any|
|?:	|条件运算符|	R	|bool,any,any->any|
|=赋值 *=、/=、+=、-=	|赋值 运算且赋值|	R|	lval,any->any|
|,	|忽略第一个操作数，返回第二个操作数|	L	|any,any->any|
*****
### 标签label
JavaScript 语言允许，语句的前面有标签（label），相当于定位符，用于跳转到程序的任意位置，标签通常与break语句和continue语句配合使用，跳出特定的循环。
```
top:
  for (var i = 0; i < 3; i++){
    for (var j = 0; j < 3; j++){
      if (i === 1 && j === 1) break top;
      console.log('i=' + i + ', j=' + j);
    }
  }
// i=0, j=0
// i=0, j=1
// i=0, j=2
// i=1, j=0
```
上面代码为一个双重循环区块，break命令后面加上了top标签（注意，top不用加引号），满足条件时，直接跳出双层循环。如果break语句后面不使用标签，则只能跳出内层循环，进入下一次的外层循环。

continue语句也可以与标签配合使用。
```
top:
  for (var i = 0; i < 3; i++){
    for (var j = 0; j < 3; j++){
      if (i === 1 && j === 1) continue top;
      console.log('i=' + i + ', j=' + j);
    }
  }
// i=0, j=0
// i=0, j=1
// i=0, j=2
// i=1, j=0
// i=2, j=0
// i=2, j=1
// i=2, j=2
```
上面代码中，continue命令后面有一个标签名，满足条件时，会跳过当前循环，直接进入下一轮外层循环。如果continue语句后面不使用标签，则只能进入下一轮的内层循环。
****
### 三元运算符
JavaScript还有一个三元运算符（即该运算符需要三个运算子）?:，也可以用于逻辑判断。
(条件) ? 表达式1 : 表达式2
上面代码中，如果“条件”为true，则返回“表达式1”的值，否则返回“表达式2”的值。
```
var even = (n % 2 === 0) ? true : false;
```
上面代码中，如果n可以被2整除，则even等于true，否则等于false。它等同于下面的形式。
```
var even;
if (n % 2 === 0) {
  even = true;
} else {
  even = false;
}
```
这个三元运算符可以被视为if...else...的简写形式，因此可以用于多种场合。
```
var myVar;
console.log(
  myVar ?
  'myVar has a value' :
  'myVar do not has a value'
)
// myVar do not has a value
```
上面代码利用三元运算符，输出相应的提示。
```
var msg = '数字' + n + '是' + (n % 2 === 0 ? '偶数' : '奇数');
```
上面代码利用三元运算符，在字符串之中插入不同的值。
****
### 引用类型有哪些？非引用类型有哪些
- 非引用类型 String 类型、Null 类型、Number 类型、Undefined类型、Boolean 类型

- 引用类型（Object、Array、Function、正则）：指的是那些保存在堆内存中的对象，变量中保存的实际上只是一个指针，这个指针执行内存中的另一个位置，由该位置保存对象。

## 函数声明和函数表达式
```
 //函数声明
  function sayHello(){
    console.log('hello')
  }

  //函数调用
  sayHello()
```
```  
//函数表达式
var sayHello = function(){
    console.log('hello');
  }

  sayHello()
```
###### 相同点
函数是一个值，所以和其他值一样，函数也可以进行被输出、被赋值、作为参数传给其他函数等相关操作，不管函数是以什么方式被定义的，当然和其他值的输出还是有些区别的。
作为参数传给其他函数。
###### 不同点
- 函数声明必须有标识符，也就是常说的函数名；函数表达式可以省略函数名。
- 函数声明不用考虑顺序问题。因为他的声明会提前。
- ECMAScript规范中表示，函数声明语句可以出现在全局代码中，或者内嵌在其他函数中，但是不能出现在循环、条件判、或者try/finally以及with语句中。
- 可以创建一个函数表达式立即执行。
******
## 声明前置
###### 变量声明前置
JavaScript 引擎的工作方式是先解析代码，获取所有被声明的变量，然后再一行一行地运行。这造成的结果，就是所有的变量的声明语句，都会被提升到代码的头部，然后给他初始值undefined，然后才逐句执行程序，这就叫做“变量提升”，也即“变量的声明前置”。
###### 函数声明前置
和变量的声明会前置一样，函数声明同样会前置，如果我们使用函数表达式那么规则和变量一样。
如果我们使用函数声明的方式，那么即使函数写在最后也可以在前面语句调用，前提是函数声明部分已经被下载到本地。
*****
## arguments 对象
在 JavaScript 中，arguments 对象是比较特别的一个对象，实际上是当前函数的一个内置属性。arguments 非常类似 Array，但实际上又不是一个 Array 实例
由于 JavaScript 允许函数有不定数目的参数，所以需要一种机制，可以在函数体内部读取所有参数。这就是arguments对象的由来。
arguments 对象的长度是由实参个数而不是形参个数决定的。形参是函数内部重新开辟内存空间存储的变量，但是其与 arguments 对象内存空间并不重叠。对于 arguments 和值都存在的情况下，两者值是同步的，但是针对其中一个无值的情况下，对于此无值的情形值不会得以同步。
arguments 对象中有一个非常有用的属性：callee。arguments.callee 返回此 arguments 对象所在的当前函数引用。在使用函数递归调用时可以使用 arguments.callee 代替函数名本身。
需要注意的是，虽然arguments很像数组，但它是一个对象。数组专有的方法（比如slice和forEach），不能在arguments对象上直接使用。
****
## arguments.callee 方法
该方法本身
*****
## 函数"重载"
在 JS 中
没有重载! 同名函数会覆盖。 但可以在函数体针对不同的参数调用执行相应的逻辑。
*****
## 立即执行函数
在 Javascript 中，圆括号()是一种运算符，跟在函数名之后，表示调用该函数。比如，print()就表示调用print函数。

有时，我们需要在定义函数之后，立即调用该函数。这时，你不能在函数的定义之后加上圆括号，这会产生语法错误。
```
function(){ /* code */ }();
// SyntaxError: Unexpected token (
```
产生这个错误的原因是，function这个关键字即可以当作语句，也可以当作表达式。
```
// 语句
function f() {}

// 表达式
var f = function f() {}
```
为了避免解析上的歧义，JavaScript 引擎规定，如果function关键字出现在行首，一律解释成语句。因此，JavaScript引擎看到行首是function关键字之后，认为这一段都是函数的定义，不应该以圆括号结尾，所以就报错了。

解决方法就是不要让function出现在行首，让引擎将其理解成一个表达式。最简单的处理，就是将其放在一个圆括号里面。
```
(function(){ /* code */ }());
// 或者
(function(){ /* code */ })();
```
上面两种写法都是以圆括号开头，引擎就会认为后面跟的是一个表示式，而不是函数定义语句，所以就避免了错误。这就叫做“立即调用的函数表达式”（Immediately-Invoked Function Expression），简称 IIFE。

注意，上面两种写法最后的分号都是必须的。如果省略分号，遇到连着两个 IIFE，可能就会报错。
```
// 报错
(function(){ /* code */ }())
(function(){ /* code */ }())
```
上面代码的两行之间没有分号，JavaScript 会将它们连在一起解释，将第二行解释为第一行的参数。

推而广之，任何让解释器以表达式来处理函数定义的方法，都能产生同样的效果，比如下面三种写法。
```
var i = function(){ return 10; }();
true && function(){ /* code */ }();
0, function(){ /* code */ }();
```
甚至像下面这样写，也是可以的。
```
!function () { /* code */ }();
~function () { /* code */ }();
-function () { /* code */ }();
+function () { /* code */ }();
```
通常情况下，只对匿名函数使用这种“立即执行的函数表达式”。它的目的有两个：一是不必为函数命名，避免了污染全局变量；二是 IIFE 内部形成了一个单独的作用域，可以封装一些外部无法读取的私有变量。
```
// 写法一
var tmp = newData;
processData(tmp);
storeData(tmp);

// 写法二
(function () {
  var tmp = newData;
  processData(tmp);
  storeData(tmp);
}());
```
上面代码中，写法二比写法一更好，因为完全避免了污染全局变量。

****
## call、apply
###### call
函数名.call(指向对象，参数，参数)
###### apply
函数名.apply(指向对象，[参数，参数])
###### 运用
用在通用方法中，可以让不同的对象都能够使用该方法，且该方法的this指向该对象。而不需要该对象自身定义该方法。
*****
## 闭包
```
例一
var fnArr = [];
for (var i = 0; i < 10; i ++) {
    (function(i){
        fnArr[i] = function(){
            console.log(i)
        }
    })(i);
}
console.log(fnArr[3]());

例二
//打印结果是6个6，而不是预想的1,2,3,4,5.
for(var i=0;i<6;i++) {
    setTimeout(function() {
        console.log(i);
    },1000);
}


解决办法：
//内部声明一个函数，立刻执行，将变量i存为aa函数的局部变量，
// 该变量将会一直保存直到计时器运行完毕。实现闭包
for(var i=0;i<6;i++) {
    function aa(i) {
        setTimeout(function() {
            console.log(i);
        },1000);
    };
    aa(i);
}

//上例立刻执行函数简化写法
for(var i=0;i<6;i++) {
    ~function aa(i) {
        setTimeout(function() {
            console.log(i);
        },1000);
    }(i);
}
```

### 长文字符串的处理
如果长字符串必须分成多行，可以在每一行的尾部使用反斜杠。
```
var longString = 'Long \
long \
long \
string';
longString
// "Long long long string"
```
上面代码表示，加了反斜杠以后，原来写在一行的字符串，可以分成多行书写。但是，输出的时候还是单行，效果与写在同一行完全一样。注意，反斜杠的后面必须是换行符，而不能有其他字符（比如空格），否则会报错。

连接运算符（+）可以连接多个单行字符串，将长字符串拆成多行书写，输出的时候也是单行。
```
var longString = 'Long '
  + 'long '
  + 'long '
  + 'string';
```
如果想输出多行字符串，有一种利用多行注释的变通方法。
```
(function () { /*
line 1
line 2
line 3
*/}).toString().split('\n').slice(1, -1).join('\n')
// "line 1
// line 2
// line 3"
```
上面的例子中，输出的字符串就是多行。
*****

### 字符串API
###### replace
.replace有两个参数，第一个参数是字符串的某个序号的字符，第二个参数是将要替换第一个参数的字符。结果是得到一个新的字符串，实现上述替换。
****
### JSON格式数据
JSON 格式（JavaScript Object Notation 的缩写）是一种用于数据交换的文本格式，2001年由 Douglas Crockford 提出，目的是取代繁琐笨重的 XML 格式。
JSON 对值的类型和格式有严格的规定。

1. 复合类型的值只能是数组或对象，不能是函数、正则表达式对象、日期对象。
2. 简单类型的值只有四种：字符串、数值（必须以十进制表示）、布尔值和null（不能使用NaN, Infinity, -Infinity和undefined）。
3. 字符串必须使用双引号表示，不能使用单引号。
4. 对象的键名必须放在双引号里面。
5. 数组或对象最后一个成员的后面，不能加逗号。

###### JSON表示对象
```
{ "one": 1, "two": 2, "three": 3 }
{"names": ["张三", "李四"] }
```
**注意即使是对象的KEY也需要加双引号**
###### 如何把JSON 格式的字符串转换为 JS 对象
JSON.parse方法用于将JSON字符串转化成对象。
```
JSON.parse('{}') // {}
JSON.parse('true') // true
JSON.parse('"foo"') // "foo"
JSON.parse('[1, 5, "false"]') // [1, 5, "false"]
JSON.parse('null') // null

var o = JSON.parse('{"name": "张三"}');
o.name // 张三
```
如果传入的字符串不是有效的JSON格式，JSON.parse方法将报错。
```
JSON.parse("'String'") // illegal single quotes
// SyntaxError: Unexpected token ILLEGAL
```
上面代码中，双引号字符串中是一个单引号字符串，因为单引号字符串不符合JSON格式，所以报错。
###### 如何把 JS一个值转换为 JSON 格式的字符串?
JSON.stringify方法用于将一个值转为字符串。该字符串符合 JSON 格式，并且可以被JSON.parse方法还原。
```
JSON.stringify('abc') // ""abc""
JSON.stringify(1) // "1"
JSON.stringify(false) // "false"
JSON.stringify([]) // "[]"
JSON.stringify({}) // "{}"

JSON.stringify([1, "false", false])
// '[1,"false",false]'

JSON.stringify({ name: "张三" })
// '{"name":"张三"}'
```
上面代码将各种类型的值，转成 JSON 字符串。

### 查看对象属性
查看一个对象本身的所有属性，可以使用Object.keys方法。
```
var obj = {
  key1: 1,
  key2: 2
};
Object.keys(obj);
// ['key1', 'key2']
```
*****
### delete
delete命令用于删除对象的属性，删除成功后返回true。
```
var obj = { p: 1 };
Object.keys(obj) // ["p"]
delete obj.p // true
obj.p // undefined
Object.keys(obj) // []
```
上面代码中，delete命令删除对象obj的p属性。删除后，再读取p属性就会返回undefined，而且Object.keys方法的返回值也不再包括该属性。
注意，删除一个不存在的属性，delete不报错，而且返回true。
```
var obj = {};
delete obj.p // true
```
上面代码中，对象obj并没有p属性，但是delete命令照样返回true。因此，不能根据delete命令的结果，认定某个属性是存在的。
*****
### freeze
冻结对象，对象不可修改，但可以查看。
```
Object.freeze(o)
```

### 数组
1. push
将括号内的数据增加到数组的最后一位。length相应增加。改变原数组。
```
var arr = [1,2,3,5,6]  //undefined

arr   //(5) [1, 2, 3, 5, 6]

arr.push('adf')  //6

arr  //(6) [1, 2, 3, 5, 6, "adf"]
``` 
2.pop
将数组的最后一位去掉。length相应减少。改变原数组。
```
arr.pop()  //"adf"

arr  //(5) [1, 2, 3, 5, 6]
```
3. shift
将数组的第一位去掉。length相应减少。改变原数组。
```
arr.shift()  //1

arr  //(2) [2, 3]
```
4. unshift
在数组的第一位前面增加。length相应增加。改变原数组。
```
arr.unshift('abc')  //3

arr  //(3) ["abc", 2, 3]
```
5.join
将数组按照括号里面的规则组合成一个字符串。不改变原数组，需要赋值给一个str。
```
arr.join('+')  //"abc+2+3"

arr  //(3) ["abc", 2, 3]
```
6. splice
将数组按照（数组位置，删除数组元素个数，增加数组元素）的规则作出修改。改变原数组，length相应变化。
```
arr.splice(1,2,8)  /(2) [2, 3]

arr  //(2) ["abc", 8]
```
7. reverse
将数组的顺序颠倒过来，直接在旧数组上生成。length不变。
8. sort
将数组按照字符编码顺序进行排序，直接在旧数组上生成。length不变。
sort（）括号内无内容就是默认排序，如果想按照其他标准进行排序，就需要提供比较函数，该函数要比较两个值，然后返回一个用于说明这两个值的相对顺序的数字。比较函数应该具有两个参数 a 和 b，其返回值如下：
- 若 a 小于 b，在排序后的数组中 a 应该出现在 b 之前，则返回一个小于 0 的值。
- 若 a 等于 b，则返回 0。
- 若 a 大于 b，则返回一个大于 0 的值。
9. indexOf
查询数组里面有没有某内容。
如果查询到了会返回该内容所在的序号，如果查不到则返回-1。
10. forEach
遍历数组，将数组每一项,下标作为参数传给函数，函数进行相应计算对数组进行修改。可以代替for循环。
11. map
遍历数组，生成一个新的数组。用法与forEach类似。
12. filter
遍历数组，把符合条件的内容组成一个新数组。
13. reduce
遍历数组，从数组的前两项内容开始，将数组按函数的算法规则依次计算，最后得到一个数（number）。
14.slice
例：arr.slice(3,6)，指的是将数组的第3位到第5位生成一个新数组。
15.concat
连接两个数组，返回一个新的数组
```
var a1 = [1,2,3]
undefined
var a2 = [4,5,6]
undefined
a3 = a1.concat(a2)
(6) [1, 2, 3, 4, 5, 6]
a3
(6) [1, 2, 3, 4, 5, 6]
```
*****
### 数组.length
length属性是可写的。如果人为设置一个小于当前成员个数的值，该数组的成员会自动减少到length设置的值。
···
var arr = [ 'a', 'b', 'c' ];
arr.length // 3
arr.length = 2;
arr // ["a", "b"]
···
上面代码表示，当数组的length属性设为2（即最大的整数键只能是1）那么整数键2（值为c）就已经不在数组中了，被自动删除了。
清空数组的一个有效方法，就是将length属性设为0。

值得注意的是，由于数组本质上是一种对象，所以可以为数组添加属性，但是这不影响length属性的值。
```
var a = [];

a['p'] = 'abc';
a.length // 0

a[2.1] = 'abc';
a.length // 0
```
上面代码将数组的键分别设为字符串和小数，结果都不影响length属性。因为，length属性的值就是等于最大的数字键加1，而这个数组没有整数键，所以length属性保持为0。

使用delete命令删除一个数组成员，会形成空位，并且不会影响length属性。
删除数组且不形成空位使用arr.splice(第几位,删除多少个,添加什么元素)。

###  正则的字符
###### 元字符

|字符	|含义|
|-----|------|
|\t	|水平制表符|
|\r|	回车符|
|\n	|换行符|
|\f	|换页符|
|\v	|垂直制表符|
|\0|	空字符|
###### 字符类
但是我们可以使用元字符[]来构建一个简单的类, 比如[abcd]代表一个字符，这个字符可以是 abcd四个字符中的任意一个
取反
元字符[]组合可以创建一个类，我们还可以使用元字符^创建反向类/负向类，反向类的意思是不属于XXX类的内容，表达式 [^abc] 表示一个不是字符a或b或c的字符
###### 范围类
```
 //匹配一个字符，这个字符可以是大写字母、小写字母、数字中的任意一个
    var reg3 = /[a-zA-Z0-9]/
```
###### 预定义类
|字符	|等价类|含义|
|-----|------|------|
|.	|[^\r\n]|	除了回车符和换行符之外的所有字符|
|\d|	[0-9]|	数字字符|
|\D|	[^0-9]	|非数字字符|
|\s	|[\t\n\x0B\f\r]	|空白符|
|\S|	[^\t\n\x0B\f\r]|	非空白符|
|\w|	[a-zA-Z_0-9]|	单词字符，字母、数字下划线|
|\W|	[^a-zA-Z_0-9]|	非单词字符|
###### 边界
|字符|	含义|
|------|------|
|^	|以xxx开头|
|$|	以xxx结尾|
|\b|	单词边界|
|\B|	非单词边界|
###### 量词
|字符	|含义|
|------|------|
|?	|出现零次或一次（最多出现一次）|
|+	|出现一次或多次（至少出现一次）|
|*	|出现零次或多次（任意次）|
|{n}	|出现n次|
|{n,m}|	出现n到m次|
|{n,}	|至少出现n次|

**在量词后加上?即可执行非贪婪模式**
###### 分组
怎么把hunger作为一个整体呢？使用()就可以达到此目的，我们称为分组
```
    /(hugner){10}/
```
###### 前瞻
|表达式|	含义|
|-------|-------|
|exp1(?=exp2)	|匹配后面是exp2的exp1|
|exp1(?!exp2)	|匹配后面不是exp2的exp1|
*******
### 属性
RegExp实例对象有五个属性

- global：是否全局搜索，默认是false

- ignoreCase：是否大小写敏感，默认是false

- multiline：多行搜索，默认值是false

- lastIndex：是当前表达式模式首次匹配内容中最后一个字符的下一个位置，每次正则表达式成功匹配时， lastIndex属性值都会随之改变

- source：正则表达式的文本字符串

### 获取dom
###### 直接获取
1. 根据id：document.getElementById("id"); →单个对象
2. 根据name: document.getElementsByName("name");→满足条件的数组
3. 根据标签名: document.getElementsByTagName("tagName");→满足条件的数组
4. 根据类名: document.getElementsByClassName("className");→满足条件的数组
5. **表单根据name获取:**
document.name;document.forms[index];document.forms[name];
###### 间接获取
1. 父亲找儿子(属性): parent.childNodes;→满足条件的数组，含元素和文本
2. 父亲找儿子(属性): parent.children;→满足条件的数组，含元素
3. 其他儿子(属性): firstChild;lastChild;
4. 儿子找爹(属性): child.parentNode;
5. 获取兄弟(属性): nextSibling;previousSibling 含元素和文本
*****
### 节点类型
属性：nodeType → 返回1是元素，返回3是文本
**文本节点的nodeValue是他的文本内容，元素节点的nodeValue是null**
*****
###  dom对象的innerText和innerHTML有什么区别？

- Element.innerHTML 属性设置或获取描述元素后代的HTML语句。
- Element.innerText 是一个可写属性，返回元素内包含的文本内容，在多层次的时候会按照元素由浅到深的顺序拼接其内容
- Element.outerText 从当前元素的外部向里面查文档，若改变该值，则改变的是该元素外部的文本内容，该元素内部文本清空
- Element.outerHTML 与innerHTML相比，还要把自己的HTML语句拿到。
****
###  elem.children和elem.childNodes的区别？
[![image](http://upload-images.jianshu.io/upload_images/10942635-b6a6896967833ee8..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
通过对比我们可以发现
- children：子元素列表（HTMLCollection），当前元素的子元素的集合
- childNodes：节点列表（NodeList），当前元素的子元素以及文本(空格也算文本)的集合
*****
### 如何创建一个元素？如何给元素设置属性？如何删除属性
通过createElement()方法创建元素。通过createTextNode()方法创建文本节点。通过元素的setAttribute()(**或者对象.属性 = value**)方法给元素设置属性。通过元素的removeAttribute()(**或者对象.属性 = null**)删除属性。通过getAttribute()(**或者对象.属性**)来获得属性。
****
### 如何给页面元素添加子元素？如何删除页面元素下的子元素?
- parentNode.appendChild(新节点);
- parentNode.insertBefore(新节点，在该节点之前插入);
- parentNode.removeChild(节点);
- parentNode.replaceChild(新节点,旧节点);
*****
###  element.classList有哪些方法？如何判断一个元素的 class 列表中是包含某个 class？如何添加一个class？如何删除一个class?

```
// div是具有class =“foo bar”的<div>元素的对象引用
div.classList.remove("foo");
div.classList.add("anotherclass");

// 如果visible被设置则删除它，否则添加它
div.classList.toggle("visible");

// 添加/删除 visible，取决于测试条件，i小于10
div.classList.toggle("visible", i < 10);

alert(div.classList.contains("foo"));

//添加或删除多个类
div.classList.add("foo","bar");
div.classList.remove("foo", "bar");
```
参考如下资料
 [Document](https://developer.mozilla.org/zh-CN/docs/Web/API/Document)
 [Node](https://developer.mozilla.org/zh-CN/docs/Web/API/Node)
MDN上有比较健全的关于document和node的API
也可以参考阮一峰博客 [http://javascript.ruanyifeng.com/#dom4](http://javascript.ruanyifeng.com/#dom)
[饥人谷课件-dom](http://book.jirengu.com/fe/%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/Javascript/dom.html)


###  DOM0 事件和DOM2级在事件监听使用方式上有什么区别？
- DOM0就是直接通过 onclick写在html里面的事件, 　DOM0的事件具有极好的跨浏览器优势, 会以最快的速度绑定, 如果你通过DOM2绑定要等到JS运行, DOM0不用, 因为DOM0是写在元素上面的。

`<input onclick="alert(hehe.value)">`

- DOM2是通过addEventListener绑定的事件, 还有IE下的DOM2事件通过attachEvent绑定;事件规定了事件流包含三个阶段包括： 1：事件捕获， 2：处于目标阶段， 3：事件冒泡阶段（IE8以及更早版本不支持DOM事件流）;
*****
###  attachEvent与addEventListener的区别？
在添加事件处理程序事addEventListener和attachEvent主要有几个区别

1. 参数个数不相同，这个最直观，addEventListener有三个参数，attachEvent只有两个，attachEvent添加的事件处理程序只能发生在冒泡阶段，addEventListener第三个参数可以决定添加的事件处理程序是在捕获阶段还是冒泡阶段处理（我们一般为了浏览器兼容性都设置为冒泡阶段）

2. 第一个参数意义不同，addEventListener第一个参数是事件类型（比如click，load），而attachEvent第一个参数指明的是事件处理函数名称（onclick，onload）

3.事件处理程序的作用域不相同，addEventListener的作用域是元素本身，this是指的触发元素，而attachEvent事件处理程序会在全局变量内运行，this是window.

4. 为一个事件添加多个事件处理程序时，执行顺序不同，addEventListener添加会按照添加顺序执行，而attachEvent添加多个事件处理程序时顺序无规律(添加的方法少的时候大多是按添加顺序的反顺序执行的，但是添加的多了就无规律了)，所以添加多个的时候，不依赖执行顺序的还好，若是依赖于函数执行顺序，最好自己处理，不要指望浏览器
*****
### 解释IE事件冒泡和DOM2事件传播机制？
1. 事件冒泡：事件开始时由最具体的元素接收，然后逐级向上传播到较为不具体的元素

2. 事件捕获：不太具体的节点更早接收事件，而最具体的元素最后接收事件，和事件冒泡相反
3. DOM事件流：DOM2级事件规定事件流包括三个阶段，事件捕获阶段，处于目标阶段，事件冒泡阶段，首先发生的是事件捕获，为截取事件提供机会，然后是实际目标接收事件，最后是冒泡阶段
******
### 如何阻止事件冒泡？ 如何阻止默认事件？
- event.stopPropagation(): 阻止捕获和冒泡阶段中当前事件的进一步传播。
- event.preventDefault()：如果事件可取消，则取消该事件，而不停止事件的进一步传播。
其中event为事件接收器的参数,即该事件，亦或者可以表达为this.target.
******
### 进阶任务9还有一个博客没写
*****
###  URL 如何编码解码？为什么要编码？
一般来说，URL只能使用英文字母、阿拉伯数字和某些标点符号，不能使用其他文字和符号。比如，世界上有英文字母的网址"http://www.abc.com"，但是没有希腊字母的网址"http://www.aβγ.com"。这是因为网络标准[RFC 1738](http://www.ietf.org/rfc/rfc1738.txt)做了硬性规定。
>"只有字母和数字[0-9a-zA-Z]、一些特殊符号"$-_.+!*'(),"[不包括双引号]、以及某些保留字，才可以不经过编码直接用于URL。"
*****
###  补全如下函数，判断用户的浏览器类型
```
function isAndroid(){
    return /android/i.test(window.navigator.userAgent);
}
function isIphone(){
    return /iphone/i.test(window.navigator.userAgent);
}
function isIpad(){
    return /ipad/i.test(window.navigator.userAgent);
}
function isIOS(){
    return /ipad|iphone/i.test(window.navigator.userAgent);
}
```
*****
### bomAPI
###### window方法
- confirm(message),确认框，返回boolean类型
- setTimeOut（function,time）；延时函数
- setInterval（function,time）；无限循环
- clearInterval（对象名）；清除定时
- clearTimeOut（对象名）；清除定时
###### window属性
- location：
1. location.href = url,前往该地址
- history:
1. go(int),前往第几个页面
2. forward();
3. back();
- navigator：
1. userAgent：浏览器信息


### 说说库和框架的区别？
当你在调用library的时候， 你按照自己的意愿来control他（比如jQuery）。
而，对于framework， 那么control就是倒转过来了，是他在调用你（比如bootstrap,vue)。
参考 [framework vs library10](http://stackoverflow.com/questions/3057526/framework-vs-toolkit-vs-library)
****
### jquery 能做什么？

相较于JavaScript Jquery拥有大量简洁，兼容性能良好的api，所以可以快速的操作dom元素。
*****
### jquery 对象和 DOM 原生对象有什么区别？如何转化？

jQuery对象就是通过jQuery包装DOM对象后产生的对象(集合对象)。jQuery对象是jQuery独有的，可以使用jQuery里的方法。
因此jQuery对象和DOM对象是不一样的，不能调用对方定义的方法。
注意！dom对象须使用dom方法，jq对象需使用jq方法

转化：

普通的DOM对象可以用\$()包装起来转换为jQuery对象：\$(document.getElementById(‘#test’)).html();//正常

jQuery对象jquery对象本身是一个集合，要转换为DOM对象，可通过数组索引取出：
第一种方式：\$(‘#test’)[0]
第二种方式：\$(‘#test’).get(0)
注： eq(0)返回的还是jQuery对象,eq(0)[0]是DOM对象。
****
### jquery中如何绑定事件？bind、unbind、delegate、live、on、off都有什么作用？推荐使用哪种？使用on绑定事件使用事件代理的写法？

从jQuery 1.7开始，事件绑定只推荐on()、off()这两个方法，
具体查 [jquery API10](http://www.css88.com/jqapi-1.9/)
****
### jquery 如何展示/隐藏元素？

\$(ele).show()/\$(ele).hide()
****
### jquery 动画如何使用？

[$(ele).animate()5](http://www.css88.com/jqapi-1.9/animate/)
****
### 如何设置和获取元素内部 HTML 内容？如何设置和获取元素内部文本？

\$(ele).html()/$(ele).text()
*****
### 如何设置和获取表单用户输入或者选择的内容？如何设置和获取元素属性？
\$(ele).val（） 获取表单用户输入或者选择的内容。括号里面如果有值，则将该值设置到表单上。
\$(ele).attr（'arrtibute'）获取元素的arrtibute属性的值，若括号再添加第二个值，则是将attribute属性赋值为第二个值。用\$(ele).removeAttr（'arrtibute'）删除属性。
****
### 注：本文中所有api都可以查询 [jquery API10](http://www.css88.com/jqapi-1.9/)
****
### jQuery 中， $(document).ready()是什么意思？

当DOM准备就绪时，指定一个函数来执行。与JavaScript提供的window.onload事件的区别：

> 简单来说 `window.onload`是当页面呈现时用来执行这个事件，直到所有的东西，如图像已被完全接收前，此事件不会被触发
> `$(document).ready()`只要DOM结构已完全加载时，脚本就可以运行。传递处理函数给.ready()方法，能保证DOM准备好后就执行这个函数，等效于`$(function（） {})`
****
### \$node.html()和$node.text()的区别?

\$node.html()，返回所选择元素内的html内容，包含html标签和文本内容
\$node.text()，返回所选择元素内的文本内容，不包含html标签，只包含文本内容
****
### $.extend 的作用和用法?

语法：`jQuery.extend([deep,] target [, object1 ] [, objectN ] )`
1\. 当我们提供两个或多个对象给\$.extend()，对象的所有属性都添加到目标对象（target参数）。
2\. 如果只有一个参数提供给\$.extend()，这意味着目标参数被省略。在这种情况下，jQuery对象本身被默认为目标对象。这样，我们可以在jQuery的命名空间下添加新的功能。这对于插件开发者希望向 jQuery 中添加新函数时是很有用的

目标对象（第一个参数）将被修改，并且将通过$.extend()返回。然而，如果我们想保留原对象，我们可以通过传递一个空对象作为目标对象：

```
var object = \$.extend({}, object1, object2);
```

在默认情况下，通过$.extend()合并操作不是递归的;

如果第一个对象的属性本身是一个对象或数组，那么它将完全用第二个对象相同的key重写一个属性。这些值不会被合并。如果将 true作为该函数的第一个参数，那么会在对象上进行递归的合并。

```
var object1 = {
  apple: 0,
  banana: { weight: 52, price: 100 },
  cherry: 97
};
var object2 = {
  banana: { price: 200 },
  durian: 100
};

// Merge object2 into object1
$.extend( object1, object2 );
```
*****
### JQuery 的链式调用是什么？

链式调用：使用jQuery方法时，对象方法返回的是对象本身，可以调用对此对象的其他jQuery方法，实现连续调用多个方法

```
例：$(this).siblings().removeClass('active');
```

原理（伪代码）：

```
	window.onload = function(){
		// jQuery构造器
		var jQuery = function(selector, context) {
			return new jQuery.fn.init(selector, context);
		}
		jQuery.fn = jQuery.prototype = {
			constructor: jQuery,
			//给所有元素设置颜色
			setColor: function(color) {
				for (var i = 0; i < this.length; i++) {
					this[i].style.color = color
				}
				return this;
			},
			//返回第一个元素
			first: function() {
				return this[0]
			}
		}
		var init = jQuery.fn.init = function(selector, context) {
			var elem = document.querySelectorAll(selector)
			for (var i = 0; i < elem.length; i++) {
				this[i] = elem[i]
			}
			this.length   = elem.length;
			this.context  = document;
			this.selector = selector;
			return this;
		}
		init.prototype = jQuery.fn;
		var $div = jQuery('div')
		$div.setColor('red').first()
	}
```
*****
### jQuery 中 data 函数的作用

用法：jQuery.data( element, key, value )
jQuery.data() 方法允许我们在DOM元素上附加任意类型的数据,避免了循环引用的内存泄漏风险。如果 DOM 元素是通过 jQuery 方法删除的或者当用户离开页面时，jQuery 同时也会移除添加在上面的数据。
例：

![image](http://upload-images.jianshu.io/upload_images/10942635-e9e39a99a99994fe..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*****
### jQuery 多库共存
```$.fn.jquery```输出当前版本号。
当同时引用多个版本的jquery
```
$.noCoflict();
alert($.fn.jquery);
alert(jQuery.fn.jquery);
```
通过nocofilct函数，使$和jQuery可以作用于不同版本，代表不同版本。



### 面向对象
Object Oriented Programming（简称OOD）,面向对象的程序设计。所谓“对象”在显式支持面向对象的语言中，一般是指类在内存中装载的实例，具有相关的成员变量和成员函数（也称为：方法）。
特点：
- 封装，也叫做信息封装：确保组件不会以不可预期的方式改变其它组件的内部状态；只有在那些提供了内部状态改变方法的组件中，才可以访问其内部状态。每类组件都提供了一个与其它组件联系的接口，并规定了其它组件进行调用的方法。
- 多态性：组件的引用和类集会涉及到其它许多不同类型的组件，而且引用组件所产生的结果依据实际调用的类型。
- 继承性：允许在现存的组件基础上创建子类组件，这统一并增强了多态性和封装性。典型地来说就是用类来对组件进行分组，而且还可以定义新类为现存的类的扩展，这样就可以将类组织成树形或网状结构，这体现了动作的通用性。
*****
### prototype 是什么？有什么特性
- prototype：javascript采用prototype机制代替类机制，每个构造函数都有一个prototype，而prototype也是一个对象，prototype表示该函数的原型，也表示一个类的成员的集合。
- 特点：prototype的属性和方法会被拥有该prototype的对象所共同享有。
- 每个实例通过\_proto_指向到对应类的prototype，每个类也有\_proto_指向父类的prototype。
- Function类最高的类。所有\_proto_最终指向Function类。
*****
### 原型重写
将类的prototype直接=一个对象，则对该类的原型进行了重写。此时该原型的constructor默认是object。通过修改构造器可以改变该类的构造器
****
### apply、call、bind有什么作用，什么区别?

apply、call、bind都作用本质都是改变函数的执行环境上下文，即this指向。

区别

apply和call返回的值是函数的运行结果。
apply的参数是传入数组或类数组，而call的参数是若干个指定参数。
bind返回的是修改过this的函数
```
var getAge=function(){
    console.log(this.age);
    console.log(arguments[0]);
}

window.age="window";

var context={
    age:"19"
}

getAge()//window,

getAge.bind(context)();//19,undefined
```
****
### 继承有什么作用?
抽象事物的从一般到特殊的过程，使得更具体的事物共享那些更一般的事物的属性与方法，减少重复，通常我们将更具体的事物成为子类，比其抽象的事物称为父类。我们可以灵活的给子类添加特定的属性与方法以此来使其拓展父类的功能，而又不影响父类本身。
***
### Object.create 有什么作用？兼容性如何？
接收两个参数，第一个参数是一个原型对象，用来作为将要返回的新对象的原型，第二个参数是一个参数对象，里面的key代表了这个新对象的key，而：后面代表了这个key对应的属性，比如value, enumerable等。

```
function Person(name, sex){
    this.name = name;
    this.sex = sex;               // todo ...
}

Person.prototype.printName = function(){
    return this.name;   // todo ...
};    

function Male(name, sex, age){
    Person.call(this,name,sex);
    this.age = age;                      //todo ...
}
Male.prototype = Object.create(Person.prototype);
Male.prototype.getAge = function(){
    return this.age;    //todo ...
};
Male.prototype.constructor = Male
var ruoyu = new Male('若愚', '男', 27);
ruoyu.printName();
```

***
### 为什么要使用模块化？
- 解决命名冲突
- 可以提高代码的可读性
- 提高代码的复用性
- 避免污染全局变量
- 实现依赖管理
- 可以实现代码的异步加载，提高页面加载性能，避免网页失去响应。
- 有利于团队分工
*****
### CMD、AMD、CommonJS 规范分别指什么？有哪些应用
###### CommonJS
CommonJS是node采用的模块化规范，采用同步加载模块的方式，这个在服务端是完全没有问题的。
一个单独的文件就是一个模块，每个模块都是一个单独的作用域，在模块内部定义的变量，无法被其他模块读取，除非定义为global对象的属性
- 模块输出：模块只有一个出口，module.exports对象，我们需要把模块输出的内容放入该对象
- 加载模块：加载模块使用require方法，该方法读取一个文件并执行，返回文件内部的module.exports对象，如果请求的模块不能返回，那么"require"必须抛出一个错误。
- demo：
```
    //模块定义model.js
    
    var name = "Array"
    
    function printName(){
        console.log(name);
    }
    function printFullName(firstName){
        console.log(firstName + name)
    }
    module.exports = {
        printName: printName;
        printFullName: printFullName
    }
    
    //加载模块
    var nameModule = require("./model.js")
    nameModule.printName() // "Array"
    nameModule.printFullName("Bob") // "Bob Array"
    
    应用 ：因为require是同步的，所以主要是在服务器端使用
    浏览器断加载JavaScript是异步的，所以传统的CommonJs在浏览器环境中无法正常加载
```
###### AMD
AMD （Async Module Definition）异步模块定义,回一个浏览器断模块化开发的规范，由于有CMD在前，所以网页端也需要有专门管理模块的工具，但是网页端的模块只能是异步加载，如果是同步加载的话，那时间就呵呵了,典型的应用就是require.js。

主要解决：
js文件依赖问题
js加载时浏览器会停止页面渲染，加载文件越多，页面失去的响应时间越长
demo：
```
//定义module.js
define(['dependencies'],function(){
    var name = 'Array';
    function printName(){console.log(name)}
    return {printName: pirintName}
})

//加载模块
require(['module'],function(module){
    module.printName()
})

/*
* define:
* define(id?dependencies?,factory);
* id:可选参数，用来定义模块的标识，如果没有提供该参数，脚本文件名为模块标识符
* dependencies：是一个当前模块依赖的模块名称数组
* factory：工厂方法，模块初始化要执行的函数或对象，如果为函数，它应该只执行一次，如果是对象，此对象应该为模块的输出值
*
* require:
* require(['dependencies'],function(depName){})
* dependencies:标识所依赖的模块
* function(){}：当模块加载成功后执行的回调函数，加载的模块会以参数形式传入该函数
* require()函数在加载依赖函数时是异步加载，所以浏览器不会失去响应，它指定的回调函数，只有前面的模块都加载成功了，才会运行，解决了依赖性
*
*
*/
```
###### CMD
很多时候，我们会发现，我们在首屏的时候，其实没有必要一次性加载那么多的js文件，更多的我们希望js文件在需要的时候才会去记载，于是就有了CMD的出现，主要应用在sea.js。
```
/*
* define
* define(id?,deps?,factory)
* CMD推崇依赖就近，所以一般不再define的参数中写依赖，在factory中写
* factory有三个参数
*
* function(require,exports,module)
* require是一个方法，接受模块标识作为唯一参数，用来获取其他模块提供的接口
* exports是一个对象，用来想外提供模块接口
* module是个对象，上面存储了与当前模块相关的属性和方法
*/

//在a.js中定义模块
define(function(require,exports,module){
    var $ = require("jquery.js")
    // dosomething with jquery
    exports.foo = something
})
//在c.js中使用模块
defind(function(require,exports,module){
    var a = require('./a.js')
    a.foo//do something with a.foo函数
    exports.bar = thisthing//本模块输出的接口
})
```

### let, const
这两个的用途与var类似，都是用来声明变量的，但在实际运用中他俩都有各自的特殊用途。
- let则实际上为JavaScript新增了块级作用域。用它所声明的变量，只在let命令所在的代码块内有效。
例如常见的闭包代码：
```
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10
```
采用let后：
```
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```
就不再需要像原来一样利用带参数的立刻执行函数来实现闭包。

- const也用来声明变量，但是声明的是常量。一旦声明，常量的值就不能改变。
const有一个很好的应用场景，就是当我们引用第三方库的时声明的变量，用const来声明可以避免未来不小心重命名而导致出现bug
****
### class, extends, super
ES6提供了更接近传统语言的写法，引入了Class（类）这个概念。新的class写法让对象原型的写法更加清晰、更像面向对象编程的语法，也更加通俗易懂。
```
class Animal {
    constructor(){
        this.type = 'animal'
    }
    says(say){
        console.log(this.type + ' says ' + say)
    }
}

let animal = new Animal()
animal.says('hello') //animal says hello

class Cat extends Animal {
    constructor(){
        super()
        this.type = 'cat'
    }
}

let cat = new Cat()
cat.says('hello') //cat says hello
```
上面代码首先用class定义了一个“类”，可以看到里面有一个constructor方法，constructor内定义的方法和属性是实例对象自己的，而constructor外定义的方法和属性则是所有实例对象可以共享的。
Class之间可以通过extends关键字实现继承，super关键字，它指代父类的实例（即父类的this对象）。子类必须在constructor方法中调用super方法，否则新建实例时会报错。
****
### arrow function
用来简写函数
```
function(i){ return i + 1; } //ES5
(i) => i + 1 //ES6

function(x, y) { 
    x++;
    y--;
    return x + y;
}
(x, y) => {x++; y--; return x+y}
```
当我们使用箭头函数时，函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。
***
### template string
用反引号（\）来标识起始，用${}`来引用变量，而且所有的空格和缩进都会被保留在输出之中，是不是非常爽？！
****
### destructuring
ES6允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。
```
let cat = 'ken'
let dog = 'lili'
let zoo = {cat, dog}
console.log(zoo)  //Object {cat: "ken", dog: "lili"}
let dog = {type: 'animal', many: 2}
let { type, many} = dog
console.log(type, many)   //animal 2
```
****
### default, rest
- 一个需要参数的函数往往可能忘了传参而报错，我们就可以用这个方式设置一个默认值
```
function animal(type = 'cat'){
    console.log(type)
}
animal()
```
- rest语法也很简单，直接看例子：
```
function animals(...types){
    console.log(types)
}
animals('cat', 'dog', 'fish') //["cat", "dog", "fish"]
```
****
### import export
- 以前的js没有模块化，一方面js代码变得很臃肿，难以维护，另一方面我们常常得很注意每个script标签在html中的位置，因为它们通常有依赖关系，顺序错了可能就会出bug。
ES6的模块化写法如下：
```
//index.js
import animal from './content'

//content.js
export default 'A cat'
```

- export命令除了输出变量，还可以输出函数，甚至是类（react的模块基本都是输出类）,import引入的时候可以用花括号{}写入export的变量名，得到相应的变量。而如果想引入export default 的变量，只需要不加花括号，随意命名即可。

- 花括号{}里面的变量名如果不想在引入的时候继续使用，可以加入as换名字。
```
import animal, { say, type as animalType } from './content'  
let says = say()
console.log(`The ${animalType} says ${says} to ${animal}`)  
//The dog says Hello to A cat
```
- 除了指定加载某个输出值，还可以使用整体加载，即用星号（\*）指定一个对象，所有输出值都加载在这个对象上面。
通常星号*结合as一起使用比较合适。

