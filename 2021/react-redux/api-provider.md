---
title: React Redux 7.x 中文文档 - API Provider
date: 2021-04-07 15:32:35
categories: JavaScript
tags: [React, Redux]
---

# Provider

## 概述

`<Provider>`组件使需要访问`Redux`存储的任何嵌套组件都可以使用`Redux`存储。
> The <Provider> component makes the Redux store available to any nested components that need to access the Redux store.

因为`React Redux`应用中的任何`React`组件都可以连接到`store`，所以大多数应用都会在顶层渲染一个`<Provider>`，整个应用的组件树都在里面。
> Since any React component in a React Redux app can be connected to the store, most applications will render a <Provider> at the top level, with the entire app’s component tree inside of it.

钩子和连接`api`可以通过`React`的上下文机制访问所提供的store实例。
> The Hooks and connect APIs can then access the provided store instance via React's Context mechanism.

### Props

`store`(`Redux store`)应用程序中的单个`Redux store`。
`children`（`ReactElement`）组件层次结构的根。
你可以提供一个`context`实例。如果这样做，还需要为所有连接的组件提供相同的`context`实例。未能提供正确的`context`将导致运行时错误:
> store (Redux Store) The single Redux store in your application.
> children (ReactElement) The root of your component hierarchy.
> context You may provide a context instance. If you do so, you will need to provide the same context instance to all of your connected components as well. Failure to provide the correct context results in runtime error:

**不变的违反**
在“`Connect(MyComponent)`”上下文中找不到“`store`”。要么将根组件包装在`<Provider>`中，要么将自定义的`React`上下文提供程序传递给`<Provider>`和相应的`React context`使用者，以便在`Connect options`中连接(`Todo`)。
> Invariant Violation
> Could not find "store" in the context of "Connect(MyComponent)". Either wrap the root component in a <Provider>, or pass a custom React context provider to <Provider> and the corresponding React context consumer to Connect(Todo) in connect options.

### 示例使用

在下面的例子中，`<App />`组件是我们的根级组件。这意味着它位于组件层次结构的最顶层。
> In the example below, the <App /> component is our root-level component. This means it’s at the very top of our component hierarchy.

```js
import React from 'react'
import ReactDOM from 'react-dom'
import { Provider } from 'react-redux'

import { App } from './App'
import createStore from './createReduxStore'

const store = createStore()

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```
