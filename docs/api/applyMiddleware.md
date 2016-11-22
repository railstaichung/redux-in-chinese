# `applyMiddleware(...middlewares)`

使用包含自定義功能的 middleware 來擴充套件 Redux 是一種推薦的方式。Middleware 可以讓你包裝 store 的 [`dispatch`](Store.md#dispatch) 方法來達到你想要的目的。同時， middleware 還擁有“可組合”這一關鍵特性。多個 middleware 可以被組合到一起使用，形成 middleware 鏈。其中，每個 middleware 都不需要關心鏈中它前後的 middleware 的任何資訊。

Middleware 最常見的使用場景是無需引用大量程式碼或依賴類似 [Rx](https://github.com/Reactive-Extensions/RxJS) 的第三方庫實現非同步 actions。這種方式可以讓你像 dispatch 一般的 actions 那樣 dispatch [非同步 actions](../Glossary.md#async-action)。

例如，[redux-thunk](https://github.com/gaearon/redux-thunk) 支援 dispatch function，以此讓 action creator 控制反轉。被 dispatch 的 function 會接收 [`dispatch`](Store.md#dispatch) 作為參數，並且可以非同步呼叫它。這類的 function 就稱為 *thunk*。另一個 middleware 的示例是 [redux-promise](https://github.com/acdlite/redux-promise)。它支援 dispatch 一個非同步的 [Promise](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise) action，並且在 Promise resolve 後可以 dispatch 一個普通的 action。

Middleware 並不需要和 [`createStore`](createStore.md) 綁在一起使用，也不是 Redux 架構的基礎組成部分，但它帶來的益處讓我們認為有必要在 Redux 核心中包含對它的支援。因此，雖然不同的 middleware 可能在易用性和用法上有所不同，它仍被作為擴充套件 [`dispatch`](Store.md#dispatch) 的唯一標準的方式。

#### 參數

* `...middlewares` (*arguments*): 遵循 Redux *middleware API* 的函數。每個 middleware 接受 [`Store`](Store.md) 的 [`dispatch`](Store.md#dispatch) 和 [`getState`](Store.md#getState) 函數作為命名參數，並返回一個函數。該函數會被傳入
被稱為 `next` 的下一個 middleware 的 dispatch 方法，並返回一個接收 action 的新函數，這個函數可以直接呼叫 `next(action)`，或者在其他需要的時刻呼叫，甚至根本不去呼叫它。呼叫鏈中最後一個 middleware 會接受真實的 store 的 [`dispatch`](Store.md#dispatch) 方法作為 `next` 參數，並藉此結束呼叫鏈。所以，middleware 的函數簽名是 `({ getState, dispatch }) => next => action`。

#### 返回值

(*Function*) 一個應用了 middleware 後的 store enhancer。這個 store enhancer 的簽名是 `createStore => createStore`，但是最簡單的使用方法就是直接作為最後一個 `enhancer` 參數傳遞給 [`createStore()`](createStore.md) 函數。

#### 示例: 自定義 Logger Middleware

```js
import { createStore, applyMiddleware } from 'redux'
import todos from './reducers'

function logger({ getState }) {
  return (next) => (action) => {
    console.log('will dispatch', action)

    // 呼叫 middleware 鏈中下一個 middleware 的 dispatch。
    let returnValue = next(action)

    console.log('state after dispatch', getState())

    // 一般會是 action 本身，除非
    // 後面的 middleware 修改了它。
    return returnValue
  }
}

let store = createStore(
  todos,
  [ 'Use Redux' ],
  applyMiddleware(logger)
)

store.dispatch({
  type: 'ADD_TODO',
  text: 'Understand the middleware'
})
// (將列印如下資訊:)
// will dispatch: { type: 'ADD_TODO', text: 'Understand the middleware' }
// state after dispatch: [ 'Use Redux', 'Understand the middleware' ]
```

#### 示例: 使用 Thunk Middleware 來做非同步 Action

```js
import { createStore, combineReducers, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'
import * as reducers from './reducers'

let reducer = combineReducers(reducers)
// applyMiddleware 為 createStore 注入了 middleware:
let store = createStore(reducer, applyMiddleware(thunk))

function fetchSecretSauce() {
  return fetch('https://www.google.com/search?q=secret+sauce')
}

// 這些是你已熟悉的普通 action creator。
// 它們返回的 action 不需要任何 middleware 就能被 dispatch。
// 但是，他們只表達「事實」，並不表達「非同步資料流」

function makeASandwich(forPerson, secretSauce) {
  return {
    type: 'MAKE_SANDWICH',
    forPerson,
    secretSauce
  }
}

function apologize(fromPerson, toPerson, error) {
  return {
    type: 'APOLOGIZE',
    fromPerson,
    toPerson,
    error
  }
}

function withdrawMoney(amount) {
  return {
    type: 'WITHDRAW',
    amount
  }
}

// 即使不使用 middleware，你也可以 dispatch action：
store.dispatch(withdrawMoney(100))

// 但是怎樣處理非同步 action 呢，
// 比如 API 呼叫，或者是路由跳轉？

// 來看一下 thunk。
// Thunk 就是一個返回函數的函數。
// 下面就是一個 thunk。

function makeASandwichWithSecretSauce(forPerson) {

  // 控制反轉！
  // 返回一個接收 `dispatch` 的函數。
  // Thunk middleware 知道如何把非同步的 thunk action 轉為普通 action。

  return function (dispatch) {
    return fetchSecretSauce().then(
      sauce => dispatch(makeASandwich(forPerson, sauce)),
      error => dispatch(apologize('The Sandwich Shop', forPerson, error))
    )
  }
}

// Thunk middleware 可以讓我們像 dispatch 普通 action
// 一樣 dispatch 非同步的 thunk action。

store.dispatch(
  makeASandwichWithSecretSauce('Me')
)

// 它甚至負責回傳 thunk 被 dispatch 後返回的值，
// 所以可以繼續串連 Promise，呼叫它的 .then() 方法。

store.dispatch(
  makeASandwichWithSecretSauce('My wife')
).then(() => {
  console.log('Done!')
})

// 實際上，可以寫一個 dispatch 其它 action creator 裡
// 普通 action 和非同步 action 的 action creator，
// 而且可以使用 Promise 來控制資料流。

function makeSandwichesForEverybody() {
  return function (dispatch, getState) {
    if (!getState().sandwiches.isShopOpen) {

      // 返回 Promise 並不是必須的，但這是一個很好的約定，
      // 為了讓呼叫者能夠在非同步的 dispatch 結果上直接呼叫 .then() 方法。

      return Promise.resolve()
    }

    // 可以 dispatch 普通 action 物件和其它 thunk，
    // 這樣我們就可以在一個資料流中組合多個非同步 action。

    return dispatch(
      makeASandwichWithSecretSauce('My Grandma')
    ).then(() =>
      Promise.all([
        dispatch(makeASandwichWithSecretSauce('Me')),
        dispatch(makeASandwichWithSecretSauce('My wife'))
      ])
    ).then(() =>
      dispatch(makeASandwichWithSecretSauce('Our kids'))
    ).then(() =>
      dispatch(getState().myMoney > 42 ?
        withdrawMoney(42) :
        apologize('Me', 'The Sandwich Shop')
      )
    )
  }
}

// 這在服務端渲染時很有用，因為我可以等到資料
// 準備好後，同步的渲染應用。

import { renderToString } from 'react-dom/server'

store.dispatch(
  makeSandwichesForEverybody()
).then(() =>
  response.send(renderToString(<MyApp store={store} />))
)

// 也可以在任何導致元件的 props 變化的時刻
// dispatch 一個非同步 thunk action。

import { connect } from 'react-redux'
import { Component } from 'react'

class SandwichShop extends Component {
  componentDidMount() {
    this.props.dispatch(
      makeASandwichWithSecretSauce(this.props.forPerson)
    )
  }

  componentWillReceiveProps(nextProps) {
    if (nextProps.forPerson !== this.props.forPerson) {
      this.props.dispatch(
        makeASandwichWithSecretSauce(nextProps.forPerson)
      )
    }
  }

  render() {
    return <p>{this.props.sandwiches.join('mustard')}</p>
  }
}

export default connect(
  state => ({
    sandwiches: state.sandwiches
  })
)(SandwichShop)
```

#### 小貼士

* Middleware 只是包裝了 store 的 [`dispatch`](Store.md#dispatch) 方法。技術上講，任何 middleware 能做的事情，都可能通過手動包裝 `dispatch` 呼叫來實現，但是放在同一個地方統一管理會使整個項目的擴充套件變的容易得多。

* 如果除了 `applyMiddleware`，你還用了其它 store enhancer，一定要把 `applyMiddleware` 放到組合鏈的前面，因為 middleware 可能會包含非同步操作。比如，它應該在 [redux-devtools](https://github.com/gaearon/redux-devtools) 前面，否則 DevTools 就看不到 Promise middleware 裡 dispatch 的 action 了。

* 如果你想有條件地使用 middleware，記住只 import 需要的部分：

  ```js
  let middleware = [ a, b ]
  if (process.env.NODE_ENV !== 'production') {
    let c = require('some-debug-middleware')
    let d = require('another-debug-middleware')
    middleware = [ ...middleware, c, d ]
  }

  const store = createStore(
    reducer,
    preloadedState,
    applyMiddleware(...middleware)
  )
  ```

  這樣做有利於打包時去掉不需要的模組，減小打包檔案大小。

* 有想過 `applyMiddleware` 本質是什麼嗎？它肯定是比 middleware 還強大的擴充套件機制。實際上，`applyMiddleware` 只是被稱為 Redux 最強大的擴充套件機制的 [store enhancer](../Glossary.md#store-enhancer) 中的一個範例而已。你不太可能需要實現自己的 store enhancer。另一個 store enhancer 示例是 [redux-devtools](https://github.com/gaearon/redux-devtools)。Middleware 並沒有 store enhancer 強大，但開發起來卻是更容易的。

* Middleware 聽起來比實際難一些。真正理解 middleware 的唯一辦法是瞭解現有的 middleware 是如何工作的，並嘗試自己實現。需要的功能可能錯綜複雜，但是你會發現大部分 middleware 實際上很小，只有 10 行左右，是通過對它們的組合使用來達到最終的目的。

* 想要使用多個 store enhancer，可以使用 [`compose()`](./compose.md) 方法。
