---
title: React Redux 7.x 中文文档 - API connect()
date: 2021-04-07 17:35:35
categories: JavaScript
tags: [React, Redux]
---

# connect()

## 概述

`connect()`函数的作用是将`React`组件连接到`Redux store`。
> The connect() function connects a React component to a Redux store.

它向其连接的组件提供存储所需的数据片段，以及用于将操作分派到存储区的函数。
> It provides its connected component with the pieces of the data it needs from the store, and the functions it can use to dispatch actions to the store.

它不会修改传递给它的组件类;相反，它返回一个新的、连接的组件类，它包装了传入的组件。
> It does not modify the component class passed to it; instead, it returns a new, connected component class that wraps the component you passed in.

```js
function connect(mapStateToProps?, mapDispatchToProps?, mergeProps?, options?)
```

`mapStateToProps`和`mapDispatchToProps`分别处理`Redux`存储的状态和调度。状态和调度将作为第一个参数提供给您的`mapStateToProps`或`mapDispatchToProps`函数。
> The mapStateToProps and mapDispatchToProps deals with your Redux store’s state and dispatch, respectively. state and dispatch will be supplied to your mapStateToProps or mapDispatchToProps functions as the first argument.

`mapStateToProps`和`mapDispatchToProps`的返回值在内部分别被称为`statprops`和`dispatchProps`。如果定义了，它们将被提供给`mergeProps`，作为第一个和第二个参数，其中第三个参数将是`ownProps`。组合后的结果(通常称为`mergedProps`)将被提供给连接的组件。
> The returns of mapStateToProps and mapDispatchToProps are referred to internally as stateProps and dispatchProps, respectively. They will be supplied to mergeProps, if defined, as the first and the second argument, where the third argument will be ownProps. The combined result, commonly referred to as mergedProps, will then be supplied to your connected component.

## connect() Parameters

`connect`接受四个不同的参数，它们都是可选的。按照惯例，它们被称为:
> connect accepts four different parameters, all optional. By convention, they are called:

1. mapStateToProps?: Function
2. mapDispatchToProps?: Function | Object
3. mergeProps?: Function
4. options?: Object

### mapStateToProps?: (state, ownProps?) => Object

如果指定了`mapStateToProps`函数，新的包装器组件将订阅`Redux`存储更新。这意味着每当存储被更新时，`mapStateToProps`都会被调用。`mapStateToProps`的结果必须是一个普通对象，它将被合并到包装组件的`props`中。如果你不想订阅存储更新，传递`null`或`undefined`来代替`mapStateToProps`。
> If a mapStateToProps function is specified, the new wrapper component will subscribe to Redux store updates. This means that any time the store is updated, mapStateToProps will be called. The results of mapStateToProps must be a plain object, which will be merged into the wrapped component’s props. If you don't want to subscribe to store updates, pass null or undefined in place of mapStateToProps.

> Parameters
> 1. state: Object
> 2. ownProps?: Object

`mapStateToProps`函数最多接受两个参数。声明的函数参数的数量(也称为arity)会影响它何时被调用。这也决定了函数是否接收`ownProps`。看到笔记。
> A mapStateToProps function takes a maximum of two parameters. The number of declared function parameters (a.k.a. arity) affects when it will be called. This also determines whether the function will receive ownProps. See notes here.

**state**

如果您的`mapStateToProps`函数声明为接受一个参数，那么在存储状态发生变化时就会调用它，并将存储状态作为唯一的参数。
> If your mapStateToProps function is declared as taking one parameter, it will be called whenever the store state changes, and given the store state as the only parameter.

```js
const mapStateToProps = (state) => ({ todos: state.todos })
```

**ownProps**

如果`mapStateToProps`函数声明为接受两个参数，那么当存储状态发生变化或包装器组件接收到新的`props`时(基于浅相等性比较)，它就会被调用。第一个参数是存储状态，第二个参数是包装器组件的道具。
> If your mapStateToProps function is declared as taking two parameters, it will be called whenever the store state changes or when the wrapper component receives new props (based on shallow equality comparisons). It will be given the store state as the first parameter, and the wrapper component's props as the second parameter.

根据约定，第二个参数通常称为`ownProps`。
> The second parameter is normally referred to as ownProps by convention.

```js
const mapStateToProps = (state, ownProps) => ({
  todo: state.todos[ownProps.id],
})
```

**Returns**

`mapStateToProps`函数将返回一个对象。该对象通常称为`statprops`，它将被合并为已连接组件的`props`。如果您定义了`mergeProps`，它将作为`mergeProps`的第一个参数提供。
> Your mapStateToProps functions are expected to return an object. This object, normally referred to as stateProps, will be merged as props to your connected component. If you define mergeProps, it will be supplied as the first parameter to mergeProps.

`mapStateToProps`的返回将决定是否重新呈现连接的组件([详细信息在这里](https://react-redux.js.org/using-react-redux/connect-mapstate#return-values-determine-if-your-component-re-renders))。
> The return of the mapStateToProps determine whether the connected component will re-render (details here).

关于`mapStateToProps`推荐用法的更多细节，请参阅我们的[`mapStateToProps`使用指南](https://react-redux.js.org/using-react-redux/connect-mapstate)。
> For more details on recommended usage of mapStateToProps, please refer to our guide on using mapStateToProps.

你可以将`mapStateToProps`和`mapDispatchToProps`定义为一个工厂函数，也就是说，你可以返回一个函数而不是一个对象。在这种情况下，返回的函数将被视为真正的`mapStateToProps`或`mapDispatchToProps`，并在后续调用中被调用。你可能会看到工厂函数的注释或我们的性能优化指南。
> You may define mapStateToProps and mapDispatchToProps as a factory function, i.e., you return a function instead of an object. In this case your returned function will be treated as the real mapStateToProps or mapDispatchToProps, and be called in subsequent calls. You may see notes on Factory Functions or our guide on performance optimizations.

### mapDispatchToProps?: Object | (dispatch, ownProps?) => Object

按照惯例称为`mapDispatchToProps`, `connect()`的第二个参数可以是对象、函数，也可以不提供。
> Conventionally called mapDispatchToProps, this second parameter to connect() may either be an object, a function, or not supplied.

你的组件将默认接收分派，也就是说，当你没有为`connect()`提供第二个参数时:
> Your component will receive dispatch by default, i.e., when you do not supply a second parameter to connect():

```js
// do not pass `mapDispatchToProps`
connect()(MyComponent)
connect(mapState)(MyComponent)
connect(mapState, null, mergeProps, options)(MyComponent)
```

如果您将`mapDispatchToProps`定义为一个函数，则调用它时最多只带两个参数。
> If you define a mapDispatchToProps as a function, it will be called with a maximum of two parameters.

**Parameters**
1. dispatch: Function
2. ownProps?: Object

**dispatch**

如果`mapDispatchToProps`被声明为一个带有一个参数的函数，则会给它一个存储的分派。
> If your mapDispatchToProps is declared as a function taking one parameter, it will be given the dispatch of your store.

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

**ownProps**

如果您的`mapDispatchToProps`函数被声明为接受两个参数，那么它将被调用，`dispatch`作为第一个参数，`props`作为第二个参数传递给包装器组件，并且在连接的组件接收到新的`props`时将被重新调用。
> If your mapDispatchToProps function is declared as taking two parameters, it will be called with dispatch as the first parameter and the props passed to the wrapper component as the second parameter, and will be re-invoked whenever the connected component receives new props.

根据约定，第二个参数通常称为`ownProps`。
> The second parameter is normally referred to as ownProps by convention.

```js
// binds on component re-rendering
;<button onClick={() => this.props.toggleTodo(this.props.todoId)} />

// binds on `props` change
const mapDispatchToProps = (dispatch, ownProps) => ({
  toggleTodo: () => dispatch(toggleTodo(ownProps.todoId)),
})
```

`mapDispatchToProps`声明的函数参数的数量决定了它们是否接收`ownProps`。[详见](https://react-redux.js.org/api/connect#the-arity-of-maptoprops-functions)。
> The number of declared function parameters of mapDispatchToProps determines whether they receive ownProps. See notes here.

**Returns**

`mapDispatchToProps`函数将返回一个对象。对象的每个字段都应该是一个函数，调用它将向存储区发送一个操作。
> Your mapDispatchToProps functions are expected to return an object. Each fields of the object should be a function, calling which is expected to dispatch an action to the store.

`mapDispatchToProps`函数的返回被视为`dispatchProps`。它将被合并为连接组件的道具。如果您定义了`mergeProps`，它将作为第二个参数提供给`mergeProps`。
> The return of your mapDispatchToProps functions are regarded as dispatchProps. It will be merged as props to your connected component. If you define mergeProps, it will be supplied as the second parameter to mergeProps.

```js
const createMyAction = () => ({ type: 'MY_ACTION' })
const mapDispatchToProps = (dispatch, ownProps) => {
  const boundActions = bindActionCreators({ createMyAction }, dispatch)
  return {
    dispatchPlainObject: () => dispatch({ type: 'MY_ACTION' }),
    dispatchActionCreatedByActionCreator: () => dispatch(createMyAction()),
    ...boundActions,
    // you may return dispatch here
    dispatch,
  }
}
```

关于推荐使用的更多细节，请参阅我们的[`mapDispatchToProps`使用指南](https://react-redux.js.org/using-react-redux/connect-mapdispatch)。
> For more details on recommended usage, please refer to our guide on using mapDispatchToProps.

你可以将`mapStateToProps`和`mapDispatchToProps`定义为一个工厂函数，也就是说，你可以返回一个函数而不是一个对象。在这种情况下，返回的函数将被视为真正的`mapStateToProps`或`mapDispatchToProps`，并在后续调用中被调用。你可能会看到工厂函数的注释或我们的性能优化指南。
> You may define mapStateToProps and mapDispatchToProps as a factory function, i.e., you return a function instead of an object. In this case your returned function will be treated as the real mapStateToProps or mapDispatchToProps, and be called in subsequent calls. You may see notes on Factory Functions or our guide on performance optimizations.

**Object Shorthand Form**

`mapDispatchToProps`可能是一个对象，其中的每个字段都是一个动作创建者。
> mapDispatchToProps may be an object where each field is an action creator.

```js
import { addTodo, deleteTodo, toggleTodo } from './actionCreators'

const mapDispatchToProps = {
  addTodo,
  deleteTodo,
  toggleTodo,
}

export default connect(null, mapDispatchToProps)(TodoApp)
```

在本例中，`React-Redux`使用`bindactioncreator`将存储的分派绑定到每个操作创建者。结果将被视为`dispatchProps`，它将被直接合并到连接的组件中，或者作为第二个参数提供给`mergeProps`。
> In this case, React-Redux binds the dispatch of your store to each of the action creators using bindActionCreators. The result will be regarded as dispatchProps, which will be either directly merged to your connected components, or supplied to mergeProps as the second argument.

```js
// internally, React-Redux calls bindActionCreators
// to bind the action creators to the dispatch of your store
bindActionCreators(mapDispatchToProps, dispatch)
```

我们在`mapDispatchToProps`指南中也有一节介绍对象简写形式的[使用](https://react-redux.js.org/using-react-redux/connect-mapdispatch#defining-mapdispatchtoprops-as-an-object)。
> We also have a section in our mapDispatchToProps guide on the usage of object shorthand form here.

### mergeProps?: (stateProps, dispatchProps, ownProps) => Object

如果指定，则定义如何为自己包装的组件确定最终的道具。如果你没有提供`mergeProps`，你被包装的组件默认会收到`{ ...ownProps, ...stateProps, ...dispatchProps }`。
> If specified, defines how the final props for your own wrapped component are determined. If you do not provide mergeProps, your wrapped component receives { ...ownProps, ...stateProps, ...dispatchProps } by default.

**Parameters**

`mergeProps`最多应该指定三个参数。它们分别是`mapStateToProps()`、`mapDispatchToProps()`和包装器组件的`props`的结果:
> mergeProps should be specified with maximum of three parameters. They are the result of mapStateToProps(), mapDispatchToProps(), and the wrapper component's props, respectively:
1. stateProps
2. dispatchProps
3. ownProps

从它返回的普通对象中的字段将用作被包装组件的道具。您可以指定这个函数来根据`props`选择状态的切片，或者将动作创建者绑定到`props`中的特定变量。
> The fields in the plain object you return from it will be used as the props for the wrapped component. You may specify this function to select a slice of the state based on props, or to bind action creators to a particular variable from props.

**Returns**

`mergeProps`的返回值被称为`mergedProps`，这些字段将被用作被包装组件的道具。
> The return value of mergeProps is referred to as mergedProps and the fields will be used as the props for the wrapped component.

### options?: Object

```js
{
  context?: Object,
  pure?: boolean,
  areStatesEqual?: Function,
  areOwnPropsEqual?: Function,
  areStatePropsEqual?: Function,
  areMergedPropsEqual?: Function,
  forwardRef?: boolean,
}
```

**context: Object**

备注:仅>= v6.0版本支持此参数
> Note: This parameter is supported in >= v6.0 only

`React-Redux v6`允许您提供`React-Redux`使用的自定义上下文实例。你需要把你的上下文实例传递给`<Provider />`和你连接的组件。您可以将上下文传递给连接的组件，在这里将其作为选项字段传递，或者在呈现时将其作为连接的组件的道具传递。
> React-Redux v6 allows you to supply a custom context instance to be used by React-Redux. You need to pass the instance of your context to both <Provider /> and your connected component. You may pass the context to your connected component either by passing it here as a field of option, or as a prop to your connected component in rendering.

```js
// const MyContext = React.createContext();
connect(mapStateToProps, mapDispatchToProps, null, { context: MyContext })(
  MyComponent
)
```

**pure: boolean**
- default value: true

假设包装的组件是一个“纯”组件，不依赖于任何输入或状态，除了它的`props`和所选`Redux store`的状态。
> Assumes that the wrapped component is a “pure” component and does not rely on any input or state other than its props and the selected Redux store’s state.

当选择。如果`pure`为真，`connect`会执行一些相等性检查，以避免不必要地调用`mapStateToProps`、`mapDispatchToProps`、`mergeProps`以及最终渲染。这些包括`areStatesEqual`, `areOwnPropsEqual`, `areStatePropsEqual`和`areMergedPropsEqual`。虽然默认值可能在99%的情况下是合适的，但出于性能或其他原因，您可能希望使用自定义实现覆盖它们。
> When options.pure is true, connect performs several equality checks that are used to avoid unnecessary calls to mapStateToProps, mapDispatchToProps, mergeProps, and ultimately to render. These include areStatesEqual, areOwnPropsEqual, areStatePropsEqual, and areMergedPropsEqual. While the defaults are probably appropriate 99% of the time, you may wish to override them with custom implementations for performance or other reasons.

我们将在下面的章节中提供一些示例。
> We provide a few examples in the following sections.

**areStatesEqual: (next: Object, prev: Object) => boolean**
- default value: strictEqual: (next, prev) => prev === next

`pure`时，将传入的存储状态与之前的值进行比较。
> When pure, compares incoming store state to its previous value.

案例1

```js
const areStatesEqual = (next, prev) => prev.entities.todos === next.entities.todos
```

如果你的`mapStateToProps`函数在计算上很昂贵，而且只关心你的状态的一小部分，你可能希望重写`areStatesEqual`。上面的例子将有效地忽略除了那部分状态之外的所有状态更改。
> You may wish to override areStatesEqual if your mapStateToProps function is computationally expensive and is also only concerned with a small slice of your state. The example above will effectively ignore state changes for everything but that slice of state.

案例2

如果你有不纯的`reducers`来改变你的存储状态，你可能希望重写`areStatesEqual`以总是返回`false`:
> If you have impure reducers that mutate your store state, you may wish to override areStatesEqual to always return false:

```js
const areStatesEqual = () => false
```

这可能还会影响其他相等性检查，这取决于`mapStateToProps`函数。
> This would likely impact the other equality checks as well, depending on your mapStateToProps function.

**areOwnPropsEqual: (next: Object, prev: Object) => boolean**
- default value: shallowEqual: (objA, objB) => boolean ( returns true when each field of the objects is equal )

当`pure`时，将输入的道具与之前的值进行比较。
> When pure, compares incoming props to its previous value.

你可能希望重写`areOwnPropsEqual`作为一个传入道具的白名单。您还必须实现`mapStateToProps`、`mapDispatchToProps`和`mergeProps`，使它们也成为白名单道具。(使用其他方法可能更简单，例如使用`recompose的mapProps`。)
> You may wish to override areOwnPropsEqual as a way to whitelist incoming props. You'd also have to implement mapStateToProps, mapDispatchToProps and mergeProps to also whitelist props. (It may be simpler to achieve this other ways, for example by using recompose's mapProps.)

**areStatePropsEqual: (next: Object, prev: Object) => boolean**
- type: function
- default value: shallowEqual

当`pure`时，则将`mapStateToProps`的结果与之前的值进行比较。
> When pure, compares the result of mapStateToProps to its previous value.

**areMergedPropsEqual: (next: Object, prev: Object) => boolean**
- default value: shallowEqual

当`pure`时，则将`mergeProps`的结果与之前的值进行比较。
> When pure, compares the result of mergeProps to its previous value.

如果你的`mapStateToProps`使用了一个记忆化的选择器，只有在相关的`prop`发生变化时才会返回一个新对象，你可能希望重写`areStatePropsEqual`，使用`strictEqual`。这将是一个非常轻微的性能改进，因为它将避免每次调用`mapStateToProps`时对单个`props`进行额外的相等检查。
> You may wish to override areStatePropsEqual to use strictEqual if your mapStateToProps uses a memoized selector that will only return a new object if a relevant prop has changed. This would be a very slight performance improvement, since would avoid extra equality checks on individual props each time mapStateToProps is called.

如果你的选择器产生复杂的道具，你可能希望重写`areMergedPropsEqual`来实现`deepqual`。例如:嵌套对象，新数组，等等(深度相等检查可能比重新渲染更快。)
> You may wish to override areMergedPropsEqual to implement a deepEqual if your selectors produce complex props. ex: nested objects, new arrays, etc. (The deep equal check may be faster than just re-rendering.)

**forwardRef: boolean**

备注:仅>= v6.0版本支持此参数
> Note: This parameter is supported in >= v6.0 only

如果`{forwardRef: true}`已经被传递给了`connect`，那么给连接的包装器组件添加一个`ref`实际上会返回被包装组件的实例。
> If {forwardRef : true} has been passed to connect, adding a ref to the connected wrapper component will actually return the instance of the wrapped component.

## connect() Returns

`connect()`的返回是一个包装器函数，它接受您的组件，并返回一个包装器组件和它注入的额外道具。
> The return of connect() is a wrapper function that takes your component and returns a wrapper component with the additional props it injects.

```js
import { login, logout } from './actionCreators'

const mapState = (state) => state.user
const mapDispatch = { login, logout }

// first call: returns a hoc that you can use to wrap any component
const connectUser = connect(mapState, mapDispatch)

// second call: returns the wrapper component with mergedProps
// you may use the hoc to enable different components to get the same behavior
const ConnectedUserLogin = connectUser(Login)
const ConnectedUserProfile = connectUser(Profile)
```

在大多数情况下，包装器函数会立即被调用，而不会被保存在临时变量中:
> In most cases, the wrapper function will be called right away, without being saved in a temporary variable:

```js
import { login, logout } from './actionCreators'

const mapState = (state) => state.user
const mapDispatch = { login, logout }

// call connect to generate the wrapper function, and immediately call
// the wrapper function to generate the final wrapper component.

export default connect(mapState, mapDispatch)(Login)
```

## Example Usage

因为`connect`是如此灵活，它可能有助于看到一些额外的例子，它可以如何被调用:
- 注入只调度，不听存储
> Because connect is so flexible, it may help to see some additional examples of how it can be called:
> - Inject just dispatch and don't listen to store

```js
export default connect()(TodoApp)
```

- 在不订阅商店的情况下注入所有动作创建者(`addTodo, completeTodo，…`)
> - Inject all action creators (addTodo, completeTodo, ...) without subscribing to the store

```js
import * as actionCreators from './actionCreators'

export default connect(null, actionCreators)(TodoApp)
```

- 注入调度和全局状态中的每个字段
> - Inject dispatch and every field in the global state

不要这样做!它终止了任何性能优化，因为`TodoApp`会在每次状态更改后重新呈现。最好在视图层次结构中的几个组件上使用更细粒度的`connect()`，每个组件只侦听状态的一个相关切片。
> Don’t do this! It kills any performance optimizations because TodoApp will rerender after every state change. It’s better to have more granular connect() on several components in your view hierarchy that each only listen to a relevant slice of the state.

```js
// don't do this!
export default connect((state) => state)(TodoApp)
```

- 注入调度和待办事项
> - Inject dispatch and todos

```js
function mapStateToProps(state) {
  return { todos: state.todos }
}

export default connect(mapStateToProps)(TodoApp)
```

- 注入待办事项和所有动作创建者
> - Inject todos and all action creators

```js
import * as actionCreators from './actionCreators'

function mapStateToProps(state) {
  return { todos: state.todos }
}

export default connect(mapStateToProps, actionCreators)(TodoApp)
```

- 将`todos`和所有动作创建者(`addTodo, completeTodo，…`)注入为动作
> - Inject todos and all action creators (addTodo, completeTodo, ...) as actions

```js
import * as actionCreators from './actionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return { actions: bindActionCreators(actionCreators, dispatch) }
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

- 注入`todos`和一个特定的动作创建者(`addTodo`)
> - Inject todos and a specific action creator (addTodo)

```js
import { addTodo } from './actionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return bindActionCreators({ addTodo }, dispatch)
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

- 用简写语法注入待办事项和特定的动作创建者(`addTodo和deleteTodo`)
> - Inject todos and specific action creators (addTodo and deleteTodo) with shorthand syntax

```js
import { addTodo, deleteTodo } from './actionCreators'

function mapStateToProps(state) {
  return { todos: state.todos }
}

const mapDispatchToProps = {
  addTodo,
  deleteTodo,
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

- 将`todos、todoactioncreator`注入为`todoActions`，将`counteractioncreator`注入为`counterActions`
> - Inject todos, todoActionCreators as todoActions, and counterActionCreators as counterActions

```js
import * as todoActionCreators from './todoActionCreators'
import * as counterActionCreators from './counterActionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return {
    todoActions: bindActionCreators(todoActionCreators, dispatch),
    counterActions: bindActionCreators(counterActionCreators, dispatch),
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

- 将`todos、todoactioncreator`和`counteractioncreator`注入到一起作为动作
> - Inject todos, and todoActionCreators and counterActionCreators together as actions

```js
import * as todoActionCreators from './todoActionCreators'
import * as counterActionCreators from './counterActionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return {
    actions: bindActionCreators(
      { ...todoActionCreators, ...counterActionCreators },
      dispatch
    ),
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

直接将`todos`和所有`todoactioncreator`和`counteractioncreator`注入道具中
> - Inject todos, and all todoActionCreators and counterActionCreators directly as props

```js
import * as todoActionCreators from './todoActionCreators'
import * as counterActionCreators from './counterActionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return bindActionCreators(
    { ...todoActionCreators, ...counterActionCreators },
    dispatch
  )
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

- 根据道具注入特定用户的待办事项
> - Inject todos of a specific user depending on props

```js
import * as actionCreators from './actionCreators'

function mapStateToProps(state, ownProps) {
  return { todos: state.todos[ownProps.userId] }
}

export default connect(mapStateToProps)(TodoApp)
```

- 根据道具注入特定用户的待办事项，并注入`props.userId`转换到操作中
> - Inject todos of a specific user depending on props, and inject props.userId into the action

```js
import * as actionCreators from './actionCreators'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mergeProps(stateProps, dispatchProps, ownProps) {
  return Object.assign({}, ownProps, {
    todos: stateProps.todos[ownProps.userId],
    addTodo: (text) => dispatchProps.addTodo(ownProps.userId, text),
  })
}

export default connect(mapStateToProps, actionCreators, mergeProps)(TodoApp)
```

## Notes

### mapToProps函数的特性

`mapStateToProps`和`mapDispatchToProps`声明的函数参数的数量决定了它们是否接收`ownProps`
> The number of declared function parameters of mapStateToProps and mapDispatchToProps determines whether they receive ownProps

注意:如果函数的正式定义包含一个必选参数(函数长度为1)，则`ownProps`不会被传递给`mapStateToProps`和`mapDispatchToProps`。例如，下面定义的函数不会接收`ownProps`作为第二个参数。如果`ownProps`的传入值未定义，则使用默认实参值。
> Note: ownProps is not passed to mapStateToProps and mapDispatchToProps if the formal definition of the function contains one mandatory parameter (function has length 1). For example, functions defined like below won't receive ownProps as the second argument. If the incoming value of ownProps is undefined, the default argument value will be used.

```js
function mapStateToProps(state) {
  console.log(state) // state
  console.log(arguments[1]) // undefined
}

const mapStateToProps = (state, ownProps = {}) => {
  console.log(state) // state
  console.log(ownProps) // {}
}
```

没有强制参数或两个参数*的函数将接收`ownProps`。
> Functions with no mandatory parameters or two parameters*will receive ownProps.

```js
const mapStateToProps = (state, ownProps) => {
  console.log(state) // state
  console.log(ownProps) // ownProps
}

function mapStateToProps() {
  console.log(arguments[0]) // state
  console.log(arguments[1]) // ownProps
}

const mapStateToProps = (...args) => {
  console.log(args[0]) // state
  console.log(args[1]) // ownProps
}
```

### Factory Functions

如果你的`mapStateToProps`或`mapDispatchToProps`函数返回一个函数，它们将在组件实例化时被调用一次，它们的返回值将分别作为实际的`mapStateToProps`、`mapDispatchToProps`和函数，在它们的后续调用中使用。
> If your mapStateToProps or mapDispatchToProps functions return a function, they will be called once when the component instantiates, and their returns will be used as the actual mapStateToProps, mapDispatchToProps, functions respectively, in their subsequent calls.

工厂函数通常与记忆化的选择器一起使用。这让你能够在闭包中创建特定于组件实例的选择器:
> The factory functions are commonly used with memoized selectors. This gives you the ability to create component-instance-specific selectors inside the closure:

```js
const makeUniqueSelectorInstance = () =>
  createSelector([selectItems, selectItemId], (items, itemId) => items[itemId])
const makeMapState = (state) => {
  const selectItemForThisComponent = makeUniqueSelectorInstance()
  return function realMapState(state, ownProps) {
    const item = selectItemForThisComponent(state, ownProps.itemId)
    return { item }
  }
}
export default connect(makeMapState)(SomeComponent)
```
