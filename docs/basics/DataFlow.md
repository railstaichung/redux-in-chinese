# 資料流

**嚴格的單向資料流**是 Redux 架構的設計核心。

這意味著應用中所有的資料都遵循相同的生命週期，這樣可以讓應用變得更加可預測且容易理解。同時也鼓勵做資料正規化化，這樣可以避免使用多個且獨立的無法相互引用的重複資料。

如果這些理由還不足以令你信服，讀一下 [動機](../introduction/Motivation.md) 和 [Flux 案例](https://medium.com/@dan_abramov/the-case-for-flux-379b7d1982c6)，這裡面有更加詳細的單向資料流優勢分析。雖然 Redux 就不是嚴格意義上的 [Flux](../introduction/Relation to Other Libraries.md)，但它們有共同的設計思想。

Redux 應用中資料的生命週期遵循下面 4 個步驟：

1. **呼叫** [`store.dispatch(action)`](../api/Store.md#dispatch)。

  [Action](Actions.md) 就是一個描述“發生了什麼”的普通物件。比如：

    ```js
    { type: 'LIKE_ARTICLE', articleId: 42 };
    { type: 'FETCH_USER_SUCCESS', response: { id: 3, name: 'Mary' } };
    { type: 'ADD_TODO', text: 'Read the Redux docs.'};
    ```

  可以把 action 理解成新聞的摘要。如 “瑪麗喜歡42號文章。” 或者 “任務列表裡新增了'學習 Redux 文件'”。

  你可以在任何地方呼叫 [`store.dispatch(action)`](../api/Store.md#dispatch)，包括元件中、XHR 回撥中、甚至定時器中。

2. **Redux store 呼叫傳入的 reducer 函數。**

  [Store](Store.md) 會把兩個參數傳入 [reducer](Reducers.md)： 當前的 state 樹和 action。例如，在這個 todo 應用中，根 reducer 可能接收這樣的資料：

    ```js
    // 當前應用的 state（todos 列表和選中的過濾器）
    let previousState = {
      visibleTodoFilter: 'SHOW_ALL',
      todos: [
        {
          text: 'Read the docs.',
          complete: false
        }
      ]
    }

    // 將要執行的 action（新增一個 todo）
    let action = {
      type: 'ADD_TODO',
      text: 'Understand the flow.'
    }

    // render 返回處理後的應用狀態
    let nextState = todoApp(previousState, action);
    ```

    注意 reducer 是純函數。它僅僅用於計算下一個 state。它應該是完全可預測的：多次傳入相同的輸入必須產生相同的輸出。它不應做有副作用的操作，如 API 呼叫或路由跳轉。這些應該在 dispatch action 前發生。

3. **根 reducer 應該把多個子 reducer 輸出合併成一個單一的 state 樹。**

  根 reducer 的結構完全由你決定。Redux 原生提供[`combineReducers()`](../api/combineReducers.md)輔助函數，來把根 reducer 拆分成多個函數，用於分別處理 state 樹的一個分支。

  下面演示 [`combineReducers()`](../api/combineReducers.md) 如何使用。假如你有兩個 reducer：一個是 todo 列表，另一個是當前選擇的過濾器設定：

    ```js
    function todos(state = [], action) {
      // 省略處理邏輯...
      return nextState;
    }

    function visibleTodoFilter(state = 'SHOW_ALL', action) {
      // 省略處理邏輯...
      return nextState;
    }

    let todoApp = combineReducers({
      todos,
      visibleTodoFilter
    })
    ```

  當你觸發 action 後，`combineReducers` 返回的 `todoApp` 會負責呼叫兩個 reducer：

    ```js
    let nextTodos = todos(state.todos, action);
    let nextVisibleTodoFilter = visibleTodoFilter(state.visibleTodoFilter, action);
    ```

  然後會把兩個結果集合併成一個 state 樹：

    ```js
    return {
      todos: nextTodos,
      visibleTodoFilter: nextVisibleTodoFilter
    };
    ```

  雖然 [`combineReducers()`](../api/combineReducers.md) 是一個很方便的輔助工具，你也可以選擇不用；你可以自行實現自己的根 reducer！

4. **Redux store 儲存了根 reducer 返回的完整 state 樹。**

  這個新的樹就是應用的下一個 state！所有訂閱 [`store.subscribe(listener)`](../api/Store.md#subscribe) 的監聽器都將被呼叫；監聽器裡可以呼叫 [`store.getState()`](../api/Store.md#getState) 獲得當前 state。

  現在，可以應用新的 state 來更新 UI。如果你使用了 [React Redux](https://github.com/gaearon/react-redux) 這類的繫結庫，這時就應該呼叫 `component.setState(newState)` 來更新。

## 下一步

現在你已經理解了 Redux 如何工作，是時候[結合 React 開發應用](UsageWithReact.md)了。

>##### 高階使用者使用注意
>如果你已經熟悉了基礎概念且完成了這個教程，可以學習[高階教程](../advanced/README.md)中的[非同步資料流](../advanced/AsyncFlow.md)，你將學到如何使用 middleware 在 [非同步 action](../advanced/AsyncActions.md) 到達 reducer 前處理它們。
