---
date: 2022-01-01
title: 栈溢出（ Monday, September 6, 2021） - Mr. Panda
# category: 
# tags: 
# description:
---

# 栈溢出（ Monday, September 6, 2021） - Mr. Panda

---
栈溢出系列文章——解决技术问题，积累开发经验。

本期内容包括：React：Updating State From Properties With React Hooks、CSS：flex布局设置宽度width不生效的解决办法、Javascript：Expressions and operators等。

## React：Updating State From Properties With React Hooks

如果你想要在 FC 组件内通过 props 来更新 state，有如下的方法可以达到目的，但是其中一些可能存在一些陷阱。

首先我们知道 React 并不会让 state 随 props 变化而更新。我们先来回忆一下类组件的解决场景，声明周期函数 componentWillReceiveProps 在很长一段时间内曾是随 props 变化而更新 state 而不需要中心渲染的唯一方法，知道 React 16.3 中静态函数 getDerivedStateFromProps 的出现成为更加安全的解决方式。

### 不要使用 useEffect

使用 useEffect 来跟踪 props 的变化并更新 state 看起来是一种简洁而优雅的方式，但是这种方式成本太高，因为在内部 state 变化时引起渲染后，useEffect 中的 setInternalState 会引起应用重新渲染。

### 追踪上一次 state 的值：有效却不那么高成本

React 官方提供了一种 FC 中实现 `getDerivedStateFromProps` 的方法，参见：[How do I implement `getDerivedStateFromProps`?](https://reactjs.org/docs/hooks-faq.html#how-do-i-implement-getderivedstatefromprops)

```javascript
function ScrollView({ row }) {
  const [isScrollingDown, setIsScrollingDown] = useState(false);
  const [prevRow, setPrevRow] = useState(null);

  if (row !== prevRow) {
    // Row changed since last render. Update isScrollingDown.
    setIsScrollingDown(prevRow !== null && row > prevRow);
    setPrevRow(row);
  }

  return `Scrolling down: ${isScrollingDown}`;
}
```

### usePrevious：最低成本的方式

React 官方提供一种更好的解决方案，参见：[How to get the previous props or state?](https://reactjs.org/docs/hooks-faq.html#how-to-get-the-previous-props-or-state)。

```javascript
function Component({ value }) {
  const [internalState, setInternalState] = useState(value);

  const previousValueRef = useRef();
  const previousValue = previousValueRef.current;
  if (value !== previousValue && value !== internalState) {
    setInternalState(value);
  }

  useEffect(() => {
    previousValueRef.current = value;
  });

  return (
    <div>
      <p>Provided value: {value}</p>
      <p>
        <label>
          User updated value:
          <input
            type="text"
            value={internalState}
            onChange={(e) => setInternalState(e.currentTarget.value)}
          />
        </label>
      </p>
    </div>
  );
}
```

### useUpdatableState

参见：landisdesign/**[use-updatable-state](https://github.com/landisdesign/use-updatable-state)**；

参考：

-   React 官方博客：[You Probably Don't Need Derived State](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)；
-   React 官方文档：`[static getDerivedStateFromProps()](https://reactjs.org/docs/react-component.html#static-getderivedstatefromprops)`；

```typescript
import { Dispatch, SetStateAction, useEffect, useRef, useState } from 'react';

/**
 * `useState` that updates based on changes to provided value. If `value`
 * changes, the `state` value returned by `useUpdatableState` will reflect the
 * new value. A third array element, `changed`, will be set to `true` for the
 * render cycle that caused the change, as a snapshot indicator that a change
 * took place. Future render cycles will not retain this `true` value.
 * @param value The external value that initially populates `state` and from
 *        where `state` receives updates.
 * @param predicate An optional function (`(value, previous): boolean`) to
 *        compare the current and previous `value` properties. If it returns
 *        `false`, `state` will be updated and `changed` will be `true` until
 *        the component is rendered. If not provided, the new and previous
 *        values will be compared by a strict identity comparison (`===`).
 * @returns `[state, setState, changed]` where `state` and `setState` behave
 *          exactly like with the `useState` hook, including guarantees that
 *          `setState` will not change. `change` will be `true` when `value`
 *          changes between renders.
 */
function useUpdatableState<T>(value: T, predicate: StateComparator<T> = defaultComparator): UpdatableResult<T> {
  // I am explicitly choosing to modify the array returned by useState instead
  // of spreading its contents into a new array, in case React makes any
  // guarantees about the array's identity.
  const stateArray: UpdatableResult<T> = useState(value) as any;
  const previousValueRef = useRef(value);
  const [isChanged, setChanged] = useState(true);

  useEffect(() => {
    previousValueRef.current = value;
  });

  if (!predicate(value, previousValueRef.current) && !predicate(value, stateArray[0])) {
    setChanged(true);
    stateArray[1](value);
  } else {
    if (isChanged) {
      setChanged(false);
    }
  }
  stateArray[2] = isChanged;
  return stateArray;
}

export default useUpdatableState;

const defaultComparator = <T>(a: T, b: T) => a === b;

export interface StateComparator<T> {
  (a: T | undefined, b: T | undefined): boolean;
}

export type UpdatableResult<T> = [T, Dispatch<SetStateAction<T>>, boolean];
```

尝试给另一个元素设置 flex:1。

参见：[flex布局设置宽度width不生效的解决办法](https://www.jiangruitao.com/css/flex-width-not-work/)；

## Javascript：Expressions and operators

参见：MDN：[Expressions and operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators)；

## Javascript：Optional chaining：?.()，?.\[\]

-   OC 不是一个操作符，而是一个特别的语法糖，所以可以和函数调用和中括号共用。
-   `?.()`可以用于执行一个可能不存在的函数。
-   `?.`语法同样可以在当我们需要通过中括号去访问属性时使用，使用它可以安全的访问一个或许还不存在的对象的属性

参见：[JavaScript 新提案：optional chaining（可选链）](https://segmentfault.com/a/1190000023263123)；

## Javascript：js选中文字和获取光标的几种常用方法

参见：

-   [js选中文字和获取光标的几种常用方法](https://juejin.cn/post/6844904038991921166)；
-   MDN： experimental：[Selection Api](https://developer.mozilla.org/en-US/docs/Web/API/Selection);

## Git：remove git cache 删除git缓存

```bash
remove git cache

# remove specific file from git cache
git rm --cached filename

# remove all files from git cache
git rm -r --cached .
git add .
git commit -m ".gitignore is now working"
```

三击选段会选中段落末尾的换行符，因此在粘贴到 bash 等终端时会自动执行代码，粘贴到 input 等元素中末尾会出现一个空格。

三击时会触发一次双击，这种问题如何解决？可以在双击后等待 500s m，判断之后是否发生三击。

参考：[优雅的区分浏览器的双击选词与三击选段](https://github.com/lmk123/blog/issues/78)；

## React：React Portals

-   什么是Portal？React Portal 提供了一种**将子节点渲染到父组件以外的 DOM 节点**的解决方案。
-   Portal 的应用场景？模态对话框（Modal）、工具提示（Tooltip）、悬浮卡（Popover）、加载动画等。
-   使用方法：使用 ReactDOM.createPortal(child,container) 创建一个 Portal。这里的 child 是一个 ReactNode，container 是 Portal 要插入的 DOM 节点的位置。
-   注意：即使 Portal 是在父级 DOM 元素之外呈现的，其表现行为也跟平常我们在 React 组件中使用是一样的。它能够接受 props 以及 context API。Portal 驻留在 React Tree 层次结构内（也就是保证在同一颗 React Tree 上）。Potal 中的事件仍然遵守 React Tree 的事件委托机制，即事件冒泡仍然在 React Tree 中。
-   由于这个模态框是在根层次结构之外渲染的，因此模态框的大小不会因为继承父级组件而被更改。

### 使用 React Portals 的注意事项

-   事件冒泡正常工作：通过将事件传播到 React Tree 的祖先，事件冒泡将按预期工作，而与 DOM 中的 Portal 节点位置无关。
-   React 可以控制 Portal 节点及其生命周期：当通过 Portal 渲染子元素时，React 仍然可以控制它们的生命周期。
-   React Portal 只影响 DOM 结构：Portal 只会影响 DOM 结构，而不会影响 React 组件树。
-   预定义的 HTML 挂载点：使用 React Portal 时，你需要提前定义一个 DOM 元素作为 Portal 组件的挂载。

## Array 构造器

-   Array 的构造器函数（可以省略 new 关键字）当传入一个参数时，创建长度为 length 的稀疏数组（ _sparse_ array）,当传入超过一个参数时，就会创建包含这些参数元素的数组。如 _Array(1, 2, 3)_ 创建的就不是稀疏数组。
-   _Function.prototype.apply_ 允许你以数组的方式向函数传递参数，所以调用 Array.apply() 就等同于 _Array.apply(null, \[1, 2, 3\])_。
-   _Array.prototype.map_ 方法处理稀疏数组的方式不同，稀疏数组的 length 值比数组中值的个数要大，map 在循环稀疏数组时会跳过其中的空值，而对 _Function.prototype.apply_ 创建的数组则不会有这种特性。

## KALI：解决 The following signatures were invalid 问题

```bash
wget -q -O - https://archive.kali.org/archive-key.asc | apt-key add
```

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
