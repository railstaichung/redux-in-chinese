# Reducer

[Action](./Actions.md) 只是描述了**有事情發生了**這一事實，並沒有指明應用如何更新 state。而這正是 reducer 要做的事情。

## 設計 State 結構

在 Redux 應用中，所有的 state 都被儲存在一個單一物件中。建議在寫程式碼前先想一下這個物件的結構。如何才能以最簡的形式把應用的 state 用物件描述出來？

以 todo 應用為例，需要儲存兩種不同的資料：

* 當前選中的任務過濾條件；
* 完整的任務列表。

通常，這個 state 樹還需要存放其它一些資料，以及一些 UI 相關的 state。這樣做沒問題，但儘量把這些資料與 UI 相關的 state 分開。

```js
{
  visibilityFilter: 'SHOW_ALL',
  todos: [
    {
      text: 'Consider using Redux',
      completed: true,
    },
    {
      text: 'Keep all state in a single tree',
      completed: false
    }
  ]
}
```

>##### 處理 Reducer 關係時的注意事項

>開發複雜的應用時，不可避免會有一些資料相互引用。建議你儘可能地把 state 正規化化，不存在巢狀。把所有資料放到一個物件裡，每個資料以 ID 為主鍵，不同實體或列表間通過 ID 相互引用資料。把應用的 state 想像成資料庫。這種方法在 [normalizr](https://github.com/gaearon/normalizr) 文件裡有詳細闡述。例如，實際開發中，在 state 裡同時存放 `todosById: { id -> todo }` 和 `todos: array<id>` 是比較好的方式，本文中為了保持示例簡單沒有這樣處理。

## Action 處理

現在我們已經確定了 state 物件的結構，就可以開始開發 reducer。reducer 就是一個純函數，接收舊的 state 和 action，返回新的 state。

```js
(previousState, action) => newState
```

之所以稱作 reducer 是因為它將被傳遞給 [`Array.prototype.reduce(reducer, ?initialValue)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) 方法。保持 reducer 純淨非常重要。**永遠不要**在 reducer 裡做這些操作：

* 修改傳入參數；
* 執行有副作用的操作，如 API 請求和路由跳轉；
* 呼叫非純函數，如 `Date.now()` 或 `Math.random()`。

在[高階篇](../advanced/README.md)裡會介紹如何執行有副作用的操作。現在只需要謹記 reducer 一定要保持純淨。**只要傳入參數相同，返回計算得到的下一個 state 就一定相同。沒有特殊情況、沒有副作用，沒有 API 請求、沒有變數修改，單純執行計算。**

明白了這些之後，就可以開始編寫 reducer，並讓它來處理之前定義過的 [action](Actions.md)。

我們將以指定 state 的初始狀態作為開始。Redux 首次執行時，state 為 `undefined`，此時我們可藉機設定並返回應用的初始 state。

```js
import { VisibilityFilters } from './actions'

const initialState = {
  visibilityFilter: VisibilityFilters.SHOW_ALL,
  todos: []
};

function todoApp(state, action) {
  if (typeof state === 'undefined') {
    return initialState
  }

  // 這裡暫不處理任何 action，
  // 僅返回傳入的 state。
  return state
}
```

這裡一個技巧是使用 [ES6 參數預設值語法](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/default_parameters) 來精簡程式碼。

```js
function todoApp(state = initialState, action) {
  // 這裡暫不處理任何 action，
  // 僅返回傳入的 state。
  return state
}
```

現在可以處理 `SET_VISIBILITY_FILTER`。需要做的只是改變 state 中的 `visibilityFilter`。

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    default:
      return state
  }
}
```

注意:

1. **不要修改 `state`。** 使用 [`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 新建了一個副本。不能這樣使用 `Object.assign(state, { visibilityFilter: action.filter })`，因為它會改變第一個參數的值。你**必須**把第一個參數設定為空物件。你也可以開啟對ES7提案[物件展開運算符](../recipes/UsingObjectSpreadOperator.md)的支援, 從而使用 `{ ...state, ...newState }` 達到相同的目的。

2. **在 `default` 情況下返回舊的 `state`。**遇到未知的 action 時，一定要返回舊的 `state`。

>##### `Object.assign` 須知

>[`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 是 ES6 特性，但多數瀏覽器並不支援。你要麼使用 polyfill，[Babel 外掛](https://github.com/babel-plugins/babel-plugin-object-assign)，或者使用其它庫如 [`_.assign()`](https://lodash.com/docs#assign) 提供的幫助方法。

>##### `switch` 和樣板程式碼須知

>`switch` 語句並不是嚴格意義上的樣板程式碼。Flux 中真實的樣板程式碼是概念性的：更新必須要傳送、Store 必須要註冊到 Dispatcher、Store 必須是物件（開發同構應用時變得非常複雜）。為了解決這些問題，Redux 放棄了 event emitters（事件傳送器），轉而使用純 reducer。

>很不幸到現在為止，還有很多人存在一個誤區：根據文件中是否使用 `switch` 來決定是否使用它。如果你不喜歡 `switch`，完全可以自定義一個 `createReducer` 函數來接收一個事件處理函數列表，參照["減少樣板程式碼"](../recipes/ReducingBoilerplate.md#reducers)。

## 處理多個 action

還有兩個 action 需要處理。讓我們先處理 `ADD_TODO`。

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      })
    default:
      return state
  }
}
```

如上，不直接修改 `state` 中的欄位，而是返回新物件。新的 `todos` 物件就相當於舊的 `todos` 在末尾加上新建的 todo。而這個新的 todo 又是基於 action 中的資料建立的。

最後，`TOGGLE_TODO` 的實現也很好理解：

```js
case TOGGLE_TODO:
  return Object.assign({}, state, {
    todos: state.todos.map((todo, index) => {
      if (index === action.index) {
        return Object.assign({}, todo, {
          completed: !todo.completed
        })
      }
      return todo
    })
  })
```

我們需要修改陣列中指定的資料項而又不希望導致**突變**, 因此我們的做法是在建立一個新的陣列後, 將那些無需修改的項原封不動移入, 接著對需修改的項用新生成的物件替換。(譯者注:Javascript中的物件儲存時均是由值和指向值的引用兩個部分構成。此處**突變**指直接修改引用所指向的值, 而引用本身保持不變。) 如果經常需要這類的操作，可以選擇使用幫助類 [React-addons-update](https://facebook.github.io/react/docs/update.html)，[updeep](https://github.com/substantial/updeep)，或者使用原生支援深度更新的庫 [Immutable](http://facebook.github.io/immutable-js/)。最後，時刻謹記永遠不要在克隆 `state` 前修改它。

## 拆分 Reducer

目前的程式碼看起來有些冗長：

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      })
    case TOGGLE_TODO:
      return Object.assign({}, state, {
        todos: state.todos.map((todo, index) => {
          if(index === action.index) {
            return Object.assign({}, todo, {
              completed: !todo.completed
            })
          }
          return todo
        })
      })
    default:
      return state
  }
}
```

上面程式碼能否變得更通俗易懂？這裡的 `todos` 和 `visibilityFilter` 的更新看起來是相互獨立的。有時 state 中的欄位是相互依賴的，需要認真考慮，但在這個案例中我們可以把 `todos` 更新的業務邏輯拆分到一個單獨的函數裡：

```js
function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
    case TOGGLE_TODO:
      return Object.assign({}, state, {
        todos: todos(state.todos, action)
      })
    default:
      return state
  }
}
```

注意 `todos` 依舊接收 `state`，但它變成了一個陣列！現在 `todoApp` 只把需要更新的一部分 state 傳給 `todos` 函數，`todos` 函數自己確定如何更新這部分資料。**這就是所謂的 *reducer 合成*，它是開發 Redux 應用最基礎的模式。**

下面深入探討一下如何做 reducer 合成。能否抽出一個 reducer 來專門管理 `visibilityFilter`？當然可以：

```js
function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return action.filter
  default:
    return state
  }
}
```

現在我們可以開發一個函數來做為主 reducer，它呼叫多個子 reducer 分別處理 state 中的一部分資料，然後再把這些資料合成一個大的單一物件。主 reducer 並不需要設定初始化時完整的 state。初始時，如果傳入 `undefined`, 子 reducer 將負責返回它們的預設值。

```js
function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}

function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  }
}
```

**注意每個 reducer 只負責管理全局 state 中它負責的一部分。每個 reducer 的 `state` 參數都不同，分別對應它管理的那部分 state 資料。**

現在看起來好多了！隨著應用的膨脹，我們還可以將拆分後的 reducer 放到不同的檔案中, 以保持其獨立性並用於專門處理不同的資料域。

最後，Redux 提供了 [`combineReducers()`](../api/combineReducers.md) 工具類來做上面 `todoApp` 做的事情，這樣就能消滅一些樣板程式碼了。有了它，可以這樣重構 `todoApp`：

```js
import { combineReducers } from 'redux';

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp;
```

注意上面的寫法和下面完全等價：

```js
export default function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  }
}
```

你也可以給它們設定不同的 key，或者呼叫不同的函數。下面兩種合成 reducer 方法完全等價：

```js
const reducer = combineReducers({
  a: doSomethingWithA,
  b: processB,
  c: c
})
```

```js
function reducer(state = {}, action) {
  return {
    a: doSomethingWithA(state.a, action),
    b: processB(state.b, action),
    c: c(state.c, action)
  }
}
```

[`combineReducers()`](../api/combineReducers.md) 所做的只是生成一個函數，這個函數來呼叫你的一系列 reducer，每個 reducer **根據它們的 key 來篩選出 state 中的一部分資料並處理**，然後這個生成的函數再將所有 reducer 的結果合併成一個大的物件。[沒有任何魔法。](https://github.com/gaearon/redux/issues/428#issuecomment-129223274)

>##### ES6 使用者使用注意

>`combineReducers` 接收一個物件，可以把所有頂級的 reducer 放到一個獨立的檔案中，通過 `export` 暴露出每個 reducer 函數，然後使用 `import * as reducers` 得到一個以它們名字作為 key 的 object：

>```js
>import { combineReducers } from 'redux'
>import * as reducers from './reducers'
>
>const todoApp = combineReducers(reducers)
>```
>
>由於 `import *` 還是比較新的語法，為了避免[困惑](https://github.com/gaearon/redux/issues/428#issuecomment-129223274)，我們不會在本文件中使用它。但在一些社羣示例中你可能會遇到它們。

## 源碼

#### `reducers.js`

```js
import { combineReducers } from 'redux'
import { ADD_TODO, TOGGLE_TODO, SET_VISIBILITY_FILTER, VisibilityFilters } from './actions'
const { SHOW_ALL } = VisibilityFilters

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}

function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```

## 下一步

接下來會學習 [建立 Redux store](Store.md)。store 能維持應用的 state，並在當你發起 action 的時候呼叫 reducer。
