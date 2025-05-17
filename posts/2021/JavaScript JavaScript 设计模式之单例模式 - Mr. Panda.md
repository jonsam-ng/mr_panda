---
date: 2022-01-01
title: JavaScript 设计模式之单例模式 - Mr. Panda
# category: 
# tags: 
# description:
---

# [JavaScript] JavaScript 设计模式之单例模式 - Mr. Panda

---
## 简介

单例模式又叫单体模式，是设计模式中最常见的一种。单例模式是为了保证我们在创建对象的时候，始终使用的都是同一个对象实例。这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。

## 使用场景

-   在整个应用中使用同一个对象实例

## 为什么使用

-   防止对象被频繁创建，浪费内存。

## 如何使用

-   实例私有化，只提供对外访问的方法。

## 代码案例

我们可以通过闭包保证实例是唯一且不可修改的。

闭包是一种保护私有变量的机制，在函数执行时形成私有的作用域，保护里面的私有变量不受外界干扰。直观的说就是形成一个不销毁的栈环境。

```javascript
class SDK {} const getSdk = (() => { let sdk = null; return () => { if(sdk === null) { sdk = new SDK(); } return sdk; } })() const sdk1 = getSdk(); const sdk2 = getSdk(); console.log(sdk1 === sdk2);
```

执行结果：

```javascript
>>> true
```

如果您使用了 ESM 和 TS，您也可以通过 ES 的模块化保证实例只可读取而不可更改，同时通过ts 的 private 关键字保证实例的私有。

```javascript
class SDK {} class SDKWrapper{ private sdk = null; constructor() { this.sdk = new SDK(); } getSDK() { return sdk; } } const sdkWrapper = new SDKWrapper(); const SdkInstance = sdkWrapper.getSDK(); export default SdkInstance;
```

参考资料：

-   [单例模式-菜鸟教程](https://www.runoob.com/design-pattern/singleton-pattern.html)
-   [javascript 闭包-菜鸟教程](https://www.runoob.com/js/js-function-closures.html)

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
