# Middleware

我們已經在[非同步 Action ](../advanced/AsyncActions.md)一節的示例中看到了一些 middleware 的使用。如果你使用過 [Express](http://expressjs.com/) 或者 [Koa](http://koajs.com/) 等服務端框架, 那麼應該對 *middleware* 的概念不會陌生。 在這類框架中，middleware 是指可以被嵌入在框架接收請求到產生響應過程之中的程式碼。例如，Express 或者 Koa 的 middleware 可以完成新增 CORS headers、記錄日誌、內容壓縮等工作。middleware 最優秀的特性就是可以被鏈式組合。你可以在一個項目中使用多個獨立的第三方 middleware。

相對於 Express 或者 Koa 的 middleware，Redux middleware 被用於解決不同的問題，但其中的概念是類似的。**它提供的是位於 action 被髮起之後，到達 reducer 之前的擴充套件點。** 你可以利用 Redux middleware 來進行日誌記錄、建立崩潰報告、呼叫非同步介面或者路由等等。

這個章節分為兩個部分，前面是幫助你理解相關概念的深度介紹，而後半部分則通過[一些例項](#seven-examples)來體現 middleware 的強大能力。對文章前後內容進行結合通讀，會幫助你更好的理解枯燥的概念，並從中獲得啟發。

## 理解 Middleware

正因為 middleware 可以完成包括非同步 API 呼叫在內的各種事情，瞭解它的演化過程是一件相當重要的事。我們將以記錄日誌和建立崩潰報告為例，引導你體會從分析問題到通過構建 middleware 解決問題的思維過程。

### 問題: 記錄日誌

使用 Redux 的一個益處就是它讓 state 的變化過程變的可預知和透明。每當一個 action 發起完成後，新的 state 就會被計算並儲存下來。State 不能被自身修改，只能由特定的 action 引起變化。

試想一下，當我們的應用中每一個 action 被髮起以及每次新的 state 被計算完成時都將它們記錄下來，豈不是很好？當程式出現問題時，我們可以通過查閱日誌找出是哪個 action 導致了 state 不正確。

<img src='http://i.imgur.com/BjGBlES.png' width='70%'>

我們如何通過 Redux 實現它呢？

### 嘗試 #1: 手動記錄

最直接的解決方案就是在每次呼叫 [`store.dispatch(action)`](../api/Store.md#dispatch) 前後手動記錄被髮起的 action 和新的 state。這稱不上一個真正的解決方案，僅僅是我們理解這個問題的第一步。

>##### 注意

>如果你使用 [react-redux](https://github.com/gaearon/react-redux) 或者類似的繫結庫，最好不要直接在你的元件中操作 store 的例項。在接下來的內容中，僅僅是假設你會通過 store 顯式地向下傳遞。

假設，你在建立一個 Todo 時這樣呼叫：

```js
store.dispatch(addTodo('Use Redux'))
```

為了記錄這個 action 以及產生的新的 state，你可以通過這種方式記錄日誌：

```js
let action = addTodo('Use Redux')

console.log('dispatching', action)
store.dispatch(action)
console.log('next state', store.getState())
```

雖然這樣做達到了想要的效果，但是你並不想每次都這麼幹。

### 嘗試 #2: 封裝 Dispatch

你可以將上面的操作抽取成一個函數：

```js
function dispatchAndLog(store, action) {
  console.log('dispatching', action)
  store.dispatch(action)
  console.log('next state', store.getState())
}
```

然後用它替換 `store.dispatch()`:

```js
dispatchAndLog(store, addTodo('Use Redux'))
```
你可以選擇到此為止，但是每次都要匯入一個外部方法總歸還是不太方便。

### 嘗試 #3: Monkeypatching Dispatch

如果我們直接替換 store 例項中的 `dispatch` 函數會怎麼樣呢？Redux store 只是一個包含[一些方法](../api/Store.md)的普通物件，同時我們使用的是 JavaScript，因此我們可以這樣實現 `dispatch` 的 monkeypatch：

```js
let next = store.dispatch
store.dispatch = function dispatchAndLog(action) {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}
```

這離我們想要的已經非常接近了！無論我們在哪裡發起 action，保證都會被記錄。Monkeypatching 令人感覺還是不太舒服，不過利用它我們做到了我們想要的。

### 問題: 崩潰報告

如果我們想對 `dispatch` 附加**超過一個**的變換，又會怎麼樣呢？

我腦海中出現的另一個常用的變換就是在生產過程中報告 JavaScript 的錯誤。全局的 `window.onerror` 並不可靠，因為它在一些舊的瀏覽器中無法提供錯誤堆棧，而這是排查錯誤所需的至關重要資訊。

試想當發起一個 action 的結果是一個異常時，我們將包含呼叫堆棧，引起錯誤的 action 以及當前的 state 等錯誤資訊通通發到類似於 [Sentry](https://getsentry.com/welcome/) 這樣的報告服務中，不是很好嗎？這樣我們可以更容易地在開發環境中重現這個錯誤。

然而，將日誌記錄和崩潰報告分離是很重要的。理想情況下，我們希望他們是兩個不同的模組，也可能在不同的包中。否則我們無法構建一個由這些工具組成的生態系統。（提示：我們正在慢慢了解 middleware 的本質到底是什麼！）

如果按照我們的想法，日誌記錄和崩潰報告屬於不同的模組，他們看起來應該像這樣：

```js
function patchStoreToAddLogging(store) {
  let next = store.dispatch
  store.dispatch = function dispatchAndLog(action) {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
  }
}

function patchStoreToAddCrashReporting(store) {
  let next = store.dispatch
  store.dispatch = function dispatchAndReportErrors(action) {
    try {
      return next(action)
    } catch (err) {
      console.error('捕獲一個異常!', err)
      Raven.captureException(err, {
        extra: {
          action,
          state: store.getState()
        }
      })
      throw err
    }
  }
}
```

如果這些功能以不同的模組釋出，我們可以在 store 中像這樣使用它們：

```js
patchStoreToAddLogging(store)
patchStoreToAddCrashReporting(store)
```

儘管如此，這種方式看起來還是不是夠令人滿意。

### 嘗試 #4: 隱藏 Monkeypatching

Monkeypatching 本質上是一種 hack。“將任意的方法替換成你想要的”，此時的 API 會是什麼樣的呢？現在，讓我們來看看這種替換的本質。 在之前，我們用自己的函數替換掉了 `store.dispatch`。如果我們不這樣做，而是在函數中**返回**新的 `dispatch` 呢？

```js
function logger(store) {
  let next = store.dispatch

  // 我們之前的做法:
  // store.dispatch = function dispatchAndLog(action) {

  return function dispatchAndLog(action) {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
  }
}
```

我們可以在 Redux 內部提供一個可以將實際的 monkeypatching 應用到 `store.dispatch` 中的輔助方法：

```js
function applyMiddlewareByMonkeypatching(store, middlewares) {
  middlewares = middlewares.slice()
  middlewares.reverse()

  // 在每一個 middleware 中變換 dispatch 方法。
  middlewares.forEach(middleware =>
    store.dispatch = middleware(store)
  )
}
```

然後像這樣應用多個 middleware：

```js
applyMiddlewareByMonkeypatching(store, [ logger, crashReporter ])
```

儘管我們做了很多，實現方式依舊是 monkeypatching。
因為我們僅僅是將它隱藏在我們的框架內部，並沒有改變這個事實。

### 嘗試 #5: 移除 Monkeypatching

為什麼我們要替換原來的 `dispatch` 呢？當然，這樣我們就可以在後面直接呼叫它，但是還有另一個原因：就是每一個 middleware 都可以操作（或者直接呼叫）前一個 middleware 包裝過的 `store.dispatch`：

```js
function logger(store) {
  // 這裡的 next 必須指向前一個 middleware 返回的函數：
  let next = store.dispatch

  return function dispatchAndLog(action) {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
  }
}
```

將 middleware 串連起來的必要性是顯而易見的。

如果 `applyMiddlewareByMonkeypatching` 方法中沒有在第一個 middleware 執行時立即替換掉 `store.dispatch`，那麼 `store.dispatch` 將會一直指向原始的 `dispatch` 方法。也就是說，第二個 middleware 依舊會作用在原始的 `dispatch` 方法。

但是，還有另一種方式來實現這種鏈式呼叫的效果。可以讓 middleware 以方法參數的形式接收一個 `next()` 方法，而不是通過 store 的例項去獲取。

```js
function logger(store) {
  return function wrapDispatchToAddLogging(next) {
    return function dispatchAndLog(action) {
      console.log('dispatching', action)
      let result = next(action)
      console.log('next state', store.getState())
      return result
    }
  }
}
```
現在是[“我們該更進一步”](http://knowyourmeme.com/memes/we-need-to-go-deeper)的時刻了，所以可能會多花一點時間來讓它變的更為合理一些。這些串聯函數很嚇人。ES6 的箭頭函數可以使其 [柯里化](https://en.wikipedia.org/wiki/Currying) ，從而看起來更舒服一些:

```js
const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}

const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}
```

**這正是 Redux middleware 的樣子。**

Middleware 接收了一個 `next()` 的 dispatch 函數，並返回一個 dispatch 函數，返回的函數會被作為下一個 middleware 的 `next()`，以此類推。由於 store 中類似 `getState()` 的方法依舊非常有用，我們將 `store` 作為頂層的參數，使得它可以在所有 middleware 中被使用。

### 嘗試 #6: “單純”地使用 Middleware

我們可以寫一個 `applyMiddleware()` 方法替換掉原來的 `applyMiddlewareByMonkeypatching()`。在新的 `applyMiddleware()` 中，我們取得最終完整的被包裝過的 `dispatch()` 函數，並返回一個 store 的副本：

```js
// 警告：這只是一種“單純”的實現方式！
// 這 *並不是* Redux 的 API.

function applyMiddleware(store, middlewares) {
  middlewares = middlewares.slice()
  middlewares.reverse()

  let dispatch = store.dispatch
  middlewares.forEach(middleware =>
    dispatch = middleware(store)(dispatch)
  )

  return Object.assign({}, store, { dispatch })
}
```

這與 Redux 中 [`applyMiddleware()`](../api/applyMiddleware.md) 的實現已經很接近了，但是**有三個重要的不同之處**：

* 它只暴露一個 [store API](../api/Store.md) 的子集給 middleware：[`dispatch(action)`](../api/Store.md#dispatch) 和 [`getState()`](../api/Store.md#getState)。

* 它用了一個非常巧妙的方式來保證你的 middleware 呼叫的是 `store.dispatch(action)` 而不是 `next(action)`，從而使這個 action 會在包括當前 middleware 在內的整個 middleware 鏈中被正確的傳遞。這對非同步的 middleware 非常有用，正如我們在[之前的章節](AsyncActions.md)中提到的。

* 為了保證你只能應用 middleware 一次，它作用在 `createStore()` 上而不是 `store` 本身。因此它的簽名不是 `(store, middlewares) => store`， 而是 `(...middlewares) => (createStore) => createStore`。

### 最終的方法

這是我們剛剛所寫的 middleware：

```js
const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}

const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}
```

然後是將它們引用到 Redux store 中：

```js
import { createStore, combineReducers, applyMiddleware } from 'redux'

// applyMiddleware 接收 createStore()
// 並返回一個包含相容 API 的函數。
let createStoreWithMiddleware = applyMiddleware(logger, crashReporter)(createStore)

// 像使用 createStore() 一樣使用它。
let todoApp = combineReducers(reducers)
let store = createStoreWithMiddleware(todoApp)
```

就是這樣！現在任何被髮送到 store 的 action 都會經過 `logger` 和 `crashReporter`：

```js
// 將經過 logger 和 crashReporter 兩個 middleware！
store.dispatch(addTodo('Use Redux'))
```

## 7個示例

如果讀完上面的章節你已經覺得頭都要爆了，那就想象一下把它寫出來之後的樣子。下面的內容會讓我們放鬆一下，並讓你的思路延續。

下面的每個函數都是一個有效的 Redux middleware。它們不是同樣有用，但是至少他們一樣有趣。

```js
/**
 * 記錄所有被髮起的 action 以及產生的新的 state。
 */
const logger = store => next => action => {
  console.group(action.type)
  console.info('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  console.groupEnd(action.type)
  return result
}

/**
 * 在 state 更新完成和 listener 被通知之後傳送崩潰報告。
 */
const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}

/**
 * 用 { meta: { delay: N } } 來讓 action 延遲 N 毫秒。
 * 在這個案例中，讓 `dispatch` 返回一個取消 timeout 的函數。
 */
const timeoutScheduler = store => next => action => {
  if (!action.meta || !action.meta.delay) {
    return next(action)
  }

  let timeoutId = setTimeout(
    () => next(action),
    action.meta.delay
  )

  return function cancel() {
    clearTimeout(timeoutId)
  }
}

/**
 * 通過 { meta: { raf: true } } 讓 action 在一個 rAF 迴圈幀中被髮起。
 * 在這個案例中，讓 `dispatch` 返回一個從佇列中移除該 action 的函數。
 */
const rafScheduler = store => next => {
  let queuedActions = []
  let frame = null

  function loop() {
    frame = null
    try {
      if (queuedActions.length) {
        next(queuedActions.shift())
      }
    } finally {
      maybeRaf()
    }
  }

  function maybeRaf() {
    if (queuedActions.length && !frame) {
      frame = requestAnimationFrame(loop)
    }
  }

  return action => {
    if (!action.meta || !action.meta.raf) {
      return next(action)
    }

    queuedActions.push(action)
    maybeRaf()

    return function cancel() {
      queuedActions = queuedActions.filter(a => a !== action)
    }
  }
}

/**
 * 使你除了 action 之外還可以發起 promise。
 * 如果這個 promise 被 resolved，他的結果將被作為 action 發起。
 * 這個 promise 會被 `dispatch` 返回，因此呼叫者可以處理 rejection。
 */
const vanillaPromise = store => next => action => {
  if (typeof action.then !== 'function') {
    return next(action)
  }

  return Promise.resolve(action).then(store.dispatch)
}

/**
 * 讓你可以發起帶有一個 { promise } 屬性的特殊 action。
 *
 * 這個 middleware 會在開始時發起一個 action，並在這個 `promise` resolve 時發起另一個成功（或失敗）的 action。
 *
 * 為了方便起見，`dispatch` 會返回這個 promise 讓呼叫者可以等待。
 */
const readyStatePromise = store => next => action => {
  if (!action.promise) {
    return next(action)
  }

  function makeAction(ready, data) {
    let newAction = Object.assign({}, action, { ready }, data)
    delete newAction.promise
    return newAction
  }

  next(makeAction(false))
  return action.promise.then(
    result => next(makeAction(true, { result })),
    error => next(makeAction(true, { error }))
  )
}

/**
 * 讓你可以發起一個函數來替代 action。
 * 這個函數接收 `dispatch` 和 `getState` 作為參數。
 *
 * 對於（根據 `getState()` 的情況）提前退出，或者非同步控制流（ `dispatch()` 一些其他東西）來說，這非常有用。
 *
 * `dispatch` 會返回被髮起函數的返回值。
 */
const thunk = store => next => action =>
  typeof action === 'function' ?
    action(store.dispatch, store.getState) :
    next(action)

// 你可以使用以上全部的 middleware！（當然，這不意味著你必須全都使用。）
let createStoreWithMiddleware = applyMiddleware(
  rafScheduler,
  timeoutScheduler,
  thunk,
  vanillaPromise,
  readyStatePromise,
  logger,
  crashReporter
)(createStore)
let todoApp = combineReducers(reducers)
let store = createStoreWithMiddleware(todoApp)
```
