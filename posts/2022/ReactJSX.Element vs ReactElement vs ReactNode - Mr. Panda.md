---
date: 2022-01-01
title: JSX.Element vs ReactElement vs ReactNode - Mr. Panda
# category: 
# tags: 
# description:
---

# [React]JSX.Element vs ReactElement vs ReactNode - Mr. Panda

---
JSX.Element、ReactElement、ReactNode这三种类型通常会使新手 React 开发人员感到困惑。似乎它们是同一个东西，只是名称不同。但这并不完全正确。下面详细对比下这三种类型。

## `JSX.Element`对比`ReactElement`

这两种类型都是`React.createElement()`/`jsx()`函数调用的结果。

它们都是具有以下特征的对象：

-   type
-   props
-   key
-   一些其他“隐藏”属性，如 ref、$$typeof 等

### `ReactElement`

`ReactElement`类型是最基本的。它在 React 源代码中使用 flow 定义。

```javascript
// ./packages/shared/ReactElementType.js export type ReactElement = {| $$typeof: any, type: any, key: any, ref: any, props: any, // ReactFiber _owner: any, // __DEV__ _store: {validated: boolean, ...}, _self: React$Element<any>, _shadowChildren: any, _source: Source, |};
```

这种类型也在[DefiniteTyped 包中](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts#L146)定义。

```javascript
interface ReactElement<P = any, T extends string | JSXElementConstructor<any> = string | JSXElementConstructor<any>> { type: T; props: P; key: Key | null; }
```

### `JSX.Element`

它是更通用的类型。[主要区别在于`props`和`type`的类型`为any`](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts#L3078)。

```javascript
declare global { namespace JSX { interface Element extends React.ReactElement<any, any> { } // ... } }
```

这为不同库如何实现 JSX 提供了灵活性。例如，[Preact 有自己的实现，具有不同的 API](https://github.com/preactjs/preact/blob/master/src/jsx.d.ts#L18)。

## `ReactNode`

`ReactNode`类型是另一回事。它不是`React.createElement()`/`jsx()`函数调用的返回值。

```javascript
const Component = () => { // ReactElement return <div>Hello world!</div> } // ReactNode const Example = Component();
```

React 节点本身就是虚拟 DOM 的一种表示。`ReactNode`组件的所有可能返回值的集合也是如此。

```javascript
type ReactChild = ReactElement | ReactText; type ReactFragment = {} | Iterable<ReactNode>; interface ReactPortal extends ReactElement { key: Key | null; children: ReactNode; } type ReactNode = | ReactChild | ReactFragment | ReactPortal | boolean | null | undefined;
```

## `children`用什么类型？

一般来说，`ReactNode`是正确的`children`的类型。它提供了最大的灵活性，同时保持正确的类型检查。但它有一个警告，因为`ReactFragment`允许一个`{}`类型。

```javascript
const Item = ({ children }: { children: ReactNode }) => { return <li>{children}</li>; } const App = () => { return ( <ul> // Run-time error here, objects are not valid children! <Item>{{}}</Item> </ul> ); }
```

在使用 TypeScript 和 React 时，React 已经为带有子组件的组件提供了类型，因此也可以不必自己传入类型。所以也可以：

```javascript
import type { FC } from "react"; const Item: FC = ({ children }) => <li>{children}</li>;
```

如果你想变得更灵活，你可以使用`JSX.IntrinsicElements` “扩展”类型：

```javascript
import type { FC } from "react"; const Item: FC<JSX.IntrinsicElements["li"]> = props => <li {...props} />;
```

如果使用没有子元素的元素（如`input`），则可以使用`VFC`而不是`FC`，如下所示：

```javascript
import type { VFC } from "react"; const Input: VFC<JSX.IntrinsicElements["input"]> = props => ( <input {...props} /> );
```

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
