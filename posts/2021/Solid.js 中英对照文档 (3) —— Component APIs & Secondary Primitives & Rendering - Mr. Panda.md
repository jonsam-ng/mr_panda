---
date: 2022-01-01
title: Solid.js 中英对照文档 (3) —— Component APIs & Secondary Primitives & Rendering - Mr. Panda
# category: 
# tags: 
# description:
---

# Solid.js 中英对照文档 (3) —— Component APIs & Secondary Primitives & Rendering - Mr. Panda

---
Solid.js, a declarative, efficient and flexible JavaScript library for building user interfaces.**Solid is a purely reactive library.**It was designed from the ground up with a reactive core and built on hardened tooling in a growing ecosystem.

SolidJs 是一个声明式的、高效的、灵活地 Javasript 库，用来构架UI。Solid 是一个纯粹的响应式的库。他以响应式的核心自下而上的设计并且建立在活跃的生态和强大的工具上。

## Component APIs 组件 API

### `createContext`

```typescript
interface Context<T> {
  id: symbol;
  Provider: (props: { value: T; children: any }) => any;
  defaultValue: T;
}
export function createContext<T>(defaultValue?: T): Context<T | undefined>;
```

在 Solid 中，Context 提供了一种依赖注入的形式。它可以用来避免需要通过中间组件将数据作为 props 传递的情况。

This function creates a new context object that can be used with `**_useContext_**` and provides the `_Provider_` control flow. Default Context is used when no `Provider` is found above in the hierarchy.

该函数创建了一个新的 context 对象，可以通过 `useContext` 来使用，并提供 `Provider` 控制流。当在层次结构的上方找不到 `Provider` 时，将使用默认上下文。

```typescript
export const CounterContext = createContext([{ count: 0 }, {}]); export function CounterProvider(props) { const [state, setState] = createStore({ count: props.count || 0 }); const store = [ state, { increment() { setState("count", (c) => c + 1); }, decrement() { setState("count", (c) => c - 1); }, }, ]; return ( <CounterContext.Provider value={store}> {props.children} </CounterContext.Provider> ); }
```

The value passed to provider is passed to `_useContext_` as is. That means wrapping as a reactive expression will not work. You should pass in Signals and Stores directly instead of accessing them in the JSX.

传递给 provider 的值按原样传递给 `useContext`。这意味着包装为响应性的表达式将不起作用。你应该直接传入 Signal 和 Store，而不是在 JSX 中访问它们。

### `useContext`

```typescript
export function useContext<T>(context: Context<T>): T;
```

Used to grab context to allow for deep passing of props without having to pass them through each Component function.

用于获取上下文以允许深层传递 props，而不必通过每个组件函数传递它们。

```typescript
const [state, { increment, decrement }] = useContext(CounterContext);
```

### `children`

```typescript
export function children(fn: () => any): () => any;
```

Used to make it easier to interact with `props.children`. This helper resolves any nested reactivity and returns a memo. Recommended approach to using `props.children` in anything other than passing directly through to JSX.

用于更容易地与`props.children`交互。这个工具函数解决层级嵌套的响应性并返回一个 Memo。除了直接传递值给 JSX 这种情况之外，推荐使用 `props.children` 的方法。

```typescript
const list = children(() => props.children); // do something with them createEffect(() => list());
```

### `lazy`

```typescript
export function lazy<T extends Component<any>>( fn: () => Promise<{ default: T }> ): T & { preload: () => Promise<T> };
```

Used to lazy load components to allow for **code splitting**. Components are not loaded until rendered. Lazy loaded components can be used the same as its statically imported counterpart, receiving props etc... Lazy components trigger `<Suspense>`.

用于延迟加载组件以允许代码拆分。组件在渲染之前不会加载。延迟加载的组件可以与静态导入的组件同样使用，接收 props 等...... 延迟加载的组件还会触发 `<Suspense>`。

```typescript
// 保证导入 const ComponentA = lazy(() => import("./ComponentA")); // 在 JSX 中使用 <ComponentA title={props.title} />;
```

## Secondary Primitives 第二 Primitive

You probably won't need them for your first app, but these useful tools to have.

你的第一个 app 可能不需要它们，但这些有用的工具也不可或缺。

### `createDeferred`

```typescript
export function createDeferred<T>( source: () => T, options?: { timeoutMs?: number; name?: string; equals?: false | ((prev: T, next: T) => boolean); } ): () => T;
```

Creates a readonly that only notifies downstream changes when the browser is idle. `timeoutMs` is the maximum time to wait before forcing the update.

创建只读值，仅在浏览器空闲时通知下游变更。`timeoutMs` 是强制更新前等待的最长时间。

### `createComputed`

```typescript
export function createComputed<T>( fn: (v: T) => T, value?: T, options?: { name?: string } ): void;
```

Creates a new computation that automatically tracks dependencies and runs immediately before render. Use this to write to other reactive primitives. When possible use `createMemo` instead as writing to a signal mid update can cause other computations to need to re-calculate.

创建一个新的计算，自动跟踪依赖关系并在渲染之前立即运行。使用它来编写其他响应式 primitive。如果可能，请使用 `createMemo` 代替，因为写入中间更新的 signal 可能会导致其他计算需要重新计算。

### `createRenderEffect`

```typescript
export function createRenderEffect<T>( fn: (v: T) => T, value?: T, options?: { name?: string } ): void;
```

Creates a new computation that automatically tracks dependencies and runs during the render phase as DOM elements are created and updated but not necessarily connected. All internal DOM updates happen at this time.

创建一个新的计算，自动跟踪依赖项并在渲染阶段运行，因为 DOM 元素被创建和更新但不一定连接。所有内部 DOM 更新都在此时发生。

### `createSelector`

```typescript
export function createSelector<T, U>( source: () => T, fn?: (a: U, b: T) => boolean, options?: { name?: string } ): (k: U) => boolean;
```

Creates a **conditional signal** that only notifies subscribers when entering or exiting their key matching the value. Useful for delegated selection state. As it makes the operation O(2) instead of O(n).

创建一个条件 signal，仅在进入或退出时键与值匹配时通知订阅者。处理委托选择状态很有用。因为它使得操作复杂度是 O(2) 而不是 O(n)。

```typescript
const isSelected = createSelector(selectedId); <For each={list()}> {(item) => <li classList={{ active: isSelected(item.id) }}>{item.name}</li>} </For>;
```

## Rendering 渲染

These imports are exposed from `solid-js/web`.

以下导入是从 `solid-js/web` 暴露的。

### `render`

```typescript
export function render( code: () => JSX.Element, element: MountableElement ): () => void;
```

This is the browser app entry point. Provide a top level component definition or function and an element to mount to. It is recommended this element be empty as the returned dispose function will wipe all children.

`render` 是浏览器应用程序入口点。它需要提供顶级组件定义或函数以及挂载容器。建议挂载容器为空，因为返回的 dispose 函数将清理所有子元素。

```typescript
const dispose = render(App, document.getElementById("app"));
```

### `hydrate`

```typescript
export function hydrate( fn: () => JSX.Element, node: MountableElement ): () => void;
```

This method is similar to `render` except it attempts to rehydrate what is already rendered to the DOM. When initializing in the browser a page has already been server rendered.

此方法类似于 `render`，只是它会尝试重新注水到已经渲染到 DOM 的内容。在浏览器中初始化时，页面已被服务器渲染。

```typescript
const dispose = hydrate(App, document.getElementById("app"));
```

### `renderToString`

```typescript
export function renderToString<T>( fn: () => T, options?: { eventNames?: string[]; nonce?: string; } ): string;
```

Renders to a string synchronously. The function also generates a script tag for progressive hydration. Options include eventNames to listen to before the page loads and play back on hydration, and nonce to put on the script tag.

同步渲染为字符串。该函数还为渐进式 hydration 生成 script 标签。选项包括在页面加载之前侦听并在注水时回放的 eventNames，以及放在 script 标签上的 nonce。

```typescript
const html = renderToString(App);
```

### `renderToStringAsync`

```typescript
export function renderToStringAsync<T>( fn: () => T, options?: { eventNames?: string[]; timeoutMs?: number; nonce?: string; } ): Promise<string>;
```

Same as `renderToString` except it will wait for all `<Suspense>` boundaries to resolve before returning the results. Resource data is automatically serialized into the script tag and will be hydrated on client load.

与 `renderToString` 相同，除了它在返回结果之前会等待所有 `<Suspense>` 边界解析。资源数据会自动序列化到 script 标签中，并会在客户端加载时注水。

```typescript
const html = await renderToStringAsync(App);
```

### `pipeToNodeWritable`

```typescript
export type PipeToWritableResults = { startWriting: () => void; write: (v: string) => void; abort: () => void; }; export function pipeToNodeWritable<T>( fn: () => T, writable: { write: (v: string) => void }, options?: { eventNames?: string[]; nonce?: string; noScript?: boolean; onReady?: (r: PipeToWritableResults) => void; onComplete?: (r: PipeToWritableResults) => void | Promise<void>; } ): void;
```

This method renders to a Node stream. It renders the content synchronously including any Suspense fallback placeholders, and then continues to stream the data from any async resource as it completes.

此方法渲染出 Node 流。它同步渲染内容，包括任何 Suspense 回退占位符，然后在完成时继续从任何异步资源流式传输数据。

```typescript
pipeToNodeWritable(App, res);
```

The `onReady` option is useful for writing into the stream around the the core app rendering. Remember if you use `onReady` to manually call `startWriting`.

`onReady` 选项用来写入核心应用程序渲染的流很有用。请记住，如果你想要使用 `onReady` 手动调用 `startWriting` 的话。

### `pipeToWritable`

```typescript
export type PipeToWritableResults = { write: (v: string) => void; abort: () => void; script: string; }; export function pipeToWritable<T>( fn: () => T, writable: WritableStream, options?: { eventNames?: string[]; nonce?: string; noScript?: boolean; onReady?: ( writable: { write: (v: string) => void }, r: PipeToWritableResults ) => void; onComplete?: ( writable: { write: (v: string) => void }, r: PipeToWritableResults ) => void; } ): void;
```

This method renders to a web stream. It renders the content synchronously including any Suspense fallback placeholders, and then continues to stream the data from any async resource as it completes.

此方法渲染到 Web 流。它同步渲染内容，包括任何 Suspense 回退占位符，然后在完成时继续从任何异步资源流式传输数据。

```typescript
const { readable, writable } = new TransformStream(); pipeToWritable(App, writable);
```

The `onReady` option is useful for writing into the stream around the the core app rendering. Remember if you use `onReady` to manually call `startWriting`.

`onReady` 选项对于写入围绕核心应用程序渲染的流很有用。请记住，如果你需要使用 `onReady` 手动调用 `startWriting`

### `isServer`

```typescript
export const isServer: boolean;
```

This indicates that the code is being run as the server or browser bundle. As the underlying runtimes export this as a constant boolean it allows bundlers to eliminate the code and their used imports from the respective bundles.

这指明了代码是在服务器运行还是在浏览器运行。由于底层运行时将其导出为常量布尔值，所以它允许构建工具从相应的包中消除代码及其使用的导入代码。

```typescript
if (isServer) { // I will never make it to the browser bundle } else { // won't be run on the server; }
```

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
