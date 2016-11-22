# 搭配 React Router

現在你想在你的 Redux 應用中使用路由功能，可以搭配使用 [React Router](https://github.com/reactjs/react-router) 來實現。
Redux 和 React Router 將分別成為你資料和 URL 的事實來源（the source of truth）。
在大多數情況下， **最好** 將他們分開，除非你需要時光旅行和回放 action 來觸發 URL 改變。

## 安裝 React Router
可以使用 npm 來安裝 `React Router`。本教程基於 `react-router@^2.7.0` 。

`npm install --save react-router`

## 配置後備(fallback) URL
在整合 React Router 之前，我們需要配置一下我們的開發伺服器。
顯然，我們的開發伺服器無法感知配置在 React Router 中的 route。
比如：你想訪問並重新整理 `/todos`，由於是一個單頁面應用，你的開發伺服器需要生成並返回 `index.html`。
這裡，我們將演示如何在流行的開發伺服器上啟用這項功能。

>### 使用 Create React App 須知
> 如果你是使用 Create React App （你可以點選[這裡](https://facebook.github.io/react/blog/2016/07/22/create-apps-with-no-configuration.html)瞭解更多，譯者注）工具來生成項目，會自動為你配置好後備(fallback) URL。

### 配置 Express
如果你使用的是 Express 來返回你的 `index.html` 頁面，可以增加以下程式碼到你的項目中：

```js
app.get('/*', (req,res) => {
  res.sendfile(path.join(__dirname, 'index.html'))
})
```

### 配置 WebpackDevServer
如果你正在使用 WebpackDevServer 來返回你的 `index.html` 頁面，
你可以增加如下配置到 webpack.config.dev.js：

```js
devServer: {
  historyApiFallback: true,
}
```

## 連線 React Router 和 Redux 應用
在這一章，我們將使用 [Todos](https://github.com/reactjs/redux/tree/master/examples/todos) 作為例子。我們建議你在閱讀本章的時候，先將倉庫克隆下來。

首先，我們需要從 React Router 中匯入 `<Router />` 和 `<Route />`。程式碼如下：

```js
import { Router, Route, browserHistory } from 'react-router';
```

在 React 應用中，通常你會用 `<Router />` 包裹 `<Route />`。
如此，當 URL 變化的時候，`<Router />` 將會匹配到指定的路由，然後渲染路由繫結的元件。
`<Route />` 用來顯式地把路由對映到應用的元件結構上。
用 `path` 指定 URL，用 `component` 指定路由命中 URL 後需要渲染的那個元件。

```js
const Root = () => (
  <Router>
    <Route path="/" component={App} />
  </Router>
);
```

另外，在我們的 Redux 應用中，我們仍將使用  `<Provider />`。
`<Provider />` 是由 React Redux 提供的高階元件，用來讓你將 Redux 繫結到 React （詳見 [搭配 React](../basics/UsageWithReact.md)）。

然後，我們從 React Redux 匯入 `<Provider />`：

```js
import { Provider } from 'react-redux';
```

我們將用 `<Provider />` 包裹 `<Router />`，以便於路由處理器可以[訪問 `store`](../basics/UsageWithReact.html#passing-the-store)（暫時未找到相關中文翻譯，譯者注）。

```js
const Root = ({ store }) => (
  <Provider store={store}>
    <Router>
      <Route path="/" component={App} />
    </Router>
  </Provider>
);
```

現在，如果 URL 匹配到 '/'，將會渲染 `<App />` 元件。此外，我們將在 '/' 後面增加參數 `(:filter)`,
當我們嘗試從 URL 中讀取參數 `(:filter)`，需要以下程式碼：

```js
<Route path="/(:filter)" component={App} />
```

也許你想將 '#' 從 URL 中移除（例如：http://localhost:3000/#/?_k=4sbb0i）。
你需要從 React Router 匯入 `browserHistory` 來實現：

```js
import { Router, Route, browserHistory } from 'react-router';
```

然後將它傳給 `<Router />` 來移除 URL 中的 '#'：

```js
<Router history={browserHistory}>
  <Route path="/(:filter)" component={App} />
</Router>
```

只要你不需要相容古老的瀏覽器，比如IE9，你都可以使用 `browserHistory`。

#### `components/Root.js`

```js
import React, { PropTypes } from 'react';
import { Provider } from 'react-redux';
import { Router, Route, browserHistory } from 'react-router';
import App from './App';

const Root = ({ store }) => (
  <Provider store={store}>
    <Router history={browserHistory}>
      <Route path="/(:filter)" component={App} />
    </Router>
  </Provider>
);

Root.propTypes = {
  store: PropTypes.object.isRequired,
};

export default Root;
```

## 通過 React Router 導航

React Router 提供了 [`<Link />`](https://github.com/reactjs/react-router/blob/master/docs/API.md#link) 來實現導航功能。
下面將舉例演示。現在，修改我們的容器元件 `<FilterLink />` ，這樣我們就可以使用 `<FilterLink />` 來改變 URL。你可以通過 `activeStyle` 屬性來指定啟用狀態的樣式。

### `containers/FilterLink.js`

```js
import React from 'react';
import { Link } from 'react-router';

const FilterLink = ({ filter, children }) => (
  <Link
    to={filter === 'all' ? '' : filter}
    activeStyle={{
      textDecoration: 'none',
      color: 'black'
    }}
  >
    {children}
  </Link>
);

export default FilterLink;
```

### `containers/Footer.js`

```js
import React from 'react'
import FilterLink from '../containers/FilterLink'

const Footer = () => (
  <p>
    Show:
    {" "}
    <FilterLink filter="all">
      All
    </FilterLink>
    {", "}
    <FilterLink filter="active">
      Active
    </FilterLink>
    {", "}
    <FilterLink filter="completed">
      Completed
    </FilterLink>
  </p>
);

export default Footer
```

這時，如果你點選 `<FilterLink />`，你將看到你的 URL 在 `'/complete'`，`'/active'`，`'/'` 間切換。
甚至還支援瀏覽的回退功能，可以從歷史記錄中找到之前的 URL 並回退。

## 從 URL 中讀取資料
現在，即使 URL 改變，todo 列表也不會被過濾。
這是因為我們是在 `<VisibleTodoList />` 的 `mapStateToProps()` 函數中過濾的。
這個目前仍然是和 `state` 繫結，而不是和 URL 繫結。
`mapStateToProps` 的第二可選參數 `ownProps`，這個是一個傳遞給 `<VisibleTodoList />` 所有屬性的物件。

### `components/App.js`

```js
const mapStateToProps = (state, ownProps) => {
  return {
    todos: getVisibleTodos(state.todos, ownProps.filter) // previously was getVisibleTodos(state.todos, state.visibilityFilter)
  };
};
```

目前我們還沒有傳遞任何參數給 `<App />`，所以 `ownProps` 依然是一個空物件。
為了能夠根據 URL 來過濾我們的 todo 列表，我們需要向 `<VisibleTodoList />` 傳遞URL參數。

之前我們寫過：`<Route path="/(:filter)" component={App} />`，這使得可以在 `App` 中獲取 `params` 的屬性。

`params` 是一個包含 url 中所有指定參數的物件。
*例如：如果我們訪問 `localhost:3000/completed`，那麼 `params` 將等價於 `{ filter: 'completed' }`。
現在，我們可以在 `<App />` 中讀取 URL 參數了。*

注意，我們將使用 [ES6 的解構賦值](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)來對 `params` 進行賦值，以此傳遞給 `<VisibleTodoList />`。

### `components/App.js`

```js
const App = ({ params }) => {
  return (
    <div>
      <AddTodo />
      <VisibleTodoList
        filter={params.filter || 'all'}
      />
      <Footer />
    </div>
  );
};
```

## 下一步

現在你已經知道如何實現基礎的路由，接下來你可以閱讀 [React Router API](https://github.com/reactjs/react-router/tree/master/docs) 來學習更多知識。

>##### 其它路由庫注意點

>*Redux Router* 是一個實驗性質的庫，它使得你的 URL 的狀態和 redux store 內部狀態保持一致。它有和 React Router 一樣的 API，但是它的社羣支援比 react-router 小。

>*React Router Redux* 將你的 redux 應用和 react-router 繫結在一起，並且使它們保持同步。如果沒有這層繫結，你將不能通過時光旅行來回放 action。除非你需要這個，不然 React-router 和 Redux 完全可以分開操作。
