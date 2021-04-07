---
title: React Redux 7.x 中文文档 - API connectAdvanced()
date: 2021-04-07 18:25:35
categories: JavaScript
tags: [React, Redux]
---

# connectAdvanced()

```js
connectAdvanced(selectorFactory, connectOptions?)
```

将`React`组件连接到`Redux`存储。它是`connect()`的基础，但对于如何将状态、`props`和`dispatch`组合到最终的`props`中并不那么固执。它对默认值或结果记忆没有任何假设，将这些责任留给调用者。
> Connects a React component to a Redux store. It is the base for connect() but is less opinionated about how to combine state, props, and dispatch into your final props. It makes no assumptions about defaults or memoization of results, leaving those responsibilities to the caller.

它不会修改传递给它的组件类;相反，它返回一个新的、连接的组件类供您使用。
> It does not modify the component class passed to it; instead, it returns a new, connected component class for you to use.

大多数应用程序不需要使用这个，因为`connect`中的默认行为适用于大多数用例。
> Most applications will not need to use this, as the default behavior in connect is intended to work for most use cases.


**注意**
`connectAdvanced`是在5.0版本中添加的，`connect`作为`connectAdvanced`的特定参数集重新实现。然而，`connectAdvanced`现在已弃用，并将在未来的`React Redux`主版本中被删除。
> connectAdvanced was added in version 5.0, and connect was reimplemented as a specific set of parameters to connectAdvanced. However, connectAdvanced is now deprecated and will eventually be removed in a future major version of React Redux.

### Arguments

- `selectorFactory(dispatch, factoryOptions): selector(state, ownProps): props` (`Function`):初始化一个选择器函数(在每个实例的构造函数中)。当连接器组件由于存储状态更改或接收新道具而需要计算新道具时，就会调用该选择器函数。选择器的结果应该是一个普通对象，它作为道具传递给被包装的组件。如果对选择器的连续调用返回与之前调用相同的对象(===)，组件将不会被重新渲染。选择器的职责是在适当的时候返回前一个对象。
- [`connectOptions`] (Object)如果指定，则进一步自定义连接器的行为。
  - [`getDisplayName`](函数):计算连接器组件的`displayName`属性相对于被包装组件的`displayName`属性。通常由包装器函数覆盖。默认值:`name => 'ConnectAdvanced('+name+')'`
  - [`methodName`](字符串):显示在错误消息中。通常由包装器函数覆盖。默认值:“`connectAdvanced`”
  - [`renderCountProp`] (String):如果定义了，一个名为这个值的属性将被添加到传递给包装组件的道具中。它的值将是组件被渲染的次数，这对于跟踪不必要的重新渲染非常有用。默认值:`undefined`
  - [`shouldHandleStateChanges`] (`Boolean`):控制连接器组件是否订阅还原存储状态更改。如果设置为`false`，则只在父组件重新呈现时才重新呈现。默认值:`true`
  - [`forwardRef`] (`Boolean`):如果为真值，添加一个ref到连接的包装器组件实际上会返回被包装组件的实例。
另外，任何通过连接传递的额外选项都将通过`factoryOptions`参数传递给你的`selectorFactory`。
  - selectorFactory(dispatch, factoryOptions): selector(state, ownProps): props (Function): Initializes a selector function (during each instance's constructor). That selector function is called any time the connector component needs to compute new props, as a result of a store state change or receiving new props. The result of selector is expected to be a plain object, which is passed as the props to the wrapped component. If a consecutive call to selector returns the same object (===) as its previous call, the component will not be re-rendered. It's the responsibility of selector to return that previous object when appropriate.
> - [connectOptions] (Object) If specified, further customizes the behavior of the connector.
>  - [getDisplayName] (Function): computes the connector component's displayName property relative to that of the wrapped component. Usually overridden by wrapper functions. Default value: name => 'ConnectAdvanced('+name+')'
>  - [methodName] (String): shown in error messages. Usually overridden by wrapper functions. Default value: 'connectAdvanced'
>  - [renderCountProp] (String): if defined, a property named this value will be added to the props passed to the wrapped component. Its value will be the number of times the component has been rendered, which can be useful for tracking down unnecessary re-renders. Default value: undefined
>  - [shouldHandleStateChanges] (Boolean): controls whether the connector component subscribes to redux store state changes. If set to false, it will only re-render when parent component re-renders. Default value: true
>  - [forwardRef] (Boolean): If true, adding a ref to the connected wrapper component will actually return the instance of the wrapped component.
>  - Additionally, any extra options passed via connectOptions will be passed through to your selectorFactory in the factoryOptions argument.

## Returns

一个更高阶的`React`组件类，它从存储状态构建`props`，并将它们传递给包装好的组件。高阶组件是一个函数，它接受一个组件参数并返回一个新组件。
> A higher-order React component class that builds props from the store state and passes them to the wrapped component. A higher-order component is a function which accepts a component argument and returns a new component.

### Static Properties

- `WrappedComponent` (Component):传递给`connectAdvanced(...)(Component)`的原始组件类。
> - WrappedComponent (Component): The original component class passed to connectAdvanced(...)(Component).

### Static Methods

组件的所有原始静态方法都被提升。
> All the original static methods of the component are hoisted.

## Remarks

- 由于`connectAdvanced`返回高阶组件，因此需要调用两次。第一次使用上面描述的参数，第二次使用组件`connectAdvanced(selectorFactory)(MyComponent)`。
- `connectAdvanced`不修改传递的`React`组件。它返回一个新的、连接的组件，你应该使用它。
> - Since connectAdvanced returns a higher-order component, it needs to be invoked two times. The first time with its arguments as described above, and a second time, with the component: connectAdvanced(selectorFactory)(MyComponent).
> - connectAdvanced does not modify the passed React component. It returns a new, connected component, that you should use instead.

### 案例

**根据道具注入特定用户的待办事项，并注入道具。`userId`转换到操作中**
> Inject todos of a specific user depending on props, and inject props.userId into the action

```js
import * as actionCreators from './actionCreators'
import { bindActionCreators } from 'redux'

function selectorFactory(dispatch) {
  let ownProps = {}
  let result = {}

  const actions = bindActionCreators(actionCreators, dispatch)
  const addTodo = (text) => actions.addTodo(ownProps.userId, text)

  return (nextState, nextOwnProps) => {
    const todos = nextState.todos[nextOwnProps.userId]
    const nextResult = { ...nextOwnProps, todos, addTodo }
    ownProps = nextOwnProps
    if (!shallowEqual(result, nextResult)) result = nextResult
    return result
  }
}
export default connectAdvanced(selectorFactory)(TodoApp)
```
