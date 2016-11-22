# 初始化 State

主要有兩種方法來初始化應用的 state 。

可以使用 `createStore` 方法中的第二個可選參數 `preloadedState`。

也可以在 reducer 中為 `undefined` 的 state 參數指定的預設的初始值。這個可以通過在 reducer 中新增一個明確的檢查來完成，也可以使用 ES6 中預設參數的語法 `function myReducer(state = someDefaultValue, action)`

這兩種方法是怎麼相互影響的也許並不總是很清楚，不過幸運的是，這個過程遵循著一些可預見的規則，這裡來說明它們是如何聯絡在一起的。


## 概要

如果不使用 `combineReducers()` 或者類似的程式碼，那麼 `preloadedState` 總是會優先於在 reducer 裡面使用 `state = ...` ，因為 `state` 傳到 reducer 裡的**是** `preloadedState` 的 state **而不是** `undefined`，所以 ES6 的預設參數語法並不起作用。

如果使用 `combineReducers()` 方法，那麼這裡的行為就會有一些細微的差別了。那些指定了 `preloadedState` 的 reducer 會接收到那些對應的 state。而其他的 reducer 將會接收到 `undefined` 並因此回到了 `state = ...` 這裡去獲取指定的預設值。

**通常情況下，通過 `preloadedState` 指定的 state 要優先於通過 reducer 指定 state。這樣可以使通過 reducer 預設參數指定初始資料顯得更加的合理，並且當你從一些持久化的儲存器或伺服器更新 store 的時候，允許你更新已存在的資料（全部或者部分）。**


## 深度


### 單一簡單的 Reducer

首先，讓我們舉一個單獨的 reducer 的例子，並且不使用 `combineReducers()` 方法。

那麼你的 reducer 可能像下面這樣：

```js
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT': return state + 1;
  case 'DECREMENT': return state - 1;
  default: return state;
  }
}
```

現在讓我們建立一個 store。

```
import { createStore } from 'redux';
let store = createStore(counter);
console.log(store.getState()); // 0
```

初始的狀態是 0 。為什麼呢？因為 `createStore` 的第二個參數是 `undefined`。這是 `state` 第一次傳到了 reducer 當中。Redux 會發起（dispatch）一個“虛擬”的 action 來填充這個 state 。所以 `counter` reducer 裡獲得的 `state` 等於 `undefined`。**實際上這相當於觸發了那個預設參數的特性**。因此，現在 `state` 的值是設定的預設參數 `0` ，這個 state (`0`) 是這裡 reducer 的返回值。

接下來讓我們舉一個不同場景的例子：

```js
import { createStore } from 'redux';
let store = createStore(counter, 42);
console.log(store.getState()); // 42
```

為什麼這次這裡的值是 `42` 而不是 `0` 呢？因為 `createStore` 的第二個參數是 `42` 。這個參數會賦給 state 並伴隨著一個虛擬 action 一起傳給 reducer。**這次，傳給 reducer 的 `state` 不再是 `undefined` (是 `42` )，所以 ES6 的預設參數特性沒有起作用。**所以這裡的 `state` 是 `42`，`42` 是這個 reducer 的返回值。


### 組合多個 Reducers

現在讓我們舉一個使用 `combineReducers()` 的例子。

你有兩個 reducers：

```js
function a(state = 'lol', action) {
  return state;
}

function b(state = 'wat', action) {
  return state;
}
```

這個組合的 reducer 通過 `combineReducers({ a, b })` 生成，就像下面這樣：

```js
// const combined = combineReducers({ a, b })
function combined(state = {}, action) {
  return {
	a: a(state.a, action),
	b: b(state.b, action)
  };
}
```

如果我們呼叫 `createStore` 方法並且不使用 `preloadedState` 參數，它將會把 `state` 初始化成 `{}`。 因此，傳入 reducer 的`state.a` 和 `state.b` 的值將會是 `undefined` **不論是 `a` 還是 `b` 的值都是 `undefined` ，這時如果指定 `state` 的預設值，那麼 reducer 將會返回這個預設值。 這就是第一次請求這個組合的 reducer 時返回 state `{ a: 'lol', b: 'wat'}` 的原因。

```js
import { createStore } from 'redux';
let store = createStore(combined);
console.log(store.getState()); // { a: 'lol', b: 'wat' }
```

接下來讓我們舉一個不同的場景的例子：

```js
import { createStore } from 'redux';
let store = createStore(combined, { a: 'horse' });
console.log(store.getState()); // { a: 'horse', b: 'wat' }
```

現在我們指定了 `createStore()` 的 `preloadedState` 參數。現在這個組合的 reducer 返回的 state 結合了我們指定給 `a` 的預設值 `horse` 以及通過 reducer 本身預設參數指定的 `b` 的值 `'wat'` 。

讓我們重新看看這個組合的 reducer 做了什麼：

```js
// const combined = combineReducers({ a, b })
function combined(state = {}, action) {
  return {
	a: a(state.a, action),
	b: b(state.b, action)
  };
}
```

在這個案例中，`state` 是我們指定的，所以它並沒有被賦值為 `{}`。這是一個 `a` 的值是 `'horse'` 的物件，但是沒有 `b` 的值。這就是為什麼 `a` reducer 收到了 `'horse'` 的 state 並高興地返回它，但 `b` reducer 卻得到了 `undefined` state 並因此返回了它自己設定的預設值（在這個例子裡是`'wat'`）。這就是我們如何得到的 `{ a: 'horse', b: 'wat' }` 這個結果。


## 總結

綜上所述，如果你遵守 Redux 的約定並且要讓 reducer 中 `undefined` 的 `state` 參數返回初始 state （最簡單的實現方法就是利用 ES6 的預設參數來指定 state），那麼對於組合多個 reducers 的情況，這將是一個很有用的做法，**他們會優先選擇通過 `preloadedState` 參數傳到 `createStore()` 的物件中的相應值，但是如果你不傳任何東西，或者沒設定相應的欄位，那麼 reducer 就會選擇指定的預設 `state` 參數來取代**。這樣的方法效果很好，因為他既能用來初始化也可以用來更新現有的資料，不過如果資料沒有保護措施的話，這樣做也會使一些獨立的 reducer 的 state 被重新賦值。當然你可以遞迴地使用這個模式，比如你可以在多個層級上使用 `combineReducers()` 方法，或者甚至手動的組合這些 reducer 並且傳入對應部分的 state tree。
