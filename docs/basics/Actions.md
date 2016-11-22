# Action

首先，讓我們來給 action 下個定義。

**Action** 是把資料從應用（譯者注：這裡之所以不叫 view 是因為這些資料有可能是伺服器響應，使用者輸入或其它非 view 的資料 ）傳到 store 的有效載荷。它是 store 資料的**唯一**來源。一般來說你會通過 [`store.dispatch()`](../api/Store.md#dispatch) 將 action 傳到 store。

新增新 todo 任務的 action 是這樣的：

```js
const ADD_TODO = 'ADD_TODO'
```

```js
{
  type: ADD_TODO,
  text: 'Build my first Redux app'
}
```

Action 本質上是 JavaScript 普通物件。我們約定，action 內必須使用一個字元串類型的 `type` 欄位來表示將要執行的動作。多數情況下，`type` 會被定義成字元串常量。當應用規模越來越大時，建議使用單獨的模組或檔案來存放 action。

```js
import { ADD_TODO, REMOVE_TODO } from '../actionTypes'
```

>##### 樣板檔案使用提醒

>使用單獨的模組或檔案來定義 action type 常量並不是必須的，甚至根本不需要定義。對於小應用來說，使用字元串做 action type 更方便些。不過，在大型應用中把它們顯式地定義成常量還是利大於弊的。參照 [減少樣板程式碼](../recipes/ReducingBoilerplate.md) 獲取更多保持程式碼簡潔的實踐經驗。

除了 `type` 欄位外，action 物件的結構完全由你自己決定。參照 [Flux 標準 Action](https://github.com/acdlite/flux-standard-action) 獲取關於如何構造 action 的建議。

這時，我們還需要再新增一個 action type 來表示使用者完成任務的動作。因為資料是存放在陣列中的，所以我們通過下標 `index` 來引用特定的任務。而實際項目中一般會在新建資料的時候生成唯一的 ID 作為資料的引用標識。

```js
{
  type: TOGGLE_TODO,
  index: 5
}
```

**我們應該儘量減少在 action 中傳遞的資料**。比如上面的例子，傳遞 `index` 就比把整個任務物件傳過去要好。

最後，再新增一個 action type 來表示當前的任務展示選項。

```js
{
  type: SET_VISIBILITY_FILTER,
  filter: SHOW_COMPLETED
}
```

## Action 建立函數

**Action 建立函數** 就是生成 action 的方法。“action” 和 “action 建立函數” 這兩個概念很容易混在一起，使用時最好注意區分。

在 Redux 中的 action 建立函數只是簡單的返回一個 action:

```js
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}
```

這樣做將使 action 建立函數更容易被移植和測試。

在 [傳統的 Flux](http://facebook.github.io/flux) 實現中，當呼叫 action 建立函數時，一般會觸發一個 dispatch，像這樣：

```js
function addTodoWithDispatch(text) {
  const action = {
    type: ADD_TODO,
    text
  }
  dispatch(action)
}
```

不同的是，Redux 中只需把 action 建立函數的結果傳給 `dispatch()` 方法即可發起一次 dispatch 過程。

```js
dispatch(addTodo(text))
dispatch(completeTodo(index))
```

或者建立一個 **被繫結的 action 建立函數** 來自動 dispatch：

```js
const boundAddTodo = (text) => dispatch(addTodo(text))
const boundCompleteTodo = (index) => dispatch(completeTodo(index))
```

然後直接呼叫它們：

```
boundAddTodo(text);
boundCompleteTodo(index);
```

store 裡能直接通過 [`store.dispatch()`](../api/Store.md#dispatch) 呼叫 `dispatch()` 方法，但是多數情況下你會使用 [react-redux](http://github.com/gaearon/react-redux) 提供的 `connect()` 幫助器來呼叫。[`bindActionCreators()`](../api/bindActionCreators.md) 可以自動把多個 action 建立函數 繫結到 `dispatch()` 方法上。

Action 建立函數也可以是非同步非純函數。你可以通過閱讀 [高階教程](../advanced/README.md) 中的 [非同步 action](../advanced/AsyncActions.md)章節，學習如何處理 AJAX 響應和如何把 action 建立函陣列合進非同步控制流。因為基礎教程中包含了閱讀高階教程和非同步 action 章節所需要的一些重要基礎概念, 所以請在移步非同步 action 之前, 務必先完成基礎教程。

## 源碼

### `actions.js`

```js
/*
 * action 類型
 */

export const ADD_TODO = 'ADD_TODO';
export const TOGGLE_TODO = 'TOGGLE_TODO'
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER'

/*
 * 其它的常量
 */

export const VisibilityFilters = {
  SHOW_ALL: 'SHOW_ALL',
  SHOW_COMPLETED: 'SHOW_COMPLETED',
  SHOW_ACTIVE: 'SHOW_ACTIVE'
}

/*
 * action 建立函數
 */

export function addTodo(text) {
  return { type: ADD_TODO, text }
}

export function toggleTodo(index) {
  return { type: TOGGLE_TODO, index }
}

export function setVisibilityFilter(filter) {
  return { type: SET_VISIBILITY_FILTER, filter }
}
```

## 下一步

現在讓我們 [開發一些 reducers](Reducers.md) 來說明在發起 action 後 state 應該如何更新。
