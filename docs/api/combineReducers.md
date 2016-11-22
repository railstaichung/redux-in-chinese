# `combineReducers(reducers)`

隨著應用變得複雜，需要對 [reducer 函數](../Glossary.md#reducer) 進行拆分，拆分後的每一塊獨立負責管理 [state](../Glossary.md#state) 的一部分。

`combineReducers` 輔助函數的作用是，把一個由多個不同 reducer 函數作為 value 的 object，合併成一個最終的 reducer 函數，然後就可以對這個 reducer 呼叫 [`createStore`](createStore.md)。

合併後的 reducer 可以呼叫各個子 reducer，並把它們的結果合併成一個 state 物件。**state 物件的結構由傳入的多個 reducer 的 key 決定**。

最終，state 物件的結構會是這樣的：

```
{
  reducer1: ...
  reducer2: ...
}
```

通過為傳入物件的 reducer 命名不同來控制 state key 的命名。例如，你可以呼叫 `combineReducers({ todos: myTodosReducer, counter: myCounterReducer })` 將 state 結構變為 `{ todos, counter }`。

通常的做法是命名 reducer，然後 state 再去分割那些資訊，因此你可以使用 ES6 的簡寫方法：`combineReducers({ counter, todos })`。這與 `combineReducers({ counter: counter, todos: todos })` 一樣。

> ##### Flux 使用者使用須知

> 本函數可以幫助你組織多個 reducer，使它們分別管理自身相關聯的 state。類似於 Flux 中的多個 store 分別管理不同的 state。在 Redux 中，只有一個 store，但是 `combineReducers` 讓你擁有多個 reducer，同時保持各自負責邏輯塊的獨立性。

#### 參數

1. `reducers` (*Object*): 一個物件，它的值（value） 對應不同的 reducer 函數，這些 reducer 函數後面會被合併成一個。下面會介紹傳入 reducer 函數需要滿足的規則。

> 之前的文件曾建議使用 ES6 的 `import * as reducers` 語法來獲得 reducer 物件。這一點造成了很多疑問，因此現在建議在 `reducers/index.js` 裡使用 `combineReducers()` 來對外輸出一個 reducer。下面有示例說明。

#### 返回值

(*Function*)：一個呼叫 `reducers` 物件裡所有 reducer 的 reducer，並且構造一個與 `reducers` 物件結構相同的 state 物件。

#### 注意

本函數設計的時候有點偏主觀，就是為了避免新手犯一些常見錯誤。也因些我們故意設定一些規則，但如果你自己手動編寫根 redcuer 時並不需要遵守這些規則。

每個傳入 `combineReducers` 的 reducer 都需滿足以下規則：

* 所有未匹配到的 action，必須把它接收到的第一個參數也就是那個 `state` 原封不動返回。

* 永遠不能返回 `undefined`。當過早 `return` 時非常容易犯這個錯誤，為了避免錯誤擴散，遇到這種情況時 `combineReducers` 會拋異常。

* 如果傳入的 `state` 就是 `undefined`，一定要返回對應 reducer 的初始 state。根據上一條規則，初始 state 禁止使用 `undefined`。使用 ES6 的預設參數值語法來設定初始 state 很容易，但你也可以手動檢查第一個參數是否為 `undefined`。

雖然 `combineReducers` 自動幫你檢查 reducer 是否符合以上規則，但你也應該牢記，並儘量遵守。

#### 示例

#### `reducers/todos.js`

```js
export default function todos(state = [], action) {
  switch (action.type) {
  case 'ADD_TODO':
    return state.concat([action.text])
  default:
    return state
  }
}
```

#### `reducers/counter.js`

```js
export default function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}
```

#### `reducers/index.js`

```js
import { combineReducers } from 'redux'
import todos from './todos'
import counter from './counter'

export default combineReducers({
  todos,
  counter
})
```

#### `App.js`

```js
import { createStore } from 'redux'
import reducer from './reducers/index'

let store = createStore(reducer)
console.log(store.getState())
// {
//   counter: 0,
//   todos: []
// }

store.dispatch({
  type: 'ADD_TODO',
  text: 'Use Redux'
})
console.log(store.getState())
// {
//   counter: 0,
//   todos: [ 'Use Redux' ]
// }
```

#### 小貼士

* 本方法只是起輔助作用！你可以自行實現[不同功能](https://github.com/acdlite/reduce-reducers)的 `combineReducers`，甚至像實現其它函數一樣，明確地寫一個根 reducer 函數，用它把子 reducer 手動組裝成 state 物件。

* 在 reducer 層級的任何一級都可以呼叫 `combineReducers`。並不是一定要在最外層。實際上，你可以把一些複雜的子 reducer 拆分成單獨的孫子級 reducer，甚至更多層。
