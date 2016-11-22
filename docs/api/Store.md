# Store

Store 就是用來維持應用所有的 [state 樹](../Glossary.md#state) 的一個物件。
改變 store 內 state 的惟一途徑是對它 dispatch 一個 [action](../Glossary.md#action)。

Store 不是類。它只是有幾個方法的物件。
要建立它，只需要把根部的 [reducing 函數](../Glossary.md#reducer) 傳遞給 [`createStore`](createStore.md)。

>##### Flux 使用者使用注意

>如果你以前使用 Flux，那麼你只需要注意一個重要的區別。Redux 沒有 Dispatcher 且不支援多個 store。相反，只有一個單一的 store 和一個根級的 reduce 函數（reducer）。隨著應用不斷變大，你應該把根級的 reducer 拆成多個小的 reducers，分別獨立地操作 state 樹的不同部分，而不是新增新的 stores。這就像一個 React 應用只有一個根級的元件，這個根元件又由很多小元件構成。

### Store 方法

- [`getState()`](#getState)
- [`dispatch(action)`](#dispatch)
- [`subscribe(listener)`](#subscribe)
- [`replaceReducer(nextReducer)`](#replaceReducer)

## Store 方法

### <a id='getState'></a>[`getState()`](#getState)

返回應用當前的 state 樹。
它與 store 的最後一個 reducer 返回值相同。

#### 返回值

*(any)*: 應用當前的 state 樹。

<hr>

### <a id='dispatch'></a>[`dispatch(action)`](#dispatch)

分發 action。這是觸發 state 變化的惟一途徑。

會使用當前 [`getState()`](#getState) 的結果和傳入的 `action` 以同步方式的呼叫 store 的 reduce 函數。返回值會被作為下一個 state。從現在開始，這就成為了 [`getState()`](#getState) 的返回值，同時變化監聽器(change listener)會被觸發。

>##### Flux 使用者使用注意
>當你在 [reducer](../Glossary.md#reducer) 內部呼叫 `dispatch` 時，將會拋出錯誤提示“Reducers may not dispatch actions.（Reducer 內不能 dispatch action）”。這就相當於 Flux 裡的 “Cannot dispatch in a middle of dispatch（dispatch 過程中不能再 dispatch）”，但並不會引起對應的錯誤。在 Flux 裡，當 Store 處理 action 和觸發 update 事件時，dispatch 是禁止的。這個限制並不好，因為他限制了不能在生命週期回撥裡 dispatch action，還有其它一些本來很正常的地方。

>在 Redux 裡，只會在根 reducer 返回新 state 結束後再會呼叫事件監聽器，因此，你可以在事件監聽器裡再做 dispatch。惟一使你不能在 reducer 中途 dispatch 的原因是要確保 reducer 沒有副作用。如果 action 處理會產生副作用，正確的做法是使用非同步 [action 建立函數](../Glossary.md#action-creator)。

#### 參數

1. `action` (*Object*<sup>†</sup>): 描述應用變化的普通物件。Action 是把資料傳入 store 的惟一途徑，所以任何資料，無論來自 UI 事件，網路回撥或者是其它資源如 WebSockets，最終都應該以 action 的形式被 dispatch。按照約定，action 具有 `type` 欄位來表示它的類型。type 也可被定義為常量或者是從其它模組引入。最好使用字元串，而不是 [Symbols](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol) 作為 action，因為字元串是可以被序列化的。除了 `type` 欄位外，action 物件的結構完全取決於你。參照 [Flux 標準 Action](https://github.com/acdlite/flux-standard-action) 獲取如何組織 action 的建議。

#### 返回值

(Object<sup>†</sup>): 要 dispatch 的 action。

#### 注意

<sup>†</sup> 使用 [`createStore`](createStore.md) 建立的 “純正” store 只支援普通物件類型的 action，而且會立即傳到 reducer 來執行。

但是，如果你用 [`applyMiddleware`](applyMiddleware.md) 來套住 [`createStore`](createStore.md) 時，middleware 可以修改 action 的執行，並支援執行 dispatch [intent（意圖）](../Glossary.md#intent)。Intent 一般是非同步操作如 Promise、Observable 或者 Thunk。

Middleware 是由社羣建立，並不會同 Redux 一起發行。你需要手動安裝 [redux-thunk](https://github.com/gaearon/redux-thunk) 或者 [redux-promise](https://github.com/acdlite/redux-promise) 庫。你也可以建立自己的 middleware。

想學習如何描述非同步 API 呼叫？看一下 action 建立函數裡當前的 state，執行一個有副作用的操作，或者以鏈式操作執行它們，參照 [`applyMiddleware`](applyMiddleware.md) 中的示例。

#### 示例

```js
import { createStore } from 'redux'
let store = createStore(todos, [ 'Use Redux' ])

function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}

store.dispatch(addTodo('Read the docs'))
store.dispatch(addTodo('Read about the middleware'))
```

<hr>

### <a id='subscribe'></a>[`subscribe(listener)`](#subscribe)

新增一個變化監聽器。每當 dispatch action 的時候就會執行，state 樹中的一部分可能已經變化。你可以在回撥函數裡呼叫 [`getState()`](#getState) 來拿到當前 state。

這是一個底層 API。多數情況下，你不會直接使用它，會使用一些 React（或其它庫）的繫結。如果你想讓回撥函數執行的時候使用當前的 state，你可以 [把 store 轉換成一個 Observable 或者寫一個定製的 `observeStore` 工具](https://github.com/rackt/redux/issues/303#issuecomment-125184409)。

如果需要解綁這個變化監聽器，執行 `subscribe` 返回的函數即可。

#### 參數

1. `listener` (*Function*): 每當 dispatch action 的時候都會執行的回撥。state 樹中的一部分可能已經變化。你可以在回撥函數裡呼叫 [`getState()`](#getState) 來拿到當前 state。store 的 reducer 應該是純函數，因此你可能需要對 state 樹中的引用做深度比較來確定它的值是否有變化。

##### 返回值

(*Function*): 一個可以解綁變化監聽器的函數。

##### 示例

```js
function select(state) {
  return state.some.deep.property
}

let currentValue
function handleChange() {
  let previousValue = currentValue
  currentValue = select(store.getState())

  if (previousValue !== currentValue) {
    console.log('Some deep nested property changed from', previousValue, 'to', currentValue)
  }
}

let unsubscribe = store.subscribe(handleChange)
handleChange()
```

<hr>

### <a id='replaceReducer'></a>[`replaceReducer(nextReducer)`](#replaceReducer)

替換 store 當前用來計算 state 的 reducer。

這是一個高階 API。只有在你需要實現程式碼分隔，而且需要立即載入一些 reducer 的時候才可能會用到它。在實現 Redux 熱載入機制的時候也可能會用到。

#### 參數

1. `reducer` (*Function*) store 會使用的下一個 reducer。
