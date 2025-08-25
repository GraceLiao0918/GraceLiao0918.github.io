---
title: 📣 手撕代码 & 项目题全掌握：前端
tags:
  - web
---

### 🍕数组扁平化

#### ES6 的 flat 方法

> flat() 方法将以指定的深度递归遍历数组，并将所有元素与遍历的子数组中的元素合并到一个新数组中以返回。

```js
const arr = [1, [2, [3, [4, 5]]], 6]
//数组自带的扁平化方法,flat的参数代表的是需要展开几层。Infinity不管嵌套几层，全部都展开
console.log(arr.flat(Infinity))
// [1, 2, 3, 4, 5, 6]
```

#### 使用正则

> 1. 首先是使用 JSON.stringify 把 arr 转为字符串
> 2. 接着使用正则把字符串里面的 [ 和 ] 去掉(`/\[|\]/g`,`g`是全局匹配)
> 3. 然后再拼接数组括号转为数组对象

```js
const arr = [1, [2, [3, [4, 5]]], 6]
const res = JSON.stringify(arr).replace(/\[|\]/g, "")
const res2 = JSON.parse("[" + res + "]")
console.log(res2) //[1, 2, 3, 4, 5, 6]
```

#### 使用递归

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

#### 使用 reduce

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

#### 使用栈的思想实现 flat 函数

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

### 🍔深拷贝和浅拷贝

### 🍟防抖节流

#### 防抖

#### 节流

### 🌭for...in和for...of

### 🍿从数组 [1,2,3,4,5,6] 中找出值为 2 的元素

#### filter()

> filter() 方法创建一个新数组, 包含通过所提供函数实现的测试的所有元素。

```js
function isBigEnough(element) {
  return element == 2
}
let filtered = [1, 2, 3, 4, 5, 6].filter(isBigEnough)
console.log(filtered) //[2]
```

#### find()

> find() 方法返回数组中满足提供的测试函数的第一个元素的值。否则返回undefined。

```js
let arr = [1, 2, 3, 4, 5, 6]
function finds(x) {
  return x == "2"
}
console.log(arr.find(finds)) //2
```

### 🧂两栏布局和三栏布局

### 🥓实现多行文本溢出的省略

### 🍳用 CSS 实现三角符号

### 🫔实现九宫格布局

### 🍚单行多行文字隐藏显示省略号

### 🍤函数柯里化

### 🎂图片懒加载

### 🍬bind、apply、call 的用法

### 🍭手写 new

### 🍻promise 的用法

### 🧁解析 URL

### 🍣音乐播放器

### 🥘组件通信

### 🍲购物车

### 🍨Mock 模拟数据

### 🍡轮播图

### 🍹路由跳转

### 🧋登录注册

### 🍩井字棋
