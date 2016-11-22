# 非同步 Action

在[基礎教程](../basics/README.md)中，我們建立了一個簡單的 todo 應用。它只有同步操作。每當 dispatch action 時，state 會被立即更新。

在本教程中，我們將開發一個不同的，非同步的應用。它將使用 Reddit API 來獲取並顯示指定 subreddit 下的帖子列表。那麼 Redux 究竟是如何處理非同步資料流的呢？

## Action

當呼叫非同步 API 時，有兩個非常關鍵的時刻：發起請求的時刻，和接收到響應的時刻 （也可能是超時）。

這兩個時刻都可能會更改應用的 state；為此，你需要 dispatch 普通的同步 action。一般情況下，每個 API 請求都需要 dispatch 至少三種 action：

* **一種通知 reducer 請求開始的 action。**

  對於這種 action，reducer 可能會切換一下 state 中的 `isFetching` 標記。以此來告訴 UI 來顯示載入介面。

* **一種通知 reducer 請求成功的 action。**

  對於這種 action，reducer 可能會把接收到的新資料合併到 state 中，並重置 `isFetching`。UI 則會隱藏載入介面，並顯示接收到的資料。

* **一種通知 reducer 請求失敗的 action。**

  對於這種 action，reducer 可能會重置 `isFetching`。另外，有些 reducer 會儲存這些失敗資訊，並在 UI 裡顯示出來。

為了區分這三種 action，可能在 action 裡新增一個專門的 `status` 欄位作為標記位：

```js
{ type: 'FETCH_POSTS' }
{ type: 'FETCH_POSTS', status: 'error', error: 'Oops' }
{ type: 'FETCH_POSTS', status: 'success', response: { ... } }
```

又或者為它們定義不同的 type：

```js
{ type: 'FETCH_POSTS_REQUEST' }
{ type: 'FETCH_POSTS_FAILURE', error: 'Oops' }
{ type: 'FETCH_POSTS_SUCCESS', response: { ... } }
```

究竟使用帶有標記位的同一個 action，還是多個 action type 呢，完全取決於你。這應該是你的團隊共同達成的約定。使用多個 type 會降低犯錯誤的機率，但是如果你使用像 [redux-actions](https://github.com/acdlite/redux-actions) 這類的輔助庫來生成 action 建立函數和 reducer 的話，這就完全不是問題了。

無論使用哪種約定，一定要在整個應用中保持統一。
在本教程中，我們將使用不同的 type 來做。

## 同步 Action 建立函數（Action Creator）

下面先定義幾個同步的 action 類型 和 action 建立函數。比如，使用者可以選擇要顯示的 subreddit：

#### `actions.js`

```js
export const SELECT_SUBREDDIT = 'SELECT_SUBREDDIT'

export function selectSubreddit(subreddit) {
  return {
    type: SELECT_SUBREDDIT,
    subreddit
  }
}
```

也可以按 "重新整理" 按鈕來更新它：

```js
export const INVALIDATE_SUBREDDIT = 'INVALIDATE_SUBREDDIT'

export function invalidatesubreddit(subreddit) {
  return {
    type: INVALIDATE_SUBREDDIT,
    subreddit
  }
}
```

這些是使用者操作來控制的 action。也有另外一類 action，是由網路請求來控制。後面會介紹如何使用它們，現在，我們只是來定義它們。

當需要獲取指定 subreddit 的帖子的時候，需要 dispatch `REQUEST_POSTS` action：

```js
export const REQUEST_POSTS = 'REQUEST_POSTS'

export function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}
```

把 `REQUEST_POSTS` 和 `SELECT_SUBREDDIT` 或 `INVALIDATE_SUBREDDIT` 分開很重要。雖然它們的發生有先後順序，但隨著應用變得複雜，有些使用者操作（比如，預載入最流行的 subreddit，或者一段時間後自動重新整理過期資料）後需要馬上請求資料。路由變化時也可能需要請求資料，所以一開始如果把請求資料和特定的 UI 事件耦合到一起是不明智的。

最後，當收到請求響應時，我們會 dispatch `RECEIVE_POSTS`：

```js
export const RECEIVE_POSTS = 'RECEIVE_POSTS'

export function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}
```

以上就是現在需要知道的所有內容。稍後會介紹如何把 dispatch action 與網路請求結合起來。

>##### 錯誤處理須知

>在實際應用中，網路請求失敗時也需要 dispatch action。雖然在本教程中我們並不做錯誤處理，但是這個 [真實場景的案例](../introduction/Examples.html#real-world) 會演示一種實現方案。

## 設計 state 結構

就像在基礎教程中，在功能開發前你需要 [設計應用的 state 結構](../basics/Reducers.md#designing-the-state-shape)。在寫非同步程式碼的時候，需要考慮更多的 state，所以我們要仔細考慮一下。

這部分內容通常讓初學者感到迷惑，因為選擇哪些資訊才能清晰地描述非同步應用的 state 並不直觀，還有怎麼用一個樹來把這些資訊組織起來。

我們以最通用的案例來打頭：列表。Web 應用經常需要展示一些內容的列表。比如，帖子的列表，朋友的列表。首先要明確應用要顯示哪些列表。然後把它們分開儲存在 state 中，這樣你才能對它們分別做快取並且在需要的時候再次請求更新資料。

"Reddit 頭條" 應用會長這個樣子：

```js
{
  selectedsubreddit: 'frontend',
  postsBySubreddit: {
    frontend: {
      isFetching: true,
      didInvalidate: false,
      items: []
    },
    reactjs: {
      isFetching: false,
      didInvalidate: false,
      lastUpdated: 1439478405547,
      items: [
        {
          id: 42,
          title: 'Confusion about Flux and Relay'
        },
        {
          id: 500,
          title: 'Creating a Simple Application Using React JS and Flux Architecture'
        }
      ]
    }
  }
}
```

下面列出幾個要點：

* 分開儲存 subreddit 資訊，是為了快取所有 subreddit。當使用者來回切換 subreddit 時，可以立即更新，同時在不需要的時候可以不請求資料。不要擔心把所有帖子放到記憶體中（會浪費記憶體）：除非你需要處理成千上萬條帖子，同時使用者還很少關閉標籤頁，否則你不需要做任何清理。

* 每個帖子的列表都需要使用 `isFetching` 來顯示進度條，`didInvalidate` 來標記資料是否過期，`lastUpdated` 來存放資料最後更新時間，還有 `items` 存放列表資訊本身。在實際應用中，你還需要存放 `fetchedPageCount` 和 `nextPageUrl` 這樣分頁相關的 state。

>##### 巢狀內容須知

>在這個示例中，接收到的列表和分頁資訊是存在一起的。但是，這種做法並不適用於有互相引用的巢狀內容的場景，或者使用者可以編輯列表的場景。想像一下使用者需要編輯一個接收到的帖子，但這個帖子在 state tree 的多個位置重複出現。這會讓開發變得非常困難。

>如果你有巢狀內容，或者使用者可以編輯接收到的內容，你需要把它們分開存放在 state 中，就像資料庫中一樣。在分頁資訊中，只使用它們的 ID 來引用。這可以讓你始終保持資料更新。[真實場景的案例](../introduction/Examples.html#real-world) 中演示了這種做法，結合 [normalizr](https://github.com/gaearon/normalizr) 來把巢狀的 API 響應資料正規化化，最終的 state 看起來是這樣：

>```js
> {
>   selectedsubreddit: 'frontend',
>   entities: {
>     users: {
>       2: {
>         id: 2,
>         name: 'Andrew'
>       }
>     },
>     posts: {
>       42: {
>         id: 42,
>         title: 'Confusion about Flux and Relay',
>         author: 2
>       },
>       100: {
>         id: 100,
>         title: 'Creating a Simple Application Using React JS and Flux Architecture',
>         author: 2
>       }
>     }
>   },
>   postsBySubreddit: {
>     frontend: {
>       isFetching: true,
>       didInvalidate: false,
>       items: []
>     },
>     reactjs: {
>       isFetching: false,
>       didInvalidate: false,
>       lastUpdated: 1439478405547,
>       items: [ 42, 100 ]
>     }
>   }
> }
>```

>在本教程中，我們不會對內容進行正規化化，但是在一個複雜些的應用中你可能需要使用。

## 處理 Action

在講 dispatch action 與網路請求結合使用細節前，我們為上面定義的 action 開發一些 reducer。

>##### Reducer 組合須知

>這裡，我們假設你已經學習過 [`combineReducers()`](../api/combineReducers.md) 並理解 reducer 組合，還有 [基礎章節](../basics/README.md) 中的 [拆分 Reducer](../basics/Reducers.md#splitting-reducers)。如果還沒有，請[先學習](../basics/Reducers.md#splitting-reducers)。

#### `reducers.js`

```js
import { combineReducers } from 'redux'
import {
  SELECT_SUBREDDIT, INVALIDATE_SUBREDDIT,
  REQUEST_POSTS, RECEIVE_POSTS
} from '../actions'

function selectedsubreddit(state = 'reactjs', action) {
  switch (action.type) {
    case SELECT_SUBREDDIT:
      return action.subreddit
    default:
      return state
  }
}

function posts(state = {
  isFetching: false,
  didInvalidate: false,
  items: []
}, action) {
  switch (action.type) {
    case INVALIDATE_SUBREDDIT:
      return Object.assign({}, state, {
        didInvalidate: true
      })
    case REQUEST_POSTS:
      return Object.assign({}, state, {
        isFetching: true,
        didInvalidate: false
      })
    case RECEIVE_POSTS:
      return Object.assign({}, state, {
        isFetching: false,
        didInvalidate: false,
        items: action.posts,
        lastUpdated: action.receivedAt
      })
    default:
      return state
  }
}

function postsBySubreddit(state = {}, action) {
  switch (action.type) {
    case INVALIDATE_SUBREDDIT:
    case RECEIVE_POSTS:
    case REQUEST_POSTS:
      return Object.assign({}, state, {
        [action.subreddit]: posts(state[action.subreddit], action)
      })
    default:
      return state
  }
}

const rootReducer = combineReducers({
  postsBySubreddit,
  selectedsubreddit
})

export default rootReducer
```

上面程式碼有兩個有趣的點：

* 使用 ES6 計算屬性語法，使用 `Object.assign()` 來簡潔高效地更新 `state[action.subreddit]`。這個：

  ```js
  return Object.assign({}, state, {
    [action.subreddit]: posts(state[action.subreddit], action)
  })
  ```
  與下面程式碼等價：

  ```js
  let nextState = {}
  nextState[action.subreddit] = posts(state[action.subreddit], action)
  return Object.assign({}, state, nextState)
  ```

* 我們提取出 `posts(state, action)` 來管理指定帖子列表的 state。這僅僅使用 [reducer 組合](../basics/Reducers.md#splitting-reducers)而已！我們還可以藉此機會把 reducer 分拆成更小的 reducer，這種情況下，我們把物件內列表的更新代理到了 `posts` reducer 上。在[真實場景的案例](../introduction/Examples.html#real-world)中甚至更進一步，裡面介紹瞭如何做一個 reducer 工廠來生成參數化的分頁 reducer。

記住 reducer 只是函數而已，所以你可以盡情使用函陣列合和高階函數這些特性。

## 非同步 action 建立函數

最後，如何把[之前定義](#synchronous-action-creators)的同步 action 建立函數和 網路請求結合起來呢？標準的做法是使用 [Redux Thunk middleware](https://github.com/gaearon/redux-thunk)。要引入 `redux-thunk` 這個專門的庫才能使用。我們[後面](Middleware.md)會介紹 middleware 大體上是如何工作的；目前，你只需要知道一個要點：通過使用指定的 middleware，action 建立函數除了返回 action 物件外還可以返回函數。這時，這個 action 建立函數就成為了 [thunk](https://en.wikipedia.org/wiki/Thunk)。

當 action 建立函數返回函數時，這個函數會被 Redux Thunk middleware 執行。這個函數並不需要保持純淨；它還可以帶有副作用，包括執行非同步 API 請求。這個函數還可以 dispatch action，就像 dispatch 前面定義的同步 action 一樣。

我們仍可以在 `actions.js` 裡定義這些特殊的 thunk action 建立函數。

#### `actions.js`

```js
import fetch from 'isomorphic-fetch'

export const REQUEST_POSTS = 'REQUEST_POSTS'
function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}

export const RECEIVE_POSTS = 'RECEIVE_POSTS'
function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}

// 來看一下我們寫的第一個 thunk action 建立函數！
// 雖然內部操作不同，你可以像其它 action 建立函數 一樣使用它：
// store.dispatch(fetchPosts('reactjs'))

export function fetchPosts(subreddit) {

  // Thunk middleware 知道如何處理函數。
  // 這裡把 dispatch 方法通過參數的形式傳給函數，
  // 以此來讓它自己也能 dispatch action。

  return function (dispatch) {

    // 首次 dispatch：更新應用的 state 來通知
    // API 請求發起了。

    dispatch(requestPosts(subreddit))

    // thunk middleware 呼叫的函數可以有返回值，
    // 它會被當作 dispatch 方法的返回值傳遞。

    // 這個案例中，我們返回一個等待處理的 promise。
    // 這並不是 redux middleware 所必須的，但這對於我們而言很方便。

    return fetch(`http://www.subreddit.com/r/${subreddit}.json`)
      .then(response => response.json())
      .then(json =>

        // 可以多次 dispatch！
        // 這裡，使用 API 請求結果來更新應用的 state。

        dispatch(receivePosts(subreddit, json))
      )

      // 在實際應用中，還需要
      // 捕獲網路請求的異常。
  }
}
```

>##### `fetch` 使用須知

>本示例使用了 [`fetch` API](https://developer.mozilla.org/en/docs/Web/API/Fetch_API)。它是替代 `XMLHttpRequest` 用來傳送網路請求的非常新的 API。由於目前大多數瀏覽器原生還不支援它，建議你使用 [`isomorphic-fetch`](https://github.com/matthew-andrews/isomorphic-fetch) 庫：

>```js
// 每次使用 `fetch` 前都這樣呼叫一下
>import fetch from 'isomorphic-fetch'
>```

>在底層，它在瀏覽器端使用 [`whatwg-fetch` polyfill](https://github.com/github/fetch)，在伺服器端使用 [`node-fetch`](https://github.com/bitinn/node-fetch)，所以如果當你把應用改成[同構](https://medium.com/@mjackson/universal-javascript-4761051b7ae9)時，並不需要改變 API 請求。

>注意，`fetch` polyfill 假設你已經使用了 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 的 polyfill。確保你使用 Promise polyfill 的一個最簡單的辦法是在所有應用程式碼前啟用 Babel 的 ES6 polyfill：

>```js
>// 在應用中其它任何程式碼執行前呼叫一次
>import 'babel-polyfill'
>```

我們是如何在 dispatch 機制中引入 Redux Thunk middleware 的呢？我們使用了 [`applyMiddleware()`](../api/applyMiddleware.md)，如下：

#### `index.js`

```js
import thunkMiddleware from 'redux-thunk'
import createLogger from 'redux-logger'
import { createStore, applyMiddleware } from 'redux'
import { selectSubreddit, fetchPosts } from './actions'
import rootReducer from './reducers'

const loggerMiddleware = createLogger()

const store = createStore(
  rootReducer,
  applyMiddleware(
    thunkMiddleware, // 允許我們 dispatch() 函數
    loggerMiddleware // 一個很便捷的 middleware，用來列印 action 日誌
  )
)

store.dispatch(selectSubreddit('reactjs'))
store.dispatch(fetchPosts('reactjs')).then(() =>
  console.log(store.getState())
)
```

thunk 的一個優點是它的結果可以再次被 dispatch：

#### `actions.js`

```js
import fetch from 'isomorphic-fetch'

export const REQUEST_POSTS = 'REQUEST_POSTS'
function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}

export const RECEIVE_POSTS = 'RECEIVE_POSTS'
function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}

function fetchPosts(subreddit) {
  return dispatch => {
    dispatch(requestPosts(subreddit))
    return fetch(`http://www.reddit.com/r/${subreddit}.json`)
      .then(response => response.json())
      .then(json => dispatch(receivePosts(subreddit, json)))
  }
}

function shouldFetchPosts(state, subreddit) {
  const posts = state.postsBySubreddit[subreddit]
  if (!posts) {
    return true
  } else if (posts.isFetching) {
    return false
  } else {
    return posts.didInvalidate
  }
}

export function fetchPostsIfNeeded(subreddit) {

  // 注意這個函數也接收了 getState() 方法
  // 它讓你選擇接下來 dispatch 什麼。

  // 當快取的值是可用時，
  // 減少網路請求很有用。

  return (dispatch, getState) => {
    if (shouldFetchPosts(getState(), subreddit)) {
      // 在 thunk 裡 dispatch 另一個 thunk！
      return dispatch(fetchPosts(subreddit))
    } else {
      // 告訴呼叫程式碼不需要再等待。
      return Promise.resolve()
    }
  }
}
```

這可以讓我們逐步開發複雜的非同步控制流，同時保持程式碼整潔如初：

#### `index.js`

```js
store.dispatch(fetchPostsIfNeeded('reactjs')).then(() =>
  console.log(store.getState())
)
```

>##### 服務端渲染須知

>非同步 action 建立函數對於做服務端渲染非常方便。你可以建立一個 store，dispatch 一個非同步 action 建立函數，這個 action 建立函數又 dispatch 另一個非同步 action 建立函數來為應用的一整塊請求資料，同時在 Promise 完成和結束時才 render 介面。然後在 render 前，store 裡就已經存在了需要用的 state。

[Thunk middleware](https://github.com/gaearon/redux-thunk) 並不是 Redux 處理非同步 action 的唯一方式：
* 你可以使用 [redux-promise](https://github.com/acdlite/redux-promise) 或者 [redux-promise-middleware](https://github.com/pburtchaell/redux-promise-middleware) 來 dispatch Promise 替代函數。
* 你可以使用 [redux-observable](https://github.com/redux-observable/redux-observable) 來 dispatch Observable。
* 你可以使用 [redux-saga](https://github.com/yelouafi/redux-saga/) 中介軟體來建立更加複雜的非同步 action。
* 你甚至可以寫一個自定義的 middleware 來描述 API 請求，就像這個[真實場景的案例](../introduction/Examples.html#real-world)中的做法一樣。

你也可以先嚐試一些不同做法，選擇喜歡的，並使用下去，不論有沒有使用到 middleware 都行。

## 連線到 UI

Dispatch 同步 action 與非同步 action 間並沒有區別，所以就不展開討論細節了。參照 [搭配 React](../basics/UsageWithReact.md) 獲得 React 元件中使用 Redux 的介紹。參照 [示例：Reddit API](ExampleRedditAPI.md) 來獲取本例的完整程式碼。

## 下一步

閱讀 [非同步資料流](AsyncFlow.md) 來整理一下非同步 action 是如何適用於 Redux 資料流的。
