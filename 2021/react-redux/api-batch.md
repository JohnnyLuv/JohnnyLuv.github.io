---
title: React Redux 7.x 中文文档 - API batch()
date: 2021-04-07 18:45:35
categories: JavaScript
tags: [React, Redux]
---

# batch()

```js
batch((fn: Function))
```

added in v7.0.0

`React`的`unstable_batchedUpdates()` `API`允许任何在事件循环中的`React`更新被批处理到一个渲染传递中。`React`已经在内部将其用于自己的事件处理程序回调。这个`API`实际上是`React dom`和`React Native`等渲染器包的一部分，而不是`React core`本身。
> React's unstable_batchedUpdates() API allows any React updates in an event loop tick to be batched together into a single render pass. React already uses this internally for its own event handler callbacks. This API is actually part of the renderer packages like ReactDOM and React Native, not the React core itself.

因为`React-redux`需要在`ReactDOM`和`React`原生环境中工作，所以我们在构建时从正确的渲染器导入了这个`API`，供我们自己使用。现在我们自己也公开地重新导出这个函数，并重命名为`batch()`。你可以使用它来确保在`React`之外分派的多个动作只会导致一个渲染更新，像这样:
> Since React-Redux needs to work in both ReactDOM and React Native environments, we've taken care of importing this API from the correct renderer at build time for our own use. We also now re-export this function publicly ourselves, renamed to batch(). You can use it to ensure that multiple actions dispatched outside of React only result in a single render update, like this:

```js
import { batch } from 'react-redux'

function myThunk() {
  return (dispatch, getState) => {
    // should only result in one combined re-render, not two
    batch(() => {
      dispatch(increment())
      dispatch(increment())
    })
  }
}
```
