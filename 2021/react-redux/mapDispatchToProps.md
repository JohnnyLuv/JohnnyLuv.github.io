---
title: React Redux 7.x 中文文档 - mapDispatchToProps
date: 2021-04-07 15:32:35
categories: JavaScript
tags: [React, Redux]
---

# Connect: 使用mapDispatchToProps来调度操作

作为传入`connect`的第二个参数，`mapDispatchToProps`用于将操作分派到存储。
> As the second argument passed in to connect, mapDispatchToProps is used for dispatching actions to the store.

调度是`Redux store`的一个函数。你调用`store.dispatch`派发一个`action`。这是触发状态更改的唯一方法。
> dispatch is a function of the Redux store. You call store.dispatch to dispatch an action. This is the only way to trigger a state change.

使用`React Redux`，您的组件永远不会直接访问存储区 - `connect`为您做到了这一点。`React Redux`提供了两种让组件分派动作的方法:
- 默认情况下，已连接组件接收`props`。调度和能够调度操作本身。
- `connect`可以接受一个名为`mapDispatchToProps`的参数，它允许您创建在调用时进行分派的函数，并将这些函数作为`props`传递给组件。
> With React Redux, your components never access the store directly - connect does it for you. React Redux gives you two ways to let components dispatch actions:
> - By default, a connected component receives props.dispatch and can dispatch actions itself.
> - connect can accept an argument called mapDispatchToProps, which lets you create functions that dispatch when called, and pass those functions as props to your component.

`mapDispatchToProps`函数通常简称为`mapDispatch`，但实际使用的变量名可以是您想要的任何名称。
> The mapDispatchToProps functions are normally referred to as mapDispatch for short, but the actual variable name used can be whatever you want.

## 方法调度

### 默认:作为`Prop`调度

如果没有为`connect()`指定第二个参数，组件将默认接收分派。例如:
> If you don't specify the second argument to connect(), your component will receive dispatch by default. For example:

```js
connect()(MyComponent)
// which is equivalent with
connect(null, null)(MyComponent)

// or
connect(mapStateToProps /** no second argument */)(MyComponent)
```

一旦以这种方式连接了组件，组件就会接收`props.dispatch`。您可以使用它将操作分派到`store`。
> Once you have connected your component in this way, your component receives props.dispatch. You may use it to dispatch actions to the store.

```js
function Counter({ count, dispatch }) {
  return (
    <div>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <span>{count}</span>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>reset</button>
    </div>
  )
}
```

### 提供一个`mapDispatchToProps`参数

通过提供`mapDispatchToProps`，您可以指定组件可能需要分派哪些操作。它允许你提供动作分派功能作为道具。因此，你可以直接调用`props.increment()`，而不是调用`props.dispatch(() => increment())`。有几个原因可以解释为什么你想这样做。
> Providing a mapDispatchToProps allows you to specify which actions your component might need to dispatch. It lets you provide action dispatching functions as props. Therefore, instead of calling props.dispatch(() => increment()), you may call props.increment() directly. There are a few reasons why you might want to do that.

**更多的声明**

首先，将分派逻辑封装到函数中使实现更具声明性。分派一个操作并让`Redux store`处理数据流是如何实现该行为，而不是它做什么。
> First, encapsulating the dispatch logic into function makes the implementation more declarative. Dispatching an action and letting the Redux store handle the data flow is how to implement the behavior, rather than what it does.

一个很好的例子就是在单击按钮时分派操作。直接连接按钮在概念上可能没有意义，拥有按钮参考分派也没有意义。
> A good example would be dispatching an action when a button is clicked. Connecting the button directly probably doesn't make sense conceptually, and neither does having the button reference dispatch.

```js
// button needs to be aware of "dispatch"
<button onClick={() => dispatch({ type: "SOMETHING" })} />

// button unaware of "dispatch",
<button onClick={doSomething} />
```

一旦你用分派动作的函数包装了所有的动作创建者，组件就不需要分派了。因此，如果您定义自己的`mapDispatchToProps`，则连接的组件将不再接收分派。
> Once you've wrapped all our action creators with functions that dispatch the actions, the component is free of the need of dispatch. Therefore, if you define your own mapDispatchToProps, the connected component will no longer receive dispatch.

**将动作分派逻辑传递给(未连接的)子组件**

此外，您还可以将操作分派函数传递给子组件(可能是未连接的)。这允许更多的组件分派操作，同时保持它们“`unaware`”`Redux`。
> In addition, you also gain the ability to pass down the action dispatching functions to child ( likely unconnected ) components. This allows more components to dispatch actions, while keeping them "unaware" of Redux.

```js
// pass down toggleTodo to child component
// making Todo able to dispatch the toggleTodo action
const TodoList = ({ todos, toggleTodo }) => (
  <div>
    {todos.map((todo) => (
      <Todo todo={todo} onClick={toggleTodo} />
    ))}
  </div>
)
```

这就是`React Redux`的`connect`所做的——它封装了与`Redux store`对话的逻辑，让你不必担心它。这是你在实现中应该充分利用的。
> This is what React Redux’s connect does — it encapsulates the logic of talking to the Redux store and lets you not worry about it. And this is what you should totally make full use of in your implementation.

## mapDispatchToProps的两种形式

`mapDispatchToProps`参数有两种形式。虽然函数形式允许更多的定制，但对象形式很容易使用。
> The mapDispatchToProps parameter can be of two forms. While the function form allows more customization, the object form is easy to use.

- 函数形式:允许更多的定制，获得对分派和可选的`ownProps`的访问权
- 对象简写形式:声明性更强，更容易使用
> - Function form: Allows more customization, gains access to dispatch and optionally ownProps
> - Object shorthand form: More declarative and easier to use

⭐注意:我们推荐使用`mapDispatchToProps`的对象形式，除非你特别需要以某种方式自定义分派行为。
> ⭐ Note: We recommend using the object form of mapDispatchToProps unless you specifically need to customize dispatching behavior in some way.

## 将mapDispatchToProps定义为函数

将`mapDispatchToProps`定义为函数可以让您在定制组件接收的函数以及它们如何分派操作时拥有最大的灵活性。你可以访问`dispatch`和`ownProps`。您可以利用这个机会编写由连接的组件调用的自定义函数。
> Defining mapDispatchToProps as a function gives you the most flexibility in customizing the functions your component receives, and how they dispatch actions. You gain access to dispatch and ownProps. You may use this chance to write customized functions to be called by your connected components.

### Arguments

1. dispatch
2. ownProps (optional)

**dispatch**

`mapDispatchToProps`函数将以`dispatch`作为第一个参数被调用。你通常会通过返回在自身内部调用`dispatch()`的新函数来使用它，或者直接传入一个普通的动作对象，或者传入一个动作创建者的结果。
> The mapDispatchToProps function will be called with dispatch as the first argument. You will normally make use of this by returning new functions that call dispatch() inside themselves, and either pass in a plain action object directly or pass in the result of an action creator.

```js
const mapDispatchToProps = (dispatch) => {
  return {
    // dispatching plain actions
    increment: () => dispatch({ type: 'INCREMENT' }),
    decrement: () => dispatch({ type: 'DECREMENT' }),
    reset: () => dispatch({ type: 'RESET' }),
  }
}
```

你也可能想要把参数转发给你的`action creators`:
> You will also likely want to forward arguments to your action creators:

```js
const mapDispatchToProps = (dispatch) => {
  return {
    // explicitly forwarding arguments
    onClick: (event) => dispatch(trackClick(event)),

    // implicitly forwarding arguments
    onReceiveImpressions: (...impressions) =>
      dispatch(trackImpressions(impressions)),
  }
}
```

**ownProps ( optional )**

如果您的`mapDispatchToProps`函数被声明为接受两个参数，那么它将被调用，`dispatch`作为第一个参数，`props`作为第二个参数传递给连接的组件，并且当连接的组件接收到新的`props`时将被重新调用。
> If your mapDispatchToProps function is declared as taking two parameters, it will be called with dispatch as the first parameter and the props passed to the connected component as the second parameter, and will be re-invoked whenever the connected component receives new props.

这意味着，当组件的道具发生变化时，你可以不需要在组件重新渲染时重新绑定新的道具到动作调度程序。
> This means, instead of re-binding new props to action dispatchers upon component re-rendering, you may do so when your component's props change.

**组件挂载绑定**

```js
render() {
  return <button onClick={() => this.props.toggleTodo(this.props.todoId)} />
}

const mapDispatchToProps = dispatch => {
  return {
    toggleTodo: todoId => dispatch(toggleTodo(todoId))
  }
}
```

**绑定`props`的改变**

```js
render() {
  return <button onClick={() => this.props.toggleTodo()} />
}

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    toggleTodo: () => dispatch(toggleTodo(ownProps.todoId))
  }
}
```

### Return

你的`mapDispatchToProps`函数应该返回一个普通对象:
- 对象中的每个字段将成为你自己组件的一个单独的道具，值通常应该是一个函数，在调用时分派一个动作。
- 如果你在分派内部使用动作创建者(相对于普通对象动作)，按照惯例，只需将字段键命名为与动作创建者相同的名称:
> Your mapDispatchToProps function should return a plain object:
> - Each field in the object will become a separate prop for your own component, and the value should normally be a function that dispatches an action when called.
> - If you use action creators ( as oppose to plain object actions ) inside dispatch, it is a convention to simply name the field key the same name as the action creator:

```js
const increment = () => ({ type: 'INCREMENT' })
const decrement = () => ({ type: 'DECREMENT' })
const reset = () => ({ type: 'RESET' })

const mapDispatchToProps = (dispatch) => {
  return {
    // dispatching actions returned by action creators
    increment: () => dispatch(increment()),
    decrement: () => dispatch(decrement()),
    reset: () => dispatch(reset()),
  }
}
```

`mapDispatchToProps`函数的返回将作为`props`合并到已连接的组件中。你可以直接调用他们来分派它的行动。
> The return of the mapDispatchToProps function will be merged to your connected component as props. You may call them directly to dispatch its action.

```js
function Counter({ count, increment, decrement, reset }) {
  return (
    <div>
      <button onClick={decrement}>-</button>
      <span>{count}</span>
      <button onClick={increment}>+</button>
      <button onClick={reset}>reset</button>
    </div>
  )
}
```

### 用bindactioncreator定义mapDispatchToProps函数

手工包装这些函数非常繁琐，因此`Redux`提供了一个函数来简化这一过程。
> Wrapping these functions by hand is tedious, so Redux provides a function to simplify that.

`bindactioncreator`将值为操作创建者的对象转换为具有相同键的对象，但将每个操作创建者包装到一个分派调用中，以便可以直接调用它们。请参阅关于`bindactioncreator`的`Redux`文档
> bindActionCreators turns an object whose values are action creators, into an object with the same keys, but with every action creator wrapped into a dispatch call so they may be invoked directly. See Redux Docs on bindActionCreators

`bindactioncreator`接受两个参数:
- 一个函数(`action creator`)或一个对象(每个字段都是操作创建者)
- `dispatch`
> bindActionCreators accepts two parameters:
> - A function (an action creator) or an object (each field an action creator)
> - dispatch

由`bindactioncreator`生成的包装器函数将自动转发它们的所有参数，因此您不需要手工转发。
> The wrapper functions generated by bindActionCreators will automatically forward all of their arguments, so you don't need to do that by hand.

```js
import { bindActionCreators } from 'redux'

const increment = () => ({ type: 'INCREMENT' })
const decrement = () => ({ type: 'DECREMENT' })
const reset = () => ({ type: 'RESET' })

// binding an action creator
// returns (...args) => dispatch(increment(...args))
const boundIncrement = bindActionCreators(increment, dispatch)

// binding an object full of action creators
const boundActionCreators = bindActionCreators(
  { increment, decrement, reset },
  dispatch
)
// returns
// {
//   increment: (...args) => dispatch(increment(...args)),
//   decrement: (...args) => dispatch(decrement(...args)),
//   reset: (...args) => dispatch(reset(...args)),
// }
```

在我们的`mapDispatchToProps`函数中使用`bindactioncreator`:
> To use bindActionCreators in our mapDispatchToProps function:

```js
import { bindActionCreators } from 'redux'
// ...

function mapDispatchToProps(dispatch) {
  return bindActionCreators({ increment, decrement, reset }, dispatch)
}

// component receives props.increment, props.decrement, props.reset
connect(null, mapDispatchToProps)(Counter)
```

### 手动注入dispatch

如果提供了`mapDispatchToProps`参数，组件将不再接收默认的分派。你可以手动将它添加到`mapDispatchToProps`的返回值中，但大多数时候你不需要这样做:
> If the mapDispatchToProps argument is supplied, the component will no longer receive the default dispatch. You may bring it back by adding it manually to the return of your mapDispatchToProps, although most of the time you shouldn’t need to do this:

```js
import { bindActionCreators } from 'redux'
// ...

function mapDispatchToProps(dispatch) {
  return {
    dispatch,
    ...bindActionCreators({ increment, decrement, reset }, dispatch),
  }
}
```

## 将mapDispatchToProps定义为一个对象

你已经看到在`React`组件中调度`Redux actions`的设置遵循一个非常相似的过程:定义一个动作创建者，将它包装在另一个看起来像`(…args) => dispatch(actionCreator(…args))`的函数中，并将这个包装函数作为一个道具传递给你的组件。
> You’ve seen that the setup for dispatching Redux actions in a React component follows a very similar process: define an action creator, wrap it in another function that looks like (…args) => dispatch(actionCreator(…args)), and pass that wrapper function as a prop to your component.

因为这很常见，`connect`支持`mapDispatchToProps`参数的“对象简写”形式:如果你传递的是一个充满动作创建者的对象，而不是一个函数，`connect`会自动在内部为你调用`bindactioncreator`。
> Because this is so common, connect supports an “object shorthand” form for the mapDispatchToProps argument: if you pass an object full of action creators instead of a function, connect will automatically call bindActionCreators for you internally.

我们建议总是使用`mapDispatchToProps`的“对象简写”形式，除非你有特定的理由来定制分派行为。
> We recommend always using the “object shorthand” form of mapDispatchToProps, unless you have a specific reason to customize the dispatching behavior.

注意:
- `mapDispatchToProps`对象的每个字段都被假定为一个操作创建者
- 你的组件将不再接收分派作为一个道具
> Note that:
> - Each field of the mapDispatchToProps object is assumed to be an action creator
> - Your component will no longer receive dispatch as a prop

```js
// React Redux does this for you automatically:
;(dispatch) => bindActionCreators(mapDispatchToProps, dispatch)
```

因此，我们的`mapDispatchToProps`可以简单地写成:
> Therefore, our mapDispatchToProps can simply be:

```js
const mapDispatchToProps = {
  increment,
  decrement,
  reset,
}
```

因为变量的实际名称由你决定，你可能想给它一个像`actioncreator`这样的名称，或者甚至在连接调用中内联定义对象:
> Since the actual name of the variable is up to you, you might want to give it a name like actionCreators, or even define the object inline in the call to connect:

```js
import { increment, decrement, reset } from './counterActions'

const actionCreators = {
  increment,
  decrement,
  reset,
}

export default connect(mapState, actionCreators)(Counter)

// or
export default connect(mapState, { increment, decrement, reset })(Counter)
```

## 常见问题

### 为什么我的组件不能接收分派?

也被称为
> Also known as

```js
// TypeError: this.props.dispatch is not a function
```

这是一个常见的错误，当你尝试调用`this.props.dispatch`，但`dispatch`没有注入到你的组件中。
> This is a common error that happens when you try to call this.props.dispatch , but dispatch is not injected to your component.

只有在以下情况下，分派才会被注入到组件中:
> dispatch is injected to your component only when:

1. 你没有提供`mapDispatchToProps`
> 1. You do not provide mapDispatchToProps

默认的`mapDispatchToProps`只是简单地`dispatch => ({ dispatch })`。如果您不提供`mapDispatchToProps`, `dispatch`将如上所述提供。
> The default mapDispatchToProps is simply dispatch => ({ dispatch }). If you do not provide mapDispatchToProps, dispatch will be provided as mentioned above.

换句话说，如果你这样做:
> In another words, if you do:

```js
// component receives `dispatch`
connect(mapStateToProps /** no second argument*/)(Component)
```

2. 您自定义的`mapDispatchToProps`函数的返回值特别包含`dispatch`
> 2. Your customized mapDispatchToProps function return specifically contains dispatch

你可以通过提供自定义的`mapDispatchToProps`函数来恢复调度:
> You may bring back dispatch by providing your customized mapDispatchToProps function:

```js
const mapDispatchToProps = (dispatch) => {
  return {
    increment: () => dispatch(increment()),
    decrement: () => dispatch(decrement()),
    reset: () => dispatch(reset()),
    dispatch,
  }
}
```

或者，使用`bindactioncreator`:
> Or alternatively, with bindActionCreators:

```js
import { bindActionCreators } from 'redux'

function mapDispatchToProps(dispatch) {
  return {
    dispatch,
    ...bindActionCreators({ increment, decrement, reset }, dispatch),
  }
}
```

当你指定`mapDispatchToProps`时，是否要为你的组件提供分派([Dan Abramov对#255的回应](https://github.com/reduxjs/react-redux/issues/255#issuecomment-172089874))。为了进一步理解当前的实现意图，您可以阅读它们。
> There are discussions regarding whether to provide dispatch to your components when you specify mapDispatchToProps ( Dan Abramov’s response to #255 ). You may read them for further understanding of the current implementation intention.

### 我可以mapDispatchToProps没有mapStateToProps在Redux?

是的。您可以通过传递`undefined`或`null`跳过第一个参数。您的组件不会订阅存储，仍然会接收`mapDispatchToProps`定义的分派道具。
> Yes. You can skip the first parameter by passing undefined or null. Your component will not subscribe to the store, and will still receive the dispatch props defined by mapDispatchToProps.

```js
connect(null, mapDispatchToProps)(MyComponent)
```

### 我可以调用store.dispatch吗？

在`React`组件中直接与商店交互是一种反模式，无论是显式导入商店还是通过上下文访问它(有关商店设置的详细信息，请参阅`Redux`常见问题解答条目)。让`React Redux`的`connect`处理对商店的访问，并使用它传递给道具的分派来分派动作。
> It's an anti-pattern to interact with the store directly in a React component, whether it's an explicit import of the store or accessing it via context (see the Redux FAQ entry on store setup for more details). Let React Redux’s connect handle the access to the store, and use the dispatch it passes to the props to dispatch actions.
