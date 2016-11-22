# 實現撤銷歷史

在應用中內建撤消和重做功能往往需要開發者有意識的做出一些努力。對於經典的 MVC 框架來說這不是一個簡單的問題，因為你需要通過克隆所有相關的 model 來追蹤每一個歷史狀態。此外，你需要關心整個撤消堆棧，因為使用者初始化的更改也應該是可撤消的。

這意味著在一個 MVC 應用中實現撤消和重做，通常迫使你用一些類似於 [Command](https://en.wikipedia.org/wiki/Command_pattern) 的特殊的資料修改模式來重寫應用中的部分程式碼。

而在 Redux 中，實現撤銷歷史卻是輕而易舉的。有以下三個原因：

* 你想要跟蹤的 state 子樹不會包含多個模型（models—just）。
* state 是不可變的，所有修改已經被描述成分離的 action，而這些 action 與預期的撤銷堆棧模型很接近了。
* reducer 的簽名 `(state, action) => state` 讓它可以自然的實現 “reducer enhancers” 或者 “higher order reducers”。它們可以讓你在為 reducer 新增額外的功能時保持這個簽名。撤消歷史就是一個典型的應用場景。

在動手之前，確認你已經閱讀過[基礎教程](../basics/README.md)並且良好掌握了 [reducer 合成](../basics/Reducers.md)。本文中的程式碼會構建於 [基礎教程](../basics/README.md) 的示例之上。

文章的第一部分，我們將會解釋實現撤消和重做功能所用到的基礎概念。

在第二部分中，我們會展示如何使用 [Redux Undo](https://github.com/omnidan/redux-undo) 庫來無縫地實現撤消和重做。

[![demo of todos-with-undo](http://i.imgur.com/lvDFHkH.gif)](https://twitter.com/dan_abramov/status/647038407286390784)


## 理解撤消歷史

### 設計狀態結構

撤消歷史也是你的應用 state 的一部分，我們沒有任何原因通過不同的方式實現它。無論 state 如何隨著時間不斷變化，當你實現撤消和重做這個功能時，你就必須追蹤 state 在不同時刻的**歷史記錄**。

例如，一個計數器應用的 state 結構看起來可能是這樣：

```js
{
  counter: 10
}
```

如果我們希望在這樣一個應用中實現撤消和重做的話，我們必須儲存更多的 state 以解決下面幾個問題：

* 撤消或重做留下了哪些資訊？
* 當前的狀態是什麼？
* 撤銷堆棧中過去（和未來）的狀態是什麼？

這是一個對於 state 結構的修改建議，可以回答上述問題的：

```js
{
  counter: {
    past: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9],
    present: 10,
    future: []
  }
}
```

現在，如果我們按下“撤消”，我們希望恢復到過去的狀態：

```js
{
  counter: {
    past: [0, 1, 2, 3, 4, 5, 6, 7, 8],
    present: 9,
    future: [10]
  }
}
```

再來一次：

```js
{
  counter: {
    past: [0, 1, 2, 3, 4, 5, 6, 7],
    present: 8,
    future: [9, 10]
  }
}
```

當我們按下“重做”，我們希望往未來的狀態移動一步：

```js
{
  counter: {
    past: [0, 1, 2, 3, 4, 5, 6, 7, 8],
    present: 9,
    future: [10]
  }
}
```

最終，如果處於撤銷堆棧中，使用者發起了一個操作（例如，減少計數），我們將會丟棄所有未來的資訊：

```js
{
  counter: {
    past: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9],
    present: 8,
    future: []
  }
}
```

有趣的一點是，我們在撤銷堆棧中儲存數字，字元串，陣列或是物件都沒有關係。整個結構始終完全一致：

```js
{
  counter: {
    past: [0, 1, 2],
    present: 3,
    future: [4]
  }
}
```

```js
{
  todos: {
    past: [
      [],
      [{ text: 'Use Redux' }],
      [{ text: 'Use Redux', complete: true }]
    ],
    present: [{ text: 'Use Redux', complete: true }, { text: 'Implement Undo' }],
    future: [
      [{ text: 'Use Redux', complete: true }, { text: 'Implement Undo', complete: true }]
    ]
  }
}
```

它看起來通常都是這樣：

```js
{
  past: Array<T>,
  present: T,
  future: Array<T>
}
```

我們可以儲存單一的頂層歷史記錄：

```js
{
  past: [
    { counterA: 1, counterB: 1 },
    { counterA: 1, counterB: 0 },
    { counterA: 0, counterB: 0 }
  ],
  present: { counterA: 2, counterB: 1 },
  future: []
}
```

也可以分離的歷史記錄，使用者可以獨立地執行撤消和重做操作：

```js
{
  counterA: {
    past: [1, 0],
    present: 2,
    future: []
  },
  counterB: {
    past: [0],
    present: 1,
    future: []
  }
}
```

接下來我們將會看到如何選擇合適的撤消和重做的顆粒度。

### 設計演算法

無論何種特定的資料類型，重做歷史記錄的 state 結構始終一致：

```js
{
  past: Array<T>,
  present: T,
  future: Array<T>
}
```

讓我們討論一下如何通過演算法來操作上文所述的 state 結構。我們可以定義兩個 action 來操作該 state：`UNDO` 和 `REDO`。在 reducer 中，我們希望以如下步驟處理這兩個 action：

#### 處理 Undo

* 移除 `past` 中的**最後一個**元素。
* 將上一步移除的元素賦予 `present`。
* 將原來的 `present` 插入到 `future` 的**最前面**。

#### 處理 Redo

* 移除 `future` 中的**第一個**元素。
* 將上一步移除的元素賦予 `present`。
* 將原來的 `present` 追加到 `past` 的**最後面**。

#### 處理其他 Action

* 將當前的 `present` 追加到 `past` 的**最後面**。
* 將處理完 action 所產生的新的 state 賦予 `present`。
* 清空 `future`。

### 第一次嘗試: 編寫 Reducer

```js
const initialState = {
  past: [],
  present: null, // (?) 我們如何初始化當前狀態?
  future: []
};

function undoable(state = initialState, action) {
  const { past, present, future } = state;

  switch (action.type) {
  case 'UNDO':
    const previous = past[past.length - 1];
    const newPast = past.slice(0, past.length - 1);
    return {
      past: newPast,
      present: previous,
      future: [present, ...future]
    };
  case 'REDO':
    const next = future[0];
    const newFuture = future.slice(1);
    return {
      past: [...past, present],
      present: next,
      future: newFuture
    };
  default:
    // (?) 我們如何處理其他 action？
    return state;
  }
}
```

這個實現是無法使用的，因為它忽略了下面三個重要的問題：

* 我們從何處獲取初始的 `present` 狀態？我們無法預先知道它。
* 當外部 action 被處理完畢後，我們在哪裡完成將 `present` 儲存到 `past` 的工作？
* 我們如何將 `present` 狀態的控制委託給一個自定義的 reducer？

看起來 reducer 並不是正確的抽象方式，但是我們已經非常接近了。

### 遇見 Reducer Enhancers

你可能已經熟悉 [higher order function](https://en.wikipedia.org/wiki/Higher-order_function) 了。如果你使用過 React，也應該熟悉 [higher order component](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750)。對於 reducer 來說，也有一種對應的實現模式。

一個 **reducer enhancer**（或者一個 **higher order reducer**）作為一個函數，接收 reducer 作為參數，並返回一個新的 reducer，這個新的 reducer 可以處理新的 action，或者維護更多的 state，亦或者將它無法處理的 action 委託給原始的 reducer 處理。這不是什麼新的模式技術（pattern—technically），[`combineReducers()`](../api/combineReducers.md)就是一個 reducer enhancer，因為它同樣接收多個 reducer 並返回一個新的 reducer。

這是一個沒有任何額外功能的 reducer enhancer 的示例：

```js
function doNothingWith(reducer) {
  return function (state, action) {
    // 僅僅是呼叫被傳入的 reducer
    return reducer(state, action);
  };
}
```

一個可以組合 reducer 的 reducer enhancer 看起來應該像這樣：

```js
function combineReducers(reducers) {
  return function (state = {}, action) {
    return Object.keys(reducers).reduce((nextState, key) => {
      // 呼叫每一個 reducer，並將由它管理的部分 state 傳給它
      nextState[key] = reducers[key](state[key], action);
      return nextState;
    }, {});
  };
}
```

### 第二次嘗試: 編寫 Reducer Enhancer

現在我們對 reducer enhancer 有了更深的瞭解，我們可以明確所謂的`可撤銷`到底是什麼：

```js
function undoable(reducer) {
  // 以一個空的 action 呼叫 reducer 來產生初始的 state
  const initialState = {
    past: [],
    present: reducer(undefined, {}),
    future: []
  };

  // 返回一個可以執行撤銷和重做的新的reducer
  return function (state = initialState, action) {
    const { past, present, future } = state;

    switch (action.type) {
    case 'UNDO':
      const previous = past[past.length - 1];
      const newPast = past.slice(0, past.length - 1);
      return {
        past: newPast,
        present: previous,
        future: [present, ...future]
      };
    case 'REDO':
      const next = future[0];
      const newFuture = future.slice(1);
      return {
        past: [...past, present],
        present: next,
        future: newFuture
      };
    default:
      // 將其他 action 委託給原始的 reducer 處理
      const newPresent = reducer(present, action);
      if (present === newPresent) {
        return state;
      }
      return {
        past: [...past, present],
        present: newPresent,
        future: []
      };
    }
  };
}
```

我們現在可以將任意的 reducer 通過這個擁有`可撤銷`能力的 reducer enhancer 進行封裝，從而讓它們可以處理 `UNDO` 和 `REDO` 這兩個 action。

```js
// 這是一個 reducer。
function todos(state = [], action) {
  /* ... */
}

// 處理完成之後仍然是一個 reducer！
const undoableTodos = undoable(todos);

import { createStore } from 'redux';
const store = createStore(undoableTodos);

store.dispatch({
  type: 'ADD_TODO',
  text: 'Use Redux'
});

store.dispatch({
  type: 'ADD_TODO',
  text: 'Implement Undo'
});

store.dispatch({
  type: 'UNDO'
});
```

還有一個重要注意點：你需要記住當你恢復一個 state 時，必須把 `.present` 追加到它上面。你也不能忘了需要通過檢查 `.past.length` 和 `.future.length` 來決定撤銷和重做按鈕是否可用。

你可能聽說過 Redux 受 [Elm 架構](https://github.com/evancz/elm-architecture-tutorial/) 影響頗深。所以這個示例與 [elm-undo-redo package](http://package.elm-lang.org/packages/TheSeamau5/elm-undo-redo/2.0.0) 十分相似也不會太令人吃驚。

## 使用 Redux Undo

以上這些都非常有用，但是有沒有一個庫能幫助我們實現`可撤銷`功能，而不是由我們自己編寫呢？當然有！來看看 [Redux Undo](https://github.com/omnidan/redux-undo)，它可以為你的 Redux 狀態樹中的任何部分提供撤銷和重做功能。

在這個部分中，你會學到如何讓 [示例：Todo List](../basics/ExampleTodoList.md) 擁有可撤銷的功能。你可以在 [`todos-with-undo`](https://github.com/rackt/redux/tree/master/examples/todos-with-undo)找到完整的源碼。

### 安裝

首先，你必須先執行

```
npm install --save redux-undo
```

這一步會安裝一個提供`可撤銷`功能的 reducer enhancer 的庫。

### 封裝 Reducer

你需要通過 `undoable` 函數強化你的 reducer。例如，如果使用了 [`combineReducers()`](../api/combineReducers.md)，你的程式碼看起來應該像這樣：

#### `reducers.js`

```js
import undoable, { distinctState } from 'redux-undo';

/* ... */

const todoApp = combineReducers({
  visibilityFilter,
  todos: undoable(todos, { filter: distinctState() })
});
```

`distinctState()` 過濾器將會忽略那些沒有引起 state 變化的 action。還有一些[其他選項](https://github.com/omnidan/redux-undo#configuration)來配置你可撤銷的 reducer，例如為撤銷和重做動作指定 action 的類型。

你可以在 reducer 合併層次中的任何級別對一個或多個 reducer 執行 `undoable`。由於 `visibilityFilter` 的變化並不會影響撤銷歷史，我們選擇只對 `todos` reducer 進行封裝，而不是整個頂層的 reducer。

### 更新 Selector

現在 `todos` 相關的 state 看起來應該像這樣：

```js
{
  visibilityFilter: 'SHOW_ALL',
  todos: {
    past: [
      [],
      [{ text: 'Use Redux' }],
      [{ text: 'Use Redux', complete: true }]
    ],
    present: [{ text: 'Use Redux', complete: true }, { text: 'Implement Undo' }],
    future: [
      [{ text: 'Use Redux', complete: true }, { text: 'Implement Undo', complete: true }]
    ]
  }
}
```

這意味著你必須要通過 `state.todos.present` 操作 state，而不是原來的
 `state.todos`：

#### `containers/App.js`

```js
function select(state) {
  const presentTodos = state.todos.present;
  return {
    visibleTodos: selectTodos(presentTodos, state.visibilityFilter),
    visibilityFilter: state.visibilityFilter
  };
}
```

為了確認撤銷和重做按鈕是否可用，你必須檢查 `past` 和 `future` 陣列是否為空：

#### `containers/App.js`

```js
function select(state) {
  return {
    undoDisabled: state.todos.past.length === 0,
    redoDisabled: state.todos.future.length === 0,
    visibleTodos: selectTodos(state.todos.present, state.visibilityFilter),
    visibilityFilter: state.visibilityFilter
  };
}
```

### 新增按鈕

現在，你需要做的全部事情就只是為撤銷和重做操作新增按鈕了。

首先，你需要從 `redux-undo` 中匯入 `ActionCreators`，並將他們傳遞給 `Footer` 元件：

#### `containers/App.js`

```js
import { ActionCreators } from 'redux-undo';

/* ... */

class App extends Component {
  render() {
    const { dispatch, visibleTodos, visibilityFilter } = this.props;
    return (
      <div>
        {/* ... */}
        <Footer
          filter={visibilityFilter}
          onFilterChange={nextFilter => dispatch(setVisibilityFilter(nextFilter))}
          onUndo={() => dispatch(ActionCreators.undo())}
          onRedo={() => dispatch(ActionCreators.redo())}
          undoDisabled={this.props.undoDisabled}
          redoDisabled={this.props.redoDisabled} />
      </div>
    );
  }
}
```

在 footer 中渲染它們：

#### `components/Footer.js`

```js
export default class Footer extends Component {

  /* ... */

  renderUndo() {
    return (
      <p>
        <button onClick={this.props.onUndo} disabled={this.props.undoDisabled}>Undo</button>
        <button onClick={this.props.onRedo} disabled={this.props.redoDisabled}>Redo</button>
      </p>
    );
  }

  render() {
    return (
      <div>
        {this.renderFilters()}
        {this.renderUndo()}
      </div>
    );
  }
}
```

就是這樣！在[示例資料夾](https://github.com/rackt/redux/tree/master/examples/todos-with-undo)下執行 `npm install` 和 `npm start` 試試看吧！
