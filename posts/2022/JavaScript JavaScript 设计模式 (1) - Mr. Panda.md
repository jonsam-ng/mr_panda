---
date: 2021-01-01
title: JavaScript 设计模式 (1) - Mr. Panda
# category: 
# tags: 
# description:
---

# [JavaScript] JavaScript 设计模式 (1) - Mr. Panda

---
设计模式是软件开发人员在软件开发过程中面临的一般问题的解决方案。这些解决方案是众多软件开发人员经过相当长的一段时间的试验和错误总结出来的。JavaScript 中有 20 多种设计模式，我们将讨论使用 JavaScript ES6 来实现设计模式中的一部分。

## 什么是设计模式？

设计模式是软件设计中常见问题的解决方案。这些模式易使用且富有表现力。

> 在软件工程中，软件设计模式是针对软件设计中给定上下文中的常见问题的通用可重用的解决方案。它不是可以直接转换为源代码或机器代码的设计，它是关于如何解决问题的描述或模板，可以在许多不同的情况下使用。
> 
> 维基百科

## 设计模式的类型

-   创建型
-   结构型
-   行为型

## 创建型设计模式

创建型设计模式的作用是创建对象，而不是直接实例化对象。

> 在软件工程中，创建型设计模式是处理对象创建机制的设计模式，试图以适合情况的方式创建对象。对象创建的基本形式可能会导致设计问题或增加设计的复杂性。创建型设计模式通过以某种方式控制此对象的创建来解决此问题。
> 
> 维基百科

### 工厂模式

它定义了一个用于创建单个对象的接口，并让子类决定要实例化哪个类。

> 在基于类的编程中，工厂方法模式是一种创建模式，它使用工厂方法来处理创建对象的问题，而不必指定将要创建的对象的确切类。这是通过调用工厂方法（在接口中指定并由子类实现，或在基类中实现并可选地由派生类覆盖）而不是通过调用构造函数创建对象来完成的。
> 
> 维基百科

#### 例子

举一个的例子。我们有一类 Point，我们必须创建笛卡尔坐标点和极坐标点。我们将定义一个 Point 工厂来完成这项工作。

```javascript
CoordinateSystem = { CARTESIAN: 0, POLAR: 1, }; class Point { constructor(x, y) { this.x = x; this.y = y; } static get factory() { return new PointFactory(); } }
```

现在我们将创建 Point 工厂：

```javascript
class PointFactory { static newCartesianPoint(x, y) { return new Point(x, y); } static newPolarPoint(rho, theta) { return new Point(rho * Math.cos(theta), rho * Math.sin(theta)); } }
```

我们现在将使用我们的工厂:

```javascript
let point = PointFactory.newPolarPoint(5, Math.PI/2); let point2 = PointFactory.newCartesianPoint(5, 6) console.log(point); console.log(point2);
```

### 抽象工厂模式

它创建公共对象的族或组，而不指定它们的具体类。

> 抽象工厂模式提供了一种封装一组具有共同主题的独立工厂的方法，而无需指定它们的具体类。
> 
> 维基百科

#### 例子

我们将使用饮料和饮料工厂的示例。

```javascript
class Drink { consume() {} } class Tea extends Drink { consume() { console.log('This is Tea'); } } class Coffee extends Drink { consume() { console.log(`This is Coffee`); } }
```

饮料工厂：

```javascript
class DrinkFactory { prepare(amount) } class TeaFactory extends DrinkFactory { makeTea() { console.log(`Tea Created`); return new Tea(); } } class CoffeeFactory extends DrinkFactory { makeCoffee() { console.log(`Coffee Created`); return new Coffee(); } }
```

我们现在将使用我们的工厂：

```javascript
let teaDrinkFactory = new TeaFactory(); let tea = teaDrinkFactory.makeTea() tea.consume()
```

### 建造者模式

从简单对象构造复杂对象。

> 构建器模式是一种设计模式，旨在为面向对象编程中的各种对象创建问题提供灵活的解决方案。
> 
> 维基百科

#### 例子

使用存储 Person 信息的 person 类的示例。

```javascript
class Person { constructor() { this.streetAddress = this.postcode = this.city = ""; this.companyName = this.position = ""; this.annualIncome = 0; } toString() { return ( `Person lives at ${this.streetAddress}, ${this.city}, ${this.postcode}\n` + `and works at ${this.companyName} as a ${this.position} earning ${this.annualIncome}` ); } }
```

现在创建 Person Builder：

```javascript
class PersonBuilder { constructor(person = new Person()) { this.person = person; } get lives() { return new PersonAddressBuilder(this.person); } get works() { return new PersonJobBuilder(this.person); } build() { return this.person; } }
```

现在创建 PersonJobBuilder，它将设置 Person 的 Job 信息：

```javascript
class PersonJobBuilder extends PersonBuilder { constructor(person) { super(person); } at(companyName) { this.person.companyName = companyName; return this; } asA(position) { this.person.position = position; return this; } earning(annualIncome) { this.person.annualIncome = annualIncome; return this; } }
```

PersonAddressBuilder 设置人员的地址信息：

```javascript
class PersonAddressBuilder extends PersonBuilder { constructor(person) { super(person); } at(streetAddress) { this.person.streetAddress = streetAddress; return this; } withPostcode(postcode) { this.person.postcode = postcode; return this; } in(city) { this.person.city = city; return this; } }
```

现在使用构建器：

```javascript
let personBuilder = new PersonBuilder(); let person = personBuilder.lives .at("ABC Road") .in("Multan") .withPostcode("66000") .works.at("Octalogix") .asA("Engineer") .earning(10000) .build(); console.log(person.toString());
```

### 原型模式

从现有对象创建新对象。

> 原型模式是软件开发中的一种创建型的设计模式。当要创建的对象类型由原型实例确定时使用，该实例被克隆以生成新对象。
> 
> 维基百科

#### 例子

我们将使用汽车的例子：

```javascript
class Car { constructor(name, model) { this.name = name; this.model = model; } SetName(name) { console.log(`${name}`) } clone() { return new Car(this.name, this.model); } }
```

如何使用：

```javascript
let car = new Car(); car.SetName('Audi); let car2 = car.clone() car2.SetName('BMW')
```

### 单例模式

它确保只有为特定类创建单一对象实例。

> 在软件工程中，单例模式是一种软件设计模式，它将类的实例化限制为一个“单一”实例。当整个系统只需要一个对象时使用。
> 
> 维基百科

#### 例子

创建单例类：

```javascript
class Singleton { constructor() { const instance = this.constructor.instance; if (instance) { return instance; } this.constructor.instance = this; } say() { console.log('Saying...') } }
```

如何使用：

```javascript
let s1 = new Singleton(); let s2 = new Singleton(); console.log('Are they same? ' + (s1 === s2)); s1.say();
```

## 结构型设计模式

这些模式涉及类和对象组合。他们使用继承来组合接口。

> 在软件工程中，结构型设计模式是通过识别实体之间关系的方法来简化设计的设计模式。
> 
> 维基百科

### 适配器模式

这种模式允许具有不兼容接口的类通过将自己的接口包装在现有类周围来一起工作。

> 在软件工程中，适配器模式是一种软件设计模式，它允许将现有类的接口通过另一个接口进行包装。它通常用于使现有类在不修改其源代码的情况下与其他类一起使用。
> 
> 维基百科

#### 例子

使用计算器的示例。Calculator1 是旧接口，Calculator2 是新接口。我们将构建一个适配器，它将包装新接口并使用它的新方法为我们提供结果。

```javascript
class Calculator1 { constructor() { this.operations = function(value1, value2, operation) { switch (operation) { case 'add': return value1 + value2; case 'sub': return value1 - value2; } }; } } class Calculator2 { constructor() { this.add = function(value1, value2) { return value1 + value2; }; this.sub = function(value1, value2) { return value1 - value2; }; } }
```

创建适配器类：

```javascript
class CalcAdapter { constructor() { const cal2 = new Calculator2(); this.operations = function(value1, value2, operation) { switch (operation) { case 'add': return cal2.add(value1, value2); case 'sub': return cal2.sub(value1, value2); } }; } }
```

如何使用：

```javascript
const adaptedCalc = new CalcAdapter(); console.log(adaptedCalc.operations(10, 55, 'sub'));
```

### 桥接模式

将抽象与实现分开，以便两者可以独立变化。

> 桥接模式是一种结构型设计模式，它允许您将一个大类或一组密切相关的类拆分为两个独立的层次结构——抽象和实现——它们可以相互独立地开发。
> 
> 维基百科

#### 例子

创建渲染器类来渲染多个形状：

```javascript
class VectorRenderer { renderCircle(radius) { console.log(`Drawing a circle of radius ${radius}`); } } class RasterRenderer { renderCircle(radius) { console.log(`Drawing pixels for circle of radius ${radius}`); } } class Shape { constructor(renderer) { this.renderer = renderer; } } class Circle extends Shape { constructor(renderer, radius) { super(renderer); this.radius = radius; } draw() { this.renderer.renderCircle(this.radius); } resize(factor) { this.radius *= factor; } }
```

如何使用：

```javascript
let raster = new RasterRenderer(); let vector = new VectorRenderer(); let circle = new Circle(vector, 5); circle.draw(); circle.resize(2); circle.draw();
```

### 组合模式

组合对象，以便可以将它们作为单个对象进行操作。

> 复合模式描述了一组对象，这些对象的处理方式与相同类型对象的单个实例相同。
> 
> 维基百科

#### 例子

使用员工示例:

```javascript
class Employer{ constructor(name, role){ this.name=name; this.role=role; } print(){ console.log("name:" +this.name + " relaxTime: " ); } }
```

创建 GroupEmployer：

```javascript
class EmployerGroup{ constructor(name, composite=[]){ console.log(name) this.name=name; this.composites=composite; } print(){ console.log(this.name); this.composites.forEach(emp=>{ emp.print(); }) } }
```

如何使用：

```javascript
let zee= new Employer("zee","developer") let shan= new Employer("shan","developer") let groupDevelopers = new EmployerGroup( "Developers", [zee,shan] );
```

### 装饰者模式

动态地添加或覆盖对象的行为。

> 装饰器模式是一种设计模式，它允许将行为动态添加到单个对象，而不会影响同一类中其他对象的行为。
> 
> 维基百科

#### 例子

我们将以颜色和形状为例。

```javascript
class Shape { constructor(color) { this.color = color; } } class Circle extends Shape { constructor(radius = 0) { super(); this.radius = radius; } resize(factor) { this.radius *= factor; } toString() { return `A circle ${this.radius}`; } }
```

创建 ColoredShape 类：

```javascript
class ColoredShape extends Shape { constructor(shape, color) { super(); this.shape = shape; this.color = color; } toString() { return `${this.shape.toString()}` + `has the color ${this.color}`; } }
```

如何使用：

```javascript
let circle = new Circle(2); console.log(circle); let redCircle = new ColoredShape(circle, "red"); console.log(redCircle.toString());
```

### 外观模式

它为复杂代码提供了一个简化的接口。

> 外观模式是面向对象编程中常用的一种软件设计模式。与架构中的外观类似，外观是一个对象，它充当前端接口，掩盖更复杂的底层或结构代码。
> 
> 维基百科

#### 例子

以客户端与计算机的连接为例。

```javascript
class CPU { freeze() {console.log("Freezed....")} jump(position) { console.log("Go....")} execute() { console.log("Run....") } } class Memory { load(position, data) { console.log("Load....") } } class HardDrive { read(lba, size) { console.log("Read....") } }
```

创建外观：

```javascript
class ComputerFacade { constructor() { this.processor = new CPU(); this.ram = new Memory(); this.hd = new HardDrive(); } start() { this.processor.freeze(); this.ram.load(this.BOOT_ADDRESS, this.hd.read(this.BOOT_SECTOR, this.SECTOR_SIZE)); this.processor.jump(this.BOOT_ADDRESS); this.processor.execute(); } }
```

如何使用：

```javascript
let computer = new ComputerFacade(); computer.start();
```

### 享元模式

它降低了创建类似对象的内存成本。

> 享元是一种通过与其他类似对象共享尽可能多的数据来最小化内存使用的对象。
> 
> 维基百科

#### 例子

让我们以用户为例。

```javascript
class User { constructor(fullName) { this.fullName = fullName; } } class User2 { constructor(fullName) { let getOrAdd = function(s) { let idx = User2.strings.indexOf(s); if (idx !== -1) return idx; else { User2.strings.push(s); return User2.strings.length - 1; } }; this.names = fullName.split(' ').map(getOrAdd); } } User2.strings = []; function getRandomInt(max) { return Math.floor(Math.random() * Math.floor(max)); } let randomString = function() { let result = []; for (let x = 0; x < 10; ++x) result.push(String.fromCharCode(65 + getRandomInt(26))); return result.join(''); };
```

如何使用：  
现在我们将通过创建 10k 个用户来在没有享元和享元的情况下进行内存占用比较。

```javascript
let users = []; let users2 = []; let firstNames = []; let lastNames = []; for (let i = 0; i < 100; ++i) { firstNames.push(randomString()); lastNames.push(randomString()); } // making 10k users for (let first of firstNames) for (let last of lastNames) { users.push(new User(`${first} ${last}`)); users2.push(new User2(`${first} ${last}`)); } console.log(`10k users take up approx ` + `${JSON.stringify(users).length} chars`); let users2length = [users2, User2.strings].map(x => JSON.stringify(x).length) .reduce((x,y) => x+y); console.log(`10k flyweight users take up approx ` + `${users2length} chars`);
```

### 代理模式

通过使用代理模式，一个类可以代理另一个类的功能。

> 代理模式是一种软件设计模式。代理，在其最一般的形式中，是一个作为与其他事物的接口的类。
> 
> 维基百科

#### 例子

以 value 代理为例：

```javascript
class Percentage { constructor(percent) { this.percent = percent; } toString() { return `${this.percent}&`; } valueOf() { return this.percent / 100; } }
```

使用方式：

```javascript
let fivePercent = new Percentage(5); console.log(fivePercent.toString()); console.log(`5% of 50 is ${50 * fivePercent}`);
```

## 行为型设计模式

行为型设计模式特别关注对象之间的通信。

> 在软件工程中，行为设计模式是识别对象之间常见通信模式的设计模式，增加了执行通信的灵活性。
> 
> 维基百科

### 责任链模式

它创建对象链。从一个点开始，直到找到某个条件才会停止。

> 在面向对象的设计中，责任链模式是由一个命令对象源和一系列处理对象组成的设计模式。
> 
> 维基百科

#### 例子

使用一个生物的示例。生物达到一定程度时会增加防御和攻击。它会形成一个链条，攻击和防御会增加和减少。

```javascript
class Creature { constructor(name, attack, defense) { this.name = name; this.attack = attack; this.defense = defense; } toString() { return `${this.name} (${this.attack}/${this.defense})`; } } class CreatureModifier { constructor(creature) { this.creature = creature; this.next = null; } add(modifier) { if (this.next) this.next.add(modifier); else this.next = modifier; } handle() { if (this.next) this.next.handle(); } } class NoBonusesModifier extends CreatureModifier { constructor(creature) { super(creature); } handle() { console.log("No bonuses for you!"); } }
```

增加攻击力：

```javascript
class DoubleAttackModifier extends CreatureModifier { constructor(creature) { super(creature); } handle() { console.log(`Doubling ${this.creature.name}'s attack`); this.creature.attack *= 2; super.handle(); } }
```

增加防御力：

```javascript
class IncreaseDefenseModifier extends CreatureModifier { constructor(creature) { super(creature); } handle() { if (this.creature.attack <= 2) { console.log(`Increasing ${this.creature.name}'s defense`); this.creature.defense++; } super.handle(); } }
```

如何使用：

```javascript
let peekachu = new Creature("Peekachu", 1, 1); console.log(peekachu.toString()); let root = new CreatureModifier(peekachu); root.add(new DoubleAttackModifier(peekachu)); root.add(new IncreaseDefenseModifier(peekachu)); root.handle(); console.log(peekachu.toString());
```

### 命令模式

它创建将动作封装在对象中的对象。

> 在面向对象的编程中，命令模式是一种行为设计模式，其中对象用于封装执行操作或稍后触发事件所需的所有信息。此信息包括方法名称、拥有该方法的对象和方法参数的值。
> 
> 维基百科

#### 例子

举一个简单的银行帐户示例。

```javascript
class BankAccount { constructor(balance = 0) { this.balance = balance; } deposit(amount) { this.balance += amount; console.log(`Deposited ${amount} Total balance ${this.balance}`); } withdraw(amount) { if (this.balance - amount >= BankAccount.overdraftLimit) { this.balance -= amount; console.log("Widhdrawn"); } } toString() { return `Balance ${this.balance}`; } } BankAccount.overdraftLimit = -500; let Action = Object.freeze({ deposit: 1, withdraw: 2, });
```

创建命令：

```javascript
class BankAccountCommand { constructor(account, action, amount) { this.account = account; this.action = action; this.amount = amount; } call() { switch (this.action) { case Action.deposit: this.account.deposit(this.amount); break; case Action.withdraw: this.account.withdraw(this.amount); break; } } undo() { switch (this.action) { case Action.deposit: this.account.withdraw(this.amount); break; case Action.withdraw: this.account.deposit(this.amount); break; } } }
```

如何使用：

```javascript
let bankAccount = new BankAccount(100); let cmd = new BankAccountCommand(bankAccount, Action.deposit, 50); cmd.call(); console.log(bankAccount.toString()); cmd.undo(); console.log(bankAccount.toString());
```

### 迭代器模式

迭代器访问对象的元素而不暴露其底层表示。

> 在面向对象编程中，迭代器模式是一种设计模式，其中迭代器用于遍历容器并访问容器的元素。
> 
> 维基百科

#### 例子

将举一个数组的例子，在其中打印数组的值，然后使用迭代器打印它的值。

```javascript
class Stuff { constructor() { this.a = 11; this.b = 22; } [Symbol.iterator]() { let i = 0; let self = this; return { next: function() { return { done: i > 1, value: self[i++ === 0 ? 'a' : 'b'] }; } } } get backwards() { let i = 0; let self = this; return { next: function() { return { done: i > 1, value: self[i++ === 0 ? 'b' : 'a'] }; }, // make iterator iterable [Symbol.iterator]: function() { return this; } } } }
```

如何使用：

```javascript
let values = [100, 200, 300]; for (let i in values) { console.log(`Element at pos ${i} is ${values[i]}`); } for (let v of values) { console.log(`Value is ${v}`); } let stuff = new Stuff(); for (let item of stuff) console.log(`${item}`); for (let item of stuff.backwards) console.log(`${item}`);
```

### 中介者模式

中介者模式添加了一个第三方对象来控制两个对象之间的交互。它允许类之间的松散耦合，因为它是唯一详细了解其方法的类。

> 中介者模式定义了一个对象，该对象封装了一组对象如何交互。这种模式被认为是一种行为模式，因为它可以改变程序的运行行为。在面向对象编程中，程序通常由许多类组成。
> 
> 维基百科

#### 例子

使用聊天室的例子。在这里，聊天室充当了两个人交流的中介。

```javascript
class Person { constructor(name) { this.name = name; this.chatLog = []; } receive(sender, message) { let s = `${sender}: '${message}'`; console.log(`[${this.name}'s chat session] ${s}`); this.chatLog.push(s); } say(message) { this.room.broadcast(this.name, message); } pm(who, message) { this.room.message(this.name, who, message); } }
```

创建聊天室：

```javascript
class ChatRoom { constructor() { this.people = []; } broadcast(source, message) { for (let p of this.people) if (p.name !== source) p.receive(source, message); } join(p) { let joinMsg = `${p.name} joins the chat`; this.broadcast("room", joinMsg); p.room = this; this.people.push(p); } message(source, destination, message) { for (let p of this.people) if (p.name === destination) p.receive(source, message); } }
```

如何使用：

```javascript
let room = new ChatRoom(); let zee = new Person("Zee"); let shan = new Person("Shan"); room.join(zee); room.join(shan); zee.say("Hello!!"); let doe = new Person("Doe"); room.join(doe); doe.say("Hello everyone!");
```

### 备忘录模式

将对象恢复到之前的状态。

> 备忘录模式是一种软件设计模式，它提供了将对象恢复到其先前状态的能力。备忘录模式由三个对象实现：发起者、看守者和备忘录。
> 
> 维基百科

#### 例子

举一个银行账户的例子，我们在其中存储我们以前的状态，并将具有撤消功能。

```javascript
class Memento { constructor(balance) { this.balance = balance; } }
```

添加银行账户：

```javascript
class BankAccount { constructor(balance = 0) { this.balance = balance; } deposit(amount) { this.balance += amount; return new Memento(this.balance); } restore(m) { this.balance = m.balance; } toString() { return `Balance: ${this.balance}`; } }
```

如何使用：

```javascript
let bankAccount = new BankAccount(100); let m1 = bankAccount.deposit(50); console.log(bankAccount.toString()); // restore to m1 bankAccount.restore(m1); console.log(bankAccount.toString());
```

### 观察者模式

它允许许多观察者对象关注一个事件。

> 观察者模式是一种软件设计模式，其中一个名为 subject 的对象维护其依赖项列表，称为观察者，并自动通知它们任何状态更改，通常通过调用它们的监听方法来实现。
> 
> 维基百科

#### 例子

举一个人的例子，如果一个人生病了，它会显示一个通知。

```javascript
class Event { constructor() { this.handlers = new Map(); this.count = 0; } subscribe(handler) { this.handlers.set(++this.count, handler); return this.count; } unsubscribe(idx) { this.handlers.delete(idx); } fire(sender, args) { this.handlers.forEach((v, k) => v(sender, args)); } } class FallsIllArgs { constructor(address) { this.address = address; } } class Person { constructor(address) { this.address = address; this.fallsIll = new Event(); } catchCold() { this.fallsIll.fire(this, new FallsIllArgs(this.address)); } }
```

如何使用：

```javascript
let person = new Person("ABC road"); let sub = person.fallsIll.subscribe((s, a) => { console.log(`A doctor has been called ` + `to ${a.address}`); }); person.catchCold(); person.catchCold(); person.fallsIll.unsubscribe(sub); person.catchCold();
```

### 访问者模式

向对象添加操作而无需修改它们。

> 访问者设计模式是一种将算法与其操作的对象结构分离的方法。这种分离的实际结果是能够在不修改结构的情况下向现有对象结构添加新操作。
> 
> 维基百科

#### 例子

举一个 NumberExpression 的例子，它给出给定表达式的结果。

```javascript
class NumberExpression { constructor(value) { this.value = value; } print(buffer) { buffer.push(this.value.toString()); } }
```

创建 AdditionExpression：

```javascript
class AdditionExpression { constructor(left, right) { this.left = left; this.right = right; } print(buffer) { buffer.push('('); this.left.print(buffer); buffer.push('+'); this.right.print(buffer); buffer.push(')'); } }
```

如何使用：

```javascript
// 5 + (1+9) let e = new AdditionExpression( new NumberExpression(5), new AdditionExpression( new NumberExpression(1), new NumberExpression(9) ) ); let buffer = []; e.print(buffer); console.log(buffer.join(''));
```

### 策略模式

它允许在特定情况下选择一种算法。

> 策略模式是一种行为软件设计模式，可以在运行时选择算法。代码不是直接实现单个算法，而是接收运行时指令，以确定要使用一系列算法中的哪一个。
> 
> 维基百科

#### 例子

举一个例子，有一个文本处理器，它将根据策略（HTML 或 Markdown）显示数据。

```javascript
let OutputFormat = Object.freeze({ markdown: 0, html: 1, }); class ListStrategy { start(buffer) {} end(buffer) {} addListItem(buffer, item) {} } class MarkdownListStrategy extends ListStrategy { addListItem(buffer, item) { buffer.push(` * ${item}`); } } class HtmlListStrategy extends ListStrategy { start(buffer) { buffer.push("<ul>"); } end(buffer) { buffer.push("</ul>"); } addListItem(buffer, item) { buffer.push(` <li>${item}</li>`); } }
```

创建 TextProcessor：

```javascript
class TextProcessor { constructor(outputFormat) { this.buffer = []; this.setOutputFormat(outputFormat); } setOutputFormat(format) { switch (format) { case OutputFormat.markdown: this.listStrategy = new MarkdownListStrategy(); break; case OutputFormat.html: this.listStrategy = new HtmlListStrategy(); break; } } appendList(items) { this.listStrategy.start(this.buffer); for (let item of items) this.listStrategy.addListItem(this.buffer, item); this.listStrategy.end(this.buffer); } clear() { this.buffer = []; } toString() { return this.buffer.join("\n"); } }
```

如何使用：

```javascript
let tp = new TextProcessor(); tp.setOutputFormat(OutputFormat.markdown); tp.appendList(["one", "two", "three"]); console.log(tp.toString()); tp.clear(); tp.setOutputFormat(OutputFormat.html); tp.appendList(["one", "two", "three"]); console.log(tp.toString());
```

### 状态模式

当对象的内部状态发生变化时，它会改变对象的行为。

> 状态模式是一种行为软件设计模式，它允许对象在其内部状态发生变化时改变其行为。这种模式接近于有限状态机的概念。
> 
> 维基百科

#### 例子

举一个电灯开关的例子，如果我们打开或关闭开关，它的状态就会改变。  

```javascript
class Switch { constructor() { this.state = new OffState(); } on() { this.state.on(this); } off() { this.state.off(this); } } class State { constructor() { if (this.constructor === State) throw new Error("abstract!"); } on(sw) { console.log("Light is already on."); } off(sw) { console.log("Light is already off."); } }
```

创建状态类：

```javascript
class OnState extends State { constructor() { super(); console.log("Light turned on."); } off(sw) { console.log("Turning light off..."); sw.state = new OffState(); } } class OffState extends State { constructor() { super(); console.log("Light turned off."); } on(sw) { console.log("Turning light on..."); sw.state = new OnState(); } }
```

如何使用：

```javascript
let switch = new Switch(); switch.on(); switch.off();
```

### 模板模式

将算法的骨架定义为一个抽象类，以描述应该如何执行。

> 模板模式是超类中的一个方法，通常是一个抽象超类，并根据许多高级步骤定义了操作的框架。
> 
> 维基百科

#### 例子

以国际象棋游戏为例。

```javascript
class Game { constructor(numberOfPlayers) { this.numberOfPlayers = numberOfPlayers; this.currentPlayer = 0; } run() { this.start(); while (!this.haveWinner) { this.takeTurn(); } console.log(`Player ${this.winningPlayer} wins.`); } start() {} get haveWinner() {} takeTurn() {} get winningPlayer() {} }
```

创建 Chess：

```javascript
class Chess extends Game { constructor() { super(2); this.maxTurns = 10; this.turn = 1; } start() { console.log( `Starting a game of chess with ${this.numberOfPlayers} players.` ); } get haveWinner() { return this.turn === this.maxTurns; } takeTurn() { console.log(`Turn ${this.turn++} taken by player ${this.currentPlayer}.`); this.currentPlayer = (this.currentPlayer + 1) % this.numberOfPlayers; } get winningPlayer() { return this.currentPlayer; } }
```

如何使用：

```javascript
let chess = new Chess(); chess.run();
```

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
