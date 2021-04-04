---
title: React Redux 7.x 中文文档 - 快速开始
date: 2021-03-03 14:41:35
categories: JavaScript
tags: [React, Redux]
---

# 快速开始

`React Redux`是`Redux`的官方`React UI`绑定层。它允许`React`组件从`Redux`存储中读取数据，并将操作分派到存储中以更新状态。
> React Redux is the official React UI bindings layer for Redux. It lets your React components read data from a Redux store, and dispatch actions to the store to update state.

## 安装

### 使用`Create React App`

使用`React Redux`启动新应用的推荐方式是使用`Create React App`的官方`Redux+JS`模板，它利用了`Redux`工具包。
> The recommended way to start new apps with React Redux is by using the official Redux+JS template for Create React App, which takes advantage of Redux Toolkit.

```bash
npx create-react-app my-app --template redux
```

### 现有React应用

要使用`React Redux`与你的`React`应用程序，安装它作为一个依赖:
> To use React Redux with your React app, install it as a dependency:

```bash
# If you use npm:
npm install react-redux

# Or if you use Yarn:
yarn add react-redux
```

您还需要安装`Redux`，并在应用程序中设置`Redux store`。
> You'll also need to install Redux and set up a Redux store in your app.

如果你在使用`TypeScript`, `React Redux`类型在`definelytyped`中是单独维护的。你也需要安装这些:
> If you are using TypeScript, the React Redux types are maintained separately in DefinitelyTyped. You'll need to install those as well:

```bash
npm install @types/react-redux
```

## Provider

`React Redux`包括一个`<Provider />`组件，它使`Redux store`对你的应用程序的其余部分可用:
> React Redux includes a <Provider /> component, which makes the Redux store available to the rest of your app:

```js
import React from 'react'
import ReactDOM from 'react-dom'

import { Provider } from 'react-redux'
import store from './store'

import App from './App'

const rootElement = document.getElementById('root')
ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  rootElement
)
```

## Hooks

`React Redux`提供了一对自定义的`React`钩子，允许您的`React`组件与`Redux`存储交互。
> React Redux provides a pair of custom React hooks that allow your React components to interact with the Redux store.

`useSelector`从存储状态中读取一个值并订阅更新，而`useDispatch`返回存储的分派方法以让您分派操作。
> useSelector reads a value from the store state and subscribes to updates, while useDispatch returns the store's dispatch method to let you dispatch actions.

```js
import React from 'react'
import { useSelector, useDispatch } from 'react-redux'
import {
  decrement,
  increment,
  incrementByAmount,
  incrementAsync,
  selectCount
} from './counterSlice'
import styles from './Counter.module.css'

export function Counter() {
  const count = useSelector(selectCount)
  const dispatch = useDispatch()

  return (
    <div>
      <div className={styles.row}>
        <button
          className={styles.button}
          aria-label="Increment value"
          onClick={() => dispatch(increment())}
        >
          +
        </button>
        <span className={styles.value}>{count}</span>
        <button
          className={styles.button}
          aria-label="Decrement value"
          onClick={() => dispatch(decrement())}
        >
          -
        </button>
      </div>
      {/* omit additional rendering output here */}
    </div>
  )
}
```

## 帮助和讨论

`reactflux Discord`社区的`#redux`频道是我们关于学习和使用`redux`的所有问题的官方资源。`reactflux`是一个很棒的地方来闲逛，问问题和学习-来加入我们吧!
> The #redux channel of the Reactiflux Discord community is our official resource for all questions related to learning and using Redux. Reactiflux is a great place to hang out, ask questions, and learn - come join us!

您还可以使用`#redux`标签询问关于`Stack Overflow`的问题。
> You can also ask questions on Stack Overflow using the #redux tag.
