---
title: React Redux 7.x 中文文档 - 联合TypeScript
date: 2021-04-04 14:20:35
categories: JavaScript
tags: [React, Redux]
---

# Usage with TypeScript

`React Redux`本身目前是用纯`JavaScript`编写的。不过，它在静态类型系统(比如`TypeScript`)中工作得很好。
> React Redux itself is currently written in plain JavaScript. However, it works well with static type systems such as TypeScript.

`React Redux`类型定义在NPM上是一个单独的`@types/ React - Redux typedefs`包。除了键入库函数之外，这些类型还导出了一些辅助程序，使在`Redux store`和`React`组件之间编写类型安全接口变得更容易。
> The React Redux type definitions are a separate @types/react-redux typedefs package on NPM. In addition to typing the library functions, the types also export some helpers to make it easier to write typesafe interfaces between your Redux store and your React components.

在`React Redux v7.2.3`版本中，`React - Redux`包依赖于`@types/ React - Redux`，因此类型定义将自动与库一起安装。否则，你需要自己手动安装它们(`npm install @types/react-redux`)。
> As of React Redux v7.2.3, the react-redux package has a dependency on @types/react-redux, so the type definitions will be automatically installed with the library. Otherwise, you'll need to manually install them yourself ( npm install @types/react-redux ).

## 标准的Redux Toolkit项目设置与TypeScript

我们假设一个典型的`Redux`项目是将`Redux`工具包和`React Redux`一起使用。
> We assume that a typical Redux project is using Redux Toolkit and React Redux together.

`Redux Toolkit (RTK)`是编写现代`Redux`逻辑的标准方法。`RTK`已经是用`TypeScript`编写的，它的`API`被设计成为提供一个很好的使用`TypeScript`的体验。
> Redux Toolkit (RTK) is the standard approach for writing modern Redux logic. RTK is already written in TypeScript, and its API is designed to provide a good experience for TypeScript usage.

`Create-React-App`的`Redux+TS`模板附带了一个已经配置好的这些模式的工作示例。
> The Redux+TS template for Create-React-App comes with a working example of these patterns already configured.

### 定义根状态和调度类型

使用`configureStore`不需要任何额外的类型。但是，您将需要提取根状态类型和分派类型，以便在需要时可以引用它们。从存储本身推断这些类型意味着，当您添加更多的状态片或修改中间件设置时，它们将正确更新。
> Using configureStore should not need any additional typings. You will, however, want to extract the RootState type and the Dispatch type so that they can be referenced as needed. Inferring these types from the store itself means that they correctly update as you add more state slices or modify middleware settings.

因为这些都是类型，所以可以从`app/store.ts`等商店设置文件中直接导出它们，然后直接导入其他文件。
> Since those are types, it's safe to export them directly from your store setup file such as app/store.ts and import them directly into other files.

```js
// app/store.ts

import { configureStore } from '@reduxjs/toolkit'
// ...

const store = configureStore({
  reducer: {
    posts: postsReducer,
    comments: commentsReducer,
    users: usersReducer,
  }
})

// Infer the `RootState` and `AppDispatch` types from the store itself
export type RootState = ReturnType<typeof store.getState>
// Inferred type: {posts: PostsState, comments: CommentsState, users: UsersState}
export type AppDispatch = typeof store.dispatch
```

### 定义类型的钩子

虽然可以将`RootState`和`AppDispatch`类型导入到每个组件中，但最好创建`useDispatch`和`useSelector`钩子的预类型版本，以便在应用程序中使用。这一点很重要，原因如下:
> While it's possible to import the RootState and AppDispatch types into each component, it's better to create pre-typed versions of the useDispatch and useSelector hooks for usage in your application. This is important for a couple reasons:

- 对于`useSelector`，它为您节省了每次键入(`state: RootState`)的需要
- 对于`useDispatch`，默认的分派类型不知道`thunks`或其他中间件。为了正确地分发`thunk`，您需要从包含`thunk`中间件类型的商店中使用特定的定制`AppDispatch`类型，并与`useDispatch`一起使用。添加一个预类型`useDispatch`钩子可以防止你忘记在需要的地方导入`AppDispatch`。
> - For useSelector, it saves you the need to type (state: RootState) every time
> - For useDispatch, the default Dispatch type does not know about thunks or other middleware. In order to correctly dispatch thunks, you need to use the specific customized AppDispatch type from the store that includes the thunk middleware types, and use that with useDispatch. Adding a pre-typed useDispatch hook keeps you from forgetting to import AppDispatch where it's needed.

因为这些是实际的变量，而不是类型，所以在单独的文件中定义它们是很重要的，比如`app/hooks.ts`，而不是存储安装文件。这允许您将它们导入到任何需要使用钩子的组件文件中，并避免潜在的循环导入依赖问题。
> Since these are actual variables, not types, it's important to define them in a separate file such as app/hooks.ts, not the store setup file. This allows you to import them into any component file that needs to use the hooks, and avoids potential circular import dependency issues.

```js
// app/hooks.ts

import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux'
import type { RootState, AppDispatch } from './store'

// Use throughout your app instead of plain `useDispatch` and `useSelector`
export const useAppDispatch = () => useDispatch<AppDispatch>()
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector
```

## 手动键入Hooks

我们建议使用上面所示的预类型的`useAppSelector`和`useAppDispatch`钩子。如果您不喜欢使用它们，下面是如何自己键入钩子。
> We recommend using the pre-typed useAppSelector and useAppDispatch hooks shown above. If you prefer not to use those, here is how to type the hooks by themselves.

### Typing the useSelector hook

当编写与`useSelector`一起使用的选择器函数时，您应该显式定义状态形参的类型。然后`TS`应该能够推断选择器的返回类型，它将被重用为`useSelector`钩子的返回类型:
> When writing selector functions for use with useSelector, you should explicitly define the type of the state parameter. TS should be able to then infer the return type of the selector, which will be reused as the return type of the useSelector hook:

```js
interface RootState {
  isOn: boolean
}

// TS infers type: (state: RootState) => boolean
const selectIsOn = (state: RootState) => state.isOn

// TS infers `isOn` is boolean
const isOn = useSelector(selectIsOn)
```

这也可以内联完成:
> This can also be done inline as well:

```js
const isOn = useSelector( (state: RootState) => state.isOn)
```

### Typing the useDispatch hook

默认情况下，`useDispatch`的返回值是`Redux`核心类型定义的标准分派类型，所以不需要声明:
> By default, the return value of useDispatch is the standard Dispatch type defined by the Redux core types, so no declarations are needed:

```js
const dispatch = useDispatch()
```

如果你有`Dispatch`类型的自定义版本，你可以显式地使用该类型:
> If you have a customized version of the Dispatch type, you may use that type explicitly:

```js
// store.ts
export type AppDispatch = typeof store.dispatch

// MyComponent.tsx
const dispatch: AppDispatch = useDispatch()
```

## 键入`connect`高阶组件

### Inferring The Connected Props Automatically

`connect`由两个顺序调用的函数组成。第一个函数接受`mapState`和`mapDispatch`作为参数，并返回第二个函数。第二个函数接受要包装的组件，并返回一个新的包装组件，该组件从`mapState`和`mapDispatch`传递道具。通常，这两个函数是一起调用的，比如`connect(mapState, mapDispatch)(MyComponent)`。
> connect consists of two functions that are called sequentially. The first function accepts mapState and mapDispatch as arguments, and returns a second function. The second function accepts the component to be wrapped, and returns a new wrapper component that passes down the props from mapState and mapDispatch. Normally, both functions are called together, like connect(mapState, mapDispatch)(MyComponent).

在v7.1.2中，`@types/react-redux`包公开了一个帮助器类型`ConnectedProps`，它可以从第一个函数中提取`mapStateToProps`和`mapDispatchToProps`的返回类型。这意味着，如果您将`connect`调用分成两个步骤，那么所有来自`Redux`的“`props`”都可以自动推断出来，而不需要手工编写它们。虽然如果您已经使用`React-Redux`一段时间了，这种方法可能会感觉不太寻常，但它确实在很大程度上简化了类型声明。
> As of v7.1.2, the @types/react-redux package exposes a helper type, ConnectedProps, that can extract the return types of mapStateToProps and mapDispatchToProps from the first function. This means that if you split the connect call into two steps, all of the "props from Redux" can be inferred automatically without having to write them by hand. While this approach may feel unusual if you've been using React-Redux for a while, it does simplify the type declarations considerably.

```js
import { connect, ConnectedProps } from 'react-redux'

interface RootState {
  isOn: boolean
}

const mapState = (state: RootState) => ({
  isOn: state.isOn,
})

const mapDispatch = {
  toggleOn: () => ({ type: 'TOGGLE_IS_ON' }),
}

const connector = connect(mapState, mapDispatch)

// The inferred type will look like:
// {isOn: boolean, toggleOn: () => void}
type PropsFromRedux = ConnectedProps<typeof connector>
```

`ConnectedProps`的返回类型可以用于输入`props`对象的类型。
> The return type of ConnectedProps can then be used to type your props object.

```js
interface Props extends PropsFromRedux {
  backgroundColor: string
}

const MyComponent = (props: Props) => (
  <div style={{ backgroundColor: props.backgroundColor }}>
    <button onClick={props.toggleOn}>
      Toggle is {props.isOn ? 'ON' : 'OFF'}
    </button>
  </div>
)

export default connector(MyComponent)
```

因为类型可以以任何顺序定义，所以如果需要，您仍然可以在声明连接器之前声明组件。
> Because types can be defined in any order, you can still declare your component before declaring the connector if you want.

```js
// alternately, declare `type Props = PropsFromRedux & {backgroundColor: string}`
interface Props extends PropsFromRedux {
  backgroundColor: string;
}

const MyComponent = (props: Props) => /* same as above */

const connector = connect(/* same as above*/)

type PropsFromRedux = ConnectedProps<typeof connector>

export default connector(MyComponent)
```

### 手动键入`connect`

`connect`高阶组件的类型有点复杂，因为`props`有3个来源:`mapStateToProps`、`mapDispatchToProps`和从父组件传入的`props`。下面是一个完整的示例，说明手动执行该操作的效果。
> The connect higher-order component is somewhat complex to type, because there are 3 sources of props: mapStateToProps, mapDispatchToProps, and props passed in from the parent component. Here's a full example of what it looks like to do that manually.

```js
import { connect } from 'react-redux'

interface StateProps {
  isOn: boolean
}

interface DispatchProps {
  toggleOn: () => void
}

interface OwnProps {
  backgroundColor: string
}

type Props = StateProps & DispatchProps & OwnProps

const mapState = (state: RootState) => ({
  isOn: state.isOn,
})

const mapDispatch = {
  toggleOn: () => ({ type: 'TOGGLE_IS_ON' }),
}

const MyComponent = (props: Props) => (
  <div style={{ backgroundColor: props.backgroundColor }}>
    <button onClick={props.toggleOn}>
      Toggle is {props.isOn ? 'ON' : 'OFF'}
    </button>
  </div>
)

// Typical usage: `connect` is called after the component is defined
export default connect<StateProps, DispatchProps, OwnProps>(
  mapState,
  mapDispatch
)(MyComponent)
```

也可以通过推断`mapState`和`mapDispatch`的类型来略为缩短:
> It is also possible to shorten this somewhat, by inferring the types of mapState and mapDispatch:

```js
const mapState = (state: RootState) => ({
  isOn: state.isOn,
})

const mapDispatch = {
  toggleOn: () => ({ type: 'TOGGLE_IS_ON' }),
}

type StateProps = ReturnType<typeof mapState>
type DispatchProps = typeof mapDispatch

type Props = StateProps & DispatchProps & OwnProps
```

然而，如果`mapDispatch`定义为对象并同时指向`thunk`，那么用这种方式推断`mapDispatch`的类型将会失败。
> However, inferring the type of mapDispatch this way will break if it is defined as an object and also refers to thunks.

## 建议

`hook API`通常更容易与静态类型一起使用。如果您正在寻找与`React-Redux`一起使用静态类型的最简单的解决方案，请使用`hooks API`。
> The hooks API is generally simpler to use with static types. If you're looking for the easiest solution for using static types with React-Redux, use the hooks API.

如果您正在使用`connect`，我们建议使用`ConnectedProps<T>`方法从`Redux`推断`props`，因为这需要最少的显式类型声明。
> If you're using connect, we recommend using the ConnectedProps<T> approach for inferring the props from Redux, as that requires the fewest explicit type declarations.
