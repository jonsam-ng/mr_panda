---
date: 2022-01-01
title: Solid.js 中英对照文档 (2) —— Reactive Utilities & Stores - Mr. Panda
# category: 
# tags: 
# description:
---

# Solid.js 中英对照文档 (2) —— Reactive Utilities & Stores - Mr. Panda

---
Solid.js, a declarative, efficient and flexible JavaScript library for building user interfaces.**Solid is a purely reactive library.**It was designed from the ground up with a reactive core and built on hardened tooling in a growing ecosystem.

SolidJs 是一个声明式的、高效的、灵活地 Javasript 库，用来构架UI。Solid 是一个纯粹的响应式的库。他以响应式的核心自下而上的设计并且建立在活跃的生态和强大的工具上。

## Reactive Utilities 响应式工具函数

These helpers provide the ability to better schedule updates and control how reactivity is tracked.

有了这些工具函数，我们可以更好地调度更新以及控制响应式跟踪行为。

### `untrack`

```typescript
export function untrack<T>(fn: () => T): T; 
```

忽略跟踪执行代码块中的任何依赖项并返回值。

### `batch`

```typescript
export function batch<T>(fn: () => T): T;
```

Holds committing updates within the block until the end to prevent unnecessary recalculation. This means that reading values on the next line will not have updated yet. [Solid Store](https://www.jonsam.site/2021/09/10/solid-js-reactive-utilities-stores/#createstore)'s set method and Effects automatically wrap their code in a batch.

在块作用域结束时提交更新以避免不必要的重新计算。这意味着下一行的读取到的值将没有被更新。Solid Store 的 set 方法和 Effect 会自动将它们的代码打包成一个批次来进行更新。

### `on`

```typescript
export function on<T extends Array<() => any> | (() => any), U>( deps: T, fn: (input: T, prevInput: T, prevValue?: U) => U, options: { defer?: boolean } = {} ): (prevValue?: U) => U | undefined;
```

`on` is designed to be passed into a computation to make its dependencies explicit. If an array of dependencies is passed, `input` and `prevInput` are arrays.

`on` 主要用来将其传递到计算行为中以使其依赖项更加清晰明了。如果传递依赖项是数组，则 `input` 和 `prevInput` 也是数组。

```typescript
createEffect(on(a, (v) => console.log(v, b()))); // 等同于 createEffect(() => { const v = a(); untrack(() => console.log(v, b())); });
```

You can also not run the computation immediately and instead opt in for it to only run on change by setting the defer option to true.

你也可以不用立即执行计算，而是通过将 defer 选项设置为 true 来选择仅在更改时运行计算。

```typescript
// 不会立即运行 createEffect(on(a, (v) => console.log(v), { defer: true })); setA("new"); // 现在会运行了
```

### `createRoot`

```typescript
export function createRoot<T>(fn: (dispose: () => void) => T): T;
```

Creates a new non-tracked context that doesn't auto-dispose. This is useful for nested reactive contexts that you do not wish to release when the parent re-evaluates. It is a powerful pattern for caching.

All Solid code should be wrapped in one of these top level as they ensure that all memory/computations are freed up. Normally you do not need to worry about this as `createRoot` is embedded into all `render` entry functions.

创建一个崭新的，不自动处理的，非跟踪上下文。在嵌套响应式上下文的情况下，如果你不希望在父级重新求值时释放资源这个特性会很有用。这是一种强大的缓存模式。

所有 Solid 代码都应被 createRoot 包裹，因为它们确保释放所有内存/计算。通常你不需要担心这个，因为 `createRoot` 被嵌入到所有的 `render` 入口函数中。

### `mergeProps`

```typescript
export function mergeProps(...sources: any): any;
```

A reactive object `merge` method. Useful for setting default props for components in case caller doesn't provide them. Or cloning the props object including reactive properties.

This method works by using a proxy and resolving properties in reverse order. This allows for dynamic tracking of properties that aren't present when the prop object is first merged.

响应式对象的合并 `merge` 方法。用于为组件设置默认 props 以防调用者不提供这些属性值。或者克隆包含响应式的属性的 props 对象。

此方法的运作原理是使用代理并以相反的顺序解析属性。这可以对首次合并 props 对象时不存在的属性进行动态跟踪。

```typescript
// 设置默认 props props = mergeProps({ name: "Smith" }, props); // 克隆 props newProps = mergeProps(props); // 合并 props props = mergeProps(props, otherProps);
```

### `splitProps`

```typescript
export function splitProps<T>( props: T, ...keys: Array<(keyof T)[]> ): [...parts: Partial<T>];
```

This is the replacement for destructuring. It splits a reactive object by keys while maintaining reactivity.

`splitProps` 是解构的替代品。`splitProps` 在保持响应性的同时通过键来拆分响应式对象。

```typescript
const [local, others] = splitProps(props, ["children"]); <> <Child {...others} /> <div>{local.children}<div> </>
```

### `useTransition`

```typescript
export function useTransition(): [ () => boolean, (fn: () => void, cb?: () => void) => void ];
```

Used to batch async updates in a transaction deferring commit until all async processes are complete. This is tied into Suspense and only tracks resources read under Suspense boundaries.

用于在所有异步处理完成后在延迟提交事务中批量异步更新。这与 Suspense 有所关联，并且仅跟踪在 Suspense 边界下读取的资源。

```typescript
const [isPending, start] = useTransition(); // 检查是否在 transition 中 isPending(); // 包裹在 transition 中 start(() => setSignal(newValue), () => /* transition 完成 */)
```

### `observable`

```typescript
export function observable<T>(input: () => T): Observable<T>;
```

This method takes a signal and produces a simple Observable. Consume it from the Observable library of your choice with typically with the `from` operator.

这个方法接受一个 signal 并产生一个简单的 Observable。从你选择的 Observable 库中使用它，通常使用 `from` 操作符。

```typescript
import { from } from "rxjs"; const [s, set] = createSignal(0); const obsv$ = from(observable(s)); obsv$.subscribe((v) => console.log(v));
```

### `mapArray`

```typescript
export function mapArray<T, U>( list: () => readonly T[], mapFn: (v: T, i: () => number) => U ): () => U[];
```

Reactive map helper that caches each item by reference to reduce unnecessary mapping on updates. It only runs the mapping function once per value and then moves or removes it as needed. The index argument is a signal. The map function itself is not tracking.

Underlying helper for the `<For>` control flow.

响应式映射工具函数，通过引用缓存每个子项，以减少不必要的映射更新。它只为每个值运行一次映射函数，然后根据需要移动或删除它。index 参数是一个 signal。映射函数本身没有被跟踪。

`mapArray` 也是`<For>` 组件控制流程的底层工具函数

```typescript
const mapped = mapArray(source, (model) => { const [name, setName] = createSignal(model.name); const [description, setDescription] = createSignal(model.description); return { id: model.id, get name() { return name(); }, get description() { return description(); } setName, setDescription } });
```

### `indexArray`

```typescript
export function indexArray<T, U>( list: () => readonly T[], mapFn: (v: () => T, i: number) => U ): () => U[];
```

Similar to `mapArray` except it maps by index. The item is a signal and the index is now the constant.Underlying helper for the `<Index>` control flow.

类似于 `mapArray`，除了它按索引映射。每个子项都是 signal，索引是常量。

`indexArray` 也是 `<Index>` 组件控制流程的底层工具函数。

```typescript
const mapped = indexArray(source, (model) => { return { get id() { return model().id } get firstInitial() { return model().firstName[0]; }, get fullName() { return `${model().firstName} ${model().lastName}`; }, } });
```

## Stores 状态存储

These APIs are available at `solid-js/store`.

以下 API 可以从`solid-js/store` 导入。

### `createStore`

```typescript
export function createStore<T extends StoreNode>( state: T | Store<T>, options?: { name?: string } ): [get: Store<T>, set: SetStoreFunction<T>];
```

This creates a tree of Signals as proxy that allows individual values in nested data structures to be independently tracked. The create function returns a readonly proxy object, and a setter function.

`createStore` 创建一个 Signal 树作为代理，允许独立跟踪嵌套数据结构中的各个值。create 函数返回一个只读代理对象和一个 setter 函数。

```typescript
const [state, setState] = createStore(initialValue); // 读取值 state.someValue; // 设置值 setState({ merge: "thisValue" }); setState("path", "to", "value", newValue);
```

Store objects being proxies only track on property access. And on access Stores recursively produces nested Store objects on nested data. However it only wraps arrays and plain objects. Classes are not wrapped. So things like `Date`, `HTMLElement`, `RegExp`, `Map`, `Set` are not granularly reactive. Additionally, the top level state object cannot be tracked without accessing a property on it. So it is not suitable to use for things you iterate over as adding new keys or indexes cannot trigger updates. So put any lists on a key of state rather than trying to use the state object itself.

Store 代理对象仅跟踪属性访问。并在访问 Store 时递归地生成嵌套数据上的嵌套 Store 对象。但是它只包装数组和普通对象。但是只包装数组和原生对象。类对象并不被包装。所以像 `Date`、`HTMLElement`、`RegExp`、`Map`、`Set` 之类的东西都不是响应式粒度的。此外，如果不访问对象上的属性，则无法跟踪顶级状态对象。因此它不适用于迭代对象，因为添加新键或索引无法触发更新。因此，将数组放在键上，而不是尝试使用状态对象本身。

```typescript
// 将列表作为状态对象的键 const [state, setState] = createStore({ list: [] }); // 访问 state 对象上的 `list` 属性 <For each={state.list}>{item => /*...*/}</For>
```

#### Getters

Store objects support the use of getters to store calculated values.

Store 对象支持使用 getter 来存储计算值。

```typescript
const [state, setState] = createStore({ user: { firstName: "John", lastName: "Smith", get fullName() { return `${this.firstName} ${this.lastName}`; }, }, });
```

These are simple getters, so you still need to use a Memo if you want to cache a value;

以下是简单的 getter，所以如果你想缓存一个值，你仍然需要使用 Memo；

```typescript
let fullName; const [state, setState] = createStore({ user: { firstName: "John", lastName: "Smith", get fullName() { return fullName(); }, }, }); fullName = createMemo(() => `${state.user.firstName} ${state.user.lastName}`);
```

#### Updating Stores

Changes can take the form of function that passes previous state and returns new state or a value. Objects are always shallowly merged. Set values to `undefined` to delete them from the Store.

更改状态可以采用传递先前状态并返回新状态或值的函数的形式。对象总是浅合并的。将值设置为 `undefined` 以将属性从 Store 中删除。

```typescript
const [state, setState] = createStore({ firstName: "John", lastName: "Miller", }); setState({ firstName: "Johnny", middleName: "Lee" }); // ({ firstName: 'Johnny', middleName: 'Lee', lastName: 'Miller' }) setState((state) => ({ preferredName: state.firstName, lastName: "Milner" })); // ({ firstName: 'Johnny', preferredName: 'Johnny', middleName: 'Lee', lastName: 'Milner' })
```

It supports paths including key arrays, object ranges, and filter functions.setState also supports nested setting where you can indicate the path to the change. When nested the state you are updating may be other non Object values. Objects are still merged but other values (including Arrays) are replaced.

setState 支持的路径包括数组索引、对象范围和过滤器函数。setState 还支持嵌套设置，你可以在其中指明要修改的路径。在嵌套的情况下，要更新的状态可能是非对象值。对象仍然合并，但其他值（包括数组）将会被替换。

```typescript
const [state, setState] = createStore({ counter: 2, list: [ { id: 23, title: 'Birds' } { id: 27, title: 'Fish' } ] }); setState('counter', c => c + 1); setState('list', l => [...l, {id: 43, title: 'Marsupials'}]); setState('list', 2, 'read', true); // { // counter: 3, // list: [ // { id: 23, title: 'Birds' } // { id: 27, title: 'Fish' } // { id: 43, title: 'Marsupials', read: true } // ] // }
```

Path can be string keys, array of keys, iterating objects ({from, to, by}), or filter functions. This gives incredible expressive power to describe state changes.

路径可以是字符串键、数组索引、迭代对象（{from、to、by}）或过滤器函数。这为描述状态变化提供了令人难以置信的表达能力。

```typescript
const [state, setState] = createStore({ todos: [ { task: 'Finish work', completed: false } { task: 'Go grocery shopping', completed: false } { task: 'Make dinner', completed: false } ] }); setState('todos', [0, 2], 'completed', true); // { // todos: [ // { task: 'Finish work', completed: true } // { task: 'Go grocery shopping', completed: false } // { task: 'Make dinner', completed: true } // ] // } setState('todos', { from: 0, to: 1 }, 'completed', c => !c); // { // todos: [ // { task: 'Finish work', completed: false } // { task: 'Go grocery shopping', completed: true } // { task: 'Make dinner', completed: true } // ] // } setState('todos', todo => todo.completed, 'task', t => t + '!') // { // todos: [ // { task: 'Finish work', completed: false } // { task: 'Go grocery shopping!', completed: true } // { task: 'Make dinner!', completed: true } // ] // } setState('todos', {}, todo => ({ marked: true, completed: !todo.completed })) // { // todos: [ // { task: 'Finish work', completed: true, marked: true } // { task: 'Go grocery shopping!', completed: false, marked: true } // { task: 'Make dinner!', completed: false, marked: true } // ] // }
```

### `produce`

```typescript
export function produce<T>( fn: (state: T) => void ): ( state: T extends NotWrappable ? T : Store<T> ) => T extends NotWrappable ? T : Store<T>;
```

Immer inspired API for Solid's Store objects that allows for localized mutation.

Immer 启发了 Solid 的 Store 对象的 `produce` API，它允许本地修改状态。

```typescript
setState( produce((s) => { s.user.name = "Frank"; s.list.push("Pencil Crayon"); }) );
```

### `reconcile`

```typescript
export function reconcile<T>( value: T | Store<T>, options?: { key?: string | null; merge?: boolean; } = { key: "id" } ): ( state: T extends NotWrappable ? T : Store<T> ) => T extends NotWrappable ? T : Store<T>;
```

Diffs data changes when we can't apply granular updates. Useful for when dealing with immutable data from stores or large API responses.The key is used when available to match items. By default `merge` false does referential checks where possible to determine equality and replaces where items are not referentially equal. `merge` true pushes all diffing to the leaves and effectively morphs the previous data to the new value.

当不能应用粒度更新时对比数据变更。`reconcile` 在处理来自 store 或巨大 API 响应这些不可变数据时很有用。

key 在可用于匹配项目时使用。默认情况下，`merge` 为 `false` 会在可能的情况下进行引用检查，并替换引用不相等的数据。`merge` 为 `true` 时，`reconcile` 会将所有差异推送到叶子节点，并高效地将先前的数据修改为新值。

```typescript
// 订阅一个 observable const unsubscribe = store.subscribe(({ todos }) => ( setState('todos', reconcile(todos))); ); onCleanup(() => unsubscribe());
```

### `createMutable`

```typescript
export function createMutable<T extends StoreNode>( state: T | Store<T>, options?: { name?: string } ): Store<T> {
```

Creates a new mutable Store proxy object. Stores only trigger updates on values changing. Tracking is done by intercepting property access and automatically tracks deep nesting via proxy.

Useful for integrating external systems or as a compatibility layer with MobX/Vue.

`createMutable` 创建一个新的可变 Store 代理对象。Store 仅在值更改时触发更新。跟踪是通过拦截属性访问来完成的，并通过代理自动跟踪深度嵌套数据。

`createMutable` 用于集成外部系统或作为与 MobX/Vue 的兼容层会很有用。

> **Note:** A mutable state can be passed around and mutated anywhere, which can make it harder to follow and easier to break unidirectional flow. It is generally recommended to use `createStore` instead. The `produce` modifier can give many of the same benefits without any of the downsides.
> 
> **注意：** _由于可变状态可以在任何地方传递和修改，这会使其更难以遵循并且更容易打破单向流，因此通常建议使用_ `createStore` _代替。_`produce` _修饰符可以提供许多相同的好处而没有任何缺点。_

```typescript
const state = createMutable(initialValue); // 读取值 state.someValue; // 设置值 state.someValue = 5; state.list.push(anotherValue);
```

Mutables support setters along with getters.

```typescript
const user = createMutable({ firstName: "John", lastName: "Smith", get fullName() { return `${this.firstName} ${this.lastName}`; }, set fullName(value) { [this.firstName, this.lastName] = value.split(" "); }, });
```

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
