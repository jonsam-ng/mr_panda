---
date: 2022-01-01
title: Solid.js 中英对照文档 (4) —— Control Flow & Special JSX Attributes - Mr. Panda
# category: 
# tags: 
# description:
---

# Solid.js 中英对照文档 (4) —— Control Flow & Special JSX Attributes - Mr. Panda

---
Solid.js, a declarative, efficient and flexible JavaScript library for building user interfaces.**Solid is a purely reactive library.**It was designed from the ground up with a reactive core and built on hardened tooling in a growing ecosystem.

SolidJs 是一个声明式的、高效的、灵活地 Javasript 库，用来构架UI。Solid 是一个纯粹的响应式的库。他以响应式的核心自下而上的设计并且建立在活跃的生态和强大的工具上。

## Control Flow 控制流

Solid uses components for control flow. The reason is that with reactivity to be performant we have to control how elements are created. For example with lists, a simple `map` is inefficient as it always maps everything. This means helper functions.

Wrapping these in components is convenient way for terse templating and allows users to compose and build their own control flows.

These built-in control flows will be automatically imported. All except `Portal` and `Dynamic` are exported from `solid-js`. Those two which are DOM specific are exported by `solid-js/web`.

Solid 使用组件来控制流。原因是为了提高响应式性能，我们必须控制元素的创建方式。例如，对于列表而言，简单的 `map` 效率低下，因为它总是映射所有内容。这意味着需要一个辅助函数。

将这些包装在组件中既能很方便地简化模板，也允许用户组合和构建自己的控制流。

这些内置的控制流将被自动导入。除了 `Portal` 和 `Dynamic` 之外的所有内容都是从 `solid-js` 导出的。这两个 DOM 特定的组件由 `solid-js/web` 导出。

> Note: All callback/render function children of control flow are non-tracking. This allows for nesting state creation, and better isolates reactions.
> 
> _注意：控制流的所有回调/渲染函数子项都是非跟踪性的。这允许创建嵌套状态，并更好地隔离响应。_

### `<For>`

```typescript
export function For<T, U extends JSX.Element>(props: {
  each: readonly T[];
  fallback?: JSX.Element;
  children: (item: T, index: () => number) => U;
}): () => U[];
```

简单的引用键控循环控制流程。

```typescript
<For each={state.list} fallback={<div>Loading...</div>}> {(item) => <div>{item}</div>} </For>;
```

Optional second argument is an index signal:

第二个可选参数是索引:

```typescript
<For each={state.list} fallback={<div>Loading...</div>}> {(item, index) => ( <div> #{index()} {item} </div> )} </For>;
```

### `<Show>`

```typescript
function Show<T>(props: { when: T | undefined | null | false; fallback?: JSX.Element; children: JSX.Element | ((item: T) => JSX.Element); }): () => JSX.Element;
```

The Show control flow is used to conditional render part of the view: it renders `children` when the `when` is truthy, an `fallback` otherwise. It is similar to the ternary operator (`when ? children : fallback`) but is ideal for templating JSX.

Show 控制流用于有条件地渲染视图的一部分。它跟三元运算符（`a ? b : c`）类似，但非常适合模板 JSX。

```typescript
<Show when={state.count > 0} fallback={<div>Loading...</div>}> <div>My Content</div> </Show>;
```

Show can also be used as a way of keying blocks to a specific data model. Ex the function is re-executed whenever the user model is replaced.

Show 还可以用来将区块控到特定数据模型。每当用户数据模型被替换时，该函数就会重新执行。

```typescript
<Show when={state.user} fallback={<div>Loading...</div>}> {(user) => <div>{user.firstName}</div>} </Show>;
```

### `<Switch>`/`<Match>`

```typescript
export function Switch(props: { fallback?: JSX.Element; children: JSX.Element; }): () => JSX.Element; type MatchProps<T> = { when: T | undefined | null | false; children: JSX.Element | ((item: T) => JSX.Element); }; export function Match<T>(props: MatchProps<T>);
```

Useful for when there are more than 2 mutual exclusive conditions. Can be used to do things like simple routing.

当有 2 个以上的互斥条件时会很有用。可以用来做一些简单的路由之类的事情。

```typescript
<Switch fallback={<div>Not Found</div>}> <Match when={state.route === "home"}> <Home /> </Match> <Match when={state.route === "settings"}> <Settings /> </Match> </Switch>;
```

Match also supports function children to serve as keyed flow.

Match 还支持函数子项作为键映射流程。

### `<Index>`

```typescript
export function Index<T, U extends JSX.Element>(props: { each: readonly T[]; fallback?: JSX.Element; children: (item: () => T, index: number) => U; }): () => U[];
```

Non-keyed list iteration (rows keyed to index). This is useful when there is no conceptual key, like if the data consists of primitives and it is the index that is fixed rather than the value.The item is a signal:

`<Index>` 在无 key 列表迭代时（或者说 index 作为 key）时很有用，比如数据是 primitive 并且是固定的索引而不是值。该项是一个 signal：

```typescript
<Index each={state.list} fallback={<div>Loading...</div>}> {(item) => <div>{item()}</div>} </Index>;
```

Optional second argument is an index number:

第二个可选参数是索引号：

```typescript
<Index each={state.list} fallback={<div>Loading...</div>}> {(item, index) => ( <div> #{index} {item()} </div> )} </Index>;
```

### `<ErrorBoundary>`

```typescript
function ErrorBoundary(props: { fallback: JSX.Element | ((err: any, reset: () => void) => JSX.Element); children: JSX.Element; }): () => JSX.Element;
```

Catches uncaught errors and renders fallback content.

捕获未捕获的错误并渲染回退内容。

```typescript
<ErrorBoundary fallback={<div>Something went terribly wrong</div>}> <MyComp /> </ErrorBoundary>
```

Also supports callback form which passes in error and a reset function.

还支持回调函数的形式传参，函数传入了错误和重置函数。

```typescript
<ErrorBoundary fallback={(err, reset) => <div onClick={reset}>Error: {err}</div>} > <MyComp /> </ErrorBoundary>;
```

### `<Suspense>`

```typescript
export function Suspense(props: { fallback?: JSX.Element; children: JSX.Element; }): JSX.Element;
```

A component that tracks all resources read under it and shows a fallback placeholder state until they are resolved. What makes `Suspense` different than `Show` is it is non-blocking in that both branches exist at the same time even if not currently in the DOM.

`<Suspense>` 是一个跟踪其下所有读取资源并显示回退占位符状态的组件，直到它们被解析。`Suspense` 与 `Show` 的不同之处在于它是非阻塞的，即使当前不在 DOM 中，两个分支也可以同时存在。

```typescript
<Suspense fallback={<div>Loading...</div>}> <AsyncComponent /> </Suspense>
```

### `<SuspenseList>` (Experimental)

```typescript
function SuspenseList(props: { children: JSX.Element; revealOrder: "forwards" | "backwards" | "together"; tail?: "collapsed" | "hidden"; }): JSX.Element;
```

`SuspenseList` allows for coordinating multiple parallel `Suspense` and `SuspenseList` components. It controls the order in which content is revealed to reduce layout thrashing and has an option to collapse or hide fallback states.

`SuspenseList` 可以协调多个并行的 `Suspense` 和 `SuspenseList` 组件。它控制显示内容的顺序以减少布局抖动，并且可以通过选项控制折叠或隐藏回退状态。

```typescript
<SuspenseList revealOrder="forwards" tail="collapsed"> <ProfileDetails user={resource.user} /> <Suspense fallback={<h2>Loading posts...</h2>}> <ProfileTimeline posts={resource.posts} /> </Suspense> <Suspense fallback={<h2>Loading fun facts...</h2>}> <ProfileTrivia trivia={resource.trivia} /> </Suspense> </SuspenseList>
```

SuspenseList is still experimental and does not have full SSR support.

SuspenseList 仍处于试验阶段，并没有完整的 SSR 支持。

### `<Dynamic>`

```typescript
function Dynamic<T>( props: T & { children?: any; component?: Component<T> | string | keyof JSX.IntrinsicElements; } ): () => JSX.Element;
```

This component lets you insert an arbitrary Component or tag and passes the props through to it.

该组件允许你插入任意组件或标签并将 props 传递给它。

```typescript
<Dynamic component={state.component} someProp={state.something} />
```

### `<Portal>`

```typescript
export function Portal(props: { mount?: Node; useShadow?: boolean; isSVG?: boolean; children: JSX.Element; }): Text;
```

This inserts the element in the mount node. Useful for inserting Modals outside of the page layout. Events still propagate through the Component Hierarchy.The portal is mounted in a `<div>` unless the target is the document head. `useShadow` places the element in a Shadow Root for style isolation, and `isSVG` is required if inserting into an SVG element so that the `<div>` is not inserted.

`<Portal>` 会在挂载节点中插入元素。用于在页面布局之外插入模态框。事件仍然通过组件层次结构传播。

除非目标是 document head，否则 portal 挂载在`<div>` 中。`useShadow` 将元素放在 Shadow Root 中以进行样式隔离，如果插入到 SVG 元素中，则需要 `isSVG` 避免不插入 `<div>`。

```typescript
<Portal mount={document.getElementById("modal")}> <div>My Content</div> </Portal>
```

## Special JSX Attributes 特殊的 JSX 属性

In general Solid attempts to stick to DOM conventions. Most props are treated as attributes on native elements and properties on Web Components, but a few of them have special behavior.For custom namespaced attributes with TypeScript you need to extend Solid's JSX namespace:

一般来说，Solid 试图和 DOM 习惯保持一致。大多数 props 被视为原生元素的属性和 Web Components 的属性，但其中一些具有特殊的行为。使用 TypeScript 自定义命名空间属性时，你需要扩展 Solid 的 JSX 命名空间：

```typescript
declare module "solid-js" { namespace JSX { interface Directives { // use:____ } interface ExplicitProperties { // prop:____ } interface ExplicitAttributes { // attr:____ } interface CustomEvents { // on:____ } interface CustomCaptureEvents { // oncapture:____ } } }
```

### `ref`

Refs are a way of getting access to underlying DOM elements in our JSX. While it is true one could just assign an element to a variable, it is more optimal to leave components in the flow of JSX. **Refs are assigned at render time but before the elements are connected to the DOM**. They come in 2 flavors.

Refs 是一种访问 JSX 中底层 DOM 元素的方式。虽然确实可以将一个元素分配给一个变量，但将组件留在 JSX 流中更为理想。Refs 在渲染时（在元素连接到 DOM 之前）分配。它有 2 种写法。

```typescript
// 简单赋值 let myDiv; // 连接到 DOM 后使用 onMount 或 createEffect 进行读取 onMount(() => console.log(myDiv)); <div ref={myDiv} /> // 或者，使用回调函数（在元素连接到 DOM 之前调用） <div ref={el => console.log(el)} />
```

Refs can also be used on Components. They still need to be attached on the otherside.

Refs 也可以用于组件。它们仍然需要连接到另一侧。

```typescript
function MyComp(props) { return <div ref={props.ref} />; } function App() { let myDiv; onMount(() => console.log(myDiv.clientWidth)); return <MyComp ref={myDiv} />; }
```

### `classList`

A helper that leverages `element.classList.toggle`. It takes an object whose keys are class names and assigns them when the resolved value is true.

`classList` 借助于 `element.classList.toggle`。它接受一个键为 class 名的对象，并在解析值为 true 时分配它们。

```typescript
<div classList={{ active: state.active, editing: state.currentId === row.id }} />
```

### `style`

Solid's style helper works with either a string or with an object. Unlike React's version Solid uses `element.style.setProperty` under the hood. This means support for CSS vars, but it also means we use the lower, dash-case version of properties. This actually leads to better performance and consistency with SSR output.

Solid 的样式工具可以处理字符串或对象。与 React 的版本不同，Solid 在底层使用了 `element.style.setProperty`。这意味着支持 CSS 变量，但也意味着我们使用较底层的、破折号版本的属性。这实际上会带来更好的性能并能 SSR 输出保持一致。

```typescript
// 字符串 <div style={`color: green; background-color: ${state.color}; height: ${state.height}px`} /> // 变量 <div style={{ color: "green", "background-color": state.color, height: state.height + "px" }} /> // css 变量 <div style={{ "--my-custom-color": state.themeColor }} />
```

### `innerHTML`/`textContent`

These work the same as their property equivalent. Set a string and they will be set. **Be careful!!** Setting `innerHTML` with any data that could be exposed to an end user as it could be a vector for malicious attack. `textContent` while generally not needed is actually a performance optimization when you know the children will only be text as it bypasses the generic diffing routine.

它们的工作原理与它们的等效属性相同。设置一个字符串，它们将被设置到 HTML 中。**小心!!** 任何数据设置为 `innerHTML` 都可能暴露给终端用户，因此它可能成为恶意攻击的载体。`textContent` 虽然通常不需要，但实际上是一种性能优化，因为它绕过了通用对比差异例程，因此子项将只是文本。

```typescript
<div textContent={state.text} />
```

### `on___`

Event handlers in Solid typically take the form of `onclick` or `onClick` depending on style. The event name is always lowercased. Solid uses semi-synthetic event delegation for common UI events that are composed and bubble. This improves performance for these common events.

Solid 中的事件处理程序通常采用 `onclick` 或 `onClick` 形式，具体取决于风格。事件名称总是小写。Solid 对组合和冒泡的常见 UI 事件使用半合成事件委托。这样提高了这些常见事件的性能。

```typescript
<div onClick={(e) => console.log(e.currentTarget)} />
```

Solid also supports passing an array to the event handler to bind a value to the first argument of the event handler. This doesn't use `bind` or create an additional closure, so it is highly optimized way delegating events.

Solid 还支持将数组传递给事件处理句柄以将值绑定到事件处理句柄的第一个参数。这不用使用`bind` 或创建额外的闭包，因此它是一种高度优化的事件委托方式。

```typescript
function handler(itemId, e) { /*...*/ } <ul> <For each={state.list}>{(item) => <li onClick={[handler, item.id]} />}</For> </ul>;
```

Events cannot be rebound and the bindings are not reactive. The reason is that it is generally more expensive to attach/detach listeners. Since events naturally are called there is no need for reactivity simply shortcut your handler if desired.

事件不能被重新绑定并且绑定不是响应式。原因是添加/移除侦听器通常更消耗性能。由于事件自然地会被调用，因此不需要响应性，如果需要，只需跟下面一样简单处理你的事件句柄。

```typescript
// if defined call it, otherwised don't. <div onClick={() => props.handleClick?.()} />
```

### `on:___`/`oncapture:___`

For any other events, perhaps ones with unusual names, or ones you wish not to be delegated there are the `on` namespace events. This simply adds an event listener verbatim.

对其他的事件，可能是名称不寻常或者你不希望被委托，且有 `on` 命名空间。那你只需要逐字添加事件侦听器。

```typescript
<div on:Weird-Event={(e) => alert(e.detail)} />
```

### `use:___`

These are custom directives. In a sense this is just syntax sugar over ref but allows us to easily attach multiple directives to a single element. A directive is simply a function with the following signature:

`use:___` 是自定义指令。从某种意义上说，这只是 ref 上的语法糖，但允许我们轻松地将多个指令附加到单个元素。指令只是一个具有以下签名的函数：

```typescript
function directive(element: Element, accessor: () => any): void;
```

Directive functions are called at render time but before being added to the DOM. You can do whatever you'd like in them including create signals, effects, register clean-up etc.

这些函数在渲染时运行，你可以在其中执行任何操作。创建 signal 和 effects，注册清理函数，随心所欲。

```typescript
const [name, setName] = createSignal(""); function model(el, value) { const [field, setField] = value(); createRenderEffect(() => (el.value = field())); el.addEventListener("input", (e) => setField(e.target.value)); } <input type="text" use:model={[name, setName]} />;
```

To register with TypeScript extend the JSX namespace.

注册 TypeScript 扩展 JSX 命名空间。

```typescript
declare module "solid-js" { namespace JSX { interface Directives { model: [() => any, (v: any) => any]; } } }
```

### `prop:___`

Forces the prop to be treated as a property instead of an attribute.

强制将 prop 视为 property 而不是 attribute。

```typescript
<div prop:scrollTop={props.scrollPos + "px"} />
```

### `attr:___`

Forces the prop to be treated as a attribute instead of an property. Useful for Web Components where you want to set attributes.

强制将 prop 视为 attribute 而不是 property。对于要设置 attribute 的 Web 组件很有用。

```typescript
<my-element attr:status={props.status} />
```

### `/* @once */`

Solid's compiler uses a simple heuristic for reactive wrapping and lazy evaluation of JSX expressions. Does it contain a function call, a property access, or JSX? If yes we wrap it in a getter when passed to components or in an effect if passed to native elements.Knowing this we can reduce overhead of things we know will never change simply by accessing them outside of the JSX. A simple variable will never be wrapped. We can also tell the compiler not to wrap them by starting the expression with a comment decorator `/_ @once _/`.

Solid 的编译器使用简单的启发式方法对 JSX 表达式进行响应式包装和惰性求值。判断它是否包含函数调用、属性访问或 JSX？ 如果是，我们在传递给组件时将其包装在 getter 中，或者如果传递给原生元素，则将其包装在 effect 中。

知道了这一点，我们可以通过在 JSX 之外访问它们来减少我们知道永远不会改变的东西的开销。一个简单的变量永远不会被包装。我们还可以通过以注释修饰符 `/_ @once _/` 开头的表达式来告诉编译器不要包装它们。

```typescript
<MyComponent static={/*@once*/ state.wontUpdate} />
```

This also works on children.

对 children 同样有效。

```typescript
<MyComponent>{/*@once*/ state.wontUpdate}</MyComponent>
```

## 参考链接

-   [Solid 官网](https://www.solidjs.com/guides/getting-started)

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
