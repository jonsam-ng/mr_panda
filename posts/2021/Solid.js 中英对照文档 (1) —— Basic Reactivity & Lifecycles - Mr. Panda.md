---
date: 2022-01-01
title: Solid.js 中英对照文档 (1) —— Basic Reactivity & Lifecycles - Mr. Panda
# category: 
# tags: 
# description:
---

# Solid.js 中英对照文档 (1) —— Basic Reactivity & Lifecycles - Mr. Panda

---
Solid.js, a declarative, efficient and flexible JavaScript library for building user interfaces.**Solid is a purely reactive library.**It was designed from the ground up with a reactive core and built on hardened tooling in a growing ecosystem.

SolidJs 是一个声明式的、高效的、灵活地 Javasript 库，用来构架UI。Solid 是一个纯粹的响应式的库。他以响应式的核心自下而上的设计并且建立在活跃的生态和强大的工具上。

## Basic Reactivity 响应式基础

### `createSignal`

```javascript
export function createSignal(
   value: T,
   options?: { name?: string; equals?: false | ((prev: T, next: T) => boolean) }
 ): [get: () => T, set: (v: T) => T];
```

这是最基本的反应式原语 (primitive)，用于跟踪随时间变化的单个值。 create 函数返回一对 get 和 set 函数来访问和更新信号 (signal)。

```javascript
const [getValue, setValue] = createSignal(initialValue); // read value getValue(); // set value setValue(nextValue); // set value with a function setter setValue((prev) => prev + next);
```

Remember to access signals under a **tracking scope** if you wish them to react to updates. **Tracking scopes are functions that are passed to computations** like `createEffect` or JSX expressions.

如果您希望值对更新做出响应，请记住在跟踪范围内访问信号。跟踪范围是传递给计算的函数，如 `createEffect` 或 JSX 表达式。

If you wish to store a function in a Signal you must use the function form:

如果你希望在 Signal 中存储函数，则必须使用函数的形式：

```javascript
setValue(() => myFunction);
```

### `createEffect`

```javascript
export function createEffect<T>( fn: (v: T) => T, value?: T, options?: { name?: string } ): void;
```

Creates a new computation that automatically tracks dependencies and runs after each render where a dependency has changed. Ideal for using `ref`s and managing other side effects.

创建一个新的计算来自动跟踪依赖项并在依赖项发生变化的每次渲染之后运行。非常适合使用 `ref`s 和管理其他副作用。

```javascript
const [a, setA] = createSignal(initialValue); // effect that depends on signal `a` createEffect(() => doSideEffect(a()));
```

The effect function is called with the value returned from the effect function's last execution. This value can be initialized as an optional 2nd argument. This can be useful for diffing without creating an additional closure.

effect 函数可以拿到上次执行返回的值。可以在第二个可选参数设置该值得初始化值。这可以让我们不用创建额外闭包的情况下就可以进行差异对比。

```javascript
createEffect((prev) => { const sum = a() + b(); if (sum !== prev) console.log(sum); return sum; }, 0);
```

### `createMemo`

```javascript
export function createMemo( fn: (v: T) => T, value?: T, options?: { name?: string; equals?: false | ((prev: T, next: T) => boolean) } ): () => T;
```

Creates a readonly derived signal that recalculates its value whenever the executed code's dependencies update.

创建一个只读派生的 signal，每当执行代码的依赖项被更新时，该 signal 就会重新计算其值。

```javascript
const getValue = createMemo(() => computeExpensiveValue(a(), b())); // read value getValue();
```

The memo function is called with the value returned from the memo function's last execution. This value can be initialized as an optional 2nd argument. This is useful for reducing computations.

使用 memo 函数上次执行返回的值调用 memo 函数。该值可以初始化为可选的第二个参数。这对于减少计算很有用。

```javascript
const sum = createMemo((prev) => input() + prev, 0);
```

### `createResource`

```javascript
type ResourceReturn = [ { (): T | undefined; loading: boolean; error: any; }, { mutate: (v: T | undefined) => T | undefined; refetch: () => void; } ]; export function createResource( fetcher: (k: U, getPrev: () => T | undefined) => T | Promise, options?: { initialValue?: T; name?: string } ): ResourceReturn; export function createResource( source: U | false | null | (() => U | false | null), fetcher: (k: U, getPrev: () => T | undefined) => T | Promise, options?: { initialValue?: T; name?: string } ): ResourceReturn;
```

Creates a signal that can manage async requests. The `fetcher` is an async function that accepts return value of the `source` if provided and returns a Promise whose resolved value is set in the resource. The fetcher is not reactive so use the optional first argument if you want it to run more than once. If the source resolves to false, null, or undefined will not to fetch.

创建一个可以管理异步请求的 signal。`fetcher` 是一个异步函数，它接收 `source` 的返回值（如果提供）并返回一个 Promise，其解析值设置在 resource 中。fetcher 不是响应式的，因此如果希望它运行多次，请传入第一个可选参数。如果源解析为 false、null 或 undefined，则不会执行获取操作。

```javascript
const [data, { mutate, refetch }] = createResource(getQuery, fetchData); // 读取值 data(); // 检查是否加载中 data.loading; // 检查是否加载错误 data.error; // 无需创建 promise 直接设置值 mutate(optimisticValue); // 重新执行最后的请求 refetch();
```

`loading` and `error` are reactive getters and can be tracked.

`loading` 和 `error` 是响应式 getter，可以被跟踪。

## Lifecycles 生命周期

### `onMount`

```javascript
export function onMount(fn: () => void): void;
```

Registers a method that runs after initial render and elements have been mounted. Ideal for using `ref`s and managing other one time side effects. It is equivalent to a `createEffect` which does not have any dependencies.

注册一个在初始话化渲染和元素挂载完成后运行的方法。非常适合使用 `ref` 或者管理其他的一次性副作用。它相当于一个没有任何依赖的 `createEffect`。

### `onCleanup`

```javascript
export function onCleanup(fn: () => void): void;
```

Registers a cleanup method that executes on disposal or recalculation of the current reactive scope. Can be used in any Component or Effect.

注册一个清理方法，它会在当前响应范围内执行销毁或重新计算时候被触发。可用于任何组件或 Effect。

### `onError`

```javascript
export function onError(fn: (err: any) => void): void;
```

Registers an error handler method that executes when child scope errors. Only the nearest scope error handlers execute. Rethrow to trigger up the line.

注册一个错误处理函数，它会在子作用域抛出错误时执行。但是最近的范围的错误处理函数才会执行。重新抛出可以向上触发。

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
