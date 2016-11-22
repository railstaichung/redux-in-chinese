# 示例

Redux [源碼](https://github.com/reactjs/redux/tree/master/examples) 中同時包含了一些示例。

## Counter Vanilla

執行 [Counter Vanilla](https://github.com/reactjs/redux/tree/master/examples/counter-vanilla) 示例:

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/counter-vanilla
open index.html
```

該示例不需搭建系統或檢視框架，展示了基於 ES5 的原生 Redux API。

## Counter

執行 [Counter](https://github.com/reactjs/redux/tree/master/examples/counter) 示例：

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/counter
npm install
npm start

open http://localhost:3000/
```

Redux 結合 React 使用的最基本示例。出於簡化，當 store 發生變化，React 元件會手動重新渲染。在實際的項目中，可以使用 React 和 Redux 已繫結、且更高效的 [React Redux](https://github.com/reactjs/react-redux)。

該示例包含測試程式碼。

## Todos

執行 [Todos](https://github.com/reactjs/redux/tree/master/examples/todos) 示例：

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/todos
npm install
npm start

open http://localhost:3000/
```

深入理解在 Redux 中 state 的更新與元件是如何共同運作的例子。展示了 reducer 如何委派 action 給其它 reducer，也展示瞭如何使用 [React Redux](https://github.com/reactjs/react-redux) 從展示元件中生成容器元件。

該示例包含測試程式碼。

## Todos with Undo

執行 [Todos-with-undo](https://github.com/reactjs/redux/tree/master/examples/todos-with-undo) 示例:

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/todos-with-undo
npm install
npm start

open http://localhost:3000/
```

前一個示例的衍生。基本相同但額外展示瞭如何使用 [Redux Undo](https://github.com/omnidan/redux-undo) 打包 reducer，僅增加幾行程式碼實現撤銷/重做功能。

## TodoMVC

執行 [TodoMVC](https://github.com/reactjs/redux/tree/master/examples/todomvc) 示例:

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/todomvc
npm install
npm start

open http://localhost:3000/
```

經典的 [TodoMVC](http://todomvc.com/) 示例。與 Todos 示例的目的相同，為了兩者間比較羅列在此。

該示例包含測試程式碼。

## Shopping Cart

執行 [Shopping Cart](https://github.com/reactjs/redux/tree/master/examples/shopping-cart) 示例：

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/shopping-cart
npm install
npm start

open http://localhost:3000/
```

該示例展示了隨著應用升級變得愈發重要的常用的 Redux 模式。尤其展示了，如何使用 ID 來標準化儲存資料實體，如何在不同層級將多個 reducer 組合使用，如何利用 reducer 定義選擇器以封裝 state 的相關內容。該示例也展示了使用 [Redux Logger](https://github.com/fcomb/redux-logger) 生成日誌，以及使用 [Redux Thunk](https://github.com/gaearon/redux-thunk) 中介軟體進行 action 的條件性分發。

## Tree View

執行 [Tree View](https://github.com/reactjs/redux/tree/master/examples/tree-view) 示例:

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/tree-view
npm install
npm start

open http://localhost:3000/
```

該示例展示了深層巢狀樹狀檢視的渲染，以及為了方便 reducer 更新，state 的標準化寫法。優良的渲染表現，來自於容器元件細粒度的、僅針對需要渲染的 tree node 的繫結。

該示例包含測試程式碼。

## Async

執行 [Async](https://github.com/reactjs/redux/tree/master/examples/async) 示例：

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/async
npm install
npm start

open http://localhost:3000/
```

該示例包含了：從非同步 API 的讀取操作、基於使用者的輸入來獲取資料、顯示正在載入的提示、快取響應、以及使快取過期失效。使用 [Redux Thunk](https://github.com/gaearon/redux-thunk) 中介軟體來封裝非同步帶來的附帶作用。

## Universal

執行 [Universal](https://github.com/reactjs/redux/tree/master/examples/universal) 示例:

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/universal
npm install
npm start

open http://localhost:3000/
```

展示了基於 Redux 和 React 的 [server rendering](../recipes/ServerRendering.md)。怎樣在伺服器端準備 store 中的初始 state 並傳遞到客戶端，使客戶端中的 store 可以從現有的 state 啟動。

## Real World

執行 [Real World](https://github.com/reactjs/redux/tree/master/examples/real-world) 示例：

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/real-world
npm install
npm start

open http://localhost:3000/
```

最為高階的示例。濃縮化的設計。包含了持續性地從標準化快取中批量獲取資料例項，針對 API 呼叫的自定義中介軟體的實現，逐步渲染已載入的資料、分頁器、快取響應，展示錯誤資訊，以及路由。同時，包含了 Redux DevTools 的使用。

## 更多示例

前往 [Awesome Redux](https://github.com/xgrommx/awesome-redux) 獲取更多 Redux 示例。
