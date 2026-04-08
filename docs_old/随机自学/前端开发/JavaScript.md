# JavaScript

## 基本介绍

JS 与 HTML + CSS 一同作为网页界面框架的一部分，是一门真正的编程语言。



### 书写位置

共有 3 种书写位置：

* 行内 JS：直接写在标签内

```html
<!DOCTYPE html>

<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <button onclick="alert('Hello World!')">点击打招呼</button>
</body>

</html>
```

* 内部 JS：直接写在 HTML 文件中，一般放在 `body` 标签中

```html
<!DOCTYPE html>

<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <!-- 在这里书写 JS 代码 -->
    <script></script>
</body>

</html>
```

放在底部是因为浏览器按照代码顺序加载 HTML，如果 JS 代码会修改 HTML，就可能导致 HTML 没有完全加载之前就调用 JS 代码，导致加载失败。

* 外部 JS：将 JS 代码放在单独的文件中

```js
alert("Hello World!");
```

然后链接对应的文件

```html
<!DOCTYPE html>

<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <!-- 在这里书写 JS 代码 -->
    <script src="my.js">
    	// 中间不能写代码
    </script>
</body>

</html>
```

在链接外部 JS 代码的标签中不能写代码，其内容将被忽略。



### 书写格式

JS 的注释格式与 C/C++ 基本一致

```js
// 输出 hello world
alert('hello world');
```

语句末可以省略 `;`，因为浏览器可以自动推断语句结束位置（不过根据微信小程序开发的经验，还是加上）。



### 输入输出

#### 文档输出

JS 允许向当前页面 `body` 中写入内容，例如

```js
document.write("Hello World!");
// 写入一个标签
document.write("<h1>Welcome to my website.</h1>");
```



#### 警告框

使用 `alert` 页面弹出警告对话框

```js
alert('Hello World');
```



#### 控制台输出

使用 `console.log` 进行控制台输出

```js
console.log('Hello World');
```



#### 弹窗输入

使用 `prompt` 弹出输入框

```js
let name = prompt('Enter your name:');
```



### 单步调试

#### 执行顺序

按照代码顺序执行，除了 `alert` 和 `prompt` 会跳过页面渲染预先执行。



#### 浏览器调试

在开发者模式下查看 JS 代码，可以添加断点单步调试。找到代码位置，添加断点后刷新即可。

![](image-20240711231801830.png)



## 变量

### 声明变量
#### let

使用 `let` 声明变量

```js
let age = 18;
```



#### const

使用 `const` 声明常量

```js
const name = 'Li';
```

常量必须在声明时初始化，之后不能改变。



#### var

旧的代码中常常使用 `var` 声明变量

```js
// var 允许先使用后声明
num = 10;
console.log(num);
var num;

// var 允许重复声明
var num = 10;
var num = 20;

// var 会造成变量提升、全局变量和没有块级作用域等问题
```

但由于 `var` 有很多问题，例如

```js
console.log(num);
var num = 10;
```

程序首先解析变量声明，将所有 `var` 声明的变量提升到当前作用域的开头，得到下面的结果

```js
var num;
console.log(num);
num = 10;
```

由于这种特性造成的混淆，`var` 逐渐被 `let` 取代。



### 作用域

局部作用域分为函数作用域和块作用域，对应函数定义的范围和 `{}` 包括的范围；全局作用域是 `<script>` 标签和 `.js` 最外层范围，在此声明的变量在任何局部作用域中都可以访问。函数执行时，首先查找当前作用域中的变量，然后逐级向父级查找直到全局作用域。



### 数组

#### 基本操作

直接用 `[]` 声明数组

```js
let arr = [0, 1, 2, 3];
```

索引方式与 C/C++ 一致。可以直接获得数组长度

```js
console.log(arr.length);
```



#### foreach

使用 `foreach` 对数组元素遍历操作

```js
const arr = [10, 20, 30];

arr.forEach(function (element, index, array) {
    console.log('Index: ' + index + ', Element: ' + element);
    console.log('Array: ' + array);
});
```

可以使用箭头函数简写

```js
arr.forEach(element => {
    console.log(element);
});
```



#### map

使用 `map` 映射数组并返回新的数组，与 `foreach` 的区别就在于它有返回值

```js
const arr = [10, 20, 30];
const narr = arr.map(function(element, index){
    console.log(element, index);
    return element * 2;
});

const narr = arr.map(element => {
    return element * 2;
});
```



#### join

使用 `join` 将数组中的所有元素拼接为一个字符串

```js
const arr = [10, 20, 30];
console.log(arr.join('-'));
```

使用 `-` 连接转换为字符串的元素。



### 基本数据类型

JS 中的基本数据类型包括

* number 数字：可以是整数或小数
* string 字符串：使用双引号 `""` 或单引号 ` '' ` 或反引号声明
* boolean 布尔
* undefined 未定义
* null 空



#### 字符串

可用 `+` 连接字符串和数字

```js
let age = 10;
let str = 'My age is ' + age + '.';
```

使用反引号包围的字符串是模板字符串，可以直接嵌入变量值

```js
let str = `My age is ${age}.`;
```



利用模板字符串配合弹窗输入来动态修改网页内容

```html
<!DOCTYPE html>

<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <script>
        let age = prompt("What is your age?");
        document.write(`My age is ${age}.`);
    </script>
</body>

</html>
```



#### 未定义

只声明但未赋值的变量就是未定义

```js
let num;
```

一般用于检测一个变量是否被赋值。未定义值参与数字运算得到 `NaN`

```js
let num;
num = num + 1;
// num 为 NaN 
```



#### 空值

空值表示被赋值但是值为空

```js
let obj = null;
```

它通常表示未创建的对象，用于预先创建。空值与数字运算可以得到数字

```js
obj = obj + 1;
// obj 为 1
```



### 数据类型

使用 `typeof` 获得数据类型

```js
let x = 10;
typeof x;
typeof(x);
```

两种写法等价。数据类型判断还可使用

* `IsNaN(n)` 判断是否为 `NaN` 类型
* `Instanceof` 检测是否为对象类型的实例



#### 简单数据类型

简单数据类型也就是基本数据类型，变量直接保存对应的值。



#### 复杂数据类型

复杂数据类型包括对象、数组等，是通过 `new` 关键字创建的对象，变量只保存对应地址的引用，因此又叫做引用数据类型。



### 类型转换

#### 隐式转换

对于特殊的运算符，系统自动进行转换

```js
let a = '10';
let b = 10;
console.log(a + b);		// 得到 '1010'
console.log(a - b);		// 得到 0
```

对于加号，只要两边有一个字符串，就会全部转换为字符串；对于其余四则运算，只要两边有一个数字，就会转换为数字。用 `+` 可以直接将字符串转换为数字

```js
let a = '10';
let b = +'10';	// b 是数字
```



#### 显式转换

使用 `Number` 显式转换为数字

```js
let a = Number('1231');
Number(null);		// 0
Number(undefined);	// NaN
```

如果转换失败会得到 `NaN`，它是非数字的 number 类型。



使用 `parseInt,parseFloat` 进行更细致的转换

```js
parseInt('1.281px');		// 只保留整数 1
parseFloat('1.281px');		// 保留小数 1.281
```

要求输入字符串必须以数字开头。



使用 `Boolean` 转换为布尔值

```js
Boolean(1);				// true
Boolean(0);				// false
Boolean(undefined);		// false
```



### 运算符

#### 比较运算符

主要的运算符与其它语言一致，不同的是全等和不全等运算符。

| 运算符  | 作用   | 运算符   | 作用      |
| :--- | :--- | :---- | :------ |
| `>`  | 大于   | `<`   | 小于      |
| `>=` | 大于等于 | `<=`  | 小于等于    |
| `==` | 值相等  | `===` | 类型和值都相等 |
| `!=` | 值不等  | `!==` | 不全等     |

比较符基本规则

* 比较运算符有隐式转换：当数字字符与数字比较时，会将字符转换为数字
* `NaN` 不等于任何值，包括它自身
* 字符串按照字典序比较每一位的 ASCII 码

尽量不要比较小数，避免浮点精度问题。



#### 优先级

| 优先级 | 运算符   | 顺序              |
| :-- | :---- | :-------------- |
| 1   | 括号    | `()`            |
| 2   | 一元运算符 | `++ - !`        |
| 3   | 算术运算符 | `*,/ +,-`       |
| 4   | 关系运算符 | `> >= < <=`     |
| 5   | 相等运算符 | `== != === !==` |
| 6   | 逻辑运算符 | && \|\|         |
| 7   | 赋值运算符 | `=`             |
| 8   | 逗号运算符 | `,`             |



#### 逻辑中断

逻辑运算返回的不一定是布尔值，而是其中表达式的值

```js
let x = 11 && 0;	// x == 11
let y = 11 || 0;	// y == 11
let z = 0 & 11;		// z == 0
let w = 0 || 11;	// w == 11
let v = 10 || 11;	// v == 11
```



### 循环语法

#### for

普通的 for 循环

```js
for (let index = 0; index < array.length; index++) {
    const element = array[index];
}
```



#### for in

遍历对象的循环

```js
// 遍历键
for (const key in object) {

}
```



#### for of

遍历可迭代对象的循环

```js
for (let char of str) {

}


for (let value of set) {

}

for (let [key, value] of map) {

}
```



## 函数

### 声明函数

函数声明格式为

```js
function funcname(arg1, arg2, ...)
{
	// code
}
```

参数表不是必须的，函数可以定义在调用之后。



### 匿名函数

可以将函数赋给一个变量

```js
let fn = function (arg1, arg2, ...) {
	// code
};
```

可以在声明函数时立即执行

```js
// 第一种写法
(function (x, y) {
	console.log(x + y);
})(1, 2);

// 第二种写法
(function (x, y) {
	console.log(x + y);
}(1, 2));
```

通过立即执行函数可以封装具名函数，防止命名污染

```js
(function fn(x, y) {
	console.log(x + y);
}(x, y));
```



### 回调函数

函数作为参数传递时就是回调函数，例如定时器

```js
function fn() {
	console.log('一秒调用一次');
};

// fn 就是回调函数
let timerId = setInterval(fn, 1000);
```



可以在回调函数内部设置回调函数

```js
let x = 0;
function fn() {
	if (x === 0)  {
		x = 1;
		// 这里回调自身
		setInterval(fn, 1000);
	}
};

fn();
```



## 对象

### 基本用法

#### 声明对象

声明语法为

```js
let obj = {
	name: "Li",
	age: 10,
	'tel': 180,
	hello: function() {
		console.log('hello');
	}
};
```

可以同时声明属性和方法。



#### 使用对象

可以用 `.` 和 `[]` 访问属性，并且可以添加/删除声明时没有的属性

```js
let tel = obj['tel'];	// 访问属性
obj.name = "Liu";		// 修改属性
obj.home = "USA";		// 新增属性
delete obj.age;			// 删除属性
```

属性名为字符串时，就需要使用 `[]` 访问属性值。



#### 遍历对象

以数组对象为例，可以用 `in` 关键字遍历

```js
let arr = ['pink', 'red', 'blue'];
for (let i in arr) {
	console.log(i);			// 这里 i 是索引的字符串
	console.log(arr[i]);
}
```

由于这种方式遍历数组得到的是索引的字符串，并不方便。



一般对象可以使用这种方式遍历

```js
for (let k in obj) {
	console.log(obj[k]);
}
```



#### 常量对象

常量对象的属性可以修改和添加，但是不能给对象本身重新赋值

```js
const car = { name: "A", color: "B" };
Car.Color = "C";
Car.Type = "D";
Car = {};	// 报错
```



### 内置对象

#### 数学对象

可以创建数学对象或者直接通过 `Math` 调用方法。

| 方法            | 作用    | 方法           | 作用   |
| :------------ | :---- | ------------ | ---- |
| `Math.Min`    | 返回最小值 | `Math.Floor` | 向下取整 |
| `Math.Max`    | 返回最小值 | `Math.Round` | 四舍五入 |
| `Math.Ceil`   | 向上取整  | `Math.Abs`   | 绝对值  |
| `Math.Random` | 随机数   |              |      |



#### 日期对象

创建日期对象

```js
const date = new Date()
console.log(date)
```

然后可以调用相应的方法。

| 方法              | 作用          | 方法              | 作用                 |
| :-------------- | :---------- | :-------------- | :----------------- |
| `getFullYear()` | 返回四位数年份     | `setFullYear()` | 设置四位数年份            |
| `getMonth()`    | 返回月份 `0~11` | `setMonth()`    | 设置月份 `0~11`，0 表示一月 |
| `getDate()`     | 返回月份中的天数    | `setDate()`     | 设置月份中的天数           |
| `getDay()`      | 返回星期 `0~6`  | `setDay()`      | 设置星期 `0~6`，0 表示星期日 |
| `getHours()`    | 返回小时        | `setHours()`    | 设置小时               |
| `getMinutes()`  | 返回分钟        | `setMinutes()`  | 设置分钟               |
| `getSeconds()`  | 返回秒         | `setSeconds()`  | 设置秒                |
| `getTime()`     | 返回表示日期的毫秒数  | `setTime()`     | 设置日期的毫秒数           |



## DOM

JS 通过 Web APIs 控制 HTML 和浏览器，主要分为 DOM (文档对象模型) 和 BOM (浏览器对象模型) 。DOM 是用来呈现以及与任意 HTML 或 XML 文档交互的 API，即用于操作网页内容。



### DOM 对象

将 HTML 文档以树状结构表现，就得到文档树或 DOM 树。浏览器根据 HTML 标签会生成 JS 对象，所有的标签属性都保存在对象中，因此 DOM 将网页内容当做对象来处理。通过 `document` 对象操作网页内容

```js
const div = document.querySelector('div');
```

上述代码将获得 DOM 中的第一个 div 标签。



#### 获得 DOM 节点

###### CSS 选取

可以通过标签名、类名、ID 名获取

```js
const div = document.querySelector('div');
const box = document.querySelector('.box');
const nav = document.querySelector('#nav');
```

实际上就是通过 CSS 语法选取

```js
const fstChild = document.querySelector('ul li:first-child');
```

默认情况下返回匹配的第一个 HTML 元素对象。



要获取多个对象，使用

```js
const lis = document.querySelectorAll('div');
```

获取所有 div 标签对象的集合。



还可以指定通过类、标签或 ID 来查找

| 方法                                      | 描述            |
| --------------------------------------- | ------------- |
| `document.GetElementById(id)`           | 通过元素 id 来查找元素 |
| `document.GetElementsByTagName(name)`   | 通过标签名来查找元素    |
| `document.GetElementsByClassName(name)` | 通过类名来查找元素     |



###### 父节点

查找父节点

```js
const body = document.querySelector('.body');
body.parentNode
```

它返回最近一级的父节点。



###### 子节点

查找子节点

```js
body.childNodes		// 获得所有子节点，包括文本节点、注释节点
body.children		// 获得所有元素节点（标签），返回字典
```

其中 `children` 属性最为常用。



###### 兄弟节点

查找兄弟节点

```js
body.nextElementSibling			// 下一个兄弟节点
body.previousElementSibling		// 上一个兄弟节点
```



#### 节点操作

###### 追加节点

通过 `document` 创建元素，然后追加到指定元素后

```js
// 创建并添加元素
const ul = document.createElement('ul');
document.body.appendChild(ul);

// 创建并添加子元素
const li1 = document.createElement('li');
const li2 = document.createElement('li');
li1.innerHTML = '1';
li2.innerHTML = '2';
ul.appendChild(li1);          // 追加节点
ul.insertBefore(li2, li1);    // 插入到第二个位置
```



###### 克隆节点

有时需要将节点克隆然后插入到其它位置

```js
ul.cloneNode(false);    // 克隆当前节点
ul.cloneNode(true);     // 克隆当前节点及其所有子节点
```



###### 删除节点

在原生 DOM 操作中，必须通过父元素删除

```js
ul.removeChild(li1);
ul.removeChild(li2);
```



#### 访问属性

获得元素后，标签的各种属性就是元素对象的成员，可以直接修改。

| 成员          | 含义        |
| :---------- | :-------- |
| `innerText` | 文本（不解析标签） |
| `innerHTML` | 文本（解析标签）  |
| `style`     | 样式属性      |
| `src`       | 链接源       |
| `...`       |           |

可以修改文字内容，其中嵌入标签

```js
const div = document.querySelector('div');
div.style.color = 'red';
div.innerText = '<strong>Hello</strong>';	// 显示 <strong>Hello</strong>
div.innerHTML = '<strong>Hello</strong>';	// 显示加粗的 Hello
```

> 使用 `style` 属性添加的是行内样式，具有极高的权重。



###### 元素尺寸

对于标签，可以获得其宽度

```js
const div = document.querySelector('div');
console.log(div.clientWidth);
console.log(div.offsetWidth);
```

其中 `clientWidth` 是元素本身的宽度，`offsetWidth` 包含内边距和边框宽度。



###### 元素位置

可以获得**相对于具有定位的父元素**的偏移值

```js
div.offsetLeft;
div.offsetTop;
```

如果父元素没有定位，则向上追溯。如果都没有，则以文档左上角为基准。



可以获得**相对于视口**的矩形

```js
div.getBoundingClientRect();
```

视口是显示区域，当页面滚动时，视口位置相对于文档发生滚动，通过此方法获得的矩形也会随之变化。



#### 修改类名

如果想要修改一类标签样式，可以预先定义一个类的样式，然后用 JS 将其添加到指定标签

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <!--[if lt IE 9]-->
    <script src="" https://oss.maxcdn.com/html5shiv/3.7.3/html5shiv.min.js"></script>
    <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <!--[endif]-->
    <!-- bootstrap 核心样式 -->
    <link rel="stylesheet" href="bootstrap/css/bootstrap.min.css">
    <title>Document</title>
    <style>
        .box {
            background-color: #f2f2f2;
            padding: 20px;
            margin-top: 20px;
        }
    </style>
</head>

<body>
    <div class="container"></div>
    <script>
    	// 添加类名 box
        const div = document.querySelector('div');
        div.className = 'box container';
    </script>
</body>

</html>
```

由于 `className` 会整个替换，如果需要保留之前的类名，需要在赋值时添加。如果想要追加和删除，使用

```js
div.classList.add('box');		// 追加
div.classList.remove('box');	// 删除
div.classList.toggle('box');	// 切换：如果当前标签是 box 类，则追加 box 类；否则移除 box 类
```



#### 自定义属性

在 HTML5 中新增了自定义属性，以 `data-` 开头，可通过 `dataset` 成员获得自定义属性。

> 自定义属性的典型应用就是标记标签的序号。

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <div data-id="1" data-spm="nothing">1</div>
    <div data-id="2">2</div>
    <div data-id="3">3</div>
    <div data-id="4">4</div>
    <div data-id="5">5</div>
    <script>
        const div = document.querySelector('div');
        console.log(div.dataset.id); 	// 1
        console.log(div.dataset.spm); 	// nothing
    </script>
</body>

</html>
```



### 事件监听

#### 添加事件

可以为元素添加事件

```js
// 添加鼠标点击事件
div.addEventListener('click', function () {
	alert('click');
});
```

也可以使用 `onclick` 属性来添加事件

```html
<div onclick="alert('Hello')">1</div>
```

等价于

```js
div.onclick = function () {
	alert('Hello');
};
```

这种方式已经不推荐使用，因为将会覆盖原本的事件函数；而 `addEventListener` 可添加多个事件。

| **事件**      | **描述**         |
| ----------- | -------------- |
| onchange    | HTML 元素已被改变    |
| onclick     | 点击了 HTML 元素    |
| onmouseover | 鼠标移动到 HTML 元素上 |
| onmouseout  | 鼠标移开 HTML 元素   |
| onmouseup   | 鼠标松开 HTML 元素   |
| onload      | 浏览器已经完成页面加载    |
| onunload    | 退出页面           |
| onkeydown   | 用户按下键盘按键       |



#### 事件类型

| 鼠标事件         | 描述   | 焦点事件    | 描述   | 键盘事件      | 描述   | 文本事件    | 描述  |
| :----------- | :--- | :------ | :--- | :-------- | :--- | :------ | :-- |
| `click`      | 鼠标点击 | `focus` | 获得焦点 | `keydown` | 键盘按下 | `input` | 输入  |
| `mouseenter` | 鼠标经过 | `blur`  | 失去焦点 | `keyup`   | 键盘抬起 |         |     |
| `mouseleave` | 鼠标离开 |         |      |           |      |         |     |



#### 事件对象

事件对象中包含了事件类型、触发位置、键值等事件信息，作为事件函数的第一个参数传入

```js
const btn = document.querySelector('button');
btn.addEventListener('click', function (e) {
	console.log(e);
 	console.log(e.type);
	console.log(e.offsetX);
});
```



#### 环境对象

环境对象是函数内部的特殊变量 `this`，代表当前函数的运行环境。例如

```js
const btn = document.querySelector('button');
btn.addEventListener('click', function (e) {
	this.style.color = 'red';
});
```



### 事件流

事件流是指事件完整执行过程中的流动路径。当触发事件时，会经历捕获阶段和冒泡阶段，分别从父到子、从子到父。

> 实际工作以考虑事件冒泡为主。

![](image-20240713150549672.png|600)



#### 事件捕获

当一个父元素触发事件时，会首先触发父元素的事件函数，然后依次触发子元素的事件函数。例如

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>

<body>
    <div class="grandfather">
        <div class="father">
            <div class="son">
                hello world
            </div>
        </div>
    </div>

    <script>
        const grandfather = document.querySelector('.grandfather');
        const father = document.querySelector('.father');
        const son = document.querySelector('.son');
        
        // 指定第三个参数为 true，表示捕获阶段触发事件
        grandfather.addEventListener('click', function() {
            console.log('Grandfather clicked');
        }, true);

        father.addEventListener('click', function() {
            console.log('Father clicked');
        }, true);

        son.addEventListener('click', function() {
            console.log('Son clicked');
        }, true);
    </script>
</body>

</html>
```

当点击子元素时，由于开启捕获，因此会依次触发 `grandfather, father, son` 的事件函数。



#### 事件冒泡

默认情况下在冒泡阶段触发事件

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>

<body>
    <div class="grandfather">
        <div class="father">
            <div class="son">
                hello world
            </div>
        </div>
    </div>

    <script>
        const grandfather = document.querySelector('.grandfather');
        const father = document.querySelector('.father');
        const son = document.querySelector('.son');
        
        grandfather.addEventListener('click', function() {
            console.log('Grandfather clicked');
        });

        father.addEventListener('click', function() {
            console.log('Father clicked');
        });

        son.addEventListener('click', function() {
            console.log('Son clicked');
        });
    </script>
</body>

</html>
```



###### 阻止冒泡

如果想阻止事件向上传递，就要阻止事件冒泡。使用

```js
son.addEventListener('click', function(e) {
    console.log('Son clicked');
	e.stopPropagation(); // 阻止事件冒泡
});
```

这里 `stopPropagation` 阻止事件流动，因此也会阻止事件捕获。



###### 鼠标移动

事件 `mouseover, mouseout` 是鼠标经过和离开事件，在事件冒泡的情况下，这两个事件会产生不必要的重复调用。例如

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        .father {
            height: 200px;
            width: 200px;
            background-color: red;
        }

        .son {
            height: 100px;
            width: 100px;
            background-color: blue;
        }
    </style>
</head>

<body>
    <div class="grandfather">
        <div class="father">
            <div class="son">
            </div>
        </div>
    </div>

    <script>
        const father = document.querySelector('.father');
        
        father.addEventListener('mouseover', function() {
            console.log('Father hovered');
        });

        father.addEventListener('mouseout', function() {
            console.log('Father unhovered');
        });
    </script>
</body>

</html>
```

当鼠标从蓝色移动到红色区域时，由于离开子元素，会触发子元素的 `mouseout` 消息，通过冒泡传递给父元素；同时鼠标进入父元素，触发 `mouseover` 消息。最终导致父元素同时触发 `mouseover, mouseout` 消息（从红色移动到蓝色区域时同理），这可能导致交互上的问题。

![](image-20240713153344786.png)

为了避免这个消息，最好使用 `mouseenter, mouseleave` 消息，它们能够自动阻止冒泡。



#### 事件解绑

使用 `onclick` 属性直接移除所有事件

```js
son.onclick = null;
```

也可以使用

```js
function fn() {
	console.log('Son clicked');	
};

son.addEventListener('click', fn);
son.removeEventListener('click', fn);
```

这种解绑方式需要保留函数名，因此匿名函数不能解绑。



#### 事件委托

将事件委托给父元素执行，通过事件对象操作子元素。例如当子元素被点击时，事件冒泡到父元素，然后通过父元素处理子元素

```js
father.addEventListener('click', function(e) {
	// 指定子元素类型
	if (e.target.tagName === 'DIV') {
		e.target.style.color = 'red';
	}
});
```

这种做法用于批量处理子元素，例如

```html
<div class="father">
    <div class="son">1</div>
    <div class="son">2</div>
    <div class="son">3</div>
    <div class="son">4</div>
    <p class="son">5</p>
    <p class="son">6</p>
</div>
```



#### 阻止默认行为

表单和链接标签具有其默认行为，有时希望能够满足一定要求时才执行，这就需要阻止默认行为。例如

```js
const form = document.querySelector('form');
form.addEventListener('submit', function(e) {
	// 阻止默认提交
    e.preventDefault();
    console.log('Form submitted');
});
```

当表单触发提交事件时，阻止其默认提交。



#### 页面事件

###### 加载事件

老版本的代码通常将 `script` 块放在 `body` 前面，因此如果 JS 代码对下面的标签进行操作，由于页面尚未加载完成，就有可能报错。有时我们也希望在页面加载完成之前进行一些操作，这就需要处理页面加载事件。借助窗口加载事件

```js
window.addEventListener('load', function () {
	// 页面加载完成后调用
	const btn = document.querySelector('button');
	btn.addEventListener('click', function () {
		alert('click');
	});
});
```



窗口的加载事件 `load` 会在整个页面都加载完成后触发，但是我们可能希望更快执行代码。借助事件 `DOMContentLoaded`，能够在初始的 HTML 文档被完全加载和解析后立即触发，不需要等待样式表和图像完成加载。此消息添加在 `document` 上

```js
document.addEventListener('DOMContentLoaded', function () {
	// 页面加载完成后调用
	const btn = document.querySelector('button');
	btn.addEventListener('click', function () {
		alert('click');
	});
});
```



###### 滚动事件

页面滚动时触发此事件，添加在窗口上

```js
window.addEventListener('scroll', function () {
	// 页面滚动时触发
});
```

具有滚动条的标签均可以触发此事件

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        body {
            height: 2000px;
        }

        .box {
            overflow: scroll;
            width: 200px;
            height: 200px;
        }
    </style>
</head>

<body>
    <div class="box">
        这几年，随着拍摄设备、渲染方法和显示设备的发展，HDR慢慢会成为标配。照相机和摄像机可以捕捉到HDR的影响，渲染过程中可以产生HDR的画面。这些内容如果需要显示到LDR的设备上，就需要一个称为tone mapping的过程，把HDR变成LDR。现在高端的显示器和电视也可以直接显示出HDR的内容。然而和LDR不同之处在于，LDR就是一个确定的范围，HDR是一个非常宽广的概念。即便两个都是HDR的，但它们的范围仍可能不同。因此有人把这个称为Variable Dynamic Range（VDR），可变动态范围，因为此H不一定是彼H。所以，即便在一个HDR世界，也仍然需要tone mapping来改变动态范围。
    </div>

    <script>
        const box = document.querySelector('.box');
        box.addEventListener('scroll', function() {
            // 获得盒子滚动高度
            const n = box.scrollTop;
            console.log(n);
        });

        window.addEventListener('scroll', function() {
            // 获得页面滚动高度
            const n = document.documentElement.scrollTop;
            console.log(n);
        })
    </script>
</body>

</html>
```

滚动高度 `scrollTop` 属性可以修改，因此可用于实现返回顶部等效果。例如

```js
document.documentElement.scrollTop = 0;
window.scrollTo(0, 0);
```

上述两行代码效果等价。



###### 尺寸事件

页面尺寸修改时触发此事件，添加在窗口上

```js
window.addEventListener('resize', function() {
	// 获得屏幕宽度
	let w = document.documentElement.clientWidth;
});
```





## Window 对象

### BOM

BOM (浏览器对象模型) 通过 window 全局对象，即 JS 中的顶级对象管理属性。

![](image-20240819235322189.png|800)



### 定时器

#### 间歇

间歇函数每隔指定时间就执行一次

```js
setInterval(function () {
	console.log('一秒调用一次');
}, 1000);

function fn() {
	console.log('一秒调用一次');
};

// 可以获得定时器 id，用于关闭定时器
let timerId = setInterval(fn, 1000);
clearInterval(timerId);
```



#### 延迟

延迟函数等待指定时间后执行一次

```js
setTimeout(function () {
    console.log('延迟 1 秒执行');
}, 1000);
```



### 执行机制

JS 的特点之一是单线程，同一时间只能做一件事。如果执行时间过长，就会造成页面的渲染不连贯，导致页面渲染加载阻塞的效果。为了能够利用多核 CPU 的计算能力，在 HTML5 中提出 Web Worker 标准，允许 JS 脚本创建多个线程。JS 任务进行分类

- 同步任务：在主线程上执行，形成一个执行栈
- 异步任务：通过回调函数实现，添加到任务队列（消息队列）中。主要包括
	- 普通事件：如 `click, resize` 等
	- 资源加载：如 `load, error` 等
	- 定时器：包括 `setInterval, setTimeout` 等

先执行执行栈中的同步任务，将异步任务放入任务队列。当同步任务执行完毕，就会按次序读取异步任务，放入执行栈。

> 主线程不断重复获得任务、执行任务的过程，称为事件循环。



### BOM 对象
#### location

使用 `location` 对象获得页面信息

```js
location.href = "https://www.baidu.com"; 	// 跳转到百度首页
location.search								// 获得地址 ? 后面的部分
location.hash								// 获得地址 # 后面的部分
location.hostname							// 返回 web 主机的域名
location.pathname							// 返回当前页面的路径或文件名
location.protocol							// 返回使用的 web 协议（http: 或 https:）
location.assign("https://www.baidu.com"); 	// 加载新的页面
location.reload								// 刷新页面（传入 true 强制刷新）
```



#### navigator

使用 `navigator` 获得浏览器信息

```js
navigator.cookieEnabled // 返回 cookie 是否启用
navigator.appName       // 返回浏览器应用程序名
navigator.appCodeName   // 返回浏览器应用程序代码名
navigator.product       // 返回浏览器引擎产品名
navigator.appVersion    // 返回浏览器版本信息
navigator.userAgent     // 返回由浏览器发送到服务器的用户代理报头
navigator.platform      // 返回浏览器平台（操作系统）
navigator.language      // 返回浏览器语言
navigator.onLine        // 返回浏览器是否在线
navigator.javaEnabled   // 返回 java 是否启用
```



#### history

使用 `history` 获得历史页面

```js
history.back()      // 等同于在浏览器点击后退按钮
history.forward()   // 等同于在浏览器中点击前进按钮
history.go(n)		// 前进 n 个页面，n 可以为负，表示后退
```



### 本地存储

#### 存储信息

使用 `localStorage` 对象保存信息，不会随着网页刷新消失

```js
localStorage.setItem('name', 'John');
console.log(localStorage.getItem('name'));
localStorage.removeItem('name');
```

保存后的数据可以在 Application 选项卡中找到。

![](image-20240820002628166.png)



#### 分类

使用 `sessionStorage` 存储信息

- 关闭浏览器窗口后丢失
- 在同一个窗口下数据共享

其基本用法与 `localStorage` 一致。



#### 复杂信息

使用 `localStorage` 只能保存字符串，如果要保存对象，需要进行转换

```js
const obj = {
    name: 'John',
    age: 30,
    city: 'New York'
};

// 转换为 JSON 字符串
localStorage.setItem('user', JSON.stringify(obj));

// 重新解析
const user = JSON.parse(localStorage.getItem('user'));
console.log(user);
```



## 正则表达式

### 语法

正则表达式语法为

```js
const reg = /pattern/modifiers;
```

例如

```js
const str = 'hello';
const reg = /he/;
console.log(reg.test(str));     		// 是否匹配 true
console.log(reg.exec(str));     		// 匹配到的内容 ["he"]
console.log(str.match(reg));    		// 匹配到的内容 ["he"]
console.log(str.replace(reg, 'hi'));  	// 替换后的内容 "hillo"
```



### 元字符

可以使用元字符（特殊字符）进行匹配。例如

```js
const reg = /[a-z]/;
```



#### 边界符

表示位置，定位开头和结尾。例如

```js
const reg = /^a/;   // 以 a 开头
const reg = /$a/;   // 以 a 结尾
```

如果同时指定开头和结尾，则表示精确匹配

```js
const reg = '/^hello$/';	// 严格匹配 hello
```



#### 量词

量词指定模式的重复次数

| 字符       | 作用                     |
| -------- | ---------------------- |
| `*`      | 前面的字符进行任意（0 到多次）次匹配    |
| `+`      | 前面的字符进行一次或多次匹配         |
| `？`      | 前面的字符进行 0 次或一次匹配       |
| `{n}`    | 前面的字符进行 n 次（非负）匹配      |
| `{n,}`   | 前面的字符进行至少 n 次（非负）匹配    |
| `{n, m}` | 前面的字符进行至少 n 次，最多 m 次匹配 |



#### 字符类

字符类指定匹配的字符类型

| 字符       | 作用                             |
| -------- | ------------------------------ |
| `[abc]`  | [] 中给出一个字符集合，可以匹配其中的**任意一个**字符 |
| `[^abc]` | 与该集合外的任意一个字符匹配                 |
| `[a-z]`  | 匹配 a-z 范围的一个字符匹配               |
| `[^a-z]` | 与该集合外的一个字符匹配                   |
| `.`      | 匹配除换行符外的**任意一个**字符             |



使用 `\` 转义字符来匹配格式

| 转义符  | 作用                    |
| ---- | --------------------- |
| `\d` | 匹配一个数字字符              |
| `\D` | 匹配一个非数字字符             |
| `\w` | 匹配一个单词（包括数字、字母、下划线）   |
| `\W` | 匹配一个非单词字符             |
| `\s` | 匹配一个不可见字符（空格、制表符、换行符） |
| `\S` | 匹配一个可见字符              |



#### 修饰符

| **修饰符** | **描述**                      |
| ------- | --------------------------- |
| `i`     | 执行对大小写不敏感的匹配                |
| `g`     | 执行全局匹配（查找所有匹配而非在找到第一个匹配后停止） |
| `m`     | 执行多行匹配                      |



## 进阶语法

### 垃圾回收

#### 引用计数

引用计数法考虑每个对象的引用次数，当引用次数为 0 时则回收内存。但是在某些情况下将会导致内存泄漏

```js
function fn() {
    let o1 = {};
    let o2 = {};
    o1.a = o2;
    o2.a = o1;
}

fn();
```

如果两个对象相互引用，就会导致无法回收内存。



#### 标记清除

现代浏览器已经弃用引用计数法，转而使用标记清除。它从全局作用域出发，逐步扫描局部作用域，检查可以到达的对象。如果一个局部作用域不能从全局作用域到达，则会清除该作用域的内存。



### 闭包

闭包可以理解为内部嵌套的函数，其保存了外层函数的变量。例如

```js
function outer() {
    const a = 1;
    function f() {
        console.log(a);
    }
    f();
}
```

借助闭包，可以从外部访问内部变量

```js
function outer() {
    const a = 1;
    function f() {
        return a;
    }
    return f;
}

const g = outer();
console.log(g()); // 1
```

局部变量 `a` 被引用到 `f` 中，因此即使 `outer` 执行完毕也不会销毁。利用这一特性，可以实现变量私有化

```js
function outer() {
    let i = 0;
    function f() {
        i++;
        return i;
    }
    return f;
}

const g = outer();
console.log(g()); // 1 
console.log(g()); // 2
```




### 函数进阶

#### 函数提升

函数执行前，会将所有函数声明提升到当前作用域开头，因此可以在声明前调用

```js
fn();

function fn() {
    console.log('fn');
}
```

函数表达式不会提升

```js
fn();

// 报错
var fn = function () {
    console.log('fn');
}
```

这里遵循 `var` 声明的变量提升规则，得到

```js
var fn;

// fn 不是函数
fn();

fn = function () {
    console.log('fn');
}
```



#### 变长参数

可以在函数内部获得传入的动态参数，因此可以指定变长参数

```js
function getSum() {
    let sum = 0;
    // arguments 是一个保存了动态参数的字典
    for (let arg of arguments) {
        sum += arg;
    }
    return sum;
}

// 传入多个参数
console.log(getSum(1, 2, 3, 4, 5));
```



#### 剩余参数

使用 `...` 获得除了具名参数以外的剩余参数，更为常用

```js
// arr 是剩余参数的数组
function getSum(a, b, ...arr) {
    let sum = a + b;
    for (let i of arr) {
        sum += i;
    }
    return sum;
}

console.log(getSum(1, 2));
console.log(getSum(1, 2, 3, 4, 5));
```



#### 展开运算符

展开运算符 `...` 将数组内容展开后传入函数

```js
const arr = [1, 2, 3, 4, 5];
console.log(Math.max(...arr));

const arr2 = [6, 7, 8];
const arr3 = [...arr, ...arr2];
console.log(arr3);
```

常用于获得数组最大最小值、合并数组。



#### 箭头函数

箭头函数是为了更简短的函数写法且不绑定 `this`，用于本来需要匿名函数的地方。例如

```js
// 函数表达式
const fn = function(x) {
    console.log(x);
};

// 箭头函数
const arrow = (x) => {
    console.log(x);
};

// 只有一个形参时可以省略 ()
const arrow = x => {
    console.log(x);
};

// 只有一行时可以省略 {}
const arrow = (x, y) => console.log(x, y);

// 如果有返回值，且只有一行，可以省略 return
const arrow = (x, y) => x + y;

// 可以直接返回对象，需要用 () 包围
const arrow = (uname) => ({name: uname});
```



#### this 变量

普通函数内部具有 `this` 变量，指向所在的对象。例如

```js
const fn = function() {
    console.log(this);
};

// this 是所在对象 window
fn();

const obj = {
    name: 'John',
    say: function () {
        console.log(this);
    }
};


// this 是所在对象 obj
obj.say();
```

而箭头函数内部没有 `this`，因此从上一层作用域获得

```js
const fn = () => {
    console.log(this);
};

// window
fn();

const obj = {
    name: 'John',
    say: () => {
        console.log(this);
    }
};

// window
obj.say();

const obj = {
    name: 'John',
    say: function() {
        let i = 10;
        // 箭头函数
        const count = () => {
            console.log(this);
        }
        count();
    }
};

// 上层作用域 say 中有 this，因此 count 获得 say 中的 this
obj.say();
```



#### 修改 this

通过 `call` 方法可以指定 `this` 变量

```js
const obj = {
    name: 'John',
    age: 30,
    city: 'New York'
};

let fn = function (x, y) {
    // this 指向 obj
    console.log(this);
};

fn.call(obj, 1, 2);
```

也可以使用 `apply` 指定，它将参数通过数组传入

```js
fn.apply(obj, [1, 2]);
```



#### 绑定参数

使用 `bind` 可以给函数绑定参数，返回一个新的函数

```js
const obj = {
    name: 'John',
    age: 30,
    city: 'New York'
};

let fn = function (x, y) {
    // this 指向 obj
    console.log(this);
};

// 绑定 this 和一个参数，得到 func(y) 只需要一个参数
const func = fn.bind(obj, 1);
func(2);
```

借助绑定参数，可以统一嵌套调用时 `this` 的指向

```js
<body>
	// 按钮点击后等待 3 秒后可以再次点击
    <button>按钮</button>
    <script>
        document.querySelector('button').addEventListener('click', function() {
        	// this 指向 button
            this.disabled = true;
            window.setTimeout(function() {
				// this 指向 button                
                this.disabled = false;
            }.bind(this), 3000);
            // bind 修改 this 指向为 this，即 button
        });
    </script>
</body>
```



### 解构赋值

#### 数组解构

使用 `[]` 将数组元素解构赋值给多个变量

```js
const [x, y, z] = [1, 2, 3];
console.log(x, y, z);

// 交换赋值
let a = 1;
let b = 2;
[a, b] = [b, a];
console.log(a, b);

function value() {
    return [1, 2];
}

const [a, b] = value();
console.log(a, b);
```

如果赋值变量少于传入值，则多余的值会被丢弃；如果希望获得所有值，可以使用剩余参数

```js
const [x, y, z] = [1, 2, 3, 4];
console.log(x, y, z);

const [a, b, ...c] = [1, 2, 3, 4];
console.log(a, b, c);
```

可以空出位置表示跳过赋值

```js
const [x, y, , z] = [1, 2, 3, 4];
console.log(x, y, z);
```

多维数组解构

```js
const [a, b, [c, d]] = [1, 2, [3, 4]];
console.log(a, b, c, d);
```



#### 对象解构

使用 `{}` 将对象属性赋给多个值

```js
const obj = {
    name: 'John',
    age: 30,
    city: 'New York'
};

// 解构变量名要与对象属性名一致
const { name, age, city } = obj;

console.log(name); 	// John
console.log(age); 	// 30
console.log(city); 	// New York
```

为了避免变量名重冲突，可以在解构时指定新的变量名

```js
// username 为新的变量名
const { name: username, age, city } = obj;
console.log(username);
```

对象解构和数组解构都可以多级嵌套使用

```js
const obj = ['hello', {
    name: 'John',
    age: 30,
    city: 'New York'
}];

// 解构变量名要与对象属性名一致
const [str, { name: username, age, city }] = obj;
```



### 对象进阶

#### 构造函数

定义一个返回对象的函数，作为构造函数

```js
function Pig(name, age) {
    this.name = name;
    this.age = age;
    this.eat = function() {
        console.log(this.name + " is eating");
    }
    return this;
}

const pig = new Pig('Piggy', 2);
console.log(pig.name);  // Piggy
console.log(pig.age);   // 2
pig.eat();              // Piggy is eating
```

但是构造函数封装可能造成内存浪费，例如其中的 `eat` 方法保存在每一个实例中，占用了多余的内存。



#### 原型对象

为了解决构造函数封装方法造成的内存浪费，可以利用原型对象。

- 构造函数通过原型分配的函数是所有对象共享的
- 每个构造函数都有一个 `prototype` 属性，指向另一个对象，称为原型对象，它可以挂载函数
- 不变的方法可以定义在原型对象上
- 构造函数和原型对象中的 `this` 都指向实例化的对象

例如

```js
function Pig(name, age) {
    this.name = name;
    this.age = age;
    // 单独的属性
    this.eat = function () {
        console.log(this.name + ' is eating');
    }
    return this;
}

// 将方法定义到原型上
Pig.prototype.say = function () {
    console.log(this.name + ' says hello');
}

const pig1 = new Pig('Piggy1', 2);
const pig2 = new Pig('Piggy2', 4);
console.log(pig1.eat === pig2.eat); // false
console.log(pig1.say === pig2.say); // true
```



原型对象保存了构造器

```js
Pig.prototype.constructor === Pig
```

可以直接在原型对象中添加方法

```js
Pig.prototype = {
    // 需要将构造函数指向原型上
    constructor: Pig,
    eat: function () {
        console.log(this.name + ' is eating');
    },
    say: function () {
        console.log(this.name + ' says hello');
    }
}
```



#### 对象原型

每个对象实例中都保存一个对象原型 `__proto__`，它指向构造器的原型

```js
pig.__proto__ === Pig.prototype
```

因此可以在实例中调用原型对象中添加的方法

```js
Pig.prototype.say = function () {
    console.log(this.name + ' says hello');
}

const pig = new Pig('Piggy', 2);
// 通过 __proto__ 调用 Pig.prototype.say
pig.say();
```



#### 原型继承

利用原型对象来实现继承

```js
function Person() {
    this.head = 1;
    this.eyes = 2;
    return this;
}

function Man() {

}

// 通过原型继承 Person，使用 new Person 来创建不同的对象
Man.prototype = new Person();

// 重新指定构造函数，并添加新的方法
Man.prototype.constructor = Man;
Man.prototype.speak = function() {
    console.log('man speak');
}

function Woman() {

}

Woman.prototype = new Person();

Woman.prototype.constructor = Woman;
Woman.prototype.speak = function() {
    console.log('woman speak');
}

const man = new Man();
const woman = new Woman();

man.speak();
woman.speak();
```

使用 `new Person` 的目的在于，要向 `prototype` 中添加新的方法，如果使用同一个 `Person` 对象，就会导致 `Man, Woman` 共用所有的方法。



#### 原型链

原型对象本身作为一个对象，也具有对象原型 `__proto__`，这个对象原型指向其构造器 `Object` 的原型，即

```js
Person.prototype.__proto__ === Object.prototype
```

同样的，`Object` 的原型也具有对象原型

```js
Object.prototype.__proto__
```

由于 `Object` 构造器没有原型，因此其对象原型为空。这种原型对象之间的链状关系称为原型链。



根据原型链传递关系，JS 依次查找对象属性或方法

1. 首先检查对象 `person` 本身是否具有相应的属性
2. 如果没有，则检查对象的原型对象 `Person.prototype`
3. 如果没有，则检查原型对象的原型 `Person.prototype.__proto__ => Object.prototype`

对于具有继承关系的对象，将会按照此流程向上查询。可以使用 `instanceof` 判断一个实例是否是某个对象的实例

```js
person instanceof Person	// true
person instanceof Object	// true
Person instanceof Object	// true
```

这里 `instanceof` 会按照上述流程查询判断。



#### 静态成员

构造函数的属性和方法称为静态成员。例如

```js
// 构造函数
function Pig(name, age) {
    this.name = name;
    this.age = age;
    return this;
}

// 静态属性
Pig.eyes = 2;
Pig.ears = 2;
Pig.nose = 1;

// 静态方法
Pig.sayHello = function() {
    console.log("Hello");
};

Pig.sayHello();
```



#### Object

在 `Object` 中存在几个常用的静态方法

```js
const obj = {
    name: 'John',
    age: 30,
    city: 'New York'
};

// 获得所有属性和值的列表
const keys = Object.keys(obj);
const values = Object.values(obj);
console.log(keys, values);

// 拷贝对象，常用于添加属性
const oo = {};
Object.assign(oo, obj);
Object.assign(oo, { gender: 'woman' });
console.log(oo);
```



#### Array

在 `Array` 中常用的静态方法

| 方法        | 作用   | 说明             |
| :-------- | :--- | :------------- |
| `forEach` | 遍历数组 | 用于查找数组元素       |
| `filter`  | 过滤数组 | 返回满足条件的新数组     |
| `map`     | 迭代数组 | 返回处理后的新数组      |
| `reduce`  | 累计器  | 返回累计处理的结果，例如求和 |

例如数组求和

```js
const arr = [1, 2, 3, 4];
const sum = arr.reduce(function (prev, cur) {
    // 累计前面的值和当前值
    return prev + cur;
}, 10);
console.log(sum);

// 使用箭头函数简写
const sum = arr.reduce((prev, cur) => prev + cur, 10);
```

其中第二个参数指定起始值。



### 拷贝

#### 浅拷贝

有两种方式实现浅拷贝

```js
const obj = {
    name: 'John',
    age: 30,
    city: 'New York'
};

// 解构赋值
let o = { ...obj };
o.name = 'Jane';
console.log(o);
console.log(obj);

// assign 方法
let o2 = {};
Object.assign(o2, obj);
o2.name = 'Jenny';
console.log(o2);
console.log(obj);
```



#### 深拷贝

利用函数递归手动实现深拷贝

```js
function deepCopy(newObj, oldObj) {
    for (let key in oldObj) {
        // 处理对象和数组
        if (oldObj[key] instanceof Array) {
            newObj[key] = [];
            deepCopy(newObj[key], oldObj[key]);
        } else if (oldObj[key] instanceof Object) {
            newObj[key] = {};
            deepCopy(newObj[key], oldObj[key]);
        } else {
            newObj[key] = oldObj[key];
        }
    }
}

const obj1 = {
    name: "John",
    age: 30,
    address: {
        street: "123 Main St",
        city: "New York",
        state: "NY"
    },
    hobbies: ["reading", "swimming", "running"]
};

const obj2 = {};
deepCopy(obj2, obj1);

console.log(obj2);
```

由于数组本身也是对象，因此需要先处理数组再处理对象（其实可以统一作为对象处理）。



可以利用 JSON 转换实现深拷贝

```js
const obj1 = {
    name: "John",
    age: 30,
    address: {
        street: "123 Main St",
        city: "New York",
        state: "NY"
    },
    hobbies: ["reading", "swimming", "running"]
};

const obj2 = JSON.parse(JSON.stringify(obj1));
console.log(obj2);
```



### 异常处理

#### throw

使用 `throw` 抛出异常

```js
throw 'something wrong';
throw new Error('something wrong');
```



#### try

使用 `try...catch` 语法捕获错误

```js
try {
    let box = document.querySelector('.box');
    box.scrollTop = 1000;
    console.log(box.scrollTop);
} catch (error) {
    console.log('错误：', error);
    throw new Error('something wrong');
} finally {
	alert('警告');
}
```

其中 `finally` 即使在 `catch` 中报错的情况下都会执行。



#### debugger

用于手动添加断点，当在浏览器中调试时，程序会暂停在 `debugger` 位置。例如

```js
debugger;
const obj = {
    name: 'John',
    age: 30,
    city: 'New York'
};
```



