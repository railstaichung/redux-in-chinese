# 排錯

這裡會列出常見的問題和對應的解決方案。
雖然使用 React 做示例，但是即使你使用了其它庫，這些問題和解決方案仍然對你有所幫助。

### dispatch action 後什麼也沒有發生

有時，你 dispatch action 後，view 卻沒有更新。這是為什麼呢？可能有下面幾種原因。

#### 永遠不要直接修改 reducer 的參數

如果你想修改 Redux 給你傳入的 `state` 或 `action`，請住手！

Redux 假定你永遠不會修改 reducer 裡傳入的物件。**任何時候，你都應該返回一個新的 state 物件。**即使你沒有使用 [Immutable](https://facebook.github.io/immutable-js/) 這樣的庫，也要保證做到不修改物件。

不變性（Immutability）可以讓 [react-redux](https://github.com/gaearon/react-redux) 高效的監聽 state 的細粗度更新。它也讓 [redux-devtools](http://github.com/gaearon/redux-devtools) 能提供“時間旅行”這類強大特性。

例如，下面的 reducer 就是錯誤的，因為它改變了 state：

```js
function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      // 錯誤！這會改變 state.actions。
      state.push({
        text: action.text,
        completed: false
      })
      return state
    case 'COMPLETE_TODO':
      // 錯誤！這會改變 state[action.index]。
      state[action.index].completed = true
      return state
    default:
      return state
  }
}
```

應該重寫成這樣：

```js
function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      // 返回新陣列
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case 'COMPLETE_TODO':
      // 返回新陣列
      return [
        ...state.slice(0, action.index),
        // 修改之前複製陣列
        Object.assign({}, state[action.index], {
          completed: true
        }),
        ...state.slice(action.index + 1)
      ]
    default:
      return state
  }
}
```

雖然需要寫更多程式碼，但是讓 Redux 變得可具有可預測性和高效。如果你想減少程式碼量，你可以用一些輔助方法類似
 [`React.addons.update`](https://facebook.github.io/react/docs/update.html) 來讓這樣的不可變轉換操作變得更簡單：

```js
// 修改前
return [
  ...state.slice(0, action.index),
  Object.assign({}, state[action.index], {
    completed: true
  }),
  ...state.slice(action.index + 1)
]

// 修改後
return update(state, {
  [action.index]: {
    completed: {
      $set: true
    }
  }
})
```

最後，如果需要更新 object，你需要使用 Underscore 提供的 `_.extend` 方法，或者更好的，使用 [`Object.assign`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 的 polyfill

要注意 `Object.assign` 的使用方法。例如，在 reducer 裡不要這樣使用 `Object.assign(state, newData)`，應該用 `Object.assign({}, state, newData)`。這樣它才不會覆蓋以前的 `state`。

你也可以通過使用 [Babel transform-object-rest-spread 外掛](http://babeljs.io/docs/plugins/transform-object-rest-spread/)來開啟 [ES7 物件的 spread 操作](https://github.com/sebmarkbage/ecmascript-rest-spread)：

```js
// 修改前：
return [
  ...state.slice(0, action.index),
  Object.assign({}, state[action.index], {
    completed: true
  }),
  ...state.slice(action.index + 1)
]

// 修改後：
return [
  ...state.slice(0, action.index),
  { ...state[action.index], completed: true },
  ...state.slice(action.index + 1)
]
```

注意還在實驗階段的特性註定經常改變，最好不要在大的項目裡過多依賴它們。

#### 不要忘記呼叫 [`dispatch(action)`](api/Store.md#dispatch)

如果你定義了一個 action 建立函數，呼叫它並**不**會自動 dispatch 這個 action。比如，下面的程式碼什麼也不會做：

#### `TodoActions.js`

```js
export function addTodo(text) {
  return { type: 'ADD_TODO', text }
}
```

#### `AddTodo.js`

```js
import { Component } from 'react'
import { addTodo } from './TodoActions'

class AddTodo extends Component {
  handleClick() {
    // 不起作用！
    addTodo('Fix the issue')
  }

  render() {
    return (
      <button onClick={() => this.handleClick()}>
        Add
      </button>
    )
  }
}
```

它不起作用是因為你的 action 建立函數只是一個**返回** action 的函數而已。你需要手動 dispatch 它。我們不能在定義時把 action 建立函數繫結到指定的 Store 上，因為應用在服務端渲染時需要為每個請求都對應一個獨立的 Redux store。

解法是呼叫 [store](api/Store.md) 例項上的 [`dispatch()`](api/Store.md#dispatch) 方法。

```js
handleClick() {
  // 生效！（但你需要先以某種方式拿到 store）
  store.dispatch(addTodo('Fix the issue'))
}
```

如果元件的層級非常深，把 store 一層層傳下去很麻煩。因此 [react-redux](https://github.com/gaearon/react-redux) 提供了 `connect` 這個 [高階元件](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750)，它除了可以幫你監聽 Redux store，還會把 `dispatch` 注入到元件的 props 中。

修復後的程式碼是這樣的：
#### `AddTodo.js`
```js
import React, { Component } from 'react'
import { connect } from 'react-redux'
import { addTodo } from './TodoActions'

class AddTodo extends Component {
  handleClick() {
    // 生效！
    this.props.dispatch(addTodo('Fix the issue'))
  }

  render() {
    return (
      <button onClick={() => this.handleClick()}>
        Add
      </button>
    )
  }
}

// 除了 state，`connect` 還把 `dispatch` 放到 props 裡。
export default connect()(AddTodo)
```

如果你想的話也可以把 `dispatch` 手動傳給其它元件。

## 其它問題

在 Discord [Reactiflux](http://reactiflux.com/) 裡的 **redux** 頻道里提問，或者[提交一個 issue](https://github.com/rackt/redux/issues)。
如果問題終於解決了，請把解法[寫到文件裡](https://github.com/rackt/redux/edit/master/docs/Troubleshooting.md)，以便別人遇到同樣問題時參考。
