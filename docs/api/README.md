# API 文件

Redux 的 API 非常少。Redux 定義了一系列的約定（contract）來讓你來實現（例如 [reducers](../Glossary.md#reducer)），同時提供少量輔助函數來把這些約定整合到一起。

這一章會介紹所有的 Redux API。記住，Redux 只關心如何管理 state。在實際的項目中，你還需要使用 UI 繫結庫如 [react-redux](https://github.com/gaearon/react-redux)。

### 頂級暴露的方法

* [createStore(reducer, [initialState])](createStore.md)
* [combineReducers(reducers)](combineReducers.md)
* [applyMiddleware(...middlewares)](applyMiddleware.md)
* [bindActionCreators(actionCreators, dispatch)](bindActionCreators.md)
* [compose(...functions)](compose.md)

### Store API

* [Store](Store.md)
  * [getState()](Store.md#getState)
  * [dispatch(action)](Store.md#dispatch)
  * [subscribe(listener)](Store.md#subscribe)
  * [getReducer()](Store.md#getReducer)
  * [replaceReducer(nextReducer)](Store.md#replaceReducer)

### 引入

上面介紹的所有函數都是頂級暴露的方法。都可以這樣引入：

#### ES6

```js
import { createStore } from 'redux';
```

#### ES5 (CommonJS)

```js
var createStore = require('redux').createStore;
```

#### ES5 (UMD build)

```js
var createStore = Redux.createStore;
```
