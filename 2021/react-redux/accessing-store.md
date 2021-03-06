---
title: React Redux 7.x 中文文档 - 访问 Store
date: 2021-04-07 15:32:35
categories: JavaScript
tags: [React, Redux]
---

# 访问 Store

`React Redux`提供的`api`允许您的组件分派操作和订阅来自`store`的数据更新。
> React Redux provides APIs that allow your components to dispatch actions and subscribe to data updates from the store.

作为其中的一部分，`React Redux`抽象了您正在使用哪个`store`的细节，以及如何处理`store`交互的确切细节。在典型的用法中，您自己的组件永远不需要关心这些细节，也永远不会直接引用存储。`React Redux`还在内部处理如何将存储和状态传播到连接组件的细节，以便在默认情况下按预期工作。
> As part of that, React Redux abstracts away the details of which store you are using, and the exact details of how that store interaction is handled. In typical usage, your own components should never need to care about those details, and won't ever reference the store directly. React Redux also internally handles the details of how the store and state are propagated to connected components, so that this works as expected by default.

然而，在某些用例中，您可能需要自定义如何将`store`和`state`传播到连接的组件，或者直接访问`store`。下面是一些如何做到这一点的例子。
> However, there may be certain use cases where you may need to customize how the store and state are propagated to connected components, or access the store directly. Here are some examples of how to do this.

## 理解Context使用

在内部，`React Redux`使用了`React`的“`context`”特性，使深层次嵌套连接的组件可以访问`Redux`存储。在`ReactRedux`版本6中，这通常由`React.createcontext()`生成的单个默认`context`对象实例来处理，称为`ReactReduxContext`。
> Internally, React Redux uses React's "context" feature to make the Redux store accessible to deeply nested connected components. As of React Redux version 6, this is normally handled by a single default context object instance generated by React.createContext(), called ReactReduxContext.

`ReactRedux`的`<Provider>`组件使用`<ReactReduxContext.Provider>`将还原存储和当前存储状态放入`context`，而`connect`使用`<ReactReduxContext.Consumer>`读取这些值并处理更新。
> React Redux's <Provider> component uses <ReactReduxContext.Provider> to put the Redux store and the current store state into context, and connect uses <ReactReduxContext.Consumer> to read those values and handle updates.

## 使用 useStore Hook

`useStore`钩子从默认的`ReactReduxContext`返回当前的`store`实例。如果您确实需要访问`store`，这是推荐的方法。
> The useStore hook returns the current store instance from the default ReactReduxContext. If you truly need to access the store, this is the recommended approach.

## 提供自定义Context

您可以提供自己的自定义`context`实例，而不是使用`React Redux`的默认上下文实例。
> Instead of using the default context instance from React Redux, you may supply your own custom context instance.

```js
<Provider context={MyContext} store={store}>
  <App />
</Provider>
```

如果您提供了一个自定义`context`，`React Redux`将使用该`context`实例，而不是它在默认情况下创建和导出的实例。
> If you supply a custom context, React Redux will use that context instance instead of the one it creates and exports by default.

在你为`<Provider />`提供了自定义`context`之后，你需要为所有连接到同一个`store`的组件提供这个`context`实例:
> After you’ve supplied the custom context to <Provider />, you will need to supply this context instance to all of your connected components that are expected to connect to the same store:

```js
// You can pass the context as an option to connect
export default connect(
  mapState,
  mapDispatch,
  null,
  { context: MyContext }
)(MyComponent)

// or, call connect as normal to start
const ConnectedComponent = connect(
  mapState,
  mapDispatch
)(MyComponent)

// Later, pass the custom context as a prop to the connected component
<ConnectedComponent context={MyContext} />
```

当React Redux在它正在查找的上下文中没有找到存储时，会发生以下运行时错误。例如:
您为`<Provider />`提供了一个自定义`context`实例，但没有为所连接的组件提供相同的实例(或没有提供任何实例)。
您为所连接的组件提供了一个自定义`context`，但没有向`<Provider />`提供相同的实例(或没有提供任何实例)。
> The following runtime error occurs when React Redux does not find a store in the context it is looking. For example:
> - You provided a custom context instance to <Provider />, but did not provide the same instance (or did not provide any) to your connected components.
> - You provided a custom context to your connected component, but did not provide the same instance (or did not provide any) to <Provider />.

**不变的违反**
在“`Connect(MyComponent)`”`context`中找不到“`store`”。要么将根组件包装在`<Provider>`中，要么将自定义的`React`上下文提供程序传递给`<Provider>`和相应的`React`上下文消费者，以便在`Connect options`中连接(`Todo`)。
> Invariant Violation
> Could not find "store" in the context of "Connect(MyComponent)". Either wrap the root component in a <Provider>, or pass a custom React context provider to <Provider> and the corresponding React context consumer to Connect(Todo) in connect options.

## 多个Stores

`Redux`是专为单一`store`设计的。但是，如果不可避免地需要使用多个`store`，从v6开始，可以通过提供(多个)自定义`contexts`来实现。这还提供了`store`的自然隔离，因为它们存在于单独的`store`实例中。
> Redux was designed to use a single store. However, if you are in an unavoidable position of needing to use multiple stores, as of v6 you may do so by providing (multiple) custom contexts. This also provides a natural isolation of the stores as they live in separate context instances.

```js
// a naive example
const ContextA = React.createContext();
const ContextB = React.createContext();

// assuming reducerA and reducerB are proper reducer functions
const storeA = createStore(reducerA);
const storeB = createStore(reducerB);

// supply the context instances to Provider
function App() {
  return (
    <Provider store={storeA} context={ContextA} />
      <Provider store={storeB} context={ContextB}>
        <RootModule />
      </Provider>
    </Provider>
  );
}

// fetch the corresponding store with connected components
// you need to use the correct context
connect(mapStateA, null, null, { context: ContextA })(MyComponentA)

// You may also pass the alternate context instance directly to the connected component instead
<ConnectedMyComponentA context={ContextA} />

// it is possible to chain connect()
// in this case MyComponent will receive merged props from both stores
compose(
  connect(mapStateA, null, null, { context: ContextA }),
  connect(mapStateB, null, null, { context: ContextB })
)(MyComponent);
```

## 直接使用ReactReduxContext

在极少数情况下，您可能需要在自己的组件中直接访问`Redux store`。这可以通过自己呈现适当的`context`使用者，并在上下文值之外访问`store`字段来实现。
> In rare cases, you may need to access the Redux store directly in your own components. This can be done by rendering the appropriate context consumer yourself, and accessing the store field out of the context value.

**注意**
这不是`React Redux`公共`API`的一部分，可能会在不通知的情况下中断。我们确实认识到社区有必要使用的用例，并将努力让用户在`React Redux`之上构建额外的功能，但我们对上下文的具体使用被认为是实现细节。如果你有额外的用例没有被当前的API充分覆盖，请提交一个问题来讨论可能的`API`改进。
> CAUTION
> This is not considered part of the React Redux public API, and may break without notice. We do recognize that the community has use cases where this is necessary, and will try to make it possible for users to build additional functionality on top of React Redux, but our specific use of context is considered an implementation detail. If you have additional use cases that are not sufficiently covered by the current APIs, please file an issue to discuss possible API improvements.

```js
import { ReactReduxContext } from 'react-redux'

// Somewhere inside of a <Provider>
function MyConnectedComponent() {
  // Access the store via the `useContext` hook
  const {store} = useContext(ReactReduxContext)

  // alternately, use the render props form of the context
  /*
  return (
    <ReactReduxContext.Consumer>
      {({ store }) => {
        // do something useful with the store, like passing it to a child
        // component where it can be used in lifecycle methods
      }}
    </ReactReduxContext.Consumer>
  )
  */
}
```
