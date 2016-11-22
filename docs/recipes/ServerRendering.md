# 服務端渲染

服務端渲染一個很常見的場景是當使用者（或搜尋引擎爬蟲）第一次請求頁面時，用它來做_初始渲染_。當伺服器接收到請求後，它把需要的元件渲染成 HTML 字元串，然後把它返回給客戶端（這裡統指瀏覽器）。之後，客戶端會接手渲染控制權。

下面我們使用 React 來做示例，對於支援服務端渲染的其它 view 框架，做法也是類似的。

### 服務端使用 Redux

當在伺服器使用 Redux 渲染時，一定要在響應中包含應用的 state，這樣客戶端可以把它作為初始 state。這點至關重要，因為如果在生成 HTML 前預載入了資料，我們希望客戶端也能訪問這些資料。否則，客戶端生成的 HTML 與伺服器端返回的 HTML 就會不匹配，客戶端還需要重新載入資料。

把資料傳送到客戶端，需要以下步驟：

* 為每次請求建立全新的 Redux store 例項；
* 按需 dispatch 一些 action；
* 從 store 中取出 state；
* 把 state 一同返回給客戶端。

在客戶端，使用伺服器返回的 state 建立並初始化一個全新的 Redux store。
Redux 在服務端**惟一**要做的事情就是，提供應用所需的**初始 state**。

## 安裝

下面來介紹如何配置服務端渲染。使用極簡的 [Counter 計數器應用](https://github.com/rackt/redux/tree/master/examples/counter) 來做示例，介紹如何根據請求在服務端提前渲染 state。

### 安裝依賴庫

本例會使用 [Express](http://expressjs.com/) 來做小型的 web 伺服器。還需要安裝 Redux 對 React 的繫結庫，Redux 預設並不包含。

```
npm install --save express react-redux
```

## 服務端開發

下面是服務端程式碼大概的樣子。使用 [app.use](http://expressjs.com/api.html#app.use) 掛載 [Express middleware](http://expressjs.com/guide/using-middleware.html) 處理所有請求。如果你還不熟悉 Express 或者 middleware，只需要瞭解每次伺服器收到請求時都會呼叫 handleRender 函數。

##### `server.js`

```js
import path from 'path';
import Express from 'express';
import React from 'react';
import { createStore } from 'redux';
import { Provider } from 'react-redux';
import counterApp from './reducers';
import App from './containers/App';

const app = Express();
const port = 3000;

// 每當收到請求時都會觸發
app.use(handleRender);

// 接下來會補充這部分程式碼
function handleRender(req, res) { /* ... */ }
function renderFullPage(html, initialState) { /* ... */ }

app.listen(port);
```

### 處理請求

第一件要做的事情就是對每個請求建立一個新的 Redux store 例項。這個 store 惟一作用是提供應用初始的 state。

渲染時，使用 `<Provider>` 來包住根元件 `<App />`，以此來讓元件樹中所有元件都能訪問到 store，就像之前的[搭配 React](../basics/UsageWithReact.md) 教程講的那樣。

服務端渲染最關鍵的一步是在**傳送響應前**渲染初始的 HTML。這就要使用 [React.renderToString()](https://facebook.github.io/react/docs/top-level-api.html#react.rendertostring).

然後使用 [`store.getState()`](../api/Store.md#getState) 從 store 得到初始 state。`renderFullPage` 函數會介紹接下來如何傳遞。

```js
import { renderToString } from 'react-dom/server'

function handleRender(req, res) {
  // 建立新的 Redux store 例項
  const store = createStore(counterApp);

  // 把元件渲染成字元串
  const html = renderToString(
    <Provider store={store}>
      <App />
    </Provider>
  )

  // 從 store 中獲得初始 state
  const initialState = store.getState();

  // 把渲染後的頁面內容傳送給客戶端
  res.send(renderFullPage(html, initialState));
}
```

### 注入初始元件的 HTML 和 State

服務端最後一步就是把初始元件的 HTML 和初始 state 注入到客戶端能夠渲染的模板中。如何傳遞 state 呢，我們新增一個 `<script>` 標籤來把 `initialState` 賦給 `window.__INITIAL_STATE__`。

客戶端可以通過 `window.__INITIAL_STATE__` 獲取 `initialState`。

同時使用 script 標籤來引入打包後的 js bundle 檔案。之前引入的 `serve-static` middleware 會處理它的請求。下面是程式碼。

```js
function renderFullPage(html, initialState) {
  return `
    <!doctype html>
    <html>
      <head>
        <title>Redux Universal Example</title>
      </head>
      <body>
        <div id="root">${html}</div>
        <script>
          window.__INITIAL_STATE__ = ${JSON.stringify(initialState)}
        </script>
        <script src="/static/bundle.js"></script>
      </body>
    </html>
    `
}
```

>##### 字元串插值語法須知

>上面的示例使用了 ES6 的[模板字元串](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/template_strings)語法。它支援多行字元串和字元串插補特性，但需要支援 ES6。如果要在 Node 端使用 ES6，參考 [Babel require hook](https://babeljs.io/docs/usage/require/) 文件。你也可以繼續使用 ES5。

## 客戶端開發

客戶端程式碼非常直觀。只需要從 `window.__INITIAL_STATE__` 得到初始 state，並傳給 [`createStore()`](../api/createStore.md) 函數即可。

程式碼如下:

#### `client.js`

```js
import React from 'react'
import { render } from 'react-dom'
import { createStore } from 'redux'
import { Provider } from 'react-redux'
import App from './containers/App'
import counterApp from './reducers'

// 通過服務端注入的全局變數得到初始 state
const initialState = window.__INITIAL_STATE__

// 使用初始 state 建立 Redux store
const store = createStore(counterApp, initialState)

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

你可以選擇自己喜歡的打包工具（Webpack, Browserify 或其它）來編譯並打包檔案到 `dist/bundle.js`。

當頁面載入時，打包後的 js 會啟動，並呼叫 [`React.render()`](https://facebook.github.io/react/docs/top-level-api.html#react.render)，然後會與服務端渲染的 HTML 的 `data-react-id` 屬性做關聯。這會把新生成的 React 例項與服務端的虛擬 DOM 連線起來。因為同樣使用了來自 Redux store 的初始 state，並且 view 元件程式碼是一樣的，結果就是我們得到了相同的 DOM。

就是這樣！這就是實現服務端渲染的所有步驟。

但這樣做還是比較原始的。只會用動態程式碼渲染一個靜態的 View。下一步要做的是動態建立初始 state 支援動態渲染 view。

## 準備初始 State

因為客戶端只是執行收到的程式碼，剛開始的初始 state 可能是空的，然後根據需要獲取 state。在服務端，渲染是同步執行的而且我們只有一次渲染 view 的機會。在收到請求時，可能需要根據請求參數或者外部 state（如訪問 API 或者資料庫），計算後得到初始 state。

### 處理 Request 參數

服務端收到的惟一輸入是來自瀏覽器的請求。在伺服器啟動時可能需要做一些配置（如執行在開發環境還是生產環境），但這些配置是靜態的。

請求會包含 URL 請求相關資訊，包括請求參數，它們對於做 [React Router](https://github.com/rackt/react-router) 路由時可能會有用。也可能在請求頭裡包含 cookies，鑑權資訊或者 POST 內容資料。下面演示如何基於請求參數來得到初始 state。

#### `server.js`

```js
import qs from 'qs'; // 新增到檔案開頭
import { renderToString } from 'react-dom/server'

function handleRender(req, res) {
  // 如果存在的話，從 request 讀取 counter
  const params = qs.parse(req.query)
  const counter = parseInt(params.counter) || 0

  // 得到初始 state
  let initialState = { counter }

  // 建立新的 Redux store 例項
  const store = createStore(counterApp, initialState)

  // 把元件渲染成字元串
  const html = renderToString(
    <Provider store={store}>
      <App />
    </Provider>
  )

  // 從 Redux store 得到初始 state
  const finalState = store.getState()

  // 把渲染後的頁面發給客戶端
  res.send(renderFullPage(html, finalState))
}
```

上面的程式碼首先訪問 Express 的 `Request` 物件。把參數轉成數字，然後設定到初始 state 中。如果你在瀏覽器中訪問 [http://localhost:3000/?counter=100](http://localhost:3000/?counter=100)，你會看到計數器從 100 開始。在渲染後的 HTML 中，你會看到計數顯示 100 同時設定進了 `__INITIAL_STATE__` 變數。

### 獲取非同步 State

服務端渲染常用的場景是處理非同步 state。因為服務端渲染天生是同步的，因此非同步的資料獲取操作對應到同步操作非常重要。

最簡單的做法是往同步程式碼裡傳遞一些回撥函數。在這個回撥函數裡引用響應物件，把渲染後的 HTML 發給客戶端。不要擔心，並沒有想像中那麼難。

本例中，我們假設有一個外部資料來源提供計算器的初始值（所謂的把計算作為一種服務）。我們會模擬一個請求並使用結果建立初始 state。API 請求程式碼如下：

#### `api/counter.js`

```js
function getRandomInt(min, max) {
  return Math.floor(Math.random() * (max - min)) + min
}

export function fetchCounter(callback) {
  setTimeout(() => {
    callback(getRandomInt(1, 100))
  }, 500)
}
```

再次說明一下，這只是一個模擬的 API，我們使用 `setTimeout` 模擬一個需要 500 毫秒的請求（實現項目中 API 請求一般會更快）。傳入一個回撥函數，它非同步返回一個隨機數字。如果你使用了基於 Promise 的 API 工具，那麼要把回撥函數放到 `then` 中。

在服務端，把程式碼使用 `fetchCounter` 包起來，在回撥函數裡拿到結果：

#### `server.js`

```js
// 新增到 import
import { fetchCounter } from './api/counter'
import { renderToString } from 'react-dom/server'

function handleRender(req, res) {
  // 非同步請求模擬的 API
  fetchCounter(apiResult => {
    // 如果存在的話，從 request 讀取 counter
    const params = qs.parse(req.query)
    const counter = parseInt(params.counter) || apiResult || 0

    // 得到初始 state
    let initialState = { counter }

    // 建立新的 Redux store 例項
    const store = createStore(counterApp, initialState)

    // 把元件渲染成字元串
    const html = renderToString(
      <Provider store={store}>
        <App />
      </Provider>
    )

    // 從 Redux store 得到初始 state
    const finalState = store.getState()

    // 把渲染後的頁面發給客戶端
    res.send(renderFullPage(html, finalState))
  });
}
```

因為在回撥中使用了 `res.send()`，伺服器會保護連線開啟並在回撥函數執行前不傳送任何資料。你會發現每個請求都有 500ms 的延時。更高階的用法會包括對 API 請求出錯進行處理，比如錯誤的請求或者超時。

### 安全注意事項

因為我們程式碼中很多是基於使用者生成內容（UGC）和輸入的，不知不覺中，提高了應用可能受攻擊區域。任何應用都應該對使用者輸入做安全處理以避免跨站指令碼攻擊（XSS）或者程式碼注入。

我們的示例中，只對安全做基本處理。當從請求中拿參數時，對 `counter` 參數使用 `parseInt` 把它轉成數字。如果不這樣做，當 request 中有 script 標籤時，很容易在渲染的 HTML 中生成危險程式碼。就像這樣的：`?counter=</script><script>doSomethingBad();</script>`

在我們極簡的示例中，把輸入轉成數字已經比較安全。如果處理更復雜的輸入，比如自定義格式的文字，你應該用安全函數處理輸入，比如 [validator.js](https://www.npmjs.com/package/validator)。

此外，可能新增額外的安全層來對產生的 state 進行消毒。`JSON.stringify` 可能會造成 script 注入。鑑於此，你需要清洗 JSON 字元串中的 HTML 標籤和其它危險的字元。可能通過字元串替換或者使用複雜的庫如 [serialize-javascript](https://github.com/yahoo/serialize-javascript) 處理。

## 下一步

你還可以參考 [非同步 Actions](../advanced/AsyncActions.md) 學習更多使用 Promise 和 thunk 這些非同步元素來表示非同步資料流的方法。記住，那裡學到的任何內容都可以用於同構渲染。

如果你使用了 [React Router](https://github.com/rackt/react-router)，你可能還需要在路由處理元件中使用靜態的 `fetchData()` 方法來獲取依賴的資料。它可能返回 [非同步 action](../advanced/AsyncActions.md)，以便你的 `handleRender` 函數可以匹配到對應的元件類，對它們均 dispatch `fetchData()` 的結果，在 Promise 解決後才渲染。這樣不同路由需要呼叫的 API 請求都並置於路由處理元件了。在客戶端，你也可以使用同樣技術來避免在切換頁面時，當資料還沒有載入完成前執行路由。
