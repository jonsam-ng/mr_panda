---
date: 2022-01-01
title: Js Tricks 系列 (1) - Mr. Panda
# category: 
# tags: 
# description:
---

# [JavaScript] Js Tricks 系列 (1) - Mr. Panda

---
这是一篇分享 javascript 小技巧的系列文章，文章会持续的更新。此系列主要面向 Javascrip t新手和中级开发者。希望每个读者都能至少从中学到一个有用的技巧。能够为大家提供这些简短而实用的 JavaScript 技巧来提高大家编程能力，这对于我来说是件很开心的事。技巧虽好、重在掌握并使用起来！

## 通过 generator 生成 ID

```javascript
function* customerIDGenerator(){ let i = 0; while(true){ yield i++; } } const gengerator = customerIDGenerator(); console.info(gengerator.next().value); // 0 console.info(gengerator.next().value); // 1
```

通过生成器生成独一无二的id，就不需要定义全局变量了。

## 有缩进的 Json 格式化

```javascript
const student = { name: 'john', age: 18, sex: 1 }; JSON.stringify(student,null,2); // "{\n \"name\": \"john\",\n \"age\": 18,\n \"sex\": 1\n}" console.log(JSON.stringify({ alpha: 'A', beta: 'B' }, null, '\t'));
```

参考：[MDN：JSON.stringify()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)；

## 以对象的方式解构数组

```javascript
const student = ['john', 20]; const {1: age} = student; console.info(age); // 20
```

## nums.map(Number)

```javascript
[1, '2', [3, '4']].map(Number); // [1, 2, NaN] [1, '2', ['3']].map(Number); // [1, 2, 3] [1, '2', [['4']]].map(Number); // [1, 2, 4]
```

## 测量代码执行时间

```javascript
console.time('timer'); for (let i = 0; i < 1e6; i++) { } console.timeEnd('timer'); // timer: 15863.784912109375 ms
```

也可以使用 performance ，这个更加精确。

```javascript
const firstTime = performance.now(); for (let i = 0; i < 1e6; i++) { } const secondTime = performance.now(); console.log(The something function took ${secondTime - firstTime} milliseconds.); // The something function took 2.300000011920929 milliseconds.
```

## 数组结构交换变量的值

```javascript
let a = 1, b = 2 [a, b] = [b, a] console.log(a) // result -> 2 console.log(b) // result -> 1
```

## 函数 arguments 转为数组

```javascript
function func() { const argArray = Array.prototype.slice.call(arguments); console.log("==>", arguments, argArray); } // ==> Arguments(2) [1, "john", callee: ƒ, Symbol(Symbol.iterator): ƒ]0: 11: "john"callee: ƒ func()length: 2Symbol(Symbol.iterator): ƒ values()[[Prototype]]: Object (2) [1, "john"] func(1, 'john');
```

## event.target.valueAsNumber

通过 event.target.value 可以取出事件中的值，在输入框等场景中通常这个值是 string 类型。valueAsNumber 则可以取出数字。

![3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1595935455526/Tv1sEFRxe.png?auto=compress,format&format=webp)

## 获取链接的参数（Query Params）

```javascript
let keyword = new URLSearchParams(location.search).get('q'); // "js tricks"
```

## Float 转 Integer

通过 Math.floor()、Math.ceil()、Math.round() 等可以将 float 类型数据转为 integer ，当然你也可以使用下面的简便方法：

```javascript
'123' | 0 // 123 '123.45' | 0 // 123 '123.45f' | 0 // 0 'f35' | 0 // 0 123.345 | 0 // 123
```

通过这种方法也可以用来解析数字：

```javascript
console.log(1553 / 10 | 0) // Result: 155 console.log(1553 / 100 | 0) // Result: 15 console.log(1553 / 1000 | 0) // Result: 1
```

## void operator

**`void` 运算符** 对给定的表达式进行求值，然后返回 [`undefined`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)。void 返回 undefined 的原始值，他比 undefined 更安全。一般简写 void(0)。

```javascript
document.body.onclick = void (() => alert(0)) // 没有弹窗 document.body.onclick = () => alert(void 0) // 弹窗：undefined
```

详见：[MDN：void 运算符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/void)

## event.target 和 event.currentTarget

event.target 表示触发事件的 DOM 元素，event.currentTarget 表示监听事件的 DOM 元素。

```javascript
const list = document.querySelector(".todo-list"); list.addEventListener("click", e => { console.log(e.target); // li console.log(e.currentTarget); // ul });
```

## Curry Function

什么是 curry function？Currying is the process of turning a function with multiple arity into a function with less arity。

通过 Curry Function 可以多参数函数 pipe 化，增加代码可复用性。

通过的两层 curry 函数：

```javascript
const curry = (fn, ...args) => (..._arg) => fn(...args, ..._arg);
```

参考：[Understanding Currying in JavaScript](https://blog.bitsrc.io/understanding-currying-in-javascript-ceb2188c339)；

## Reducing Pipelines

通过 reduce 可以达到管道的效果，下面这个例子展示了 class interfaces 的效果。

```javascript
const protocols = (...ps) => ps.reduce((c, p) => p(c), Object); const Mappable = (klass) => { return class extends klass { map() { throw 'Not implemented'; } }; }; const Foldable = (klass) => { return class extends klass { fold() { throw 'Not implemented'; } }; }; class NaturalNumbers extends protocols(Mappable, Foldable) { constructor() { super(); this.elements = [1,2,3,4,5,6,7,8,9]; } map (f) { return this.elements.map(f); } fold (f) { return this.elements.reduce(f, this.elements, 0); } }
```

下面的代码实现一个 map pipeline:

```javascript
const map = f => o => o.map(f); // map helper const fold = f => o => o.fold(f); const compose = (...fns) => fns.reduce((acc, f) => (x) => acc(f(x)), id); // pineline const plus1 = x => x + 1; // mapper const div5 = x => x / 5; // mapper const plus_then_div = compose(map(div5), map(plus1)); // map pipeline console.log(plus_then_div(new NaturalNumbers)); // NaturalNumbers is mappable // => [ 0.4, 0.6, 0.8, 1, 1.2, 1.4, 1.6, 1.8, 2 ]
```

参考：[Protocols for the Brave](https://www.jstips.co/en/javascript/protocols-for-the-brave/)；

## in operator & hasOwnProperty & pure object

in operator: The **`in` operator** returns `true` if the specified property is in the specified object or its prototype chain.

```javascript
const student = {name: 'john', age: 18} 'name' in student // true 'toString' in student // true student.hasOwnProperty('name') // true student.hasOwnProperty('toString') // false Object.keys(student) // ["name", "age"] Object.getOwnPropertyNames(student) // ["name", "age"] Reflect.get(student, 'name') // "john" Reflect.get(student, 'toString') // ƒ toString() { [native code] } Reflect.has(student, 'name') // true Reflect.has(student, 'toString') // true const today = new Date() 'getDay' in today // true const arr = new Array(1,2,3) '0' in arr // true
```

在使用 in 和 Reflect.get 时，是包含原型链上的属性的。那么怎么创建一个 pure object 呢？

```javascript
const pureObj = Object.create(null) 'toString' in pureObj // false
```

参考：

-   [获取对象属性（所有属性、可](https://www.cnblogs.com/kunmomo/p/12005794.html)[枚举](https://www.cnblogs.com/kunmomo/p/12005794.html)[、不可枚举、自身属性【非原型链继承】）个数详解](https://www.cnblogs.com/kunmomo/p/12005794.html);
-   [MDN: in operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/in);

## Private Class

创建具有私有属性、私有方法的 class 对象除了借助 typescript 的 private 关键字之外，还可以使用 js 原生 '#' hash 语法。

参见：[MDN:Private class features](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields);

## Object Static Porperties

[`Object.assign()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)

Copies the values of all enumerable own properties from one or more source objects to a target object.

[`Object.create()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)

Creates a new object with the specified prototype object and properties.

[`Object.defineProperty()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

Adds the named property described by a given descriptor to an object.

[`Object.defineProperties()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties)

Adds the named properties described by the given descriptors to an object.

[`Object.entries()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/entries)

Returns an array containing all of the `[key, value]` pairs of a given object's **own** enumerable string properties.

[`Object.freeze()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)

Freezes an object. Other code cannot delete or change its properties.

[`Object.fromEntries()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/fromEntries)

Returns a new object from an iterable of `[key, value]` pairs. (This is the reverse of [`Object.entries`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/entries)).

[`Object.getOwnPropertyDescriptor()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor)

Returns a property descriptor for a named property on an object.

[`Object.getOwnPropertyDescriptors()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptors)

Returns an object containing all own property descriptors for an object.

[`Object.getOwnPropertyNames()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyNames)

Returns an array containing the names of all of the given object's **own** enumerable and non-enumerable properties.

[`Object.getOwnPropertySymbols()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertySymbols)

Returns an array of all symbol properties found directly upon a given object.

[`Object.getPrototypeOf()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf)

Returns the prototype (internal `[[Prototype]]` property) of the specified object.

[`Object.is()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)

Compares if two values are the same value. Equates all `NaN` values (which differs from both Abstract Equality Comparison and Strict Equality Comparison).

[`Object.isExtensible()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/isExtensible)

Determines if extending of an object is allowed.

[`Object.isFrozen()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/isFrozen)

Determines if an object was frozen.

[`Object.isSealed()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/isSealed)

Determines if an object is sealed.

[`Object.keys()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)

Returns an array containing the names of all of the given object's **own** enumerable string properties.

[`Object.preventExtensions()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/preventExtensions)

Prevents any extensions of an object.

[`Object.seal()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/seal)

Prevents other code from deleting properties of an object.

[`Object.setPrototypeOf()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf)

Sets the object's prototype (its internal `[[Prototype]]` property).

[`Object.values()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/values)

Returns an array containing the values that correspond to all of a given object's **own** enumerable string properties.

## \=== & Object.is()

![differences of operators in equality comparisons javascript](https://i.imgur.com/pCyqkLc.png)

## Tap Debugger

在 debugger 时，需要打印复杂运算的情况下可能要专门把复杂的运算拆开进行打印。实际上，我们可以借助 tap debugger 来解决这个问题：

```javascript
const tap = (x, fn = x => x, type = 'log') => (console[type](fn(x)), x); const someCumpute = (x) => tap((x+ 5 * 10) / 2) + 12; someCumpute(1) // 25.5 \n 37.5 tap(new Array(5)).fill(new Date().getTime()) // (5) [empty × 5] \n (5) [1630720420652, 1630720420652, 1630720420652, 1630720420652, 1630720420652] tap(new Array(5).fill(new Date().getTime())) // (5) [1630720496925, 1630720496925, 1630720496925, 1630720496925, 1630720496925] \n (5) [1630720496925, 1630720496925, 1630720496925, 1630720496925, 1630720496925] tap(new Array(5).fill(new Date().getTime()), x => x.map(x=>x*2), 'table') // 打印表格[3261441973368,3261441973368,3261441973368,3261441973368,3261441973368] \n Array(5) (5) [1630720986684, 1630720986684, 1630720986684, 1630720986684, 1630720986684]
```

## 迭代空数组

map 会跳过数组中的 empty 值，new Array() 初始化的数组中都是 empty 值。

```javascript
Array.apply(null, new Array(4)).map((v,i)=> i) // [0, 1, 2, 3]
```

## 函数传空值

```javascript
[...['parameter1', , 'parameter3']] // ["parameter1", undefined, "parameter3"] someFunction(...[,,arg]) // 省略 undefined 传值的语法糖
```

## 用 some 模拟可以 break 的 forEach

forEach 循环是不可以 break 的，因为他的每次循环都期待一个结果。但是我们可以用 some(或者 every) 来模拟可以 break 的 forEach 循环，因为 some 在得到结果时可以提前跳出循环。

```javascript
[0, 1, 2, 3, 4].some(function (val, i) { if (val === 2) return true; // break console.log(val); }); // 0, 1
```

## Comma operator

```javascript
function a(){console.log('a'); return 'a';} function b(){console.log('b'); return 'b';} function c(){console.log('c'); return 'c';} var x = (a(), b(), c()); console.log(x); // a b c c
```

逗号运算从左到右依次执行，并返回最右边的值。

## Clipboard API

[`Clipboard`](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard) 

Provides an interface for reading and writing text and data to or from the system clipboard. The specification refers to this as the 'Async Clipboard API.

'[`ClipboardEvent`](https://developer.mozilla.org/en-US/docs/Web/API/ClipboardEvent) 

Represents events providing information related to modification of the clipboard, that is [`cut`](https://developer.mozilla.org/en-US/docs/Web/API/Element/cut_event), [`copy`](https://developer.mozilla.org/en-US/docs/Web/API/Element/copy_event), and [`paste`](https://developer.mozilla.org/en-US/docs/Web/API/Element/paste_event) events. The specification refers to this as the 'Clipboard Event API'.

[`ClipboardItem`](https://developer.mozilla.org/en-US/docs/Web/API/ClipboardItem) 

Represents a single item format, used when reading or writing data.

参考：[MDN:Clipboard API](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard_API);

## 获取后缀

```javascript
// 方法 1 (/[.]/.exec(filename)) ? /[^.]+$/.exec(filename)[0] : undefined; // 方法 2 filename.split('.').pop(); // 方法 3 filename.slice((filename.lastIndexOf(".") - 1 >>> 0) + 2);
```

方法 3 中当没有 ‘.’ 时，-2 >>> 0 +2 等于 4294967296，也就是说这种方法默认字符串的长度是小于 4294967296 的，这在获取文件后缀的场景中是符合常理的。

## class 以函数的方式调用

我们在写工具类的时候喜欢用面向对象的 class 来写，但是使用的时候还是要用 new 关键词来实例化，那么能否以函数的方式来自动实例化对象呢？我们可以尝试用 Proxy 来模拟一下 moment 的写法。

```javascript
class Moment { constructor(date) { this.moment = date.getTime(); } format() { console.log(this.moment); } } const moment = new Proxy(Moment, { apply(target, thisArg, argumentsList) { return new target(...argumentsList); } }); moment(new Date()).format() // 1630729320724 new moment(new Date()).format() // 1630729351978 new Moment(new Date()).format() // 1630729454607 new Moment(new Date()) // Moment {moment: 1630729518707}moment: 1630729518707[[Prototype]]: Object 返回 Moment 实例 new moment(new Date()) // Moment {moment: 1630729518707}moment: 1630729518707[[Prototype]]: Object 返回 Moment 实例 moment(new Date()) // Moment {moment: 1630729518707}moment: 1630729518707[[Prototype]]: Object 返回 Moment 实例
```

可以看到无论是调用 new Moment() 还是 new moment() 还是 monent() 返回的都是 Moment 对象的实例。这样就解决了工具类调用的便捷性问题，同时也避免了使用 function 去模拟面向对象的麻烦。

## 简易的事件监听器

对于addEventListener() 而言，如果是命名监听就可以自动 removeEventListener，否则则需要手动的 removeEventListener，这时就需要将 handle 记住。现在我们应用上一条的知识来写一个简易的时间监听器，调用 listener.destory() 则销毁监听器。

```javascript
class EventListener { constructor(e, params = {}) { const { el = document.documentElement, cb = () => {}, useCapture = true, } = params; this.e = e; this.params = { el, cb, useCapture }; el.addEventListener(e, cb, useCapture); } destroy() { const { el, cb, useCapture } = this.params; return el.removeEventListener(this.e, cb, useCapture); } } const eventListener = new Proxy(EventListener, { apply: (target, thisArg, argumentsList) => new target(...argumentsList), }); const listener = eventListener('click', {cb: () => console.log('clicked!'), el: document.querySelector('.breadcrumbs')}) // add listener listener.destory(); // remove listener
```

## 获取 timestamp 和 unix timestamp

```javascript
new Date() // 1630733887621 Math.floor(+new Date() / 1000) //1630734001
```

## Reducer Pipeline

```javascript
const reducers = { totalInDollar: function(state, item) { // specific statements… return state.dollars += item.price; }, totalInEuros : function(state, item) { return state.euros += item.price * 0.897424392; }, totalInPounds : function(state, item) { return state.pounds += item.price * 0.692688671; }, totalInYen : function(state, item) { return state.yens += item.price * 113.852; } // more… }; const createReducerPipeLine = function (reducers) { return function (state, item) { return Object.keys(reducers).reduce( function (nextState, key) { reducers[key](state, item); return state; }, {} ); } }; const reducerPipeline = createReducerPipeLine(reducers);
```

这种方式可以将 reducers 包装为 reducerPipeline，总体来看是将小的状态机排列起来生成一个状态机。

## Document.readyState

<table><tbody><tr><td>readyState</td><td>usage</td></tr><tr><td>loading</td><td>The&nbsp;<a href="https://developer.mozilla.org/en-US/docs/Web/API/Document"><code>document</code></a>&nbsp;is still loading.</td></tr><tr><td>interactive</td><td>The document has finished loading and the document has been parsed but sub-resources such as scripts, images, stylesheets and frames are still loading.</td></tr><tr><td>complete</td><td>The document and all sub-resources have finished loading. The state indicates that the&nbsp;<code><a href="https://developer.mozilla.org/en-US/docs/Web/API/Window/load_event">load</a></code>&nbsp;event is about to fire.</td></tr></tbody></table>

readyState api

参考：[MDN:Document.readyState](https://developer.mozilla.org/en-US/docs/Web/API/Document/readyState);

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
