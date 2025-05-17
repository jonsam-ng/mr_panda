---
date: 2022-01-01
title: React18 来啦，看看有哪些新特性！ - Mr. Panda
# category: 
# tags: 
# description:
---

# [React] React18 来啦，看看有哪些新特性！ - Mr. Panda

---
在这篇文章中，我们将概述 React 18 的新特性，以及它对未来的意义。

我们最新的主要版本包括开箱即用的改进，如自动批处理、新的 API（如 startTransition）和支持 Suspense 的流式服务器端渲染。

React 18 中的许多功能都建立在我们新的并发渲染器之上，这是一个解锁强大新功能的幕后更改。Concurrent React 是可选的——它仅在您使用并发功能时启用——但我们认为它会对人们构建应用程序的方式产生重大影响。

## 什么是 Concurrent React？

React 18 中最重要的新增功能：并发性。我们认为这对于应用程序开发人员来说基本上是正确的，尽管对于库维护人员来说这个故事可能有点复杂。

并发本身不是一个特性。这是一种新的幕后机制，使 React 能够同时准备多个版本的 UI。您可以将并发视为一个实现细节——它之所以有价值，是因为它解锁了一些特性。React 在其内部实现中使用了复杂的技术，例如优先级队列和多重缓冲。但是您不会在我们的公共 API 中的任何地方看到这些概念。

当我们设计 API 时，我们试图向开发人员隐藏实现细节。作为一名 React 开发人员，您专注于您希望用户体验的样子，而 React 负责处理如何提供这种体验。所以我们不期望 React 开发人员知道并发是如何工作的。

然而，Concurrent React 比典型的实现细节更重要——它是对 React 核心渲染模型的基础更新。因此，虽然了解并发的工作原理并不是非常重要，但可能值得从高层次了解它是什么。

Concurrent React 的一个关键属性是渲染是可中断的。当你第一次升级到 React 18 时，在添加任何并发特性之前，更新的渲染方式与之前版本的 React 相同——在一个单一的、不间断的、同步的事务中。使用同步渲染，一旦更新开始渲染，在用户可以在屏幕上看到结果之前，没有任何东西可以中断它。

在并发渲染中，情况并非总是如此。React 可能会开始渲染更新，在中间暂停，然后再继续。它甚至可能完全放弃正在进行的渲染。React 保证即使渲染被中断，UI 也会保持一致。为此，它会等待执行 DOM 突变，直到完成整个树的评估。有了这个能力，React 可以在后台准备新的屏幕而不阻塞主线程。这意味着 UI 可以立即响应用户输入，即使它处于大型渲染任务的中间，从而创造流畅的用户体验。

另一个例子是可重用状态。Concurrent React 可以从屏幕上删除部分 UI，然后在重用之前的状态时将它们添加回来。例如，当用户从一个屏幕上移开并返回时，React 应该能够将前一个屏幕恢复到与之前相同的状态。在即将到来的小版本中，我们计划添加一个名为`<Offscreen>`实现此模式的新组件。同样，您将能够使用 Offscreen 在后台准备新 UI，以便在用户显示之前准备好。

并发渲染是 React 中一个强大的新工具，我们的大多数新功能都是为了利用它而构建的，包括 Suspense、过渡和流式服务器渲染。但是 React 18 只是我们在这个新基础上构建目标的开始。

## 逐渐采用并发特性

从技术上讲，并发渲染是一个突破性的变化。因为并发渲染是可中断的，所以启用它时组件的行为会略有不同。

在我们的测试中，我们已经将数千个组件升级到了 React 18。我们发现几乎所有现有组件都“可以正常工作”并发渲染，没有任何问题。但是，其中一些可能需要一些额外的迁移工作。尽管更改通常很小，但您仍然可以按照自己的节奏进行更改。React 18 中的新渲染行为**仅在应用程序中使用新功能的部分启用。**

整体升级策略是让您的应用程序在 React 18 上运行，而不会破坏现有代码。然后，您可以按照自己的节奏逐渐开始添加并发功能。您可以使用它[`<StrictMode>`](https://reactjs.org/docs/strict-mode.html)来帮助在开发过程中发现与并发相关的错误。严格模式不会影响生产行为，但在开发过程中它会记录额外的警告和预期是幂等的双重调用函数。它不会捕获所有内容，但它可以有效地防止最常见的错误类型。

升级到 React 18 后，您将能够立即开始使用并发功能。例如，您可以使用 startTransition 在屏幕之间导航，而不会阻止用户输入。或者使用DeferredValue 来限制昂贵的重新渲染。

但是，从长远来看，我们希望您向应用程序添加并发的主要方式是使用支持并发的库或框架。在大多数情况下，您不会直接与并发 API 交互。例如，不是开发人员在导航到新屏幕时调用 startTransition，路由器库将自动将导航包装在 startTransition 中。

库升级到并发兼容可能需要一些时间。我们提供了新的 API，使库更容易利用并发功能。同时，在我们逐步迁移 React 生态系统的过程中，请对维护者保持耐心。

## 数据框架中的Suspense

在 React 18 中，您可以开始使用 Suspense 在 Relay、Next.js、Hydrogen 或 Remix 等框架中获取数据。使用 Suspense 获取临时数据在技术上是可行的，但仍不建议将其作为一般策略。

将来，我们可能会公开额外的原语，使您可以更轻松地使用 Suspense 访问您的数据，也许无需使用框架。然而，当 Suspense 深度集成到您的应用程序架构中时效果最好：您的路由器、您的数据层和您的服务器渲染环境。因此，即使从长远来看，我们预计库和框架将在 React 生态系统中发挥至关重要的作用。

与之前的 React 版本一样，您还可以使用 Suspense 在客户端上通过 React.lazy 进行代码拆分。但我们对 Suspense 的愿景始终不仅仅是加载代码——目标是扩展对 Suspense 的支持，以便最终相同的声明式 Suspense 回退可以处理任何异步操作（加载代码、数据、图像等）。

## 服务器组件仍在开发中

[**服务器组件**](https://reactjs.org/blog/2020/12/21/data-fetching-with-react-server-components.html)是一项即将推出的功能，它允许开发人员构建跨服务器和客户端的应用程序，将客户端应用程序的丰富交互性与传统服务器渲染的改进性能相结合。服务器组件本身并不与 Concurrent React 耦合，但它旨在与 Suspense 和流式服务器渲染等并发功能一起工作。

服务器组件仍处于试验阶段，但我们希望在 18.x 次要版本中发布初始版本。与此同时，我们正在使用 Next.js、Hydrogen 和 Remix 等框架来推进该提案并使其为广泛采用做好准备。[](https://reactjs.org/blog/2022/03/29/react-v18.html#whats-new-in-react-18)

## React 18 的新功能

### [](https://reactjs.org/blog/2022/03/29/react-v18.html#new-feature-automatic-batching)新功能：自动批处理

批处理是 React 将多个状态更新分组到一次重新渲染中以获得更好的性能。如果没有自动批处理，我们只能在 React 事件处理程序中批处理更新。默认情况下，Promise、setTimeout、native事件处理程序或任何其他事件内部的更新不会在 React 中批处理。使用自动批处理，这些更新将自动批处理：

```javascript
// Before: only React events were batched. setTimeout(() => { setCount(c => c + 1); setFlag(f => !f); // React will render twice, once for each state update (no batching) }, 1000); // After: updates inside of timeouts, promises, // native event handlers or any other event are batched. setTimeout(() => { setCount(c => c + 1); setFlag(f => !f); // React will only re-render once at the end (that's batching!) }, 1000);
```

有关更多信息，请参阅这篇文章，了解[自动批处理以减少 React 18 中的渲染](https://github.com/reactwg/react-18/discussions/21)。

### 新功能：Transitions（过渡）

过渡是 React 中的一个新概念，用于区分紧急和非紧急更新。

-   **紧急更新**反映了直接交互，例如键入、单击、按下等。
-   **转换更新**将 UI 从一个视图转换到另一个视图。

打字、点击或按下等紧急更新需要立即响应，以符合我们对物理对象行为方式的直觉。否则他们会觉得“不对劲”。但是，Transitions 是不同的，因为用户不希望在屏幕上看到每个中间值。

例如，当您在下拉列表中选择过滤器时，您希望过滤器按钮本身在您单击时立即响应。但是，实际结果可能会单独转换。一个小的延迟将是难以察觉的并且通常是预期的。如果您在结果完成渲染之前再次更改过滤器，您只关心查看最新结果。

通常，为了获得最佳用户体验，单个用户输入应导致紧急更新和非紧急更新。你可以在输入事件中使用 startTransition API 来通知 React 哪些更新是紧急的，哪些是“转换”：

```javascript
import {startTransition} from 'react'; // Urgent: Show what was typed setInputValue(input); // Mark any state updates inside as transitions startTransition(() => { // Transition: Show the results setSearchQuery(input); });
```

包装在 startTransition 中的更新被视为非紧急更新，如果出现更紧急的更新（如点击或按键），则会中断。如果 Transitions 被用户打断（例如，通过连续输入多个字符），React 将淘汰未完成的陈旧渲染工作并仅渲染最新更新。

-   `useTransition`：一个开始转换的钩子，包括一个跟踪挂起状态的值。
-   `startTransition`：当钩子无法使用时开始转换的方法。

过渡将选择并发渲染，这允许更新被中断。如果内容重新挂起，过渡也会告诉 React 继续显示当前内容，同时在后台渲染过渡内容（有关更多信息，请参阅[Suspense RFC](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md)）。

[请参阅此处的Transitions文档](https://reactjs.org/docs/react-api.html#transitions)。[](https://reactjs.org/blog/2022/03/29/react-v18.html#new-suspense-features)

### 新的Suspense功能

如果组件树的一部分尚未准备好显示，Suspense 允许您以声明方式指定组件树的加载状态：

```javascript
<Suspense fallback={<Spinner />}> <Comments /> </Suspense>;
```

Suspense 使“UI 加载状态”成为 React 编程模型中一流的声明性概念。这让我们可以在它之上构建更高级别的功能。

几年前，我们推出了 Suspense 的限量版。但是，唯一受支持的用例是使用 React.lazy 进行代码拆分，并且在服务器上渲染时根本不支持。

在 React 18 中，我们在服务器上添加了对 Suspense 的支持，并使用并发渲染特性扩展了它的功能。

React 18 中的 Suspense 与过渡 API 结合使用时效果最佳。如果你在过渡期间suspense，React 将防止已经可见的内容被替换为后备内容。相反，React 将延迟渲染，直到加载了足够的数据以防止出现错误的加载状态。

有关更多信息，请参阅[React 18 中 Suspense](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md)的 RFC 。

### 新的客户端和服务器渲染 API

在这个版本中，我们借此机会重新设计了我们为在客户端和服务器上呈现而公开的 API。这些更改允许用户在升级到 React 18 中的新 API 时继续使用 React 17 模式下的旧 API。

#### React DOM 客户端

这些新 API 现在从以下位置导出`react-dom/client`：

-   `createRoot render`: mount 或者 unmount 根容器的新方法。使用它代替`ReactDOM.render`. 没有它，React 18 中的新功能就无法工作。
-   `hydrateRoot`: hydrate 服务器渲染应用程序的新方法。使用它而不是 `ReactDOM.hydrate`与新的 React DOM 服务器 API 结合使用。没有它，React 18 中的新功能就无法工作。

两者都接受一个新选项`onRecoverableError`，以防你想在 React 渲染期间的错误中恢复或日志记录时收到通知。默认情况下，在较旧的浏览器中 React 将使用[`reportError`](https://developer.mozilla.org/en-US/docs/Web/API/reportError), 或 `console.error`。

[在此处查看 React DOM 客户端的文档](https://reactjs.org/docs/react-dom-client.html)。

#### React DOM 服务器

这些新的 API 现在从`react-dom/server`服务器上导出并完全支持流式传输 Suspense：

-   `renderToPipeableStream`: 用于 Node 环境中的流式传输。
-   `renderToReadableStream`：适用于现代边缘运行时环境，例如 Deno 和 Cloudflare worker。

现有`renderToString`方法继续有效，但不鼓励使用。

[在此处查看 React DOM 服务器的文档](https://reactjs.org/docs/react-dom-server.html)。

### 新的严格模式行为

将来，我们希望添加一个功能，允许 React 在保留状态的同时添加和删除 UI 部分。例如，当用户从一个屏幕上移开并返回时，React 应该能够立即显示上一个屏幕。为此，React 将使用与以前相同的组件状态卸载和重新安装树。

此功能将为 React 应用程序提供更好的开箱即用性能，但要求组件能够对多次安装和销毁的效果具有弹性。大多数效果无需任何更改即可工作，但有些效果假定它们只安装或销毁一次。

为了帮助解决这些问题，React 18 为严格模式引入了一个新的仅限开发环境的检查。每当第一次安装组件时，此新检查将自动卸载并重新安装每个组件，并在第二次安装时恢复先前的状态。

在此更改之前，React 会挂载组件并创建效果：

```markup
React mounts the component. Layout effects are created. Effects are created.
```

[](https://reactjs.org/blog/2022/03/29/react-v18.html#new-strict-mode-behaviors)使用 React 18 中的严格模式，React 将在开发模式下模拟卸载和重新安装组件：

```
 React mounts the component.
  Layout effects are created.
  Effects are created.
 React simulates unmounting the component.
  Layout effects are destroyed.
  Effects are destroyed.
 React simulates mounting the component with the previous state.
  Layout effects are created.
  Effects are created. 
```

[请参阅文档以确保此处的可重用状态](https://reactjs.org/docs/strict-mode.html#ensuring-reusable-state)。

### 新的 Hook

#### useId

`useId`是一个新的钩子，用于在客户端和服务器上生成唯一 ID，同时避免hydrate不匹配。它主要用于与需要唯一 ID 的可访问性 API 集成的组件库。这解决了 React 17 及更低版本中已经存在的问题，但在 React 18 中更为重要，因为新的流式服务器渲染器如何无序交付 HTML。[请参阅此处的文档](https://reactjs.org/docs/hooks-reference.html#useid)。

#### useTransition

`useTransition`和`startTransition`让您将一些状态更新标记为不紧急。默认情况下，其他状态更新被认为是紧急的。React 将允许紧急状态更新（例如，更新文本输入）以中断非紧急状态更新（例如，呈现搜索结果列表）。[在此处查看文档](https://reactjs.org/docs/hooks-reference.html#usetransition)

#### useDeferredValue

`useDeferredValue`让您推迟重新渲染树的非紧急部分。它类似于去抖动，但与之相比有一些优点。没有固定的时间延迟，因此 React 将在第一次渲染反映在屏幕上后立即尝试延迟渲染。延迟渲染是可中断的，不会阻止用户输入。[请参阅此处的文档](https://reactjs.org/docs/hooks-reference.html#usedeferredvalue)。

#### useSyncExternalStore

`useSyncExternalStore`是一个新的钩子，它允许外部存储通过强制对存储的更新同步来支持并发读取。它在实现对外部数据源的订阅时消除了对 useEffect 的需要，并且推荐给用于与 React 外部状态集成的任何库。[请参阅此处的文档](https://reactjs.org/docs/hooks-reference.html#usesyncexternalstore)。

`useSyncExternalStore`_旨在供库使用，而不是应用程序代码。_

#### useInsertionEffect

`useInsertionEffect`是一个新的钩子，它允许 CSS-in-JS 库解决在渲染中注入样式的性能问题。除非您已经构建了 CSS-in-JS 库，否则我们不希望您使用它。这个钩子将在 DOM 发生变化之后运行，但在布局效果读取新布局之前。这解决了在 React 17 及更低版本中已经存在的问题，但在 React 18 中更为重要，因为 React 在并发渲染期间 yields 给浏览器，使其有机会重新计算布局。[请参阅此处的文档](https://reactjs.org/docs/hooks-reference.html#useinsertioneffect)。

`useInsertionEffect`_旨在供库使用，而不是应用程序代码。_[](https://reactjs.org/blog/2022/03/29/react-v18.html#changelog)

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
