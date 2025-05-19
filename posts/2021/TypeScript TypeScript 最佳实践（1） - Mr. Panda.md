---
date: 2022-01-01
title: TypeScript 最佳实践（1） - Mr. Panda
# category: 
# tags: 
# description:
---

# [TypeScript] TypeScript 最佳实践（1） - Mr. Panda

---
这篇文章时 TypeScript 最佳实践系列文章中的第一篇，此系列文章主要涵盖 TS 和 React 应用中的类型规范和编码经验。TypeScript 的出现是 javascript 的第二生命，ts 对于项目的工程化、开发规范性、便捷性和代码的可维护性都有重要的作用。弱类型到强类型的规范，使得 js 在代码的安全性、可维护性、可扩展性上有了巨大的提升。拥抱 ts，就是拥抱了 js 的未来。

## Typing Component Props

### Interface Comments

- 使用 TSDoc **/\*\* comment \*/** 来为你的接口的每一个属性添加注释，尤其是通用组件。优点是：更好的代码提示、开发文档生成（[Docz PropsTable](https://www.docz.site/docs/components-api#propstable)）。
- 添加@标注：**@desc：描述**、**@param: 参数、@default：默认值**、**@note：注意项**等。

### Namespaced Components

- 创建相似的组件或者有父子关系、附属关系的组件，可以为组件创建命名空间。
- 通过 **Object.assign()** 添加命名空间类型。

```javascript
export default Object.assign(Form, { Input }); // 通过 <Form.Input /> 使用 Input 组件
```

```typescript
export declare interface AppProps {
   children: React.ReactNode; // best, accepts everything (see edge case below)
   style?: React.CSSProperties; // to pass through style props
   onChange?: React.FormEventHandler; // form events! the generic parameter is the type of event.target
   props: Props & React.ComponentPropsWithoutRef<"button">; // to impersonate all the props of a button element and explicitly not forwarding its ref
   props2: Props & React.ComponentPropsWithRef; // to impersonate all the props of MyButtonForwardedRef and explicitly forwarding its ref
}
```

- 优先使用 interface。Use Interface until You Need Type。
- 在工具库或者三方库中使用 interface 定义类型，因为 interface 易于扩展。
- TS 官网：Type aliases and interfaces are very similar, and in many cases you can choose between them freely. Almost all features of an `interface` are available in `type`, the key distinction is that **a type cannot be re-opened to add new properties vs an interface which is always extendable.**

## Function Components

### 为什么不建议使用 React.FC?React.FC 和 React.VFC 有何区别？

- 现在的共识是 React.FunctionComponent(React.FC) 是不推荐使用的。
- FC 的具有明确的返回值类型，然而普通函数的返回值不明确的。
- FC 为一些静态属性如 **displayName**、**propTypes** 和 **defaultProps** 提供类型检查和自动补全。已知在 FC 中使用 defaultProps 有一些[已知的问题](https://github.com/typescript-cheatsheets/react-typescript-cheatsheet/issues/87)。
- FC 为 children 提供了明确的定义，但是这样会有一些已知的问题。

### 使用 React.VFC 来代替

如果你想显示定义 children 的类型，可以使用 React.VoidFunctionComponent（React.VFC）。这是在 FC 默认接受没有 children 之前的一个临时方案。

## Hooks

- **如果 state 即将被初始化并且总有一个返回值**，可以在 setState 中使用类型声明。这将暂时欺骗 ts 编译器 {} 是 IUser 类型。你应该随后更新这个状态，否则后面代码的执行依赖于 user 是 IUser 的类型可能会引起运行时错误。

```typescript
const [user, setUser] = React.useState<IUser>({} as IUser);
setUser(newUser);
```

```typescript
const divRef = useRef<HTMLDivElement>(null);
```

- 在 Custom Hooks 中，如果你返回了一个数组，想要避免 ts 的类型推断（ts 将会推断为 union type）希望数组中有不同的类型，可以使用 const 声明。这种方式可以让你在结构数组的时候得到正确的类型。当然，另一种方法就是手动写类型。事实上，React 团队推荐在自定义 hook 中，如果返回值超过两个，应该使用 objects ，而不是数组（tuples）。

```typescript
export function useLoading() {
  const [isLoading, setState] = React.useState(false);
  const load = (aPromise: Promise<any>) => {
    setState(true);
    return aPromise.finally(() => setState(false));
  };
  return [isLoading, load] as const; // infers [boolean, typeof load] instead of (boolean | typeof load)[]
}
```

- 在 ts 中，React.Component 是一个泛型（generic type）（`React.Component<PropType, StateType>`）。初始化 state 时二次声明 state 的类型，有利于类型推断。这是因为 state 的第二次泛型参数可以使 this.setState() 正常工作，因为 setState 来自于基类（React.Component），但是我们在初始化 state 时重写了基类的实现，所以二次声明 state 类型是为了告诉 ts 编译器类型没变。

## Forms and Events

- 如果不考虑性能的话，可以使用 inlining hanlders 来进行类型推断和上下文推断。
- 如果使用分开的 handlers 时，可以为 e 标注类型，当然也可以使用 React 提供的 handler 类型直接为 hander 标志类型。

```typescript
const onChange = (e: React.FormEvent<HTMLInputElement>): void => {
    this.setState({ text: e.currentTarget.value });
};

const onChange: React.ChangeEventHandler<HTMLInputElement> = (e) => {
    this.setState({text: e.currentTarget.value})
}
```

### 事件类型表

| Event Type         | Description                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| AnimationEvent     | CSS Animations. css动画                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ChangeEvent        | Changing the value of `<input>`, `<select>` and `<textarea>` element.                                                                                                                                                                                                                                                                                                                                                                            |
| ClipboardEvent     | Using copy, paste and cut events. 使用复制、粘贴和剪切事件                                                                                                                                                                                                                                                                                                                                                                                       |
| CompositionEvent   | Events that occur due to the user indirectly entering text (e.g. depending on Browser and PC setup, a popup window may appear with additional characters if you e.g. want to type Japanese on a US Keyboard) 由于用户间接输入文本而发生的事件(例如，根据浏览器和 PC 设置，如果你想在美国键盘上输入日文，弹出窗口可能会显示附加字符)                                                                                                              |
| DragEvent          | Drag and drop interaction with a pointer device (e.g. mouse). 使用指针设备(如鼠标)进行拖放交互                                                                                                                                                                                                                                                                                                                                                   |
| FocusEvent         | Event that occurs when elements gets or loses focus. 当元素获得或失去焦点时发生的事件                                                                                                                                                                                                                                                                                                                                                            |
| FormEvent          | Event that occurs whenever a form or form element gets/loses focus, a form element value is changed or the form is submitted. 当窗体或窗体元素获得/失去焦点、窗体元素值发生更改或窗体提交时发生的事件                                                                                                                                                                                                                                            |
| InvalidEvent       | Fired when validity restrictions of an input fails (e.g `<input type="number" max="10">` and someone would insert number 20). 当输入的有效性限制失败时触发                                                                                                                                                                                                                                                                                       |
| KeyboardEvent      | User interaction with the keyboard. Each event describes a single key interaction. 用户与键盘的交互。每个事件描述一个单键交互                                                                                                                                                                                                                                                                                                                    |
| MouseEvent         | Events that occur due to the user interacting with a pointing device (e.g. mouse) 由于用户与指向设备(例如鼠标)交互而发生的事件                                                                                                                                                                                                                                                                                                                   |
| PointerEvent       | Events that occur due to user interaction with a variety pointing of devices such as mouse, pen/stylus, a touchscreen and which also supports multi-touch. Unless you develop for older browsers (IE10 or Safari 12), pointer events are recommended. Extends UIEvent. 由于用户与各种设备(如鼠标、笔/触笔、触摸屏)进行交互而发生的事件，该设备还支持多点触摸。除非您是为较旧的浏览器(IE10或 Safari 12)开发的，建议使用指针事件。继承自 UIEvent。 |
| TouchEvent         | Events that occur due to the user interacting with a touch device. Extends UIEvent. 由于用户与触摸设备交互而发生的事件                                                                                                                                                                                                                                                                                                                           |
| TransitionEvent    | CSS Transition. Not fully browser supported. Extends UIEvent. CSS 变换。不完全支持所有浏览器。继承自 UIEvent。                                                                                                                                                                                                                                                                                                                                   |
| UIEvent            | Base Event for Mouse, Touch and Pointer events. 鼠标、触摸和指针事件的基本事件                                                                                                                                                                                                                                                                                                                                                                   |
| WheelEvent         | Scrolling on a mouse wheel or similar input device. 在鼠标滚轮或类似的输入设备上滚动。(`wheel` event should not be confused with the `scroll` event)                                                                                                                                                                                                                                                                                             |
| **SyntheticEvent** | **The base event for all above events. Should be used when unsure about event type** 上述所有事件的基本事件。在不确定事件类型时应使用                                                                                                                                                                                                                                                                                                            |

React 事件类型表

## Context

- 通过 helper 函数 createCtx 来防备访问 Context 时 Context 未被 Provider 提供的情况。这样的话我们就不用为 createContect 提供一个初始值，并且也不用检查 Context 为 undefined 的情况。（默认 Context 将会被提供，且初始值与业务无关）。

```typescript
import * as React from "react";

/**
 * A helper to create a Context and Provider with no upfront default value, and
 * without having to check for undefined all the time.
 */
function createCtx<A extends {} | null>() {
  const ctx = React.createContext<A | undefined>(undefined);
  function useCtx() {
    const c = React.useContext(ctx);
    if (c === undefined)
      throw new Error("useCtx must be inside a Provider with a value");
    return c;
  }
  return [useCtx, ctx.Provider] as const; // 'as const' makes TypeScript infer a tuple
}

// Usage:

// We still have to specify a type, but no default!
export const [useCurrentUserName, CurrentUserProvider] = createCtx<string>();

function EnthusasticGreeting() {
  const currentUser = useCurrentUserName();
  return <div>HELLO {currentUser.toUpperCase()}!</div>;
}

function App() {
  return (
    <CurrentUserProvider value="Anders">
      <EnthusasticGreeting />
    </CurrentUserProvider>
  );
}
```

```typescript
export function createCtx<A>(defaultValue: A) {
  type UpdateType = React.Dispatch<
    React.SetStateAction<typeof defaultValue>
  >;
  const defaultUpdate: UpdateType = () => defaultValue;
  const ctx = React.createContext({
    state: defaultValue,
    update: defaultUpdate,
  });
  function Provider(props: React.PropsWithChildren<{}>) {
    const [state, update] = React.useState(defaultValue);
    return <ctx.Provider value={{ state, update }} {...props} />;
  }
  return [ctx, Provider] as const; // alternatively, [typeof ctx, typeof Provider]
}

// usage

const [ctx, TextProvider] = createCtx("someText");
export const TextContext = ctx;
export function App() {
  return (
    <TextProvider>
      <Component />
    </TextProvider>
  );
}
export function Component() {
  const { state, update } = React.useContext(TextContext);
  return (
    <label>
      {state}
      <input type="text" onChange={(e) => update(e.target.value)} />
    </label>
  );
}
```

```typescript
export function createCtx<StateType, ActionType>(
  reducer: React.Reducer<StateType, ActionType>,
  initialState: StateType,
) {
  const defaultDispatch: React.Dispatch<ActionType> = () => initialState // we never actually use this
  const ctx = React.createContext({
    state: initialState,
    dispatch: defaultDispatch, // just to mock out the dispatch type and make it not optioanl
  })
  function Provider(props: React.PropsWithChildren<{}>) {
    const [state, dispatch] = React.useReducer<React.Reducer<StateType, ActionType>>(reducer, initialState)
    return <ctx.Provider value={{ state, dispatch }} {...props} />
  }
  return [ctx, Provider] as const
}
// usage
const initialState = { count: 0 }
type AppState = typeof initialState
type Action =
  | { type: 'increment' }
  | { type: 'add'; payload: number }
  | { type: 'minus'; payload: number }
  | { type: 'decrement' }

function reducer(state: AppState, action: Action): AppState {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 }
    case 'decrement':
      return { count: state.count - 1 }
    case 'add':
      return { count: state.count + action.payload }
    case 'minus':
      return { count: state.count - action.payload }
    default:
      throw new Error()
  }
}
const [ctx, CountProvider] = createCtx(reducer, initialState)
export const CountContext = ctx

// top level example usage
export function App() {
  return (
    <CountProvider>
      <Counter />
    </CountProvider>
  )
}

// example usage inside a component
function Counter() {
  const { state, dispatch } = React.useContext(CountContext)
  return (
    <div>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'add', payload: 5 })}>+5</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'minus', payload: 5 })}>+5</button>
    </div>
  )
}
```

```typescript
interface ProviderState {
  themeColor: string;
}

interface UpdateStateArg {
  key: keyof ProviderState;
  value: string;
}

interface ProviderStore {
  state: ProviderState;
  update: (arg: UpdateStateArg) => void;
}

const Context = React.createContext({} as ProviderStore); // type assertion on empty object

class Provider extends React.Component<{}, ProviderState> {
  public readonly state = {
    themeColor: "red",
  };

  private update = ({ key, value }: UpdateStateArg) => {
    this.setState({ [key]: value });
  };

  public render() {
    const store: ProviderStore = {
      state: this.state,
      update: this.update,
    };

    return (
      <Context.Provider value={store}>{this.props.children}</Context.Provider>
    );
  }
}

const Consumer = Context.Consumer;
```

- React.forwardRef`<Ref, Props>` 包括 Ref 类型和 Props 的类型，需要注意的是如果被 forwardRef 包裹的内层组件是一个泛型组件，应该在 forwardRef 中声明泛型。

- 从 forwardRef 中获取到的 ref 是可变的，所以你可以给它赋值。你也可以通过 `React.Ref<Ref>` 分配类型 让它不可变。

```typescript
export const FancyButton = React.forwardRef((
  props: Props,
  ref: React.Ref<Ref> // <-- here!
) => (
  <button ref={ref} className="MyClassName" type={props.type}>
    {props.children}
  </button>
));
```

泛型 forwardRef 的类型自动推导存在问题，需要手动帮助 ts 进行泛型类型推导，有如下三种方法可以达到这种目的。

- 类型声明

```typescript
const ClickableList = React.forwardRef(ClickableListInner) as <T>(
  props: ClickableListProps<T> & { ref?: React.ForwardedRef<HTMLUListElement> }
) => ReturnType<typeof ClickableListInner>;
```

尽管 ref 是 React 组件的保留字，你仍然可以使用自定义的 ref 属性来模拟 forwardRef。

```typescript
type ClickableListProps<T> = {
  items: T[],
  onSelect: (item: T) => void,
  mRef?: React.Ref<HTMLUListElement> | null,
};

export function ClickableList<T>(props: ClickableListProps<T>) {
  return (
    <ul ref={props.mRef}>
      {props.items.map((item, i) => (
        <li key={i}>
          <button onClick={(el) => props.onSelect(item)}>Select</button>
          {item}
        </li>
      ))}
    </ul>
  );
}
```

我们可以重新声明和定义全局模块、命名空间、接口的定义，利用声明合并来达到目的。

```typescript
// Redecalare forwardRef
declare module "react" {
  function forwardRef<T, P = {}>(
    render: (props: P, ref: React.Ref<T>) => React.ReactElement | null
  ): (props: P & React.RefAttributes<T>) => React.ReactElement | null;
}
```

参考：[TypeScript + React: Typing Generic forwardRefs](https://fettblog.eu/typescript-react-generic-forward-refs/)；

## Portals

- 类组件 Modal

```typescript
const modalRoot = document.getElementById("modal-root") as HTMLElement;
// assuming in your html file has a div with id 'modal-root';

export class Modal extends React.Component {
  el: HTMLElement = document.createElement("div");

  componentDidMount() {
    modalRoot.appendChild(this.el);
  }

  componentWillUnmount() {
    modalRoot.removeChild(this.el);
  }

  render() {
    return ReactDOM.createPortal(this.props.children, this.el);
  }
}
```

```typescript
import React, { useEffect, useRef } from "react";
import { createPortal } from "react-dom";

const modalRoot = document.querySelector("#modal-root") as HTMLElement;

const Modal: React.FC<{}> = ({ children }) => {
  const el = useRef(document.createElement("div"));

  useEffect(() => {
    // Use this in case CRA throws an error about react-hooks/exhaustive-deps
    const current = el.current;

    // We assume `modalRoot` exists with '!'
    modalRoot!.appendChild(current);
    return () => void modalRoot!.removeChild(current);
  }, []);

  return createPortal(children, el.current);
};

export default Modal;
```

- [React-error-boundary](https://github.com/bvaughn/react-error-boundary)；

- custom error boundary component

```typescript
import React, { Component, ErrorInfo, ReactNode } from "react";

interface Props {
  children: ReactNode;
}

interface State {
  hasError: boolean;
}

class ErrorBoundary extends Component<Props, State> {
  public state: State = {
    hasError: false
  };

  public static getDerivedStateFromError(_: Error): State {
    // Update state so the next render will show the fallback UI.
    return { hasError: true };
  }

  public componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error("Uncaught error:", error, errorInfo);
  }

  public render() {
    if (this.state.hasError) {
      return <h1>Sorry.. there was an error</h1>;
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

参考：

- React 官网：[Introducing Concurrent Mode (Experimental)](https://reactjs.org/docs/concurrent-mode-intro.html)；

- [理解 React 的下一步：Concurrent Mode 與 Suspense](https://chentsulin.medium.com/%E7%90%86%E8%A7%A3-react-%E7%9A%84%E4%B8%8B%E4%B8%80%E6%AD%A5-concurrent-mode-%E8%88%87-suspense-327b8a3df0fe)；

## 参考资料

- [React + TypeScript 实践](https://mp.weixin.qq.com/s/mUblBpj6pmdxz9mLKEDJTw)；
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/)；
- sw-yx/**[createCtx-noNullCheck.tsx](https://gist.github.com/sw-yx/f18fe6dd4c43fddb3a4971e80114a052)**；
- sw-yx/**[concurrent-react-notes](https://github.com/sw-yx/concurrent-react-notes)**;

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
