# 三大原則

Redux 可以用這三個基本原則來描述：

### 單一資料來源

**整個應用的 [state](../Glossary.md#state) 被儲存在一棵 object tree 中，並且這個 object tree 只存在於唯一一個 [store](../Glossary.md#store) 中。**

這讓同構應用開發變得非常容易。來自服務端的 state 可以在無需編寫更多程式碼的情況下被序列化並注入到客戶端中。由於是單一的 state tree ，偵錯也變得非常容易。在開發中，你可以把應用的 state 儲存在本地，從而加快開發速度。此外，受益於單一的 state tree ，以前難以實現的如“撤銷/重做”這類功能也變得輕而易舉。

```js
console.log(store.getState())

/* Prints
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
*/
```

### State 是只讀的

**惟一改變 state 的方法就是觸發 [action](../Glossary.md#action)，action 是一個用於描述已發生事件的普通物件。**

這樣確保了檢視和網路請求都不能直接修改 state，相反它們只能表達想要修改的意圖。因為所有的修改都被集中化處理，且嚴格按照一個接一個的順序執行，因此不用擔心 race condition 的出現。 Action 就是普通物件而已，因此它們可以被日誌列印、序列化、儲存、後期偵錯或測試時回放出來。

```js
store.dispatch({
  type: 'COMPLETE_TODO',
  index: 1
});

store.dispatch({
  type: 'SET_VISIBILITY_FILTER',
  filter: 'SHOW_COMPLETED'
});
```

### 使用純函數來執行修改

**為了描述 action 如何改變 state tree ，你需要編寫 [reducers](../Glossary.md#reducer)。**

Reducer 只是一些純函數，它接收先前的 state 和 action，並返回新的 state。剛開始你可以只有一個 reducer，隨著應用變大，你可以把它拆成多個小的 reducers，分別獨立地操作 state tree 的不同部分，因為 reducer 只是函數，你可以控制它們被呼叫的順序，傳入附加資料，甚至編寫可複用的 reducer 來處理一些通用任務，如分頁器。

```js

function visibilityFilter(state = 'SHOW_ALL', action) {
  switch (action.type) {
    case 'SET_VISIBILITY_FILTER':
      return action.filter
    default:
      return state
  }
}

function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case 'COMPLETE_TODO':
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: true
          })
        }
        return todo
      })
    default:
      return state
  }
}

import { combineReducers, createStore } from 'redux'
let reducer = combineReducers({ visibilityFilter, todos })
let store = createStore(reducer)
```

就是這樣，現在你應該明白 Redux 是怎麼回事了。
