# 非同步資料流

預設情況下，[`createStore()`](../api/createStore.md) 所建立的 Redux store 沒有使用 [middleware](Middleware.md)，所以只支援 [同步資料流](../basics/DataFlow.md)。

你可以使用 [`applyMiddleware()`](../api/applyMiddleware.md) 來增強 [`createStore()`](../api/createStore.md)。雖然這不是必須的，但是它可以幫助你[用簡便的方式來描述非同步的 action](AsyncActions.md)。

像 [redux-thunk](https://github.com/gaearon/redux-thunk) 或 [redux-promise](https://github.com/acdlite/redux-promise) 這樣支援非同步的 middleware 都包裝了 store 的 [`dispatch()`](../api/Store.md#dispatch) 方法，以此來讓你 dispatch 一些除了 action 以外的其他內容，例如：函數或者 Promise。你所使用的任何 middleware 都可以以自己的方式解析你 dispatch 的任何內容，並繼續傳遞 actions 給下一個 middleware。比如，支援 Promise 的 middleware 能夠攔截 Promise，然後為每個 Promise 非同步地 dispatch 一對 begin/end actions。

當 middleware 鏈中的最後一個 middleware 開始 dispatch action 時，這個 action 必須是一個普通物件。這是 [同步式的 Redux 資料流](../basics/DataFlow.md) 開始的地方（譯註：這裡應該是指，你可以使用任意多非同步的 middleware 去做你想做的事情，但是需要使用普通物件作為最後一個被 dispatch 的 action ，來將處理流程帶回同步方式）。

接著可以檢視 [非同步示例的完整源碼](ExampleRedditAPI.md)。

## 下一步

現在通過例子你對 middleware 在 Redux 中作用有了初步瞭解，是時候應用到實際開發中並自定義 middleware 了。繼續閱讀關於 [Middleware](Middleware.md) 的詳細章節。
