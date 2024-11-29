# React

## hooks

### 为什么hooks不能在条件语句中执行

1. 确保React在每次组件渲染时都能够以相同的顺序调用相同的Hooks
2. 如果Hooks的调用顺序改变，React将无法正确地将组件的状态和副作用与相应的Hooks关联起来

#### hooks机制

**调用顺序和索引**： React通过维护一个内部的“Hooks链表”来跟踪每个组件中Hooks的调用顺序。每次组件渲染时，React会按照相同的顺序调用每个Hook，并使用一个索引来标识每个Hook。这样，React可以确保状态和副作用的一致性。

**状态和副作用的存储**： 每个Hook调用都会创建一个“Hook对象”，这个对象包含了该Hook的状态或副作用。React通过组件的Fiber节点来存储这些对象。

**调度更新**： 当调用`useState`返回的setter函数时（例如`setState`），它会调度一次组件更新，触发重新渲染。React会重新执行组件函数，并通过内部的索引机制确保每个Hook调用返回相应的状态或副作用。

### useEffect 和 useLayoutEffect的区别

`useEffect` 在浏览器完成布局和绘制之后异步调用。这意味着它不会阻塞浏览器更新屏幕。React 先完成 DOM 的更新，然后再运行 `useEffect`。

`useLayoutEffect` 在浏览器执行绘制之前同步调用。这意味着它会在所有 DOM 变更之后立即执行，但在浏览器实际渲染之前。这使得它可以读取布局并同步重新渲染。这会阻塞页面的绘制，直到 effect 执行完毕，因此会使页面的更新显得更“即时”。

*ps: 字多就会阻塞；阻塞原因是先dom变更后，立马执行了*

*pps: 字多就显得长，说明是同步执行的，就会阻塞*

#### 主要区别总结

1. **执行时机**：
   - `useEffect`：在浏览器完成布局和绘制之后异步调用。
   - `useLayoutEffect`：在浏览器执行绘制之前同步调用。
2. **性能影响**：
   - `useEffect`：不会阻塞浏览器绘制，适合不需要立即影响布局的副作用操作。
   - `useLayoutEffect`：会阻塞浏览器绘制，适合需要读取布局并同步影响布局的操作。
3. **使用场景**：
   - `useEffect`：大多数副作用操作，比如数据获取、订阅等。
   - `useLayoutEffect`：需要同步读取和更新 DOM 的场景，比如获取元素尺寸并调整布局。

### useMemo&useCallback

`useMemo` 用于缓存计算结果，只有在依赖项变化时才重新计算。这样可以避免在每次渲染时都进行昂贵的计算。

`useCallback` 用于缓存函数引用，只有在依赖项变化时才返回新的函数引用。这样可以避免在子组件中因函数引用变化导致的不必要重新渲染。

**使用场景**：

- 将回调函数传递给子组件，避免子组件因函数引用变化而重新渲染。
- 将依赖于某些状态的回调函数缓存起来，只有在这些状态变化时才创建新的函数。



题目：

1. **解释 `useMemo` 和 `useCallback` 的区别。**
   - 回答要点：`useMemo` 用于缓存计算结果，`useCallback` 用于缓存函数引用。`useMemo` 返回缓存的值，而 `useCallback` 返回缓存的函数。
2. **如何判断是否应该使用 `useMemo` 或 `useCallback`？**
   - 回答要点：判断计算是否昂贵，是否依赖于某些状态；判断函数是否频繁传递给子组件，是否因函数引用变化导致子组件不必要的重新渲染。
3. **`useMemo` 和 `useCallback` 的依赖项数组应如何设置？**
   - 回答要点：依赖项数组中的值应该包括所有在计算或函数中使用的外部状态或变量。确保数组中的值变化时才重新计算或创建新的函数。
4. **如何避免 `useMemo` 和 `useCallback` 引入的性能开销？**
   - 回答要点：只在必要时使用 `useMemo` 和 `useCallback`。对于轻量级的计算和函数，不必使用这两个 Hook，因为它们自身也有一些性能开销。
5. **解释为什么在函数组件中使用 `useMemo` 和 `useCallback` 可以提升性能？**
   - 回答要点：避免不必要的计算和重新渲染。`useMemo` 缓存计算结果，`useCallback` 缓存函数引用，减少 React 重新渲染和更新的次数。

### React.memo

`React.memo` 是一个高阶组件，用于优化函数组件的性能。它通过对比组件的 props 来决定是否重新渲染组件。如果 props 没有变化，`React.memo` 会阻止组件重新渲染，从而提升性能。

使用场景：

1. **纯组件**： `React.memo` 类似于类组件中的 `PureComponent`，适用于纯组件（即只依赖于 props 的组件）。
2. **避免不必要的重新渲染**： 如在一个父组件频繁更新状态但子组件的 props 不变时，可以使用 `React.memo` 避免子组件不必要的重新渲染.
3. **优化性能**： 对于渲染开销较大的组件，使用 `React.memo` 可以显著减少重新渲染的次数，提高性能。

`React.memo` 的原理：
`React.memo` 接受一个组件并返回一个增强版组件。它使用 `Object.is` 对比新旧 props，如果 props 没有变化，则阻止重新渲染。
可以通过传递第二个参数来自定义 props 比较函数：

```react
const MyComponent = React.memo(
  ({ name }) => {
    console.log('Rendering MyComponent');
    return <div>Hello, {name}!</div>;
  },
  (prevProps, nextProps) => {
    // 只在 name prop 变化时重新渲染
    return prevProps.name === nextProps.name;
  }
);
```

React.memo 与 useCallback 结合使用

单独使用 `useCallback` 不能完全避免子组件的重新渲染。`useCallback` 只是缓存函数的引用，使得在依赖项不变的情况下，返回同一个函数引用。然而，子组件是否重新渲染，取决于其内部对 props 的对比逻辑。如果子组件没有使用 `React.memo` 或类似的优化手段，它仍然会在父组件重新渲染时重新渲染。

`useCallback` 主要用于缓存回调函数，以避免每次父组件重新渲染时都生成新的函数引用，从而减小因函数引用变化引起的性能开销。但是，如果子组件没有使用 `React.memo`，它仍然会在父组件重新渲染时重新渲染，因为默认情况下，React 认为每次传入的 props（包括函数）都是新的。

1. `useCallback` 缓存回调函数的引用，避免每次父组件重新渲染时都生成新的函数。
2. `React.memo` 对 props 进行浅比较，避免不必要的子组件重新渲染。
3. 只有将 `useCallback` 与 `React.memo` 结合使用，才能有效避免因函数引用变化导致的子组件重新渲染。

### useRef

`useRef` 是一个 Hook，用于创建一个可变对象，其 `.current` 属性初始值为传入的参数。

主要用途包括：

- 访问 DOM 元素。
- 在组件生命周期内保存可变值，不触发重新渲染。
- 在函数组件中模拟实例属性。

`useRef` 创建的引用对象不会引起重新渲染，适用于持久化不需要触发重渲染的值或 DOM 元素。

`useRef` 的高级用法有哪些？

1. 缓存上一次渲染的值
2. 控制不触发重新渲染的计时器

实现防抖

```jsx
function useDebounce(callback, delay) {
  const timerRef = useRef();

  const debouncedFunction = (...args) => {
    if (timerRef.current) {
      clearTimeout(timerRef.current);
    }
    timerRef.current = setTimeout(() => {
      callback(...args);
    }, delay);
  };

  return debouncedFunction;
}
```

计数器，不触发重新渲染

```jsx
function PersistentCounter() {
  const countRef = useRef(0);

  const increment = () => {
    countRef.current += 1;
    console.log(countRef.current);
  };

  return <button onClick={increment}>Increment</button>;
}
```

### forwardRef和useImperativeHandle

`forwardRef` 用于转发 ref，使得父组件可以访问子组件内部的 DOM 元素或组件实例。

`useImperativeHandle` 用于在函数组件内部暴露特定的函数或对象给父组件，通常用于隐藏实现细节，只暴露父组件关心的接口。

## 状态管理

### useContext

[参考1](https://mdnice.com/writing/cc652ede8e1e4c2580b53725ee4d5f97) [参考2](https://juejin.cn/post/7347905080509399067#heading-5) 

可以创建context上下文对象。并使用provider包裹组件。这样方便夸组件通信

```jsx
const Ctx = React.createContext({})
```

```jsx
function App() {
  return (
    <Ctx.Provider value={a: '11', b: '22'}>
      <MyComponent />
    </Ctx.Provider>
  );
}

function MyComponent() {
  const ctxVal = useContext(Ctx)
}
```

`useContext` 允许在不通过组件嵌套层级传递 props 的情况下，在组件之间共享数据。

**当 Context 中的值发生变化时，使用 `useContext` 的组件会重新渲染以反映最新的值。**

**优点：**

1. 简化了数据传递：不需要将数据通过组件树手动传递，提高了组件之间的解耦。
2. 更干净的代码：减少了在组件中传递和处理不必要的 props。

**缺点：**

1. 需要使用上下文对象：上下文的使用可能会让组件更难以理解和测试，因此需要谨慎使用。
2. 嵌套使用可能会导致性能问题：如果多次嵌套使用 `useContext`，可能会导致性能下降。

### useReducer

**什么是 `useReducer`？它和 `useState` 有什么区别？**

- 回答要点：`useReducer` 是 React 提供的另一种状态管理工具，它接收一个 reducer 函数和初始状态，返回当前状态和一个 dispatch 函数，用于触发状态更新。与 `useState` 相比，`useReducer` 更适合处理复杂的状态逻辑，可以将相关的状态集中处理，并且可以方便地传递给子组件。

**什么时候使用 `useReducer` 而不是 `useState`？**

- 回答要点：`useReducer` 更适合处理包含多个子值或者需要在更新时执行复杂逻辑的 state。如果状态逻辑非常简单并且不需要在更新时执行复杂逻辑，可以使用 `useState`。

**`useReducer` 的工作原理是什么？**

- 回答要点：`useReducer` 接收一个 reducer 函数和初始状态，当 `dispatch` 被调用时，会将当前状态和 action 对象传递给 reducer 函数，并根据 reducer 函数的返回值更新状态。reducer 函数接收两个参数：当前状态和 action 对象，返回新的状态。

**如何在 `useReducer` 中处理异步操作？**

- 回答要点：`useReducer` 本身不支持异步操作，但可以在 reducer 函数中使用异步操作，例如使用 `async/await`。不过，最好将异步操作放在副作用中，例如使用 `useEffect`。

**`useReducer` 与 Context 之间有什么关系？**

- 回答要点：`useReducer` 可以与 Context 结合使用，将 dispatch 函数传递给子组件，使得子组件可以通过 dispatch 发起状态更新。这种方式可以避免逐层传递回调函数的问题，特别适合在多层嵌套的组件中管理状态。

**在使用 `useReducer` 时如何避免性能问题？**

- 回答要点：可以使用 `React.memo` 来避免不必要的组件重新渲染，确保只有当状态真正发生变化时才重新渲染相关组件。另外，可以使用 `useCallback` 缓存回调函数，以避免每次渲染都生成新的回调函数。

**`useReducer` 和 Redux 有什么关系？**

- 回答要点：`useReducer` 和 Redux 都是用于管理状态的工具，但是 Redux 是一个独立的状态管理库，而 `useReducer` 是 React 提供的 Hook，用于在组件内部管理状态。Redux 可以看作是 `useReducer` 的一个更强大、更灵活的替代方案，在需要管理全局状态或者状态逻辑较为复杂时可以考虑使用 Redux。

```jsx
function TestReducer() {
    const reducer = (state, action) => {
        switch (action) {
            case 'add':
                return state + 1
            case 'double':
                return state * 2
        }
    }
    const [count, dispatch] = useReducer(reducer, 0)

    return (
        <div>
            {count}
            <button onClick={() => dispatch('add')}>点击添加</button>
            <button onClick={() => dispatch('double')}>点击*2</button>
        </div>
    )
}
```

### useContext+useReducer

[参考1](https://blog.csdn.net/weixin_37719279/article/details/121022738) [参考2](https://juejin.cn/post/7083015479190618119)

### zustand

## React Router

### React router 有两种模式

browser router 和 hash router

1.  browser router 基于h5 的history API 来实现。通过（`pushstate` 和 `replacestate`）来保持URL和UI的同步。
2. hash主要是使用路由#号后面内容，即哈希部分来实现UI与URL同步

### Route 组件 与 Switch 组件的区别

Route组件用于定义路径和对应的组件，当路由匹配时就会渲染对应的组件。

```jsx
<Route path='/user/com' component={comA} />
```

Switch组件可以包裹多个Route，只渲染第一个匹配的Route，确保路由匹配的时候，不会渲染多个组件，常用于404组件

```jsx
<Router>
  <Switch>
    <Route exact path='/' component={Home} /> // exact 完全匹配路径
    <Route exact path='/user' component={User} />
    <Route component={NotFound} /> // 放在最后，Route若没匹配到，会走到这里
  </Switch>
</Router>
```

路由重定向

1. 使用<Redirect />组件
2. 使用useHistory()



实现嵌套路由

组件内部定义子路由来实现嵌套路由



useHisotory() 用于操作 浏览器的历史记录对象

useLocation() 用于好去当前url信息

useParams() 用于获取URL路径中的动态参数



### 路由守卫

使用HOC高阶组件来实现和`<route>`的render属性来实现，高阶组件中传入两个或以上参数。头一个参数为组件。在render方法中对参数进行处理，针对结果进行加载组件或者重定向。



### 路由懒加载

使用suspense组件和React.lazy 进行实现

1. 使用React.lazy 动态引入组件
2. 使用<route> 的component属性指向，动态引入的组件
3. 使用<Switch>包裹这些<Route>
4. 使用suspense组件包裹Switch组件，并提供 fallback。在组件未加载完成的时候，提供loading样式

这样做的好处，减少初始加载的js的体积，仅在需要的时候加载需要的组件，提升应用性能。

## 其他

### react触发重新渲染的几种场景

1. 组件内部状态变化时 
2. 父组件传递给子组件的props发生变化时
3. 父组件重新渲染的时候，这时候子组件会重新接收props，所以会重新渲染。此时使用react.memo可以避免
4. 使用 React Context 时，当 Context 的值发生变化时，所有订阅了该 Context 的组件都会重新渲染。即使某个组件只使用了部分 Context 值，只要 Context 发生变化，组件就会重新渲染。
5. 调用forceUpdate的时候

### React Portals || createPortal

旧版：React Portals 是 React 提供的一种机制，用于将子组件渲染到父组件 DOM 层次结构之外的位置。它在处理一些特殊情况下的 UI 布局或交互时非常有用。

新：createPortal， `createPortal` 允许你将 JSX 作为 children 渲染至 DOM 的不同部分。

```jsx
import { createPortal } from 'react-dom';
// ...
<div>
  <p>这个子节点被放置在父节点 div 中。</p>
  {createPortal(
    <p>这个子节点被放置在 document body 中。</p>,
    document.body
  )}
</div>
```

### React高阶组件

用于复用组件逻辑的高级技术

HOC 是一个函数，它接收一个组件并返回一个新的组件。这个模式主要用于逻辑复用，而不是像 React 组件那样用于 UI 的复用。

**逻辑复用**：将多个组件间共用的逻辑抽象到一个 HOC 中，从而减少代码重复。

**增强组件**：为组件添加额外的功能，例如权限验证、数据获取、日志记录等。

**抽象和封装**：将复杂的逻辑封装在 HOC 中，使得原始组件保持简洁和专注于其主要功能。



**HOC 和 React Hooks 有什么区别和联系？**

1. 区别：
   1. HOC 是一种基于组件的模式，通过函数返回新的组件来复用逻辑。
   2. Hooks 是一种基于函数的模式，通过函数调用来在函数组件中复用状态和副作用逻辑。

2. 联系：
   1. 都是用于复用逻辑的手段，可以解决类似的问题。
   2. HOC 和 Hooks 可以结合使用，例如在 HOC 中使用 Hooks 来实现复杂的逻辑复用。

#### 综合应用

**1. 在实际项目中，你是如何使用 HOC 的？请举一个具体的例子并解释其优势。**

在一个实际项目中，我们需要在多个组件中实现权限控制。通过创建一个 `withAuthorization` 的 HOC，我们可以将权限验证逻辑提取到一个函数中，并应用到多个需要权限验证的组件中。这不仅减少了代码重复，还使得权限逻辑更加集中和易于管理。例如：

```jsx
import React from 'react';

// 高阶组件函数，接收一个组件和允许的角色列表作为参数
function withAuthorization(WrappedComponent, allowedRoles) {
  // 返回一个新的函数组件
  return function WithAuthorization(props) {
    // 从 props 中获取用户角色
    const { userRole } = props;

    // 检查用户角色是否在允许的角色列表中
    if (allowedRoles.includes(userRole)) {
      // 如果允许，返回包裹的组件
      return <WrappedComponent {...props} />;
    } else {
      // 如果不允许，返回“拒绝访问”消息
      return <div>Access Denied</div>;
    }
  };
}

// 示例组件
function AdminComponent() {
  return <div>Admin Content</div>;
}

// 使用 HOC 包装示例组件
const AdminOnlyComponent = withAuthorization(AdminComponent, ['admin']);

// 示例使用
function App() {
  return (
    <div>
      <AdminOnlyComponent userRole="admin" /> {/* 显示 Admin Content */}
      <AdminOnlyComponent userRole="user" /> {/* 显示 Access Denied */}
    </div>
  );
}

export default App;
```

### React的批量更新

React17 和以前。只能将同步事件中的状态更新合并到一起，进行批量更新。而在异步中的不行，如在定时器和promise的回调中的不行。

原理：React 内部维护了一个更新队列，当在事件处理或生命周期方法中调用 `setState` 时，React 会将更新添加到队列中而不是立即更新组件。React 的事件处理系统会在处理完事件后，统一处理这个队列，并重新渲染组件。这是“批量更新”的核心。

React 18 引入了一个新的特性，称为“自动批量更新”，这意味着批量更新的行为将扩展到大多数 React 的更新场景，包括那些异步的场景，如 setTimeout 或 promise 回调。这是通过一个新的渲染引擎架构 —— Concurrent Mode 来实现的，该模式可以更智能地管理多种类型的更新。

### fiber

**什么是 React Fiber？它解决了什么问题？**

- 回答要点：React Fiber 是 React v16 中引入的新的协调引擎。它主要解决了 React 中的渲染和调度的问题，使得 React 应用在处理大型组件树和高优先级更新时具有更好的性能和用户体验。

**React Fiber 的工作原理是什么？**

- 回答要点：React Fiber 使用了一种新的调度算法，将渲染工作分成小的单元并且可以中断和恢复，使得浏览器在处理渲染工作时可以更加灵活地响应用户输入和其他高优先级任务。

**React Fiber 的优点有哪些？**

- 回答要点：React Fiber 的引入使得 React 应用在处理大型组件树和高优先级更新时性能更好，用户体验更流畅。它还提供了更细粒度的控制，使得开发者可以更好地控制渲染的优先级和中断。

**如何利用 React Fiber 实现异步渲染？**

- 回答要点：React Fiber 允许在渲染过程中中断和恢复工作，可以利用这一特性实现异步渲染。开发者可以使用 `ReactDOM.unstable_deferredUpdates` 或者 `ReactDOM.unstable_createRoot` 来实现异步渲染。

**React Fiber 如何处理优先级？**

- 回答要点：React Fiber 使用了优先级调度算法，根据任务的优先级决定任务的执行顺序。高优先级的任务可以中断低优先级的任务，使得 React 应用可以更快地响应用户输入和其他高优先级任务。

**React Fiber 与 Virtual DOM 有何区别？**

- 回答要点：Virtual DOM 是 React 用来描述界面的一种数据结构，而 React Fiber 是 React 的协调引擎，用来处理 Virtual DOM 的更新和渲染。React Fiber 的引入使得 Virtual DOM 的更新和渲染更加高效和灵活。

**在实际项目中如何使用 React Fiber？**

- 回答要点：React Fiber 是 React v16 中的默认渲染引擎，开发者无需额外的配置就可以使用。在实际项目中，可以通过使用 React 的异步渲染 API 来利用 React Fiber 的特性，实现更加流畅和高效的用户界面。

**React Fiber 的未来发展方向是什么？**

- 回答要点：React Fiber 是 React 的一个重要的进化方向，未来可能会进一步优化和改进，以提供更好的性能和用户体验。可能会引入更多的特性和API，使得开发者可以更加灵活地控制渲染和更新过程。