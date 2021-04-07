---
title: React Redux 7.x 中文文档 - API Hooks
date: 2021-04-07 17:05:35
categories: JavaScript
tags: [React, Redux]
---

# Hooks

`React`的新“`hooks`”`api`让函数组件能够使用局部组件状态、执行副作用等等。`React`还允许我们编写自定义钩子，这让我们可以提取可重用的钩子，在`React`的内置钩子上添加我们自己的行为。
> React's new "hooks" APIs give function components the ability to use local component state, execute side effects, and more. React also lets us write custom hooks, which let us extract reusable hooks to add our own behavior on top of React's built-in hooks.

`React Redux`包括它自己的自定义`hooks api`，它允许你的`React`组件订阅`Redux`的存储和调度操作。
> React Redux includes its own custom hook APIs, which allow your React components to subscribe to the Redux store and dispatch actions.

**提示**
我们建议使用`React-redux` `hooks API`作为`React`组件的默认方法。
现有的`connect API`仍在工作，并将继续得到支持，但`hooks API`更简单，与`TypeScript`一起工作得更好。
> TIP
> We recommend using the React-Redux hooks API as the default approach in your React components.
> The existing connect API still works and will continue to be supported, but the hooks API is simpler and works better with TypeScript.

这些钩子最初是在v7.1.0中添加的。
> These hooks were first added in v7.1.0.

## 在React Redux应用程序中使用Hooks

和`connect()`一样，你应该首先把你的整个应用程序包装在`<Provider>`组件中，以使整个组件树中的存储都可用:
> As with connect(), you should start by wrapping your entire application in a <Provider> component to make the store available throughout the component tree:

```js
const store = createStore(rootReducer)

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

从那里，您可以导入任何列出的`React Redux`钩子`api`，并在您的函数组件中使用它们。
> From there, you may import any of the listed React Redux hooks APIs and use them within your function components.

### useSelector()

```js
const result: any = useSelector(selector: Function, equalityFn?: Function)
```

允许您使用选择器函数从还原存储状态提取数据。
> Allows you to extract data from the Redux store state, using a selector function.

注意
选择器函数应该是纯函数，因为它可能会在任意时间点执行多次。
> INFO
> The selector function should be pure since it is potentially executed multiple times and at arbitrary points in time.

选择器在概念上与`mapStateToProps`参数等价。调用选择器时，将整个`Redux`存储状态作为唯一参数。当函数组件呈现时，选择器就会运行(除非它的引用自组件之前的呈现后没有改变，这样缓存的结果就可以被钩子返回，而无需重新运行选择器)。`useSelector()`也将订阅`Redux`存储，并在发送操作时运行选择器。
> The selector is approximately equivalent to the mapStateToProps argument to connect conceptually. The selector will be called with the entire Redux store state as its only argument. The selector will be run whenever the function component renders (unless its reference hasn't changed since a previous render of the component so that a cached result can be returned by the hook without re-running the selector). useSelector() will also subscribe to the Redux store, and run your selector whenever an action is dispatched.

然而，传递给`useSelector()`的选择器和`mapState`函数之间有一些区别:
- 选择器可以返回任何结果值，而不仅仅是一个对象。选择器的返回值将被用作`useSelector()`钩子的返回值。
- 当一个操作被分派时，`useSelector()`将对之前的选择器结果值和当前结果值进行引用比较。如果它们不同，组件将被迫重新渲染。如果它们是相同的，组件将不会重新渲染。
- `selector`函数不接收`ownProps`参数。然而，`props`可以通过`closure`(见下面的示例)或使用`curry`过的选择器来使用。
- 在使用记忆选择符时必须格外小心(参见下面的示例了解更多细节)。
- `useSelector()`默认使用`===`严检查确保引用一致性，而不是浅相等性检查(请参阅下面一节了解更多细节)。
> However, there are some differences between the selectors passed to useSelector() and a mapState function:
> - The selector may return any value as a result, not just an object. The return value of the selector will be used as the return value of the useSelector() hook.
> - When an action is dispatched, useSelector() will do a reference comparison of the previous selector result value and the current result value. If they are different, the component will be forced to re-render. If they are the same, the component will not re-render.
> - The selector function does not receive an ownProps argument. However, props can be used through closure (see the examples below) or by using a curried selector.
> - Extra care must be taken when using memoizing selectors (see examples below for more details).
> - useSelector() uses strict === reference equality checks by default, not shallow equality (see the following section for more details).

**注意**
在选择器中使用道具存在潜在的边界情况，这可能会导致问题。有关详细信息，请参阅本页的使用警告部分。
> INFO
> There are potential edge cases with using props in selectors that may cause issues. See the Usage Warnings section of this page for further details.

你可以在一个函数组件中多次调用`useSelector()`。每次调用`useSelector()`都会创建对`Redux`存储的单独订阅。由于在`React Redux v7`中使用了`React update`批处理行为，一个分派操作如果导致同一个组件中的多个`useSelector()`返回新值，那么它只会导致一次重新渲染。
> You may call useSelector() multiple times within a single function component. Each call to useSelector() creates an individual subscription to the Redux store. Because of the React update batching behavior used in React Redux v7, a dispatched action that causes multiple useSelector()s in the same component to return new values should only result in a single re-render.

### 相等比较和更新

当函数组件呈现时，所提供的选择器函数将被调用，其结果将从`useSelector()`钩子中返回。(如果缓存的结果与组件之前渲染时的函数引用相同，钩子可以不重新运行选择器而返回该结果。)
> When the function component renders, the provided selector function will be called and its result will be returned from the useSelector() hook. (A cached result may be returned by the hook without re-running the selector if it's the same function reference as on a previous render of the component.)

然而，当一个操作被分派到`Redux`存储时，`useSelector()`只在选择器结果与上次结果不同时强制重新呈现。`v7.1.0-alpha.5`默认比较是严格的`===`引用比较。这与`connect()`不同，后者对`mapState`调用的结果使用浅层相等性检查，以确定是否需要重新呈现。这对如何使用`useSelector()`有几个暗示。
> However, when an action is dispatched to the Redux store, useSelector() only forces a re-render if the selector result appears to be different than the last result. As of v7.1.0-alpha.5, the default comparison is a strict === reference comparison. This is different than connect(), which uses shallow equality checks on the results of mapState calls to determine if re-rendering is needed. This has several implications on how you should use useSelector().

使用`mapState`，所有单独的字段都在一个组合对象中返回。返回对象是否是新的引用并不重要——`connect()`只是比较各个字段。使用`useSelector()`，每次返回一个新对象都会强制重新渲染。如果你想从存储中检索多个值，你可以:
- 多次调用`useSelector()`，每次调用都会返回单个字段值
- 使用`reelect`或类似的库创建一个`memoized`选择器，该选择器在一个对象中返回多个值，但只在其中一个值发生变化时返回一个新对象。
- 使用`React-Redux`中的`allowequal`函数作为`useSelector()`的`equalityFn`参数，例如:
> With mapState, all individual fields were returned in a combined object. It didn't matter if the return object was a new reference or not - connect() just compared the individual fields. With useSelector(), returning a new object every time will always force a re-render by default. If you want to retrieve multiple values from the store, you can:
> - Call useSelector() multiple times, with each call returning a single field value
> - Use Reselect or a similar library to create a memoized selector that returns multiple values in one object, > - but only returns a new object when one of the values has changed.
Use the shallowEqual function from React-Redux as the equalityFn argument to useSelector(), like:

```js
import { shallowEqual, useSelector } from 'react-redux'

// later
const selectedData = useSelector(selectorReturningObject, shallowEqual)
```

可选的比较函数还支持使用`Lodash`的`_.isEqual()`或`Immutable.js`的比较功能。
> The optional comparison function also enables using something like Lodash's _.isEqual() or Immutable.js's comparison capabilities.

### useSelector例子

基本用法:

```js
import React from 'react'
import { useSelector } from 'react-redux'

export const CounterComponent = () => {
  const counter = useSelector(state => state.counter)
  return <div>{counter}</div>
}
```

通过关闭使用道具来确定提取什么:
> Using props via closure to determine what to extract:

```js
import React from 'react'
import { useSelector } from 'react-redux'

export const TodoListItem = props => {
  const todo = useSelector(state => state.todos[props.id])
  return <div>{todo.text}</div>
}
```

**使用memoizing选择器**

如上所示，当`useSelector`与内联选择器一起使用时，每当组件被渲染时，都会创建一个选择器的新实例。只要选择器不维护任何状态，这就可以工作。然而，记忆选择器(例如，从`reselect`通过`createSelector`创建)确实有内部状态，因此在使用它们时必须小心。下面你可以找到记忆选择器的典型用法。
> When using useSelector with an inline selector as shown above, a new instance of the selector is created whenever the component is rendered. This works as long as the selector does not maintain any state. However, memoizing selectors (e.g. created via createSelector from reselect) do have internal state, and therefore care must be taken when using them. Below you can find typical usage scenarios for memoizing selectors.

当选择器只依赖于状态时，只需确保它是在组件外部声明的，这样每个渲染器都使用相同的选择器实例:
> When the selector does only depend on the state, simply ensure that it is declared outside of the component so that the same selector instance is used for each render:

```js
import React from 'react'
import { useSelector } from 'react-redux'
import { createSelector } from 'reselect'

const selectNumCompletedTodos = createSelector(
  state => state.todos,
  todos => todos.filter(todo => todo.completed).length
)

export const CompletedTodosCounter = () => {
  const numCompletedTodos = useSelector(selectNumCompletedTodos)
  return <div>{numCompletedTodos}</div>
}

export const App = () => {
  return (
    <>
      <span>Number of completed todos:</span>
      <CompletedTodosCounter />
    </>
  )
}
```

如果选择器依赖于组件的道具，也同样适用，但只会在单个组件的单个实例中使用:
> The same is true if the selector depends on the component's props, but will only ever be used in a single instance of a single component:

```js
import React from 'react'
import { useSelector } from 'react-redux'
import { createSelector } from 'reselect'

const selectCompletedTodosCount = createSelector(
  state => state.todos,
  (_, completed) => completed,
  (todos, completed) =>
    todos.filter(todo => todo.completed === completed).length
)

export const CompletedTodosCount = ({ completed }) => {
  const matchingCount = useSelector(state =>
    selectCompletedTodosCount(state, completed)
  )

  return <div>{matchingCount}</div>
}

export const App = () => {
  return (
    <>
      <span>Number of done todos:</span>
      <CompletedTodosCount completed={true} />
    </>
  )
}
```

然而，当选择器在多个组件实例中使用并依赖于组件的道具时，你需要确保每个组件实例都有自己的选择器实例(请参阅这里了解为什么有必要这么做):
> However, when the selector is used in multiple component instances and depends on the component's props, you need to ensure that each component instance gets its own selector instance (see here for a more thorough explanation of why this is necessary):

```js
import React, { useMemo } from 'react'
import { useSelector } from 'react-redux'
import { createSelector } from 'reselect'

const makeSelectCompletedTodosCount = () =>
  createSelector(
    state => state.todos,
    (_, completed) => completed,
    (todos, completed) =>
      todos.filter(todo => todo.completed === completed).length
  )

export const CompletedTodosCount = ({ completed }) => {
  const selectCompletedTodosCount = useMemo(makeSelectCompletedTodosCount, [])

  const matchingCount = useSelector(state =>
    selectCompletedTodosCount(state, completed)
  )

  return <div>{matchingCount}</div>
}

export const App = () => {
  return (
    <>
      <span>Number of done todos:</span>
      <CompletedTodosCount completed={true} />
      <span>Number of unfinished todos:</span>
      <CompletedTodosCount completed={false} />
    </>
  )
}
```

## useDispatch()

```js
const dispatch = useDispatch()
```

这个钩子从`Redux store`返回一个对分派函数的引用。您可以根据需要使用它来分派操作。
> This hook returns a reference to the dispatch function from the Redux store. You may use it to dispatch actions as needed.

**例子**

```js
import React from 'react'
import { useDispatch } from 'react-redux'

export const CounterComponent = ({ value }) => {
  const dispatch = useDispatch()

  return (
    <div>
      <span>{value}</span>
      <button onClick={() => dispatch({ type: 'increment-counter' })}>
        Increment counter
      </button>
    </div>
  )
}
```

当使用`dispatch`将回调函数传递给子组件时，有时可能需要使用`useCallback`来记忆它。如果子组件试图使用`React.memo()`或类似的方法优化呈现行为，这就避免了由于改变了回调引用而对子组件进行不必要的呈现。
> When passing a callback using dispatch to a child component, you may sometimes want to memoize it with useCallback. If the child component is trying to optimize render behavior using React.memo() or similar, this avoids unnecessary rendering of child components due to the changed callback reference.

```js
import React, { useCallback } from 'react'
import { useDispatch } from 'react-redux'

export const CounterComponent = ({ value }) => {
  const dispatch = useDispatch()
  const incrementCounter = useCallback(
    () => dispatch({ type: 'increment-counter' }),
    [dispatch]
  )

  return (
    <div>
      <span>{value}</span>
      <MyIncrementButton onIncrement={incrementCounter} />
    </div>
  )
}

export const MyIncrementButton = React.memo(({ onIncrement }) => (
  <button onClick={onIncrement}>Increment counter</button>
))
```

**注意**
只要同一个存储实例被传递给`<Provider>`，分派函数引用就会是稳定的。通常，这个`store`实例在应用程序中永远不会改变。
然而，`React hooks lint`规则不知道`dispatch`应该是稳定的，并且会警告说`dispatch`变量应该添加到`useEffect和useCallback`的依赖数组中。最简单的解决方法就是:
> INFO
> The dispatch function reference will be stable as long as the same store instance is being passed to the <Provider>. Normally, that store instance never changes in an application.
> However, the React hooks lint rules do not know that dispatch should be stable, and will warn that the dispatch variable should be added to dependency arrays for useEffect and useCallback. The simplest solution is to do just that:

```js
export const Todos() = () => {
  const dispatch = useDispatch();

  useEffect(() => {
    dispatch(fetchTodos())
  // Safe to add dispatch to the dependencies array
  }, [dispatch])
}
```

## useStore()

```js
const store = useStore()
```

这个钩子返回一个引用，该引用指向同一个`Redux store`，这个`Redux store`是传递给`<Provider>`组件的。
这个钩子不应该经常使用。首选`useSelector()`。然而，这对于需要访问存储的不太常见的场景可能是有用的，例如替换`reducer`。
> This hook returns a reference to the same Redux store that was passed in to the <Provider> component.
> This hook should probably not be used frequently. Prefer useSelector() as your primary choice. However, this may be useful for less common scenarios that do require access to the store, such as replacing reducers.

**例子**

```js
import React from 'react'
import { useStore } from 'react-redux'

export const CounterComponent = ({ value }) => {
  const store = useStore()

  // EXAMPLE ONLY! Do not do this in a real app.
  // The component will not automatically update if the store state changes
  return <div>{store.getState()}</div>
}
```

## Custom context

`<Provider>`组件允许你通过上下文道具来指定一个备用上下文。如果您正在构建一个复杂的可重用组件，并且不希望您的存储与消费者应用程序可能使用的任何`Redux`存储发生冲突，那么这是非常有用的。
> The <Provider> component allows you to specify an alternate context via the context prop. This is useful if you're building a complex reusable component, and you don't want your store to collide with any Redux store your consumers' applications might use.

要通过钩子`API`访问另一个上下文，请使用钩子创建者函数:
> To access an alternate context via the hooks API, use the hook creator functions:

```js
import React from 'react'
import {
  Provider,
  createStoreHook,
  createDispatchHook,
  createSelectorHook
} from 'react-redux'

const MyContext = React.createContext(null)

// Export your custom hooks if you wish to use them in other files.
export const useStore = createStoreHook(MyContext)
export const useDispatch = createDispatchHook(MyContext)
export const useSelector = createSelectorHook(MyContext)

const myStore = createStore(rootReducer)

export function MyProvider({ children }) {
  return (
    <Provider context={MyContext} store={myStore}>
      {children}
    </Provider>
  )
}
```

## Usage Warnings

### Stale Props and "Zombie Children"

**注意**
`React-Redux hooks API`自从在`v7.1.0`中发布以来就已经可以投入生产了，我们建议在组件中使用`hook API`作为默认方法。然而，也有一些可能发生的边缘情况，我们正在记录这些情况，以便您能够了解它们。
> INFO
> The React-Redux hooks API has been production-ready since we released it in v7.1.0, and we recommend using the hooks API as the default approach in your components. However, there are a couple of edge cases that can occur, and we're documenting those so that you can be aware of them.

`React Redux`实现中最困难的一个方面是确保如果您的`mapStateToProps`函数定义为(`state, ownProps`)，那么每次都会用“最新的”`props`调用它。在第4版之前，有反复报告的`bug`涉及到边界情况，例如，对于数据刚刚被删除的列表项，`mapState`函数抛出错误。
> One of the most difficult aspects of React Redux's implementation is ensuring that if your mapStateToProps function is defined as (state, ownProps), it will be called with the "latest" props every time. Up through version 4, there were recurring bugs reported involving edge case situations, such as errors thrown from a mapState function for a list item whose data had just been deleted.

从第5版开始，`React Redux`就试图保证与`ownProps`的一致性。在版本7中，这是在`connect()`中使用自定义订阅类内部实现的，它形成了一个嵌套层次结构。这将确保树中较低位置的连接组件只在最近的连接祖先更新后才接收到存储更新通知。然而，这依赖于每个`connect()`实例覆盖内部`React`上下文的一部分，提供自己唯一的订阅实例来形成嵌套，并呈现`<ReactReduxContext。提供程序>`具有新的上下文值。
> Starting with version 5, React Redux has attempted to guarantee that consistency with ownProps. In version 7, that is implemented using a custom Subscription class internally in connect(), which forms a nested hierarchy. This ensures that connected components lower in the tree will only receive store update notifications once the nearest connected ancestor has been updated. However, this relies on each connect() instance overriding part of the internal React context, supplying its own unique Subscription instance to form that nesting, and rendering the <ReactReduxContext.Provider> with that new context value.

使用钩子，无法呈现上下文提供程序，这意味着也没有订阅的嵌套层次结构。因此，“`stale props`”和“`zombie child`”问题可能会在依赖于使用钩子而不是`connect()`的应用程序中再次发生。
> With hooks, there is no way to render a context provider, which means there's also no nested hierarchy of subscriptions. Because of this, the "stale props" and "zombie child" issues may potentially re-occur in an app that relies on using hooks instead of connect().

具体来说，“`stale props`”指的是:
- 选择器函数依赖于这个组件的道具来提取数据
- 父组件将重新渲染并传递新的道具作为操作的结果
- 但是，这个组件的选择器函数在该组件有机会使用这些新道具重新渲染之前执行
> Specifically, "stale props" means any case where:
> - a selector function relies on this component's props to extract data
> - a parent component would re-render and pass down new props as a result of an action
> - but this component's selector function executes before this component has had a chance to re-render with those new props

根据所使用的道具和当前存储状态，这可能导致选择器返回不正确的数据，甚至抛出错误。
> Depending on what props were used and what the current store state is, this may result in incorrect data being returned from the selector, or even an error being thrown.

“`Zombie child`”特指以下情况:
- 多个嵌套连接的组件在第一次传递中被挂载，导致子组件在父组件之前订阅存储
- 发送一个从存储中删除数据的操作，例如`todo`项
- 因此，父组件将停止呈现子组件
- 但是，因为子进程是先订阅的，所以它的订阅会在父进程停止呈现之前运行。当它根据`props`从存储中读取一个值时，该数据将不再存在，如果提取逻辑不小心，这可能会导致抛出错误。
> "Zombie child" refers specifically to the case where:
> - Multiple nested connected components are mounted in a first pass, causing a child component to subscribe to the store before its parent
> - An action is dispatched that deletes data from the store, such as a todo item
> - The parent component would stop rendering that child as a result
> - However, because the child subscribed first, its subscription runs before the parent stops rendering it. When it reads a value from the store based on props, that data no longer exists, and if the extraction logic is not careful, this may result in an error being thrown.

`useSelector()`试图通过捕捉由于存储更新而执行选择器时抛出的所有错误来处理这个问题(但不是在呈现时执行)。当出现错误时，组件将被强制渲染，此时选择器将再次执行。只要选择器是纯函数，并且您不依赖于抛出错误的选择器，此方法就可以工作。
> useSelector() tries to deal with this by catching all errors that are thrown when the selector is executed due to a store update (but not when it is executed during rendering). When an error occurs, the component will be forced to render, at which point the selector is executed again. This works as long as the selector is a pure function and you do not depend on the selector throwing errors.

如果你更喜欢自己处理这个问题，这里有一些使用`useSelector()`来避免这些问题的可能选项:
- 不要依赖选择器函数中的道具来提取数据
- 如果你依赖于选择器函数中的道具，而这些道具可能会随着时间的推移而改变，或者你抽取的数据可能基于可以删除的道具，那么尝试编写选择器函数。不要直接进入`state.todos[props.id].name`——读取`state.todos[props.id]`。首先，在尝试读取`todo.name`之前，验证它是否存在。
- 因为连接添加必要的订阅内容提供者和延迟评价孩子订阅,直到连接组件,重新把连接组件的组件树上方的组件使用`useSelector`将防止这些问题只要连接组件被重新由于钩子一样的存储更新组件。
> If you prefer to deal with this issue yourself, here are some possible options for avoiding these problems altogether with useSelector():
> - Don't rely on props in your selector function for extracting data
> - In cases where you do rely on props in your selector function and those props may change over time, or the data you're extracting may be based on items that can be deleted, try writing the selector functions defensively. Don't just reach straight into state.todos[props.id].name - read state.todos[props.id] first, and verify that it exists before trying to read todo.name.
> - Because connect adds the necessary Subscription to the context provider and delays evaluating child subscriptions until the connected component has re-rendered, putting a connected component in the component tree just above the component using useSelector will prevent these issues as long as the connected component gets re-rendered due to the same store update as the hooks component.

### Performance

如前所述，默认情况下，`useSelector()`会在分派操作后运行`selector`函数时对所选值进行引用相等比较，只有当所选值发生变化时，才会导致组件重新呈现。然而，与`connect()`不同的是，`useSelector()`不会阻止组件因为其父组件的重新渲染而重新渲染，即使组件的道具没有改变。
> As mentioned earlier, by default useSelector() will do a reference equality comparison of the selected value when running the selector function after an action is dispatched, and will only cause the component to re-render if the selected value changed. However, unlike connect(), useSelector() does not prevent the component from re-rendering due to its parent re-rendering, even if the component's props did not change.

如果需要进一步的性能优化，你可以考虑在`React.memo()`中封装你的函数组件:
> If further performance optimizations are necessary, you may consider wrapping your function component in React.memo():

```js
const CounterComponent = ({ name }) => {
  const counter = useSelector(state => state.counter)
  return (
    <div>
      {name}: {counter}
    </div>
  )
}

export const MemoizedCounterComponent = React.memo(CounterComponent)
```

## Hooks Recipes

我们已经从最初的`alpha`版本中削减了我们的`hook API`，将重点放在一个更小的`API`原语集上。然而，您可能仍然希望使用我们在您自己的应用程序中尝试过的一些方法。这些示例应该可以复制并粘贴到您自己的代码库中。
> We've pared down our hooks API from the original alpha release, focusing on a more minimal set of API primitives. However, you may still wish to use some of the approaches we tried in your own apps. These examples should be ready to copy and paste into your own codebase.

### Recipe: useActions()

这个钩子在我们最初的`alpha`版本中，但是在`v7.1.0-alpha.4`中被删除了。根据Dan Abramov的建议。这个建议是基于“`binding action creators`”在基于钩子的用例中没有那么有用，并且导致了太多的概念开销和语法复杂性。
> This hook was in our original alpha release, but removed in v7.1.0-alpha.4, based on Dan Abramov's suggestion. That suggestion was based on "binding action creators" not being as useful in a hooks-based use case, and causing too much conceptual overhead and syntactic complexity.

您可能更喜欢在组件中调用`useDispatch`钩子来检索对`dispatch`的引用，并根据需要在回调和效果中手动调用`dispatch(someActionCreator())`。您也可以在自己的代码中使用`Redux bindactioncreator`函数来绑定动作创建者，或者“手动”绑定它们，例如`const boundAddTodo = (text) => dispatch(addTodo(text))`。
> You should probably prefer to call the useDispatch hook in your components to retrieve a reference to dispatch, and manually call dispatch(someActionCreator()) in callbacks and effects as needed. You may also use the Redux bindActionCreators function in your own code to bind action creators, or "manually" bind them like const boundAddTodo = (text) => dispatch(addTodo(text)).

不过，如果你还想自己使用这个钩子，这里有一个可复制粘贴的版本，它支持将`action creator`作为单个函数、数组或对象传递。
> However, if you'd like to still use this hook yourself, here's a copy-pastable version that supports passing in action creators as a single function, an array, or an object.

```js
import { bindActionCreators } from 'redux'
import { useDispatch } from 'react-redux'
import { useMemo } from 'react'

export function useActions(actions, deps) {
  const dispatch = useDispatch()
  return useMemo(
    () => {
      if (Array.isArray(actions)) {
        return actions.map(a => bindActionCreators(a, dispatch))
      }
      return bindActionCreators(actions, dispatch)
    },
    deps ? [dispatch, ...deps] : [dispatch]
  )
}
```

## Recipe: useShallowEqualSelector()

```js
import { useSelector, shallowEqual } from 'react-redux'

export function useShallowEqualSelector(selector) {
  return useSelector(selector, shallowEqual)
}
```

### 使用钩子时的其他注意事项

在决定是否使用钩子时，需要考虑一些架构上的权衡。Mark Erikson在他的两篇博文[《Thoughts on React Hooks, Redux, and Separation of Concerns》](https://blog.isquaredsoftware.com/2019/07/blogged-answers-thoughts-on-hooks/)和[《 Hooks, HOCs, and Tradeoffs.》](https://blog.isquaredsoftware.com/2019/09/presentation-hooks-hocs-tradeoffs/)中很好地总结了这些。
> There are some architectural trade offs to take into consideration when deciding whether to use hooks or not. Mark Erikson summarizes these nicely in his two blog posts Thoughts on React Hooks, Redux, and Separation of Concerns and Hooks, HOCs, and Tradeoffs.