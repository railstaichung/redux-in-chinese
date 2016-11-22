# Store

在前面的章節中，我們學會了使用 [action](Actions.md) 來描述“發生了什麼”，和使用 [reducers](Reducers.md) 來根據 action 更新 state 的用法。

**Store** 就是把它們聯絡到一起的物件。Store 有以下職責：

* 維持應用的 state；
* 提供 [`getState()`](../api/Store.md#getState) 方法獲取 state；
* 提供 [`dispatch(action)`](../api/Store.md#dispatch) 方法更新 state；
* 通過 [`subscribe(listener)`](../api/Store.md#subscribe) 註冊監聽器;
* 通過 [`subscribe(listener)`](../api/Store.md#subscribe) 返回的函數登出監聽器。

再次強調一下 **Redux 應用只有一個單一的 store**。當需要拆分資料處理邏輯時，你應該使用 [reducer 組合](Reducers.md#splitting-reducers) 而不是建立多個 store。

根據已有的 reducer 來建立 store 是非常容易的。在[前一個章節](Reducers.md)中，我們使用 [`combineReducers()`](../api/combineReducers.md) 將多個 reducer 合併成為一個。現在我們將其匯入，並傳遞 [`createStore()`](../api/createStore.md)。

```js
import { createStore } from 'redux'
import todoApp from './reducers'
let store = createStore(todoApp)
```

[`createStore()`](../api/createStore.md) 的第二個參數是可選的, 用於設定 state 初始狀態。這對開發同構應用時非常有用，伺服器端 redux 應用的 state 結構可以與客戶端保持一致, 那么客戶端可以將從網路接收到的服務端 state 直接用於本地資料初始化。

```js
let store = createStore(todoApp, window.STATE_FROM_SERVER)
```

## 發起 Actions

現在我們已經建立好了 store ，讓我們來驗證一下！雖然還沒有介面，我們已經可以測試資料處理邏輯了。

```js
import { addTodo, toggleTodo, setVisibilityFilter, VisibilityFilters } from './actions'

// 列印初始狀態
console.log(store.getState())

// 每次 state 更新時，列印日誌
// 注意 subscribe() 返回一個函數用來登出監聽器
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
)

// 發起一系列 action
store.dispatch(addTodo('Learn about actions'))
store.dispatch(addTodo('Learn about reducers'))
store.dispatch(addTodo('Learn about store'))
store.dispatch(toggleTodo(0))
store.dispatch(toggleTodo(1))
store.dispatch(setVisibilityFilter(VisibilityFilters.SHOW_COMPLETED))

// 停止監聽 state 更新
unsubscribe();
```

可以看到 store 裡的 state 是如何變化的：

<img src='http://i.imgur.com/zMMtoMz.png' width='70%'>

可以看到，在還沒有開發介面的時候，我們就可以定義程序的行為。而且這時候已經可以寫 reducer 和 action 建立函數的測試。不需要模擬任何東西，因為它們都是純函數。只需呼叫一下，對返回值做斷言，寫測試就是這麼簡單。

## 源碼

#### `index.js`

```js
import { createStore } from 'redux'
import todoApp from './reducers'

let store = createStore(todoApp)
```

## 下一步

在建立 todo 應用介面之前，我們先穿插學習一下[資料在 Redux 應用中如何流動的](DataFlow.md)。
