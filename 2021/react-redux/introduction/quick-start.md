---
title: React Redux 7.x 中文文档 - 快速开始
date: 2021-03-03 14:41:35
categories: JavaScript
tags: [React, Redux]
---

# 快速开始

`React Redux`是`Redux`的官方`React`绑定。它使您的`React`组件能够从`Redux`存储中读取数据，并向存储分派操作以更新数据。
> React Redux is the official React binding for Redux. It lets your React components read data from a Redux store, and dispatch actions to the store to update data.

### 安装

`React Redux 7.1`需要`React 16.8.3`或更高版本。
> React Redux 7.1 requires React 16.8.3 or later.

使用`React Redux`与您的`React`应用程序:
> To use React Redux with your React app:

```bash
npm install react-redux
```

or

```bash
yarn add react-redux
```

您还需要安装`Redux`，并在应用程序中设置`Redux store`。
> You'll also need to install Redux and set up a Redux store in your app.

### Provider

`React Redux`提供`<Provider />`，这使得`Redux store`可以用于你的应用程序的其余部分:
> React Redux provides <Provider />, which makes the Redux store available to the rest of your app:

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

### connect()

`React Redux`提供了一个`connect`函数来将组件连接到`store`。
> React Redux provides a connect function for you to connect your component to the store.

通常，你会这样调用`connect`:
> Normally, you’ll call connect in this way:

```js
import { connect } from 'react-redux'
import { increment, decrement, reset } from './actionCreators'

// const Counter = ...

const mapStateToProps = (state /*, ownProps*/) => {
  return {
    counter: state.counter,
  }
}

const mapDispatchToProps = { increment, decrement, reset }

export default connect(mapStateToProps, mapDispatchToProps)(Counter)
```
