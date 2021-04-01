---
title: React Redux 7.x 中文文档 - 基础教程
date: 2021-03-03 16:39:05
categories: JavaScript
tags: [React, Redux]
---

# 基础教程 Todo List

为了了解如何在实践中使用`React Redux`，我们将通过创建一个待办事项列表应用程序来展示一个逐步的示例。
> To see how to use React Redux in practice, we’ll show a step-by-step example by creating a todo list app.

### The React UI Components

我们已经实现了`React UI`组件，如下所示：
> We have implemented our React UI components as follows:

- `TodoApp`是我们应用程序的入口组件。它呈现`header`，`AddTodo`，`TodoList`，和`VisibilityFilters`组件。
- `AddTodo`是一个组件，允许用户输入一个`todo`项目，并在点击“add todo”按钮后添加到列表中:
  - 它使用一个受控输入，在`onChange`上设置状态。
  - 当用户单击“Add Todo”按钮时，它会分派动作(我们将使用`React Redux`提供)来将`Todo`添加到`store`。
- `TodoList`是渲染待办事项列表的组件:
  - 当其中一个可见性过滤器被选中时，它呈现过滤的待办事项列表。
- `Todo`是一个呈现单个`todo item`的组件:
  - 它呈现`todo`内容，并通过划掉它来显示一个`todo`已完成。
  - 它分派动作来在`onClick`时切换`todo`的完整状态。
- `VisibilityFilters`呈现了一组简单的过滤器:*`all`*、 *`completed`*、和 *`incomplete`*。点击它们中的每一个，就会过滤出待办事项:
  - 它接受来自父类的`activeFilter`道具，用来指示用户当前选择的是哪个过滤器。活动过滤器用下划线显示。
  - 它分派setFilter操作来更新所选的过滤器。
- `constants`保存应用程序的常量数据。
- 最后索引将我们的应用程序渲染到`DOM`。

> - TodoApp is the entry component for our app. It renders the header, the AddTodo, TodoList, and VisibilityFilters components.
> - AddTodo is the component that allows a user to input a todo item and add to the list upon clicking its “Add Todo” button:
>   - It uses a controlled input that sets state upon onChange.
>   - When the user clicks on the “Add Todo” button, it dispatches the action (that we will provide using React Redux) to add the todo to the store.
> - TodoList is the component that renders the list of todos:
>   - It renders the filtered list of todos when one of the VisibilityFilters is selected.
> - Todo is the component that renders a single todo item:
>   - It renders the todo content, and shows that a todo is completed by crossing it out.
>   - It dispatches the action to toggle the todo's complete status upon onClick.
> - VisibilityFilters renders a simple set of filters: all, completed, and incomplete. Clicking on each one of them filters the todos:
>   - It accepts an activeFilter prop from the parent that indicates which filter is currently selected by the user. An active filter is rendered with an underscore.
>   - It dispatches the setFilter action to update the selected filter.
> - constants holds the constants data for our app.
> - And finally index renders our app to the DOM.

### The Redux Store

应用程序的`Redux`部分已经使用`Redux`文档中推荐的模式设置:
> The Redux portion of the application has been set up using the patterns recommended in the Redux docs:

- `Store`
  - `todos`: `todos`的规范化简化形式。它包含所有`todos`的`byIds`映射，以及包含所有`id`列表的`allIds`。
  - `visible filters`:一个简单的字符串`all`、`completed`或`incomplete`。
- `Action Creators`
  - `addTodo`创建添加待办事项的操作。它接受单个字符串变量`content`，并返回一个`ADD_TODO`操作，其有效负载包含自增加的`id`和内容
  - `toggleTodo`创建用于切换`todos`的动作。它接受单个数字变量`id`，并返回一个`TOGGLE_TODO`动作，其负载只包含`id`
  - `setFilter`创建一个动作来设置应用程序的活动过滤器。它接受单个字符串变量`filter`，并返回包含过滤器本身的有效负载的`SET_FILTER`操作
- `Reducers`
  - `The todos reducer`
    - 将`id`附加到其`allIds`字段，并在接收到`ADD_TODO`操作时在`byIds`字段中设置`todo`
    - 在接收`TOGGLE_TODO`动作时切换`todo`的完成字段
  - `visiliityfilters`减速机将其存储片设置为它从`SET_FILTER`操作有效负载接收到的新过滤器
- `Action Types`
  - 我们使用一个`actionTypes.js`文件来保存要重用的动作类型的常量
- `Selectors`
  - `getTodoList`返回来自`todos`存储区的`allIds`列表
  - `getTodoById`在`id`给定的`store`中查找`todo`
  - `getTodos`稍微复杂一些。它从`allIds`中获取所有`id`，在`byIds`中查找每个`todo`，并返回`todo`的最终数组
  - `getTodosByVisibilityFilter`根据可见性过滤器对待办事项进行过滤

> - Store
>   - todos: A normalized reducer of todos. It contains a byIds map of all todos and a allIds that contains the list of all ids.
>   - visibilityFilters: A simple string all, completed, or incomplete.
> - Action Creators
>   - addTodo creates the action to add todos. It takes a single string variable content and returns an ADD_TODO action with payload containing a self-incremented id and content
>   - toggleTodo creates the action to toggle todos. It takes a single number variable id and returns a TOGGLE_TODO action with payload containing id only
>   - setFilter creates the action to set the app’s active filter. It takes a single string variable filter and returns a SET_FILTER action with payload containing the filter itself
> - Reducers
>   - The todos reducer
>     - Appends the id to its allIds field and sets the todo within its byIds field upon receiving the ADD_TODO action
>     - Toggles the completed field for the todo upon receiving the TOGGLE_TODO action
>   - The visibilityFilters reducer sets its slice of store to the new filter it receives from the SET_FILTER action payload
> - Action Types
>   - We use a file actionTypes.js to hold the constants of action types to be reused
> - Selectors
>   - getTodoList returns the allIds list from the todos store
>   - getTodoById finds the todo in the store given by id
>   - getTodos is slightly more complex. It takes all the ids from allIds, finds each todo in byIds, and returns the final array of todos
>   - getTodosByVisibilityFilter filters the todos according to the visibility filter

现在，我们将展示如何使用`React Redux`将这个商店连接到我们的应用程序。
> We will now show how to connect this store to our app using React Redux.

### Providing the Store

首先，我们需要让`store`可用。为了做到这一点，我们用`React Redux`提供的`<Provider />` API包装我们的应用。
> First we need to make the store available to our app. To do this, we wrap our app with the `<Provider />` API provided by React Redux.

```js
// index.js
import React from 'react'
import ReactDOM from 'react-dom'
import TodoApp from './TodoApp'

import { Provider } from 'react-redux'
import store from './redux/store'

const rootElement = document.getElementById('root')
ReactDOM.render(
  <Provider store={store}>
    <TodoApp />
  </Provider>,
  rootElement
)
```

注意我们的`<TodoApp />`现在是如何被`<Provider />`包装起来的，并将`store`作为一个道具传入。
> Notice how our `<TodoApp />` is now wrapped with the `<Provider />` with store passed in as a prop.

![Todo List](https://i.imgur.com/LV0XvwA.png)

### Connecting the Components

`React Redux`提供了一个连接函数，用于从`Redux`存储读取值(并在存储更新时重新读取值)。
> React Redux provides a connect function for you to read values from the Redux store (and re-read the values when the store updates).

`connect`函数有两个参数，都是可选的:
> The connect function takes two arguments, both optional:

- `mapStateToProps`:每次存储状态改变时调用。它接收整个存储状态，并且应该返回该组件需要的数据对象。
- `mapDispatchToProps`:这个参数可以是函数，也可以是对象。
  - 如果它是一个函数，它将在组件创建时被调用一次。它将接收`dispatch`作为参数，并且应该返回一个对象，其中包含使用`dispatch`来分派操作的函数。
  - 如果它是一个充满动作创建者的对象，那么每个动作创建者都将被转换成一个道具函数，在调用时自动分派它的动作。注意:我们建议使用这种“对象简写”形式。

> - mapStateToProps: called every time the store state changes. It receives the entire store state, and should return an object of data this component needs.
> - mapDispatchToProps: this parameter can either be a function, or an object.
>   - If it’s a function, it will be called once on component creation. It will receive dispatch as an argument, and should return an object full of functions that use dispatch to dispatch actions.
>   - If it’s an object full of action creators, each action creator will be turned into a prop function that automatically dispatches its action when called. Note: We recommend using this “object shorthand” form.

通常，你会这样调用connect:
> Normally, you’ll call connect in this way:

```js
const mapStateToProps = (state, ownProps) => ({
  // ... computed data from state and optionally ownProps
})

const mapDispatchToProps = {
  // ... normally is an object full of action creators
}

// `connect` returns a new function that accepts the component to wrap:
const connectToStore = connect(mapStateToProps, mapDispatchToProps)
// and that function returns the connected, wrapper component:
const ConnectedComponent = connectToStore(Component)

// We normally do both in one step, like this:
connect(mapStateToProps, mapDispatchToProps)(Component)
```

让我们先处理`<AddTodo />`。它需要触发对商店的更改，以添加新的待办事项。因此，它需要能够将操作分派到商店。我们是这样做的。
> Let’s work on <AddTodo /> first. It needs to trigger changes to the store to add new todos. Therefore, it needs to be able to dispatch actions to the store. Here’s how we do it.

我们的`addTodo`动作创建者看起来像这样:
> Our addTodo action creator looks like this:

```js
// redux/actions.js
import { ADD_TODO } from './actionTypes'

let nextTodoId = 0
export const addTodo = (content) => ({
  type: ADD_TODO,
  payload: {
    id: ++nextTodoId,
    content,
  },
})

// ... other actions
```

通过将它传递给`connect`，我们的组件将它作为一个道具接收，当它被调用时，它将自动分派动作。
> By passing it to connect, our component receives it as a prop, and it will automatically dispatch the action when it’s called.

```js
// components/AddTodo.js

// ... other imports
import { connect } from 'react-redux'
import { addTodo } from '../redux/actions'

class AddTodo extends React.Component {
  // ... component implementation
}

export default connect(null, { addTodo })(AddTodo)
```

现在请注意`<AddTodo />`被一个名为`<Connect(AddTodo) />`的父组件包装了。同时，`<AddTodo />`现在获得一个道具:`AddTodo`动作。
> Notice now that <AddTodo /> is wrapped with a parent component called <Connect(AddTodo) />. Meanwhile, <AddTodo /> now gains one prop: the addTodo action.

![Todo List](https://i.imgur.com/u6aXbwl.png)

我们还需要实现`handleAddTodo`函数，让它分派`addTodo`动作并重置输入
> We also need to implement the handleAddTodo function to let it dispatch the addTodo action and reset the input

```js
// components/AddTodo.js

import React from 'react'
import { connect } from 'react-redux'
import { addTodo } from '../redux/actions'

class AddTodo extends React.Component {
  // ...

  handleAddTodo = () => {
    // dispatches actions to add todo
    this.props.addTodo(this.state.input)

    // sets state back to empty string
    this.setState({ input: '' })
  }

  render() {
    return (
      <div>
        <input
          onChange={(e) => this.updateInput(e.target.value)}
          value={this.state.input}
        />
        <button className="add-todo" onClick={this.handleAddTodo}>
          Add Todo
        </button>
      </div>
    )
  }
}

export default connect(null, { addTodo })(AddTodo)
```

现在我们的`<AddTodo />`连接到存储。当我们添加`todo`时，它会分派一个动作来更改存储。我们在应用中没有看到它，因为其他组件还没有连接。如果你已经连接了`Redux DevTools`扩展，你应该会看到正在调度的动作:
> Now our <AddTodo /> is connected to the store. When we add a todo it would dispatch an action to change the store. We are not seeing it in the app because the other components are not connected yet. If you have the Redux DevTools Extension hooked up, you should see the action being dispatched:

![Todo List](https://i.imgur.com/kHvkqhI.png)

你还应该看到商店也发生了相应的变化:
> You should also see that the store has changed accordingly:

![Todo List](https://i.imgur.com/yx27RVC.png)

`<TodoList />`组件负责呈现待办事项列表。因此，它需要从存储中读取数据。我们通过调用带有`mapStateToProps`参数的`connect`来启用它，该参数是一个描述我们需要从存储中获取哪一部分数据的函数。
> The <TodoList /> component is responsible for rendering the list of todos. Therefore, it needs to read data from the store. We enable it by calling connect with the mapStateToProps parameter, a function describing which part of the data we need from the store.

我们的`<Todo />`组件将`Todo`条目作为道具。我们从`todos`的`byIds`字段中获得了这些信息。然而，我们还需要来自存储的`allIds`字段的信息，指示应该呈现哪些`todo`以及它们的顺序。我们的`mapStateToProps`函数可能是这样的:
> Our <Todo /> component takes the todo item as props. We have this information from the byIds field of the todos. However, we also need the information from the allIds field of the store indicating which todos and in what order they should be rendered. Our mapStateToProps function may look like this:

```js
// components/TodoList.js

// ...other imports
import { connect } from "react-redux";

const TodoList = // ... UI component implementation

const mapStateToProps = state => {
  const { byIds, allIds } = state.todos || {};
  const todos =
    allIds && allIds.length
      ? allIds.map(id => (byIds ? { ...byIds[id], id } : null))
      : null;
  return { todos };
};

export default connect(mapStateToProps)(TodoList);
```

幸运的是，我们有一个选择器可以做到这一点。我们可以简单地导入选择器并在这里使用它。
> Luckily we have a selector that does exactly this. We may simply import the selector and use it here.

```js
// redux/selectors.js

export const getTodosState = (store) => store.todos

export const getTodoList = (store) =>
  getTodosState(store) ? getTodosState(store).allIds : []

export const getTodoById = (store, id) =>
  getTodosState(store) ? { ...getTodosState(store).byIds[id], id } : {}

export const getTodos = (store) =>
  getTodoList(store).map((id) => getTodoById(store, id))
```

```js
// components/TodoList.js

// ...other imports
import { connect } from "react-redux";
import { getTodos } from "../redux/selectors";

const TodoList = // ... UI component implementation

export default connect(state => ({ todos: getTodos(state) }))(TodoList);
```

我们建议将任何复杂的数据查找或计算封装在选择器函数中。此外，您还可以通过使用`reselect`来写入可以跳过不必要工作的“`memoized`”选择器，从而进一步优化性能。(请参阅关于计算派生数据的Redux docs页面和博客文章惯用的Redux:使用重新选择选择器进行封装和性能，以获得关于为什么和如何使用选择器函数的更多信息。)
> We recommend encapsulating any complex lookups or computations of data in selector functions. In addition, you can further optimize the performance by using Reselect to write “memoized” selectors that can skip unnecessary work. (See the Redux docs page on Computing Derived Data and the blog post Idiomatic Redux: Using Reselect Selectors for Encapsulation and Performance for more information on why and how to use selector functions.)

现在，我们的`<TodoList />`已经连接到商店。它应该接收todo列表，映射它们，并将每个todo传递给`< todo />`组件。`<Todo />`将依次把它们渲染到屏幕上。现在尝试添加一个待办事项。它应该出现在我们的待办事项清单上!
> Now that our <TodoList /> is connected to the store. It should receive the list of todos, map over them, and pass each todo to the <Todo /> component. <Todo /> will in turn render them to the screen. Now try adding a todo. It should come up on our todo list!

![Todo List](https://i.imgur.com/N68xvrG.png)

我们将连接更多的组件。在我们这样做之前，让我们先暂停一下，了解更多关于`connect`的知识。
> We will connect more components. Before we do this, let’s pause and learn a bit more about connect first.

### Common ways of calling connect

根据您正在使用的组件类型，有不同的调用`connect`的方法，其中最常见的总结如下:
> Depending on what kind of components you are working with, there are different ways of calling connect , with the most common ones summarized as below:

| - | Do Not Subscribe to the Store | Subscribe to the Store |
| :---- | :---- | :---- |
| Do Not Inject Action Creators | connect()(Component) | 	connect(mapStateToProps)(Component) |
| Inject Action Creators | connect(null, mapDispatchToProps)(Component) | connect(mapStateToProps, mapDispatchToProps)(Component) |

不要订阅`store`，也不要注入`action creators`
> Do not subscribe to the store and do not inject action creators

如果你在不提供任何参数的情况下调用`connect`，你的组件将:
> If you call connect without providing any arguments, your component will:


- 当`store`改变时不重新呈现
- 接收`props`。可以用于手动`dispatch action`
> - not re-render when the store changes
> - receive props.dispatch that you may use to manually dispatch action

```js
// ... Component
export default connect()(Component) // Component will receive `dispatch` (just like our <TodoList />!)
```

订阅`store`，不要注入`action creators`
> Subscribe to the store and do not inject action creators

如果你只使用`mapStateToProps`来调用`connect`，你的组件就会
> If you call connect with only mapStateToProps, your component will:

- 订阅`mapStateToProps`从存储中提取的值，并只在这些值更改时重新呈现
- 接收`props`。可以用于手动`dispatch action`
> - subscribe to the values that mapStateToProps extracts from the store, and re-render only when those values have changed
> - receive props.dispatch that you may use to manually dispatch action

```js
// ... Component
const mapStateToProps = (state) => state.partOfState
export default connect(mapStateToProps)(Component)
```

不订阅`store`并注入`action creators`
> Do not subscribe to the store and inject action creators

如果你只使用`mapDispatchToProps`来调用`connect`，你的组件会:
> If you call connect with only mapDispatchToProps, your component will:

- 当`store`改变时不重新呈现
- 接收每个用`mapDispatchToProps`注入的动作创建者，并在被调用时自动分派动作
> - not re-render when the store changes
> - receive each of the action creators you inject with mapDispatchToProps as props and automatically dispatch the actions upon being called

```js
import { addTodo } from './actionCreators'
// ... Component
export default connect(null, { addTodo })(Component)
```

订阅`store`并注入`action creators`
> Subscribe to the store and inject action creators

如果你同时使用`mapStateToProps`和`mapDispatchToProps`调用`connect`，你的组件将:
> If you call connect with both mapStateToProps and mapDispatchToProps, your component will:

- 订阅`mapStateToProps`从存储中提取的值，并只在这些值更改时重新呈现
- 接收所有用`mapDispatchToProps`注入的动作创建者，并在被调用时自动分派动作。
> - subscribe to the values that mapStateToProps extracts from the store, and re-render only when those values have changed
> - receive all of the action creators you inject with mapDispatchToProps as props and automatically dispatch the actions upon being called.

```js
import * as actionCreators from './actionCreators'
// ... Component
const mapStateToProps = (state) => state.partOfState
export default connect(mapStateToProps, actionCreators)(Component)
```

这四种情况涵盖了`connect`最基本的用法。要阅读更多关于`connect`的内容，请继续阅读我们的`API`部分，该部分将更详细地解释它。
> These four cases cover the most basic usages of connect. To read more about connect, continue reading our API section that explains it in more detail.

---

现在让我们连接`<TodoApp />`的其余部分。
> Now let’s connect the rest of our <TodoApp />.

我们应该如何实现`todos`切换的交互?一个热心的读者可能已经有了答案。如果您已经设置好了环境，并一直执行到现在，那么现在是时候把它放在一边，自己实现这个特性了。用类似的方式连接`<Todo />`来分派`toggleTodo`也就不足为奇了:
> How should we implement the interaction of toggling todos? A keen reader might already have an answer. If you have your environment set up and have followed through up until this point, now is a good time to leave it aside and implement the feature by yourself. There would be no surprise that we connect our <Todo /> to dispatch toggleTodo in a similar way:

```js
// components/Todo.js

// ... other imports
import { connect } from "react-redux";
import { toggleTodo } from "../redux/actions";

const Todo = // ... component implementation

export default connect(
  null,
  { toggleTodo }
)(Todo);
```

现在我们的待办事项可以切换完成。我们快到了!
> Now our todo’s can be toggled complete. We’re almost there!

![Todo List](https://i.imgur.com/4UBXYtj.png)

最后，让我们来实现我们的`VisibilityFilters`特性。
> Finally, let’s implement our VisibilityFilters feature.

`<VisibilityFilters />`组件需要能够从过滤器当前处于活动状态的存储中读取，并将操作分派给存储。因此，我们需要传递一个`mapStateToProps`和一个`mapDispatchToProps`。这里的`mapStateToProps`可以是`visabilityfilter`状态的一个简单访问器。`mapDispatchToProps`将包含`setFilter`动作创建器。
> The <VisibilityFilters /> component needs to be able to read from the store which filter is currently active, and dispatch actions to the store. Therefore, we need to pass both a mapStateToProps and mapDispatchToProps. The mapStateToProps here can be a simple accessor of the visibilityFilter state. And the mapDispatchToProps will contain the setFilter action creator.

```js
// components/VisibilityFilters.js

// ... other imports
import { connect } from "react-redux";
import { setFilter } from "../redux/actions";

const VisibilityFilters = // ... component implementation

const mapStateToProps = state => {
  return { activeFilter: state.visibilityFilter };
};
export default connect(
  mapStateToProps,
  { setFilter }
)(VisibilityFilters);
```

同时，我们还需要更新我们的`<TodoList />`组件，根据`active filter`过滤`todos`。以前我们传递给`<TodoList />` `connect`函数调用的`mapStateToProps`只是选择器，它选择了整个`todos`列表。让我们编写另一个选择器来帮助根据状态筛选待办事项。
> Meanwhile, we also need to update our <TodoList /> component to filter todos according to the active filter. Previously the mapStateToProps we passed to the <TodoList /> connect function call was simply the selector that selects the whole list of todos. Let’s write another selector to help filtering todos by their status.

```js
// redux/selectors.js

// ... other selectors
export const getTodosByVisibilityFilter = (store, visibilityFilter) => {
  const allTodos = getTodos(store)
  switch (visibilityFilter) {
    case VISIBILITY_FILTERS.COMPLETED:
      return allTodos.filter((todo) => todo.completed)
    case VISIBILITY_FILTERS.INCOMPLETE:
      return allTodos.filter((todo) => !todo.completed)
    case VISIBILITY_FILTERS.ALL:
    default:
      return allTodos
  }
}
```

在选择器的帮助下连接到存储:
> And connecting to the store with the help of the selector:

```js
// components/TodoList.js

// ...

const mapStateToProps = (state) => {
  const { visibilityFilter } = state
  const todos = getTodosByVisibilityFilter(state, visibilityFilter)
  return { todos }
}

export default connect(mapStateToProps)(TodoList)
```

现在，我们已经用`React Redux`完成了一个非常简单的`todo`应用示例。我们所有的组件都是连接在一起的!那不是很好吗?🎉🎊
> Now we've finished a very simple example of a todo app with React Redux. All our components are connected! Isn't that nice? 🎉🎊

![Todo List](https://i.imgur.com/ONqer2R.png)
