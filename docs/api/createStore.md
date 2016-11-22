# `createStore(reducer, [initialState], enhancer)`

建立一個 Redux [store](Store.md) 來以存放應用中所有的 state。
應用中應有且僅有一個 store。

#### 參數

1. `reducer` *(Function)*: 接收兩個參數，分別是當前的 state 樹和要處理的 [action](../Glossary.md#action)，返回新的 [state 樹](../Glossary.md#state)。

2. [`initialState`] *(any)*: 初始時的 state。
在同構應用中，你可以決定是否把服務端傳來的 state 水合（hydrate）後傳給它，或者從之前儲存的使用者會話中恢復一個傳給它。如果你使用 [`combineReducers`](combineReducers.md) 建立 `reducer`，它必須是一個普通物件，與傳入的 keys 保持同樣的結構。否則，你可以自由傳入任何 `reducer` 可理解的內容。

3. `enhancer` *(Function)*: Store enhancer 是一個組合 store creator 的高階函數，返回一個新的強化過的 store creator。這與 middleware 相似，它也允許你通過複合函數改變 store 介面。

#### 返回值

([*`Store`*](Store.md)): 儲存了應用所有 state 的物件。改變 state 的惟一方法是 [dispatch](Store.md#dispatch) action。你也可以 [subscribe 監聽](Store.md#subscribe) state 的變化，然後更新 UI。

#### 示例

```js
import { createStore } from 'redux'

function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return state.concat([ action.text ])
    default:
      return state
  }
}

let store = createStore(todos, [ 'Use Redux' ])

store.dispatch({
  type: 'ADD_TODO',
  text: 'Read the docs'
})

console.log(store.getState())
// [ 'Use Redux', 'Read the docs' ]
```

#### 小貼士

* 應用中不要建立多個 store！相反，使用 [`combineReducers`](combineReducers.md) 來把多個 reducer 建立成一個根 reducer。

* 你可以決定 state 的格式。你可以使用普通物件或者 [Immutable](http://facebook.github.io/immutable-js/) 這類的實現。如果你不知道如何做，剛開始可以使用普通物件。

* 如果 state 是普通物件，永遠不要修改它！比如，reducer 裡不要使用 `Object.assign(state, newData)`，應該使用 `Object.assign({}, state, newData)`。這樣才不會覆蓋舊的 `state`。也可以使用 [Babel 階段 1](http://babeljs.io/docs/usage/experimental/) 中的 [ES7 物件的 spread 操作](https://github.com/sebmarkbage/ecmascript-rest-spread) 特性中的 `return { ...state, ...newData }`。

* 對於服務端執行的同構應用，為每一個請求建立一個 store 例項，以此讓 store 相隔離。dispatch 一系列請求資料的 action 到 store 例項上，等待請求完成後再在服務端渲染應用。

* 當 store 建立後，Redux 會 dispatch 一個 action 到 reducer 上，來用初始的 state 來填充 store。你不需要處理這個 action。但要記住，如果第一個參數也就是傳入的 state 如果是 `undefined` 的話，reducer 應該返回初始的 state 值。
