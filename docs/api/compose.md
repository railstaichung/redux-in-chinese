# `compose(...functions)`

從右到左來組合多個函數。

這是函數語言程式設計中的方法，為了方便，被放到了 Redux 裡。
當需要把多個 [store 增強器](../Glossary.md#store-enhancer) 依次執行的時候，需要用到它。

#### 參數

1. (*arguments*): 需要合成的多個函數。每個函數都接收一個函數作為參數，然後返回一個函數。

#### 返回值

(*Function*): 從右到左把接收到的函數合成後的最終函數。

#### 示例

下面示例演示瞭如何使用 `compose` 增強 [store](Store.md)，這個 store 與 [`applyMiddleware`](applyMiddleware.md) 和 [redux-devtools](https://github.com/gaearon/redux-devtools) 一起使用。

```js
import { createStore, combineReducers, applyMiddleware, compose } from 'redux';
import thunk from 'redux-thunk';
import * as reducers from '../reducers/index';

let reducer = combineReducers(reducers);
let middleware = [thunk];

let finalCreateStore;

// 生產環境中，我們希望只使用 middleware。
// 而在開發環境中，我們還希望使用一些 redux-devtools 提供的一些 store 增強器。
// UglifyJS 會在構建過程中把一些不會執行的死程式碼去除掉。

if (process.env.NODE_ENV === 'production') {
  finalCreateStore = applyMiddleware(...middleware)(createStore);
} else {
  finalCreateStore = compose(
    applyMiddleware(...middleware),
    require('redux-devtools').devTools(),
    require('redux-devtools').persistState(
      window.location.href.match(/[?&]debug_session=([^&]+)\b/)
    ),
    createStore
  );

  // 不使用 compose 來寫是這樣子：
  //
  // finalCreateStore =
  //   applyMiddleware(middleware)(
  //     devTools()(
  //       persistState(window.location.href.match(/[?&]debug_session=([^&]+)\b/))(
  //         createStore
  //       )
  //     )
  //   );
}

let store = finalCreateStore(reducer);
```

#### 小貼士

* `compose` 做的只是讓你不使用深度右括號的情況下來寫深度巢狀的函數。不要覺得它很複雜。
