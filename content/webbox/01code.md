---
title: 📣 手撕代码 & 项目题全掌握
tags:
  - web
---

## 🍕数组扁平化

### ES6 的 flat 方法

> flat() 方法将以指定的深度递归遍历数组，并将所有元素与遍历的子数组中的元素合并到一个新数组中以返回。

```js
const arr = [1, [2, [3, [4, 5]]], 6]
//数组自带的扁平化方法,flat的参数代表的是需要展开几层。Infinity不管嵌套几层，全部都展开
console.log(arr.flat(Infinity))
// [1, 2, 3, 4, 5, 6]
```

### 使用正则

> 1. 首先是使用 JSON.stringify 把 arr 转为字符串
> 2. 接着使用正则把字符串里面的 [ 和 ] 去掉(`/\[|\]/g`,`g`是全局匹配)
> 3. 然后再拼接数组括号转为数组对象

```js
const arr = [1, [2, [3, [4, 5]]], 6]
const res = JSON.stringify(arr).replace(/\[|\]/g, "")
const res2 = JSON.parse("[" + res + "]")
console.log(res2) //[1, 2, 3, 4, 5, 6]
```

### 使用递归

> 递归，在数学和计算机科学中，是指在函数的定义中使用函数本身的方法。递归这个词经常用来描述以自相似的方式重复事物的过程。通俗的理解是：我调用我自己。

```js
const arr = [1, [2, [3, [4, 5]]], 6]
const array = []
const fn = (arr) => {
  for (let i = 0; i < arr.length; i++) {
    if (Array.isArray(arr[i])) {
      fn(arr[i])
    } else {
      array.push(arr[i])
    }
  }
}
fn(arr)
console.log(array) //[1, 2, 3, 4, 5, 6]
```

### 使用 reduce

> 1. reduce 方法: 可以用来给数组求和
> 2. concat()方法用于连接两个或多个数组。
> 3. concat()方法不会更改现有数组，而是返回一个新数组，其包含已连接数组的值。

```js
const arr = [1, [2, [3, [4, 5]]], 6]
const newArr = (arr) => {
  return arr.reduce((pre, cur) => {
    return pre.concat(Array.isArray(cur) ? newArr(cur) : cur)
  }, [])
}
console.log(newArr(arr), "reduce方法") //[1, 2, 3, 4, 5, 6] 'reduce方法'
```

### 使用栈的思想实现 flat 函数

> 栈（stack）又名堆*栈*，它是一种运算受限的线性表。限定仅在表尾进行插入和删除操作的线性表。

```js
// 栈思想
function flat(arr) {
  const newArr = []
  const stack = [].concat(arr) // 将数组元素拷贝至栈，直接赋值会改变原数组
  // //如果栈不为空，则循环遍历
  while (stack.length !== 0) {
    const val = stack.pop() // 删除数组最后一个元素，并获取它
    if (Array.isArray(val)) {
      stack.push(...val) // 如果是数组再次入栈，并且展开了一层
    } else {
      newArr.unshift(val) // 如果不是数组就将其取出来放入结果数组中
    }
  }
  return newArr
}
let arr = [12, 23, [34, 56, [78, 90, 100, [110, 120, 130, 140]]]]
console.log(flat(arr))
// [12, 23, 34, 56, 78, 90, 100, 110, 120, 130, 140]
```

## 🍔深拷贝和浅拷贝

1、什么是数据类型？

- 数据分为基本数据类型 (String, Number, Boolean, Null, Undefined，Symbol) 和引用数据类型。

- 基本数据类型的特点：直接存储在栈 (stack) 中的数据

- 引用数据类型的特点：存储的是该对象在栈中引用，真实的数据存放在堆内存里

- 引用数据类型在栈中存储了指针，该指针指向堆中该实体的起始地址。

2、ES11新增了什么数据类型？

- bigint!

- bigint使用方式：

```js
let max = Number.MAX_SAFE_INTEGER
console.log(BigInt(max)) //9007199254740991n
console.log(BigInt(max) + BigInt(1)) //9007199254740992n 正确
console.log(BigInt(max) + BigInt(2)) //9007199254740993n 正确
console.log(BigInt(max) + BigInt(4)) //9007199254740995n 正确
```

3、为什么要使用深拷贝？

- 我们希望在改变新的数组（对象）的时候，不改变原数组（对象）

4、赋值和浅拷贝的区别？

- 当我们把一个对象赋值给一个新的变量时，赋的其实是该对象的在栈中的地址，而不是堆中的数据。也就是两个对象指向的是同一个存储空间，无论哪个对象发生改变，其实都是改变的存储空间的内容，因此，两个对象是联动的。

- 浅拷贝是按位拷贝对象，它会创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。

### 浅拷贝

浅拷贝是一点一点地拷贝一个对象。它将创建一个新对象。此对象具有原始对象属性值的精确副本。如果属性是基本类型，它将复制基本类型的值；如果属性是内存地址（引用类型），则复制内存地址。因此，如果一个对象更改地址，另一个对象将受到影响。也就是说，默认复制构造函数只复制浅层复制中的对象（依次复制成员），即只复制对象空间，而不复制资源。

```js
var obj1 = {
  name: "张三",
  age: 8,
  pal: ["王五", "王六", "王七"],
}
var obj3 = shallowCopy(obj1)
function shallowCopy(src) {
  var newObj = {}
  for (var prop in src) {
    console.log(prop)
    if (src.hasOwnProperty(prop)) {
      newObj[prop] = src[prop]
    }
  }
  return newObj
}
obj3.name = "李四"
obj3.pal[0] = "王麻子"

console.log("obj1", obj1) //{age: 8, name: "张三", pal: ['王麻子', '王六', '王七']}
console.log("obj3", obj3) //{age: 8, name: "李四", pal: ['王麻子', '王六', '王七']}
```

### 深拷贝

#### 方法一：递归实现深拷贝

实现**深拷贝**原理的递归方法：遍历对象、数组甚至内部都是基本的数据类型，然后复制它们，即深度复制。

```js
var obj = {
  //原数据，包含字符串、对象、函数、数组等不同的类型
  name: "test",
  main: {
    a: 1,
    b: 2,
  },
  fn: function () {},
  friends: [1, 2, 3, [22, 33]],
}

function copy(obj) {
  let newobj = null //声明一个变量用来储存拷贝之后的内容
  //判断数据类型是否是复杂类型，如果是则调用自己，再次循环，如果不是，直接赋值即可，
  //由于null不可以循环但类型又是object，所以这个需要对null进行判断
  if (typeof obj == "object" && obj !== null) {
    //声明一个变量用以储存拷贝出来的值,根据参数的具体数据类型声明不同的类型来储存
    newobj = obj instanceof Array ? [] : {}
    //循环obj 中的每一项，如果里面还有复杂数据类型，则直接利用递归再次调用copy函数
    for (var i in obj) {
      newobj[i] = copy(obj[i])
    }
  } else {
    newobj = obj
  }
  console.log("77", newobj)
  return newobj //函数必须有返回值，否则结构为undefined
}
var obj2 = copy(obj)
obj2.name = "修改成功"
obj2.main.a = 100
console.log(obj) //{name: "test",main: {a: 1, b: 2,},fn: function () {},friends: [1, 2, 3, [22, 33]],}
console.log(obj2) //{name: "修改成功",main: {a: 100, b: 2,},fn: function () {},friends: [1, 2, 3, [22, 33]],}
```

#### 方法二：递归实现深拷贝

- 递归处理 引用类型
- 保持数组类型

```js
function deepCopy(target) {
  if (typeof target === "object") {
    const newTarget = Array.isArray(target) ? [] : Object.create(null)
    for (const key in target) {
      newTarget[key] = deepCopy(target[key])
    }
    return newTarget
  } else {
    return target
  }
}
```

#### 方法三：递归实现深拷贝

- 哈希表 Map 支持 循环引用
  - Map 支持引用类型数据作为键

```js
function deepCopy(target, h = new Map()) {
  if (typeof target === "object") {
    if (h.has(target)) return h.get(target)
    const newTarget = Array.isArray(target) ? [] : Object.create(null)
    for (const key in target) {
      newTarget[key] = deepCopy(target[key], h)
    }
    h.set(target, newTarget)
    return newTarget
  } else {
    return target
  }
}
```

#### 方法四：递归实现深拷贝

- 哈希表 WeakMap 代替 Map

- WeakMap 的键是弱引用，告诉 JS 垃圾回收机制，当键回收时，对应 WeakMap 也可以回收，更适合大量数据深拷

```js
function deepCopy(target, h = new WeakMap()) {
  if (typeof target === "object") {
    if (h.has(target)) return h.get(target)
    const newTarget = Array.isArray(target) ? [] : Object.create(null)
    for (const key in target) {
      newTarget[key] = deepCopy(target[key], h)
    }
    h.set(target, newTarget)
    return newTarget
  } else {
    return target
  }
}
```

## 🍟防抖节流

### 防抖

防抖是指短时间内大量触发同一事件，只会在最后一次事件完成后延迟执行一次函数。例如，在输入用户名的过程中，需要反复验证用户名。此时，您应该等待用户停止输入，然后进行验证，否则将影响用户体验。 防抖实现的原理是在触发事件后设置计时器。在计时器延迟过程中，如果事件再次触发，则重置计时器。在没有触发事件之前，计时器将再次触发并执行相应的功能。

> - 声明定时器
> - 返回函数
> - 一定时间间隔，执行回调函数
> - 回调函数
>   - 已执行：清空定时器
>   - 未执行：重置定时器

```js
function debounce(fn, delay) {
  let timer = null
  return function (...agrs) {
    if (timer) clearTimeout(timer)
    timer = setTimeout(
      () => {
        timer = null
        fn.apply(this, args)
      },
      (delay + "") | 0 || 1000 / 60,
    )
  }
}
```

### 节流

节流是指每隔一段时间就执行一次函数。就像未拧紧的水龙头一样，水龙头每隔一段时间就会滴水。即使在这段时间管道里有更多的水，水龙头也不会掉更多的水。

节流的原理是在触发事件后设置计时器。在计时器延迟过程中，即使事件再次触发，计时器的延迟时间也不会改变。在计时器执行功能之前，计时器不会复位。

> - 声明定时器
> - 返回函数
> - 一定时间间隔，执行回调函数
> - 回调函数
>   - 已执行：清空定时器
>   - 未执行：返回

```js
function throttle(fn, interval) {
  let timer = null
  return function (...args) {
    if (timer) return
    timer = setTimeout(
      () => {
        timer = null
        fn.apply(this, args)
      },
      (interval + "") | 0 || 1000 / 60,
    )
  }
}
```

## 🌭for...in和for...of

### for..in

for...in 循环只遍历可枚举属性（包括它的原型链上的可枚举属性）。像 Array 和 Object 使用内置构造函数所创建的对象都会继承自 Object.prototype 和 String.prototype 的不可枚举属性，例如 String 的 indexOf() 方法或 Object 的 toString() 方法。循环将遍历对象本身的所有可枚举属性，以及对象从其构造函数原型中继承的属性（更接近原型链中对象的属性覆盖原型属性）。

```js
let stus = ["sum", "22", "男"]
for (let stu in stus) {
  //for..in用来遍历对象的属性
  console.log(stus[stu]) //sum 22 男
}
```

### for..of

Q: ES6 的 for of 可以遍历对象吗?

A: ES6 的 “for of” 不能遍历对象。

原因：ES6 中引入了 Iterator 接口，只有提供了 Iterator 接口的数据类型才可以使用 “for-of” 来循环遍历；而普通对象默认没有提供 Iterator 接口，因此无法用 “for-of” 来进行遍历。

```js
let letters = ["a", "b", "c"]
for (let letter of letters) {
  //for..of用于遍历一个迭代器，如数组
  console.log(letter) //a b c
}
```

## 🍿从数组 [1,2,3,4,5,6] 中找出值为 2 的元素

### filter()

> filter() 方法创建一个新数组, 包含通过所提供函数实现的测试的所有元素。

```js
function isBigEnough(element) {
  return element == 2
}
let filtered = [1, 2, 3, 4, 5, 6].filter(isBigEnough)
console.log(filtered) //[2]
```

### find()

> find() 方法返回数组中满足提供的测试函数的第一个元素的值。否则返回undefined。

```js
let arr = [1, 2, 3, 4, 5, 6]
function finds(x) {
  return x == "2"
}
console.log(arr.find(finds)) //2
```

## 🧂两栏布局和三栏布局

### 两栏布局

两栏布局非常常见，往往是以一个定宽栏和一个自适应的栏并排展示存在。

#### 浮动布局

- 使用 float 左浮左边栏
- 右边模块使用 margin-left 撑出内容块做内容展示
- 为父级元素添加 BFC，防止下方元素飞到上方内容

```html
<style>
  .box{
      overflow: hidden; 添加BFC
  }
  .left {
      float: left;
      width: 200px;
      background-color: gray;
      height: 400px;
  }
  .right {
      margin-left: 210px;
      background-color: lightgray;
      height: 200px;
  }
</style>
<div class="box">
  <div class="left">左边</div>
  <div class="right">右边</div>
</div>
```

#### flex 弹性布局

flex 容器的一个默认属性值: align-items: stretch

这个属性导致了列等高的效果。 为了让两个盒子高度自动，需要设置: align-items: flex-start

```
<style>
    .box{
        display: flex;
    }
    .left {
        width: 100px;
    }
    .right {
        flex: 1;
    }
</style>
<div class="box">
    <div class="left">左边</div>
    <div class="right">右边</div>
</div>
```

### 三栏布局

实现三栏布局中间自适应的布局方式有：

- 两边使用 float，中间使用 margin

- 两边使用 absolute，中间使用 margin
- 两边使用 float 和负 margin
- flex 实现
- grid 网格布局

#### 两边使用 float，中间使用 margin

需要将中间的内容放在 `html` 结构最后，否则右侧会臣在中间内容的下方

```html
<style>
  .wrap {
      background: #eee;
      overflow: hidden; <!-- 生成BFC，计算高度时考虑浮动的元素 -->
      padding: 20px;
      height: 200px;
  }
  .left {
      width: 200px;
      height: 200px;
      float: left;
      background: coral;
  }
  .right {
      width: 120px;
      height: 200px;
      float: right;
      background: lightblue;
  }
  .middle {
      margin-left: 220px;
      height: 200px;
      background: lightpink;
      margin-right: 140px;
  }
</style>
<div class="wrap">
  <div class="left">左侧</div>
  <div class="right">右侧</div>
  <div class="middle">中间</div>
</div>
```

#### 两边使用 absolute，中间使用 margin

基于绝对定位的三栏布局：注意绝对定位的元素脱离文档流，相对于最近的已经定位的祖先元素进行定位。无需考虑 HTML 中结构的顺序

左右两边使用绝对定位，固定在两侧。

中间占满一行，但通过 margin 和左右两边留出 10px 的间隔

```html
<style>
  .container {
    position: relative;
  }

  .left,
  .right,
  .main {
    height: 200px;
    line-height: 200px;
    text-align: center;
  }

  .left {
    position: absolute;
    top: 0;
    left: 0;
    width: 100px;
    background: green;
  }

  .right {
    position: absolute;
    top: 0;
    right: 0;
    width: 100px;
    background: green;
  }

  .main {
    margin: 0 110px;
    background: black;
    color: white;
  }
</style>

<div class="container">
  <div class="left">左边固定宽度</div>
  <div class="right">右边固定宽度</div>
  <div class="main">中间自适应</div>
</div>
```

#### 两边使用 float 和负 margin

实现过程：

- 中间使用了双层标签，外层是浮动的，以便左中右能在同一行展示

- 左边通过使用负 margin-left:-100%，相当于中间的宽度，所以向上偏移到左侧

- 右边通过使用负 margin-left:-100px，相当于自身宽度，所以向上偏移到最右侧

缺点：

- 增加了 .main-wrapper 一层，结构变复杂

- 使用负 margin，调试也相对麻烦

```html
<style>
  .left,
  .right,
  .main {
    height: 200px;
    line-height: 200px;
    text-align: center;
  }

  .main-wrapper {
    float: left;
    width: 100%;
  }

  .main {
    margin: 0 110px;
    background: black;
    color: white;
  }

  .left,
  .right {
    float: left;
    width: 100px;
    margin-left: -100%;
    background: green;
  }

  .right {
    margin-left: -100px; /* 同自身宽度 */
  }
</style>

<div class="main-wrapper">
  <div class="main">中间自适应</div>
</div>
<div class="left">左边固定宽度</div>
<div class="right">右边固定宽度</div>
```

#### flex 实现

利用 flex 弹性布局，可以简单实现中间自适应。

实现过程：

- 仅需将容器设置为 display:flex;
- 盒内元素两端对其，将中间元素设置为 100% 宽度，或者设为 flex:1 ，即可填充空白
- 盒内元素的高度撑开容器的高度

优点：

- 结构简单直观

- 可以结合 flex 的其他功能实现更多效果，例如使用 order 属性调整显示顺序，让主体内容优先加载，但展示在中间

```html
<style type="text/css">
  .wrap {
    display: flex;
    justify-content: space-between;
  }

  .left,
  .right,
  .middle {
    height: 100px;
  }

  .left {
    width: 200px;
    background: coral;
  }

  .right {
    width: 120px;
    background: lightblue;
  }

  .middle {
    background: #555;
    width: 100%;
    margin: 0 20px;
  }
</style>
<div class="wrap">
  <div class="left">左侧</div>
  <div class="middle">中间</div>
  <div class="right">右侧</div>
</div>
```

#### grid 网格布局

跟 `flex` 弹性布局一样的简单

```html
<style>
  .wrap {
    display: grid;
    width: 100%;
    grid-template-columns: 300px auto 300px;
  }

  .left,
  .right,
  .middle {
    height: 100px;
  }

  .left {
    background: coral;
  }

  .right {
    background: lightblue;
  }

  .middle {
    background: #555;
  }
</style>
<div class="wrap">
  <div class="left">左侧</div>
  <div class="middle">中间</div>
  <div class="right">右侧</div>
</div>
```

## 🥓实现多行文本溢出的省略

### 使用定位伪元素遮盖末尾文字

- 给父元素设置:

```css
overflow: hidden; /* 溢出隐藏 */
line-height: 20px; /* 设置行高与容器高度成倍数关系，这样避免文本溢出时，文字显示一半 */
text-align: justify; /* 设置文本对齐方式为两端对齐，这样在伪元素内容遮盖末尾文字时才能对齐*/
position: relative; /* 子绝父相，这里是为了给伪元素设置定位*/
```

- 给父元素设置伪元素 ::after ，并为其设置属性：

```css
content: "..."; /* 省略号是放在文本最后面的 */
width: 1em; /* 设置伪元素的宽度为1em，是为了遮盖的时候正好遮盖中原来的一个字的大小*/
background-color: pink; /* 设置与父元素相同的背景颜色，同理，也是为了和原来的内容样式保持一致*/
position: absolute; /*设置定位，其位置就是文本的右下角 */
right: 0;
bottom: 0;
```

### 利用旧版弹性盒

- 给容器元素类型转换为 display:-webkit-box ;

- 设置弹性盒子垂直排列 -webkit-box-orient:vertical ;

- 控制要显示的行数 -webkit-line-clamp:数值 ；

- 溢出隐藏 overflow:hidden ;

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      /* 多行文本溢出隐藏显示省略号方法二：
          兼容性写法,因使用了WebKit的CSS扩展属性，该方法适用于WebKit浏览器及移动端
        */
      .box {
        width: 200px;
        /* height: 100px; 如果设置了高度，且高度超过文本显示行数，在第三行会正常出现省略号，但是三行之后的仍然正常显示*/
        background-color: pink;
        /* 设置高度是行高的倍数，防止文本露出一半 */
        line-height: 20px;
        /* 旧版弹性盒 */
        display: -webkit-box;
        /* 弹性盒子元素垂直排列 */
        -webkit-box-orient: vertical;
        /* 控制要显示的行数 */
        -webkit-line-clamp: 3;
        overflow: hidden;
      }
    </style>
  </head>
  <body>
    <div class="box">
      我是文本我是文本我是文本我是文我是文我是文我是文我是文我是文我是文我是是文本我是文本我是文本我是文本我是文本我是文本我是文本我是文本我是文本
    </div>
  </body>
</html>
```

## 🍳用 CSS 实现三角符号

### 上三角

```css
#triangle-up {
  width: 0;
  height: 0;
  border-left: 50px solid transparent;
  border-right: 50px solid transparent;
  border-bottom: 100px solid red;
}
```

### 下三角

```css
#triangle-up {
  width: 0;
  height: 0;
  border-left: 50px solid transparent;
  border-right: 50px solid transparent;
  border-top: 100px solid red;
}
```

### 左三角

```css
#triangle-up {
  width: 0;
  height: 0;
  border-bottom: 50px solid transparent;
  border-top: 50px solid transparent;
  border-right: 100px solid red;
}
```

### 右三角

```css
#triangle-up {
  width: 0;
  height: 0;
  border-bottom: 50px solid transparent;
  border-top: 50px solid transparent;
  border-left: 100px solid red;
}
```

### 左下三角

```css
#triangle-bottomleft {
  width: 0;
  height: 0;
  border-bottom: 100px solid red;
  border-right: 100px solid transparent;
}
```

### 右下三角

```css
#triangle-bottomright {
  width: 0;
  height: 0;
  border-bottom: 100px solid red;
  border-left: 100px solid transparent;
}
```

### 右上三角

```css
#triangle-topright {
  width: 0;
  height: 0;
  border-top: 100px solid red;
  border-left: 100px solid transparent;
}
```

### 左上三角

```css
#triangle-topleft {
  width: 0;
  height: 0;
  border-top: 100px solid red;
  border-right: 100px solid transparent;
}
```

## 🫔实现九宫格布局

### 基本布局和样式

##### html

```html
<div class="box">
  <ul>
    <li>1</li>
    <li>2</li>
    <li>3</li>
    <li>4</li>
    <li>5</li>
    <li>6</li>
    <li>7</li>
    <li>8</li>
    <li>9</li>
  </ul>
</div>
```

#### css

```css
.box {
  background: #e4f7fd61;
  border: 2px solid #0786ada1;
  border-radius: 8px;
}
ul {
  padding: 0;
  margin: 0;
}
.box li {
  list-style: none;
  text-align: center;
  line-height: 200px;
  background: skyblue;
  border-radius: 8px;
  font-size: 20px;
  color: black;
}
```

### flex

> 使用 `flex` 布局实现需要注意一个点，就是需要用 `flex-wrap` 属性来使其换行。

```css
.box {
  width: 100%;
  overflow: hidden;
}

ul {
  width: 100%;
  height: 100%;
  display: flex;
  flex-wrap: wrap;
}

.box li {
  width: 30%;
  height: 30%;
  margin-right: 5%;
  margin-bottom: 5%;
}

.box li:nth-of-type(3n) {
  margin-right: 0;
}

.box li:nth-of-type(n + 7) {
  margin-bottom: 0;
}
```

### float

使用 `float` 来实现需要注意一个点，浮动会造成浮动崩塌，因此可以设置 `overflow: hidden;` 把 `box` 设置成 BFC 来解决浮动崩塌。

```css
.box {
  width: 100%;
  overflow: hidden;
}

ul {
  width: 100%;
  height: 100%;
}

.box li {
  width: 30%;
  height: 30%;
  margin-right: 5%;
  margin-bottom: 5%;
  float: left;
}

.box li:nth-of-type(3n) {
  margin-right: 0;
}

.box li:nth-of-type(n + 7) {
  margin-bottom: 0;
}
```

### grid

```html
<div class="grid">
  <div>1</div>
  <div>2</div>
  <div>3</div>
  <div>4</div>
  <div>5</div>
  <div>6</div>
  <div>7</div>
  <div>8</div>
  <div>9</div>
</div>
```

```css
box {
  background: #e4f7fd61;
  border: 2px solid #0786ada1;
  border-radius: 8px;
}

.grid {
  display: grid;
  width: 100%;
  grid-template-rows: repeat(3, 1fr);
  grid-template-columns: repeat(3, 1fr);
  grid-gap: 5%;
  grid-auto-flow: row;
}

.grid > div {
  list-style: none;
  text-align: center;
  line-height: 200px;
  background: skyblue;
  border-radius: 8px;
  font-size: 20px;
  color: black;
}
```

### table

```html
<div class="box">
  <ul>
    <li>
      <div>1</div>
      <div>2</div>
      <div>3</div>
    </li>
    <li>
      <div>4</div>
      <div>5</div>
      <div>6</div>
    </li>
    <li>
      <div>7</div>
      <div>8</div>
      <div>9</div>
    </li>
  </ul>
</div>
```

```css
.box {
  width: 100%;
  overflow: hidden;
}

ul {
  width: 100%;
  height: 100%;
  display: table;
  border-spacing: 10px;
}

.box li {
  display: table-row;
}

.box li div {
  display: table-cell;
  text-align: center;
  border-radius: 10px;
}
```

## 🍚单行多行文字隐藏显示省略号

### 单行文字实现

> 文字单行隐藏：给它设定一个宽和高，对于文字超出部分进行隐藏，多余的部分用省略号来表示。

```css
text-overflow: ellipsis;
overflow: hidden;
white-space: nowrap;
```

### 多行文字实现

> 多行隐藏：多行隐藏的 div 的高度不能设置，让其自动 3 行隐藏即可。

```css
overflow: hidden;
-webkit-line-clamp: 3;
-webkit-box-orient: vertical;
display: -webkit-box;
```

## 🍤函数柯里化

### 什么是函数柯里化

> 函数柯里化是指将使用多个参数的函数转化成一系列使用一个参数的函数的技术, 它返回一个新的函数, 这个新函数去处理剩余的参数

```js
// ex:
function add(a, b) {
  return a + b
}
add(1, 2) // 3
// 柯里化const addCurry = curry(add);addCurry(1)(2) // 3
```

### 函数柯里化的实现

> 实现思路: 通过函数的 length 属性获取函数的形参个数, 形参的个数就是所需参数的个数

```js
function curry(fn) {
  return _curry.call(this, fn, fn.length)
}
function _curry(fn, len, ...args) {
  return function (...params) {
    const _args = args.concat(params)
    if (_args.length >= len) {
      return fn.apply(this, _args)
    } else {
      return _curry.call(this, fn, len, ..._args)
    }
  }
}
// test
function add(a, b, c, d) {
  return a + b + c + d
}
const addCurry = curry(add)
console.log(addCurry(1)(2)(3)(4)) // 10
console.log(addCurry(1, 2, 3)(4)) // 10
```

### 函数柯里化的作用

- 参数复用: 本质上来说就是降低通用性, 提高适用性
  - 假如一个函数需要两个参数, 其中一个参数可能多次调用并不会发生更改, 比如商品打折的力度, 此时可以根据折扣进行封装

- 提前返回
  - 经典实例: 元素绑定事件监听器, 区分 IE 浏览器的 attachEvent 方法

- 延迟计算: 柯里化函数不会立即执行计算，第一次只是返回一个函数，后面的调用才会进行计算

### 经典面试题

实现一个add方法，使计算结果能够满足如下预期：
console.log(add(1)(2)(3)(4, 5)()) //15
console.log(add(1)(2)(3, 4, 5)()) //15
console.log(add(1)(2, 3, 4, 5)()) //15

```js
function add1(...args) {
  //求和
  return args.reduce((a, b) => a + b)
}
function currying(fn, ...args) {
  return function (...rest) {
    if (rest.length) {
      return currying(fn, ...args, ...rest)
    } else {
      return fn.call(this, ...args)
    }
  }
}
let add = currying(add1)
console.log(add(1)(2)(3)(4, 5)()) //15
console.log(add(1)(2)(3, 4, 5)()) //15
console.log(add(1)(2, 3, 4, 5)()) //15
```

## 🎂图片懒加载

## 🍬bind、apply、call 的用法

## 🍭手写 new

## 🍻promise 的用法

## 🧁解析 URL

## 🍣音乐播放器

## 🥘组件通信

## 🍲购物车

## 🍨Mock 模拟数据

## 🍡轮播图

## 🍹路由跳转

## 🧋登录注册

## 🍩井字棋
