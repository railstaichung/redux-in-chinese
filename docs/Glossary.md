# 詞彙表

這是 Redux 的核心概念詞彙表以及這些核心概念的類型簽名。這些類型使用了[流標註法](http://flowtype.org/docs/quick-reference.html)進行記錄。

## State

```js
type State = any
```

*State* (也稱為 *state tree*) 是一個寬泛的概念，但是在 Redux API 中，通常是指一個唯一的 state 值，由 store 管理且由 [`getState()`](api/Store.md#getState) 方法獲得。它表示了 Redux 應用的全部狀態，通常為一個多層巢狀的物件。

約定俗成，頂層 state 或為一個物件，或像 Map 那樣的鍵-值集合，也可以是任意的資料類型。然而你應儘可能確保 state 可以被序列化，而且不要把什麼資料都放進去，導致無法輕鬆地把 state 轉換成 JSON。

## Action

```js
type Action = Object
```

*Action* 是一個普通物件，用來表示即將改變 state 的意圖。它是將資料放入 store 的唯一途徑。無論是從 UI 事件、網路回撥，還是其他諸如 WebSocket 之類的資料來源所獲得的資料，最終都會被 dispatch 成 action。

約定俗成，action 必須擁有一個 `type` 域，它指明瞭需要被執行的 action type。Type 可以被定義為常量，然後從其他 module 匯入。比起用 [Symbols](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol) 表示 `type`，使用 String 是更好的方法，因為 string 可以被序列化。

除了 `type` 之外，action 物件的結構其實完全取決於你自己。如果你感興趣的話，請參考 [Flux Standard Action](https://github.com/acdlite/flux-standard-action) ，瞭解如何構建 action。

還有就是請看後面的 [非同步 action](#async-action)。

## Reducer

```js
type Reducer<S, A> = (state: S, action: A) => S
```

*Reducer* (也稱為 *reducing function*) 函數接受兩個參數：之前累積運算的結果和當前被累積的值，返回的是一個新的累積結果。該函數把一個集合歸併成一個單值。

Reducer 並不是 Redux 特有的函數 —— 它是函數語言程式設計中的一個基本概念，甚至大部分的非函數式語言比如 JavaScript，都有一個內建的 reduce API。對於 JavaScript，這個 API 是 [`Array.prototype.reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce).

在 Redux 中，累計運算的結果是 state 物件，而被累積的值是 action。Reducer 由上次累積的結果 state 與當前被累積的 action 計算得到一個新 state。這些 Reducer 必須是**純函數**，而且當輸入相同時返回的結果也會相同。它們不應該產生任何副作用。正因如此，才使得諸如熱過載和時間旅行這些很棒的功能成為可能。

Reducer 是 Redux 之中最重要的概念。

**不要在 reducer 中有 API 呼叫**

## dispatch 函數

```js
type BaseDispatch = (a: Action) => Action
type Dispatch = (a: Action | AsyncAction) => any
```

*dispatching function* (或簡言之 *dispatch function*) 是一個接收 action 或者[非同步 action](#async-action)的函數，該函數要麼往 store 分發一個或多個 action，要麼不分發任何 action。

我們必須分清一般的 dispatch function 以及由 store 例項提供的沒有 middleware 的 base [`dispatch`](api/Store.md#dispatch) function 之間的區別。

Base dispatch function **總是**同步地把 action 與上一次從 store 返回的 state 發往 reducer，然後計算出新的 state。它期望 action 會是一個可以被 reducer 消費的普通物件。

[Middleware](#middleware) 封裝了 base dispatch function，允許 dispatch function 處理 action 之外的[非同步 action](#async-action)。 Middleware 可以改變、延遲、忽略 action 或非同步 action，也可以在傳遞給下一個 middleware 之前對它們進行解釋。獲取更多資訊請往後看。

## Action Creator

```js
type ActionCreator = (...args: any) => Action | AsyncAction
```

*Action Creator* 很簡單，就是一個建立 action 的函數。不要混淆 action 和 action creator 這兩個概念。Action 是一個資訊的負載，而 action creator 是一個建立 action 的工廠。

呼叫 action creator 只會生產 action，但不分發。你需要呼叫 store 的 [`dispatch`](api/Store.md#dispatch) function 才會引起變化。有時我們講 *bound action creator*，是指一個函數呼叫了 action creator 並立即將結果分發給一個特定的 store 例項。

如果 action creator 需要讀取當前的 state、呼叫 API、或引起諸如路由變化等副作用，那麼它應該返回一個[非同步 action](#async-action)而不是 action。

## 非同步 Action

```js
type AsyncAction = any
```

*非同步 action* 是一個發給 dispatching 函數的值，但是這個值還不能被 reducer 消費。在發往 base [`dispatch()`](api/Store.md#dispatch) function 之前，[middleware](#middleware) 會把非同步 action 轉換成一個或一組 action。非同步 action 可以有多種 type，這取決於你所使用的 middleware。它通常是 Promise 或者 thunk 之類的非同步原生資料類型，雖然不會立即把資料傳遞給 reducer，但是一旦操作完成就會觸發 action 的分發事件。

##  Middleware

```js
type MiddlewareAPI = { dispatch: Dispatch, getState: () => State }
type Middleware = (api: MiddlewareAPI) => (next: Dispatch) => Dispatch
```

Middleware 是一個組合 [dispatch function](#dispatching-function) 的高階函數，返回一個新的 dispatch function，通常將[非同步 actions](#async-action) 轉換成 action。

Middleware 利用複合函數使其可以組合其他函數，可用於記錄 action 日誌、產生其他諸如變化路由的副作用，或將非同步的 API 呼叫變為一組同步的 action。

請見 [`applyMiddleware(...middlewares)`](./api/applyMiddleware.md) 獲取 middleware 的詳細內容。

## Store

```js
type Store = {
  dispatch: Dispatch
  getState: () => State
  subscribe: (listener: () => void) => () => void
  replaceReducer: (reducer: Reducer) => void
};
```

Store 維持著應用的 state tree 物件。
因為應用的構建發生於 reducer，所以一個 Redux 應用中應當只有一個 Store。

- [`dispatch(action)`](api/Store.md#dispatch) 是上述的 base dispatch function。
- [`getState()`](api/Store.md#getState) 返回當前 store 的 state。
- [`subscribe(listener)`](api/Store.md#subscribe) 註冊一個 state 發生變化時的回撥函數。
- [`replaceReducer(nextReducer)`](api/Store.md#replaceReducer) 可用於熱過載和程式碼分割。通常你不需要用到這個 API。

詳見完整的 [store API reference](api/Store.md#dispatch)。

## Store Creator

```js
type StoreCreator = (reducer: Reducer, initialState: ?State) => Store
```

Store creator 是一個建立 Redux store 的函數。就像 dispatching function 那樣，我們必須分清通過 [`createStore(reducer, initialState)`](api/createStore.md) 由 Redux 匯出的 base store creator 與從 store enhancer 返回的 store creator 之間的區別。

## Store enhancer

```js
type StoreEnhancer = (next: StoreCreator) => StoreCreator
```

Store enhancer 是一個組合 store creator 的高階函數，返回一個新的強化過的 store creator。這與 middleware 相似，它也允許你通過複合函數改變 store 介面。

Store enhancer 與 React 的高階 component 概念一致，通常也會稱為 “component enhancers”。

因為 store 並非例項，更像是一個函數集合的普通物件，所以可以輕鬆地建立副本，也可以在不改變原先的 store 的條件下修改副本。在 [`compose`](api/compose.md) 文件中有一個示例演示了這種做法。

大多數時候你基本不用編寫 store enhancer，但你可能會在 [developer tools](https://github.com/gaearon/redux-devtools) 中用到。正因為 store enhancer，應用程式才有可能察覺不到“時間旅行”。有趣的是，[Redux middleware 本身的實現](api/applyMiddleware.md)就是一個 store enhancer。
