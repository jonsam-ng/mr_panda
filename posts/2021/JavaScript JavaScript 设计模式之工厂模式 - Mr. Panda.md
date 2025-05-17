---
date: 2022-01-01
title: JavaScript 设计模式之工厂模式 - Mr. Panda
# category: 
# tags: 
# description:
---

# [JavaScript] JavaScript 设计模式之工厂模式 - Mr. Panda

---
## 简介

工厂模式主要用于创建对象实例，这在面向对象语言中所用颇多，js面向对象的写法有两种，一种是通过es5 的function和prototype来写，这种写法不够清晰，代码显得很乱，于是es6推出了第二种写法，也就是class语法糖。

class语法与java类似，熟悉java应该会很容易明白。如果再使用typescript支持，就可以做到类方法的权限细分甚至是方法重载。类的常用概念包括封装、继承、重载、多态、构造器。类实例通过new关键字创建。

## 使用场景:

-   有大量的对象实例需要统一管理时
-   需要频繁创建对象实例时

## 为什么使用

-   工厂模式可以将类与类的调用进行解耦。便于对于类进行统一的管理，同时减少在类改变时所造成的调用代码的改动。

## 如何使用

使用类工厂来管理现有的类，并且统一使用工厂类来创建已经在工厂中注册的类。

## 代码案例

```javascript
工厂模式示例


class Connection{
constructor(ip) {
this.ip = ip;
}
toString(){
return `ip:${this.ip}`;
}
}

class BeanFactory{
constructor() {
this.conn = Connection;
}
create(type, args) {
return new this[type](args);
}
}

const factory = new BeanFactory();
const conn = factory.create('conn', '193.24.56.7');
console.log(conn.toString());

>> ip:193.24.56.7
```
