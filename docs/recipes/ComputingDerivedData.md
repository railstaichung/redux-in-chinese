# 計算衍生資料

[Reselect](https://github.com/faassen/reselect.git) 庫可以建立可記憶的(Memoized)、可組合的 **selector** 函數。Reselect selectors 可以用來高效地計算 Redux store 裡的衍生資料。

### 可記憶的 Selectors 初衷

首先訪問 [Todos 列表示例](../basics/UsageWithReact.md):

#### `containers/App.js`

```js
import React, { Component, PropTypes } from 'react';
import { connect } from 'react-redux';
import { addTodo, completeTodo, setVisibilityFilter, VisibilityFilters } from '../actions';
import AddTodo from '../components/AddTodo';
import TodoList from '../components/TodoList';
import Footer from '../components/Footer';

class App extends Component {
  render() {
    // 通過 connect() 注入：
    const { dispatch, visibleTodos, visibilityFilter } = this.props;
    return (
      <div>
        <AddTodo
          onAddClick={text =>
            dispatch(addTodo(text))
          } />
        <TodoList
          todos={this.props.visibleTodos}
          onTodoClick={index =>
            dispatch(completeTodo(index))
          } />
        <Footer
          filter={visibilityFilter}
          onFilterChange={nextFilter =>
            dispatch(setVisibilityFilter(nextFilter))
          } />
      </div>
    );
  }
}

App.propTypes = {
  visibleTodos: PropTypes.arrayOf(PropTypes.shape({
    text: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired
  })),
  visibilityFilter: PropTypes.oneOf([
    'SHOW_ALL',
    'SHOW_COMPLETED',
    'SHOW_ACTIVE'
  ]).isRequired
};

function selectTodos(todos, filter) {
  switch (filter) {
  case VisibilityFilters.SHOW_ALL:
    return todos;
  case VisibilityFilters.SHOW_COMPLETED:
    return todos.filter(todo => todo.completed);
  case VisibilityFilters.SHOW_ACTIVE:
    return todos.filter(todo => !todo.completed);
  }
}

function select(state) {
  return {
    visibleTodos: selectTodos(state.todos, state.visibilityFilter),
    visibilityFilter: state.visibilityFilter
  };
}

// 打包元件，注入 dispatch 和 state
export default connect(select)(App);
```

上面的示例中，`select` 呼叫了 `selectTodos` 來計算 `visibleTodos`。執行沒問題，但有一個缺點：每當元件更新時都會重新計算 `visibleTodos`。如果 state tree 非常大，或者計算量非常大，每次更新都重新計算可能會帶來效能問題。Reselect 能幫你省去這些沒必要的重新計算。

### 建立可記憶的 Selector

我們需要一個可記憶的 selector 來替代這個 `select`，只在 `state.todos` or `state.visibilityFilter` 變化時重新計算 `visibleTodos`，而在其它部分（非相關）變化時不做計算。

Reselect 提供 `createSelector` 函數來建立可記憶的 selector。`createSelector` 接收一個 input-selectors 陣列和一個轉換函數作為參數。如果 state tree 的改變會引起 input-selector 值變化，那麼 selector 會呼叫轉換函數，傳入 input-selectors 作為參數，並返回結果。如果 input-selectors 的值和前一次的一樣，它將會直接返回前一次計算的資料，而不會再呼叫一次轉換函數。

定義一個可記憶的 selector `visibleTodosSelector` 來替代 `select`：

#### `selectors/TodoSelectors.js`

```js
import { createSelector } from 'reselect';
import { VisibilityFilters } from './actions';

function selectTodos(todos, filter) {
  switch (filter) {
  case VisibilityFilters.SHOW_ALL:
    return todos;
  case VisibilityFilters.SHOW_COMPLETED:
    return todos.filter(todo => todo.completed);
  case VisibilityFilters.SHOW_ACTIVE:
    return todos.filter(todo => !todo.completed);
  }
}

const visibilityFilterSelector = (state) => state.visibilityFilter;
const todosSelector = (state) => state.todos;

export const visibleTodosSelector = createSelector(
  [visibilityFilterSelector, todosSelector],
  (visibilityFilter, todos) => {
    return {
      visibleTodos: selectTodos(todos, visibilityFilter),
      visibilityFilter
    };
  }
);
```

在上例中，`visibilityFilterSelector` 和 `todosSelector` 是 input-selector。因為他們並不轉換資料，所以被建立成普通的非記憶的 selector 函數。但是，`visibleTodosSelector` 是一個可記憶的 selector。他接收 `visibilityFilterSelector` 和 `todosSelector` 為 input-selector，還有一個轉換函數來計算過濾的 todos 列表。

### 組合 Selector

可記憶的 selector 自身可以作為其它可記憶的 selector 的 input-selector。下面的 `visibleTodosSelector` 被當作另一個 selector 的 input-selector，來進一步通過關鍵字（keyword）過濾 todos。

```js
const keywordSelector = (state) => state.keyword

const keywordFilterSelector = createSelector(
  [ visibleTodosSelector, keywordSelector ],
  (visibleTodos, keyword) => visibleTodos.filter(
    todo => todo.indexOf(keyword) > -1
  )
)
```

### 連線 Selector 和 Redux Store

如果你在使用 react-redux，你可以使用 connect 來連線可記憶的 selector 和 Redux store。

#### `containers/App.js`

```js
import React, { Component, PropTypes } from 'react'
import { connect } from 'react-redux'
import { addTodo, completeTodo, setVisibilityFilter } from '../actions'
import AddTodo from '../components/AddTodo'
import TodoList from '../components/TodoList'
import Footer from '../components/Footer'
import { visibleTodosSelector } from '../selectors/todoSelectors'

class App extends Component {
  render() {
    // Injected by connect() call:
    const { dispatch, visibleTodos, visibilityFilter } = this.props
    return (
      <div>
        <AddTodo
          onAddClick={text =>
            dispatch(addTodo(text))
          } />
        <TodoList
          todos={this.props.visibleTodos}
          onTodoClick={index =>
            dispatch(completeTodo(index))
          } />
        <Footer
          filter={visibilityFilter}
          onFilterChange={nextFilter =>
            dispatch(setVisibilityFilter(nextFilter))
          } />
      </div>
    )
  }
}

App.propTypes = {
  visibleTodos: PropTypes.arrayOf(PropTypes.shape({
    text: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired
  })),
  visibilityFilter: PropTypes.oneOf([
    'SHOW_ALL',
    'SHOW_COMPLETED',
    'SHOW_ACTIVE'
  ]).isRequired
}

// 把 selector 傳遞給連線的元件
export default connect(visibleTodosSelector)(App)
```
