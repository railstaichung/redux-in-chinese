# `bindActionCreators(actionCreators, dispatch)`

把 [action creators](../Glossary.md#action-creator) 轉成擁有同名 keys 的物件，但使用 [`dispatch`](Store.md#dispatch) 把每個 action creator 包圍起來，這樣可以直接呼叫它們。

一般情況下你可以直接在 [`Store`](Store.md) 例項上呼叫 [`dispatch`](Store.md#dispatch)。如果你在 React 中使用 Redux，[react-redux](https://github.com/gaearon/react-redux) 會提供 [`dispatch`](Store.md#dispatch) 。

惟一使用 `bindActionCreators` 的場景是當你需要把 action creator 往下傳到一個元件上，卻不想讓這個元件覺察到 Redux 的存在，而且不希望把 Redux store 或 [`dispatch`](Store.md#dispatch) 傳給它。

為方便起見，你可以傳入一個函數作為第一個參數，它會返回一個函數。

#### 參數

1. `actionCreators` (*Function* or *Object*): 一個 [action creator](../Glossary.md#action-creator)，或者鍵值是 action creators 的物件。

2. `dispatch` (*Function*): 一個 [`dispatch`](Store.md#dispatch) 函數，由 [`Store`](Store.md) 例項提供。

#### 返回值

(*Function* or *Object*): 一個與原物件類似的物件，只不過這個物件中的的每個函數值都可以直接 dispatch action。如果傳入的是一個函數，返回的也是一個函數。

#### 示例

#### `TodoActionCreators.js`

```js
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  };
}

export function removeTodo(id) {
  return {
    type: 'REMOVE_TODO',
    id
  };
}
```

#### `SomeComponent.js`

```js
import { Component } from 'react';
import { bindActionCreators } from 'redux';
import { connect } from 'react-redux';

import * as TodoActionCreators from './TodoActionCreators';
console.log(TodoActionCreators);
// {
//   addTodo: Function,
//   removeTodo: Function
// }

class TodoListContainer extends Component {
  componentDidMount() {
    // 由 react-redux 注入：
    let { dispatch } = this.props;

    // 注意：這樣做行不通：
    // TodoActionCreators.addTodo('Use Redux');

    // 你只是呼叫了建立 action 的方法。
    // 你必須要 dispatch action 而已。

    // 這樣做行得通：
    let action = TodoActionCreators.addTodo('Use Redux');
    dispatch(action);
  }

  render() {
    // 由 react-redux 注入：
    let { todos, dispatch } = this.props;

    // 這是應用 bindActionCreators 比較好的場景：
    // 在子元件裡，可以完全不知道 Redux 的存在。

    let boundActionCreators = bindActionCreators(TodoActionCreators, dispatch);
    console.log(boundActionCreators);
    // {
    //   addTodo: Function,
    //   removeTodo: Function
    // }

    return (
      <TodoList todos={todos}
                {...boundActionCreators} />
    );

    // 一種可以替換 bindActionCreators 的做法是直接把 dispatch 函數
    // 和 action creators 當作 props
    // 傳遞給子元件
    // return <TodoList todos={todos} dispatch={dispatch} />;
  }
}

export default connect(
  state => ({ todos: state.todos })
)(TodoListContainer)
```

#### 小貼士

* 你或許要問：為什麼不直接把 action creators 繫結到 store 例項上，就像傳統 Flux 那樣？問題是這樣做的話如果開發同構應用，在服務端渲染時就不行了。多數情況下，你 每個請求都需要一個獨立的 store 例項，這樣你可以為它們提供不同的資料，但是在定義的時候繫結 action creators，你就可以使用一個唯一的 store 例項來對應所有請求了。

* 如果你使用 ES5，不能使用 `import * as` 語法，你可以把 `require('./TodoActionCreators')` 作為第一個參數傳給 `bindActionCreators`。惟一要考慮的是 `actionCreators` 的參數全是函數。模組載入系統並不重要。
