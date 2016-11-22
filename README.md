ZH-CN to ZH-TW by [leo424y](https://github.com/leo424y)

# [Redux 中文文檔](http://github.com/camsong/redux-in-chinese) [![Join the chat at https://gitter.im/camsong/redux-in-chinese](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/camsong/redux-in-chinese?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

<img src='https://camo.githubusercontent.com/f28b5bc7822f1b7bb28a96d8d09e7d79169248fc/687474703a2f2f692e696d6775722e636f6d2f4a65567164514d2e706e67' height='60'>

> :tada: :tada: :tada:
 本文檔從建立之初，幫助了非常多的小夥伴學習 Redux，也收到了 Redux 作者 Dan 的多次點贊。目前每天瀏覽量大概在一萬左右。但由於幾位作者工作都比較忙，很多文檔已經過期，卻也沒有時間更新。如何你有時間，希望你和我們一起翻譯，幫助更多人，歡迎提交 PR。不知何處下手不用怕？我們為你準備好了完善的 [ETC 翻譯文檔](https://github.com/react-guide/etc)。:point_right: 回覆 issue 認領 https://github.com/camsong/redux-in-chinese/issues :heart: :heart: :heart:
 本文檔從建立之初，幫助了非常多的小夥伴學習 Redux，也收到了 Redux 作者 Dan 的多次點贊。目前每天瀏覽量大概在一萬左右。但由於幾位作者工作都比較忙，很多文檔已經過期，卻也沒有時間更新。如何你有時間，希望你和我們一起翻譯，幫助更多人，歡迎提交 PR。不知何處下手不用怕？我們為你準備好了完善的 [ETC 翻譯文檔](https://github.com/react-guide/etc)。:point_right: 回覆 issue 認領 https://github.com/camsong/redux-in-chinese/issues :heart: :heart: :heart:

> 在線 Gitbook 地址：http://cn.redux.js.org/
> 英文原版：http://redux.js.org/
> 學了這個還不盡興？推薦極精簡的 [Redux Tutorial 教程](https://github.com/react-guide/redux-tutorial-cn#redux-tutorial)
> React 核心開發者寫的 [React 設計思想](https://github.com/react-guide/react-basic)

> :arrow_down: 離線下載：[pdf 格式](https://github.com/camsong/redux-in-chinese/raw/master/offline/redux-in-chinese.pdf)，[epub 格式](https://github.com/camsong/redux-in-chinese/raw/master/offline/redux-in-chinese.epub)，[mobi 格式](https://github.com/camsong/redux-in-chinese/raw/master/offline/redux-in-chinese.mobi)

Redux 是 JavaScript 狀態容器，提供可預測化的狀態管理。

可以讓你構建一致化的應用，運行於不同的環境（客戶端、服務器、原生應用），並且易於測試。不僅於此，它還提供
超爽的開發體驗，比如有一個[時間旅行調試器可以編輯後實時預覽](https://github.com/gaearon/redux-devtools)。

Redux 除了和 [React](https://facebook.github.io/react/) 一起用外，還支持其它界面庫。
它體小精悍（只有2kB）且沒有任何依賴。

### 評價

>[“Love what you’re doing with Redux”](https://twitter.com/jingc/status/616608251463909376)
>Jing Chen，Flux 作者

>[“I asked for comments on Redux in FB's internal JS discussion group, and it was universally praised. Really awesome work.”](https://twitter.com/fisherwebdev/status/616286955693682688)
>Bill Fisher，Flux 作者

>[“It's cool that you are inventing a better Flux by not doing Flux at all.”](https://twitter.com/andrestaltz/status/616271392930201604)
>André Staltz，Cycle 作者

### 開始之前

> 也推薦閱讀你可能並不需要Redux：
> [“You Might Not Need Redux”](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)

### 開發經歷

Redux 的開發最早開始於我在準備 React Europe 演講[熱加載與時間旅行](https://www.youtube.com/watch?v=xsSnOQynTHs)的時候，當初的目標是創建一個狀態管理庫，來提供最簡化 API，但同時做到行為的完全可預測，因此才得以實現日誌打印，熱加載，時間旅行，同構應用，錄製和重放，而不需要任何開發參與。

### 啟示

Redux 由 [Flux](http://facebook.github.io/flux/) 演變而來，但受 [Elm](http://elm-lang.org/guide/architecture) 的啟發，避開了 Flux 的複雜性。
不管你有沒有使用過它們，只需幾分鐘就能上手 Redux。

### 安裝

安裝穩定版：

```
npm install --save redux
```

以上基於使用 [npm](https://www.npmjs.com/) 來做包管理工具的情況下。

否則你可以直接在 [unpkg 上訪問這些文件](https://unpkg.com/redux/)，下載下來，或者把讓你的包管理工具指向它。

一般情況下人們認為 Redux 就是一些 [CommonJS](http://webpack.github.io/docs/commonjs.html) 模塊的集合。這些模塊就是你在使用 [Webpack](http://webpack.github.io/)、[Browserify](http://browserify.org/)、或者 Node 環境時引入的。如果你想追求時髦並使用 [Rollup](http://rollupjs.org/)，也是支持的。

你也可以不使用模塊打包工具。`redux` 的 npm 包裡 [`dist` 目錄](https://unpkg.com/redux/dist/)包含了預編譯好的生產環境和開發環境下的 [UMD](https://github.com/umdjs/umd) 文件。可以直接使用，而且支持大部分流行的 JavaScript 包加載器和環境。比如，你可以直接在頁面上的 `<script>` 標籤 中引入 UMD 文件，也可以[讓 `Bower` 來安裝](https://github.com/reactjs/redux/pull/1181#issuecomment-167361975)。UMD 文件可以讓你使用 `window.Redux` 全局變量來訪問 Redux。

Redux 源文件由 ES2015 編寫，但是會預編譯到 CommonJS 和 UMD 規範的 ES5，所以它可以支持 [任何現代瀏覽器](http://caniuse.com/#feat=es5)。你不必非得使用 Babel 或模塊打包器來使用 Redux。

#### 附加包

多數情況下，你還需要使用 [React 綁定庫](http://github.com/gaearon/react-redux)和[開發者工具](http://github.com/gaearon/redux-devtools)。

```
npm install --save react-redux
npm install --save-dev redux-devtools
```

需要提醒的是，和 Redux 不同，很多 Redux 生態下的包並不提供 UMD 文件，所以為了提升開發體驗，我們建議使用像 [Webpack](http://webpack.github.io/) 和 [Browserify](http://browserify.org/) 這樣的 CommonJS 模塊打包器。

### 要點

應用中所有的 state 都以一個對象樹的形式儲存在一個單一的 *store* 中。
惟一改變 state 的辦法是觸發 *action*，一個描述發生什麼的對象。
為了描述 action 如何改變 state 樹，你需要編寫 *reducers*。

就是這樣！

```js
import { createStore } from 'redux';

/**
 * 這是一個 reducer，形式為 (state, action) => state 的純函數。
 * 描述了 action 如何把 state 轉變成下一個 state。
 *
 * state 的形式取決於你，可以是基本類型、數組、對象、
 * 甚至是 Immutable.js 生成的數據結構。惟一的要點是
 * 當 state 變化時需要返回全新的對象，而不是修改傳入的參數。
 *
 * 下面例子使用 `switch` 語句和字符串來做判斷，但你可以寫幫助類(helper)
 * 根據不同的約定（如方法映射）來判斷，只要適用你的項目即可。
 */
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1;
  case 'DECREMENT':
    return state - 1;
  default:
    return state;
  }
}

// 創建 Redux store 來存放應用的狀態。
// API 是 { subscribe, dispatch, getState }。
let store = createStore(counter);

// 可以手動訂閱更新，也可以事件綁定到視圖層。
store.subscribe(() =>
  console.log(store.getState())
);

// 改變內部 state 惟一方法是 dispatch 一個 action。
// action 可以被序列化，用日記記錄和儲存下來，後期還可以以回放的方式執行
store.dispatch({ type: 'INCREMENT' });
// 1
store.dispatch({ type: 'INCREMENT' });
// 2
store.dispatch({ type: 'DECREMENT' });
// 1
```
你應該把要做的修改變成一個普通對象，這個對象被叫做 *action*，而不是直接修改 state。然後編寫專門的函數來決定每個 action 如何改變應用的 state，這個函數被叫做 *reducer*。

如果你以前使用 Flux，那麼你只需要注意一個重要的區別。Redux 沒有 Dispatcher 且不支持多個 store。相反，只有一個單一的 store 和一個根級的 reduce 函數（reducer）。隨著應用不斷變大，你應該把根級的 reducer 拆成多個小的 reducers，分別獨立地操作 state 樹的不同部分，而不是添加新的 stores。這就像一個 React 應用只有一個根級的組件，這個根組件又由很多小組件構成。

用這個架構開發計數器有點殺雞用牛刀，但它的美在於做複雜應用和龐大系統時優秀的擴展能力。由於它可以用 action 追溯應用的每一次修改，因此才有強大的開發工具。如錄製用戶會話並回放所有 action 來重現它。

### Redux 作者教你學

[Redux 入門](https://egghead.io/series/getting-started-with-redux) 是由 Redux 作者 Dan Abramov 講述的包含 30 個視頻的課程。它涵蓋了本文檔的“基礎”部分，同時還有不可變（immutability）、測試、Redux 最佳實踐、搭配 React 使用的講解。**這個課程將永久免費。**

還等什麼？

#### [開始觀看 30 個免費視頻！](https://egghead.io/series/getting-started-with-redux)

### 文檔

* [介紹](http://camsong.github.io/redux-in-chinese//docs/introduction/index.html)
* [基礎](http://camsong.github.io/redux-in-chinese//docs/basics/index.html)
* [高級](http://camsong.github.io/redux-in-chinese//docs/advanced/index.html)
* [技巧](http://camsong.github.io/redux-in-chinese//docs/recipes/index.html)
* [常見問題](http://camsong.github.io/redux-in-chinese//docs/FAQ.html)
* [排錯](http://camsong.github.io/redux-in-chinese//docs/Troubleshooting.html)
* [詞彙表](http://camsong.github.io/redux-in-chinese//docs/Glossary.html)
* [API 文檔](http://camsong.github.io/redux-in-chinese//docs/api/index.html)

### 示例

* [原生 Counter](http://camsong.github.io/redux-in-chinese/docs/introduction/Examples.html#counter-vanilla) ([source](https://github.com/rackt/redux/tree/master/examples/counter-vanilla))
* [Counter](http://camsong.github.io/redux-in-chinese/docs/introduction/Examples.html#counter) ([source](https://github.com/rackt/redux/tree/master/examples/counter))
* [Todos](http://camsong.github.io/redux-in-chinese/docs/introduction/Examples.html#todos) ([source](https://github.com/rackt/redux/tree/master/examples/todos))
* [可撤銷的 Todos](http://camsong.github.io/redux-in-chinese/docs/introduction/Examples.html#todos-with-undo) ([source](https://github.com/rackt/redux/tree/master/examples/todos-with-undo))
* [TodoMVC](http://camsong.github.io/redux-in-chinese/docs/introduction/Examples.html#todomvc) ([source](https://github.com/rackt/redux/tree/master/examples/todomvc))
* [購物車](http://camsong.github.io/redux-in-chinese/docs/introduction/Examples.html#shopping-cart) ([source](https://github.com/rackt/redux/tree/master/examples/shopping-cart))
* [Tree View](http://camsong.github.io/redux-in-chinese/docs/introduction/Examples.html#tree-view) ([source](https://github.com/rackt/redux/tree/master/examples/tree-view))
* [異步](http://camsong.github.io/redux-in-chinese/docs/introduction/Examples.html#async) ([source](https://github.com/rackt/redux/tree/master/examples/async))
* [Universal](http://camsong.github.io/redux-in-chinese/docs/introduction/Examples.html#universal) ([source](https://github.com/rackt/redux/tree/master/examples/universal))
* [Real World](http://camsong.github.io/redux-in-chinese/docs/introduction/Examples.html#real-world) ([source](https://github.com/rackt/redux/tree/master/examples/real-world))

如果你是 NPM 新手，創建和運行一個新的項目有難度，或者不知道上面的代碼應該放到哪裡使用，請下載 [simplest-redux-example](https://github.com/jackielii/simplest-redux-example) 這個示例，它是一個集成了 React、Browserify 和 Redux 的最簡化的示例項目。

### 交流

打開 Slack，加入 [Reactiflux](http://reactiflux.com/) 中的 **#redux** 頻道。

### 感謝

* [Elm 架構](https://github.com/evancz/elm-architecture-tutorial) 介紹了使用 reducers 來操作 state 數據；
* [Turning the database inside-out](http://blog.confluent.io/2015/03/04/turning-the-database-inside-out-with-apache-samza/) 大開腦洞;
* [ClojureScript 裡使用 Figwheel](http://www.youtube.com/watch?v=j-kj2qwJa_E) for convincing me that re-evaluation should “just work”;
* [Webpack](https://github.com/webpack/docs/wiki/hot-module-replacement-with-webpack) 熱模塊替換;
* [Flummox](https://github.com/acdlite/flummox) 教我在 Flux 裡去掉樣板文件和單例對象；
* [disto](https://github.com/threepointone/disto) 演示使用熱加載 Stores 的可行性；
* [NuclearJS](https://github.com/optimizely/nuclear-js) 證明這樣的架構性能可以很好；
* [Om](https://github.com/omcljs/om) 普及 state 惟一原子化的思想。
* [Cycle](https://github.com/staltz/cycle) 介紹了 function 是如何在很多場景都是最好的工具；
* [React](https://github.com/facebook/react) 實踐啟迪。

### 作者列表

> 定期更新，謝謝各位辛勤貢獻

* [Cam Song 會影@camsong](https://github.com/camsong)
* [Jovey Zheng@jovey-zheng](https://github.com/jovey-zheng)
* [Pandazki@pandazki](https://github.com/pandazki)
* [Yuwei Wang@yuweiw823](https://github.com/yuweiw823)
* [Yudong@hashtree](https://github.com/hashtree)
* [Arcthur@arcthur](https://github.com/arcthur)
* [Doray Hong@dorayx](https://github.com/dorayx)
* [Desen Meng@demohi](https://github.com/demohi)
* [Zhe Zhang@zhe](https://github.com/zhe)
* [alcat2008](https://github.com/alcat2008)
* [Frozenme](https://github.com/Frozenme)
* [姜楊軍@Yogi-Jiang](https://github.com/Yogi-Jiang)
* [Byron Bai@happybai](https://github.com/happybai)
* [Guo Cheng@guocheng](https://github.com/guocheng)
* [omytea](https://github.com/omytea)
* [Fred Wang](https://github.com/namelos)
* [Amo Wu](https://github.com/amowu)
* [C. T. Lin](https://github.com/chentsulin)
* [錢利江](https://github.com/timqian)
* [雲謙](https://github.com/sorrycc)
* [denvey](https://github.com/denvey)
* [三點](https://github.com/zousandian)
* [Eric Wong](https://github.com/ele828)
* [Owen Yang](https://github.com/owenyang0)
* [Cai Huanyu](https://github.com/Darmody)

**本文檔翻譯流程符合 [ETC 翻譯規範](https://github.com/react-guide/ETC)，歡迎你來一起完善**
