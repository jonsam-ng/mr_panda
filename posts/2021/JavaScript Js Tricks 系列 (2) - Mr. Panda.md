---
date: 2022-01-01
title: Js Tricks 系列 (2) - Mr. Panda
# category: 
# tags: 
# description:
---

# [JavaScript] Js Tricks 系列 (2) - Mr. Panda

---
这是一篇分享 javascript 小技巧的系列文章，文章会持续的更新。此系列主要面向 Javascrip t新手和中级开发者。希望每个读者都能至少从中学到一个有用的技巧。能够为大家提供这些简短而实用的 JavaScript 技巧来提高大家编程能力，这对于我来说是件很开心的事。技巧虽好、重在掌握并使用起来！

## 数组求中位数

```javascript
const values = [2, 56, 3, 41, 0, 4, 100, 23];
values.sort((a, b) => a - b); // sort 改变原数组
const median = (values[(values.length - 1) >> 1] + values[values.length >> 1]) / 2; // 13.5
```

## 二维数组 flat

```javascript
const arr = [].concat.apply([], myArray) const arr = myArray.reduce(function(prev, curr) { return prev.concat(curr); }) // concat 可以接受数组或者值 const arr = [].concat(...myArray) // concat 可以接受多个参数 const arr = myArray.flat() // ES10
```

## 生成数组序列数

```javascript
Array.from(new Array(4),(val,index)=>index); // [0, 1, 2, 3] Array.from(new Array(4).keys()); // [0, 1, 2, 3]
```

## 各浏览器循环对象的顺序不一致

各浏览器针对 for in 循环，以及 Object.keys 的顺序是不一致的，是无法保证一致的，如果在循环时需要保证顺序，一般看来有如下两种方法。

-   先自行排序。
-   使用 map 或者 array 代替 object，array 的顺序取决于下标，map 的顺序取决于加入 map 的时间。

## Memoization Function

在很多递归和迭代的运算中，有很多运算是重复和不必要的，其实可以通过空间换时间的方法，给之前的运算结果加一个缓存，计算之前检查缓存中是否已经计算过结果，否则才计算并将计算结果加入缓存。需要注意的时，我们需要控制缓存的大小，可以参考操作系统的缓存维护的算法。

```javascript
function memoize(func) { var memo = {}; var slice = Array.prototype.slice; return function() { var args = slice.call(arguments); if (args in memo) return memo[args]; else return (memo[args] = func.apply(this, args)); } }
```

缓存实际上是键值对的存储，其中很重要的算法也包含检索算法，可以通过优化检索算法和缓存替换算法来优化缓存的效率。

参考：[Implementing Memoization in JavaScript](https://www.sitepoint.com/implementing-memoization-in-javascript/)；

## 数组去重

-   \[...new Set(arr)\]
-   Array.from(new Set(arr))
-   arr.filter((x,i)=>arr.lastIndexOf(x) === i)

## 更安全的字符串连接

我们通常使用 + 进行字符串连接，但是加法除了字符串连接还可以是数字相加，导致出现意想不到的错误。我们可以使用下面两种方法进行更为安全的字符串连接：

-   模板字符串：\`${str1}${atr2}\`。
-   concat：''.concat(str1, str2)，推荐，将字符串当做字符数组来处理。

## 数字取整

-   Math.trunc(num):支持数字和字符串，NaN 的结果是 NaN。
-   ~~num:按位取反的 hack，按位取反可以理解为加 1 后取整变反。这个方法支持数字和字符串，NaN 的结果是 0。相比于 Math.trunc，位运算速度更快，但是语义化不是很好，建议在工具类应用中使用，而应用类的则使用语义化更好的 Math.trunc()。
-   更多位运算的技巧参考：[React 源码阅读----位运算初探](https://source.jonsam.site/react/summary/bitOperation.html)；

## 向不可传参的函数传参

有些 callback 函数是不接受传参的，我们可以采用这样的方式向函数传参。这种方法可以使回调函数的使用更为灵活。

```javascript
function addEventListener(cb) { cb(); } const cb = (type) => { console.log(type); }; addEventListener(cb.bind(this, 'click'));
```

## includes() 的 hack 写法

-   String.prototype.includes() 常用来判断数组（包括字符数组）中是否包含某一个项，这种写法兼容性不够好，不支持 ie 浏览器。
-   !!(~array.indexOf(searchWord))：通过 indexOf 和 ~ 来 hack includes 的功能，这种写法兼容性较好，可以支持 ie 浏览器。原理是 -1 经过按位取反运算是 0 ，转为布尔值刚好为 false，而其他 0-N 的下标按位取反后为-(N+1) 至 -1，转换为布尔值刚好是 true。这是 express 源码中曾使用的 hack 写法。

```javascript
!!~'fhkakfjkjavasript'.indexOf('java') // true !!~'fhkakfjkjavasript'.indexOf('javass') // false !!~[1,2,3].indexOf(1) // true !!~[1,2,3].indexOf(4) // false
```

## 必填参数

```javascript
const _err = (param) => {throw new (${param} is not defined)} const getSum = (a = _err('a'), b = _err('b')) => a + b; getSum(1) // Uncaught Error: b is not defined
```

## 伪数组转数组

函数参数列表、Dom Node List 等都是伪数组，伪数组可通过如下方法转成数组：

```javascript
const fakeArr = {0:'a',1:'b',length:2}; Array.from(fakeArr); // ["a", "b"] Array.apply(null, fakeArr); // ["a", "b"] Array.prototype.slice.call(fakeArr); // ["a", "b"]
```

## 以数组的方式处理参数

在写工具函数时，传参的灵活性可以极大的提高工具的易用性。以数组的方式处理参数允许用户传入单一元素或者数组，内部可以做统一的逻辑处理。

```javascript
function printUpperCase(words) { var elements = [].concat(words || []); for (var i = 0; i < elements.length; i++) { console.log(elements[i].toUpperCase()); } } printUpperCase("cactus"); printUpperCase(["cactus", "bear", "potato"]);
```

## switch(true) 嵌套条件语句

通过 switch(true) case 语句可以传入不同的 conditions，这样写比嵌套的 if else 语句思路更为清晰。

```javascript
switch(true) { case (typeof color === 'string' && color === 'black'): printBlackBackground(); break; case (typeof color === 'string' && color === 'red'): printRedBackground(); break; case (typeof color === 'string' && color === 'blue'): printBlueBackground(); break; case (typeof color === 'string' && color === 'green'): printGreenBackground(); break; case (typeof color === 'string' && color === 'yellow'): printYellowBackground(); break; }
```

## 生成随机字母和数字

```javascript
// 生成指定范围随机数 const x = Math.floor(Math.random() * (max - min + 1)) + min; // 生成一个随机小写字母或者数字 const genSeed = () => (~~(Math.random()*36)).toString(35); // "s" // 生成一个随机大写或者小写字母或者数字 const genSeedX = () => { const seed = genSeed(); return Math.random() < 0.5 ? seed.toUpperCase() : seed }; // "aIby9" function randomStringGenerator(length){ let s = ''; while(s.length < length) s += genSeed(); return s; } randomStringGenerator(5); // "6muky"
```

-   toString() 中所传的进制范围是2-36。
-   (0-35).toString(35) 是 0-9-a-z。

## 浮点计算的问题

```javascript
0.1 + 0.2 === 0.3 // is false，0.30000000000000004 !== 0.3 可以通过使用 toFixed() 和 toPrecision() 来解决这个问题。浮点运算一定要在确定的精度上才能判等。
```

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
