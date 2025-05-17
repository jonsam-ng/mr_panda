---
date: 2022-01-01
title: JavaScript Symbol 详解 - Mr. Panda
# category: 
# tags: 
# description:
---

# [JavaScript] JavaScript Symbol 详解 - Mr. Panda

---
这篇文章对 javascript 中 symbol 进行了详细的解读，内容包含 symbol 的含义、语法、应用场景、内置 symbol、typescript 中的 symbol 等内容。文章重点是 symbol 的使用场景、symbol 的内置对象的使用、symbol 在 typescript 中的类型推断等。建议结合参考资料学习和理解。

## 什么是 Symbol?

**symbol** 是一种基本数据类型 （[primitive data type](https://developer.mozilla.org/zh-CN/docs/Glossary/Primitive)）。`Symbol()`函数会返回**symbol**类型的值，该类型具有静态属性和静态方法。它的**静态属性会暴露几个内建的成员对象**；它的静态方法会暴露**全局的symbol注册**，且类似于内建对象类，但作为构造函数来说它并不完整，因为它不支持语法："`new Symbol()`"。

**每个从`Symbol()`返回的symbol值都是唯一的。**一个symbol值能作为对象属性的标识符；这是该数据类型仅有的目的。

## Symbol 的应用场景

### 创建匿名的对象属性

数据类型 “**symbol**” 是一种基本数据类型，该类型的性质在于这个类型的值可以用来创建匿名的对象属性。该数据类型通常被用作一个对象属性的键值——当你想让它是私有的时候。例如，**symbol** 类型的键存在于各种内置的 JavaScript 对象中。同样，自定义类也可以这样创建私有成员。

当一个 symbol 类型的值在属性赋值语句中被用作标识符，该属性（像这个 symbol 一样）是匿名的；并且是不可枚举的。因为这个属性是不可枚举的，它不会在循环结构 “`for( ... in ...)`” 中作为成员出现，也因为这个属性是匿名的，它同样不会出现在 “`Object.getOwnPropertyNames()`” 的返回数组里。这个属性可以通过创建时的原始 symbol 值访问到，或者通过遍历 “`Object.getOwnPropertySymbols()`” 返回的数组。

> **为什么字符串对象属性不是匿名的？** 
> 
> 传统方式下以字符串作为对象属性的键。这样，只要能得到这个字符串在内存中的地址，就可以访问该属性。由于同一个字符串只会在常量区生成一次，因此在任何时候通过“getName”这个字符串得到的内存中的地址是一样的。

### 消除“魔术字符串”

魔术字符串指的是，在代码之中多次出现、与代码形成强耦合的某一个具体的字符串或者数值。风格良好的代码，应该尽量消除魔术字符串，改由含义清晰的变量代替。

```javascript
const shapeType = {
  triangle: Symbol(),
};

switch (shape) {
  case shapeType.triangle: //消除了一个魔术字符串
    area = 0.5 * options.width * options.height;
    break;
  /* ... more code ... */
}
```

单例模式中往往将实例挂载到 globalThis（web 中是 window，node 中是 global） 上，如果只是普通的 key，可能会有被覆盖的风险，使用 Symbol 作为 key 可以保证实例不会被覆盖，同时可以保证 key 值在 globalThis 上的私有性，防止外界篡改。

```javascript
// mod.js
const FOO_KEY = Symbol.for("foo");

function A() {
  this.foo = "hello";
}

if (!global[FOO_KEY]) {
  global[FOO_KEY] = new A();
}

module.exports = global[FOO_KEY];
```

除了用户定义的symbols，还有一些内置symbols。 内置symbols用来表示语言内部的行为。

以下为这些symbols的列表：

-   **Symbol.hasInstance** 方法，会被instanceof运算符调用。构造器对象用来识别一个对象是否是其实例。
-   **Symbol.isConcatSpreadable** 布尔值，表示当在一个对象上调用Array.prototype.concat时，这个对象的数组元素是否可展开。
-   **Symbol.iterator** 方法，被for-of语句调用。返回对象的默认迭代器。
-   **Symbol.match** 方法，被String.prototype.match调用。正则表达式用来匹配字符串。
-   **Symbol.replace** 方法，被String.prototype.replace调用。正则表达式用来替换字符串中匹配的子串。
-   **Symbol.search** 方法，被String.prototype.search调用。正则表达式返回被匹配部分在字符串中的索引。
-   **Symbol.species** 函数值，为一个构造函数。用来创建派生对象。
-   **Symbol.split** 方法，被String.prototype.split调用。正则表达式来用分割字符串。
-   **Symbol.toPrimitive** 方法，被ToPrimitive抽象操作调用。把对象转换为相应的原始值。
-   **Symbol.toStringTag** 方法，被内置方法Object.prototype.toString调用。返回创建对象时默认的字符串描述。
-   **Symbol.unscopables** 对象，它自己拥有的属性会被with作用域排除在外。

下面将详细讲解这些内置的 Symbols。

参见：[阮一峰：ECMAScript 6 入门 - Symbol](https://es6.ruanyifeng.com/#docs/symbol)；

### Symbol.toPrimitive 和 Symbol.toStringTag

Symbol.toPrimitive 定义一个对象怎么转化为原始类型，Symbol.toStringTag 定义对象怎么实现 toString 方法。下面的代码自定义对象类型，并且自定义对象转化为原始类型的方法和 toString 方法。

```javascript
class Item {
   #item;
 constructor(item) {
     if (item?.constructor !== Number) throw new TypeError();
     this.#item = item.valueOf();
   }
 Symbol.toPrimitive {
     // hint can be "number", "string", and "default" 
     switch (hint) {
       case 'number': 
         return this.#item;
       case 'string': 
       case 'default': 
         return Item: ${this.#item};
       default:
         return null;
     }
   }
 get [Symbol.toStringTag]() {
     return this.constructor.name;
   }
 }
 const item = new Item(42);
 console.log(Number(item));
 console.log(String(item));
 console.log(item.toString());
 console.log(item);
 /* Output:
 42
 Item: 42
 [object Item]
 Item {}
 */
```

### Symbol.for()

如果使用同一个 Symbol 值，可以使用`Symbol.for()`方法。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的 Symbol 值。如果有，就返回这个 Symbol 值，否则就新建一个以该字符串为名称的 Symbol 值，并将其注册到全局。

```javascript
Symbol.for("bar") === Symbol.for("bar"); // true
Symbol("bar") === Symbol("bar"); // false
```

### Symbol.keyFor()

`Symbol.keyFor()`方法返回一个已登记的 Symbol 类型值的`key`。

```javascript
let s1 = Symbol.for("foo");
Symbol.keyFor(s1); // "foo"

let s2 = Symbol("foo");
Symbol.keyFor(s2); // undefined
```

## Symbols in TypeScript

symbol 在 typescript 中的类型是 **_symbol_**，子类型 _unique symbol_ 与具体的 symbol 强关联，通过 typeof 可以获取 _unique symbol_ 的类型。

```typescript
const PROD: unique symbol = Symbol("Production mode");
const DEV: unique symbol = Symbol("Development mode");

function showWarning(msg: string, mode: typeof DEV | typeof PROD) {
  // ...
}
```

symbols 可以重建 enum 在运行时的行为。enums 在 typescript 中是不透明的，因此不能给 enums 设置 string 值，因为 ts 会将这些值视为独一无二的。

```typescript
enum Colors {
  Red = "Red",
  Green = "Green",
  Blue = "Blue",
}

const c1: Colors = Colors.Red;
const c2: Colors = "Red"; // 💣 No direct assigment possible

enum Moods {
  Happy = "Happy",
  Blue = "Blue",
}

// 💣 This condition will always return 'false' since the
// types 'Moods.Blue' and 'Colors.Blue' have no overlap.
if (Moods.Blue === Colors.Blue) {
  // Nope
}
```

在 js 中，可以通过 symbols 创建 enum。

```javascript
// All Color symbols
const COLOR_RED: unique symbol = Symbol("RED");
const COLOR_ORANGE: unique symbol = Symbol("ORANGE");
const COLOR_YELLOW: unique symbol = Symbol("YELLOW");
const COLOR_GREEN: unique symbol = Symbol("GREEN");
const COLOR_BLUE: unique symbol = Symbol("BLUE");
const COLOR_INDIGO: unique symbol = Symbol("INDIGO");
const COLOR_VIOLET: unique symbol = Symbol("VIOLET");
const COLOR_BLACK: unique symbol = Symbol("BLACK");

// All colors except Black
const Colors = {
  COLOR_RED,
  COLOR_ORANGE,
  COLOR_YELLOW,
  COLOR_GREEN,
  COLOR_BLUE,
  COLOR_INDIGO,
  COLOR_VIOLET,
} as const;

function getHexValue(color) {
  switch (color) {
    case Colors.COLOR_RED:
      return "#ff0000";
    //...
  }
}
```

1.  symbols 标注为 unique symbol，表示不可修改。
2.  enum 标注为 const，表示在 enum 中定义的 symbols 将会被 ts 保护。ts 的推断范围将会从 symbol 限制为具体的 symbol 类型。

可以通过函数类型声明更加安全的获取 enum 中 symbol 的类型。

```typescript
type ValuesWithKeys<T, K extends keyof T> = T[K];
type Values<T> = ValuesWithKeys<T, keyof T>;

function getHexValue(color: Values<typeof Colors>) {
  switch (color) {
    case COLOR_RED:
    // super fine, is in our type
    case Colors.COLOR_BLUE:
      // also super fine, is in our type
      break;
    case COLOR_BLACK:
      // what? What is this??? TypeScript errors 💥
      break;
  }
}
```

```typescript
const ColorEnum = {
  [COLOR_RED]: COLOR_RED,
  [COLOR_YELLOW]: COLOR_YELLOW,
  [COLOR_ORANGE]: COLOR_ORANGE,
  [COLOR_GREEN]: COLOR_GREEN,
  [COLOR_BLUE]: COLOR_BLUE,
  [COLOR_INDIGO]: COLOR_INDIGO,
  [COLOR_VIOLET]: COLOR_VIOLET,
};

function getHexValueWithSymbolKeys(color: keyof typeof ColorEnum) {
  switch (color) {
    case ColorEnum[COLOR_BLUE]:
      // 👍
      break;
    case COLOR_RED:
      // 👍
      break;
    case COLOR_BLACK:
      // 💥
      break;
  }
}
```

-   MDN: [术语表 → Symbol](https://developer.mozilla.org/zh-CN/docs/Glossary/Symbol)；
-   MDN: [Symbol](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol)；
-   [阮一峰：ECMAScript 6 入门 - Symbol](https://es6.ruanyifeng.com/#docs/symbol)；
-   [Symbols in JavaScript and TypeScript](https://fettblog.eu/symbols-in-javascript-and-typescript/)；

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
