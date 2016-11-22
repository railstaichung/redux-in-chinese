# 生態系統

Redux 是一個體小精悍的庫，但它相關的內容和 API 都是精挑細選的，足以衍生出豐富的工具集和可擴充套件的生態系統。

如果需要關於 Redux 所有內容的列表，推薦移步至 [Awesome Redux](https://github.com/xgrommx/awesome-redux)。它包含了示例、樣板程式碼、中介軟體、工具庫，還有很多其它相關內容。要想學習 React 和 Redux ，[React/Redux Links](https://github.com/markerikson/react-redux-links) 包含了教程和不少有用的資源，[Redux Ecosystem Links](https://github.com/markerikson/redux-ecosystem-links) 則列出了 許多 Redux 相關的庫及外掛。

本頁將只列出由 Redux 維護者審查過的一部分內容。不要因此打消嘗試其它工具的信心！整個生態發展得太快，我們沒有足夠的時間去關注所有內容。建議只把這些當作“內部推薦”，如果你使用 Redux 建立了很酷的內容，不要猶豫，馬上發個 PR 吧。

## 學習 Redux

### 演示

* **[開始學習 Redux](https://egghead.io/series/getting-started-with-redux)** — 向作者學習 Redux 基礎知識（30 個免費的教學視訊）
* **[學習 Redux](https://learnredux.com)** — 搭建一個簡單的圖片應用，簡要使用了 Redux、React Router 和 React.js 的核心思想

### 示例應用

* [官方示例](Examples.md) — 一些官方示例，涵蓋了多種 Redux 技術
* [SoundRedux](https://github.com/andrewngu/sound-redux) — 用 Redux 構建的 SoundCloud 客戶端
* [grafgiti](https://github.com/mohebifar/grafgiti) — 在你的 Github 的 Contributor 頁上建立 graffiti

### 教程與文章

* [Redux 教程](https://github.com/happypoulp/redux-tutorial)
* [Redux Egghead 課程筆記](https://github.com/tayiorbeii/egghead.io_redux_course_notes)
* [使用 React Native 進行資料整合](http://makeitopen.com/tutorials/building-the-f8-app/data/)
* [What the Flux?! Let’s Redux.](https://blog.andyet.com/2015/08/06/what-the-flux-lets-redux)
* [Leveling Up with React: Redux](https://css-tricks.com/learning-react-redux/)
* [A cartoon intro to Redux](https://code-cartoons.com/a-cartoon-intro-to-redux-3afb775501a6)
* [Understanding Redux](http://www.youhavetolearncomputers.com/blog/2015/9/15/a-conceptual-overview-of-redux-or-how-i-fell-in-love-with-a-javascript-state-container)
* [Handcrafting an Isomorphic Redux Application (With Love)](https://medium.com/@bananaoomarang/handcrafting-an-isomorphic-redux-application-with-love-40ada4468af4)
* [Full-Stack Redux Tutorial](http://teropa.info/blog/2015/09/10/full-stack-redux-tutorial.html)
* [Getting Started with React, Redux, and Immutable](http://www.theodo.fr/blog/2016/03/getting-started-with-react-redux-and-immutable-a-test-driven-tutorial-part-2/)
* [Secure Your React and Redux App with JWT Authentication](https://auth0.com/blog/2016/01/04/secure-your-react-and-redux-app-with-jwt-authentication/)
* [Understanding Redux Middleware](https://medium.com/@meagle/understanding-87566abcfb7a)
* [Angular 2 — Introduction to Redux](https://medium.com/google-developer-experts/angular-2-introduction-to-redux-1cf18af27e6e)
* [Apollo Client: GraphQL with React and Redux](https://medium.com/apollo-stack/apollo-client-graphql-with-react-and-redux-49b35d0f2641)
* [Using redux-saga To Simplify Your Growing React Native Codebase](https://shift.infinite.red/using-redux-saga-to-simplify-your-growing-react-native-codebase-2b8036f650de)
* [Build an Image Gallery Using Redux Saga](http://joelhooks.com/blog/2016/03/20/build-an-image-gallery-using-redux-saga)
* [Working with VK API (in Russian)](https://www.gitbook.com/book/maxfarseer/redux-course-ru/details)

### 演講

* [Live React: Hot Reloading and Time Travel](http://youtube.com/watch?v=xsSnOQynTHs) — 瞭解 Redux 如何使用限制措施，讓伴隨時間旅行的熱載入變得簡單
* [Cleaning the Tar: Using React within the Firefox Developer Tools](https://www.youtube.com/watch?v=qUlRpybs7_c) — 瞭解如何從已有的 MVC 應用逐步遷移至 Redux
* [Redux: Simplifying Application State](https://www.youtube.com/watch?v=okdC5gcD-dM) — Redux 架構介紹

## 使用 Redux

### 不同框架繫結

* [react-redux](https://github.com/gaearon/react-redux) — React
* [ng-redux](https://github.com/wbuchwalter/ng-redux) — Angular
* [ng2-redux](https://github.com/wbuchwalter/ng2-redux) — Angular 2
* [backbone-redux](https://github.com/redbooth/backbone-redux) — Backbone
* [redux-falcor](https://github.com/ekosz/redux-falcor) — Falcor
* [deku-redux](https://github.com/troch/deku-redux) — Deku

### 中介軟體

* [redux-thunk](http://github.com/gaearon/redux-thunk) — 用最簡單的方式搭建非同步 action 構造器
* [redux-promise](https://github.com/acdlite/redux-promise) — 遵從 [FSA](https://github.com/acdlite/flux-standard-action) 標準的 promise 中介軟體
* [redux-axios-middleware](https://github.com/svrcekmichal/redux-axios-middleware) — 使用 axios HTTP 客戶端獲取資料的 Redux 中介軟體
* [redux-observable](https://github.com/blesh/redux-observable/) — Redux 的 RxJS 中介軟體
* [redux-rx](https://github.com/acdlite/redux-rx) — 給 Redux 用的 RxJS 工具，包括觀察變數的中介軟體
* [redux-logger](https://github.com/fcomb/redux-logger) — 記錄所有 Redux action 和下一次 state 的日誌
* [redux-immutable-state-invariant](https://github.com/leoasis/redux-immutable-state-invariant) — 開發中的狀態變更提醒
* [redux-unhandled-action](https://github.com/socialtables/redux-unhandled-action) — 開發過程中，若 Action 未使 State 發生變化則發出警告
* [redux-analytics](https://github.com/markdalgleish/redux-analytics) — Redux middleware 分析
* [redux-gen](https://github.com/weo-edu/redux-gen) — Redux middleware 生成器
* [redux-saga](https://github.com/yelouafi/redux-saga) — Redux 應用的另一種副作用 model
* [redux-action-tree](https://github.com/cerebral/redux-action-tree) — Redux 的可組合性 Cerebral-style 訊號
* [apollo-client](https://github.com/apollostack/apollo-client) — 針對 GraphQL 伺服器及基於 Redux 的 UI 框架的快取客戶端

### 路由

* [redux-simple-router](https://github.com/rackt/redux-simple-router) — 保持 React Router 和 Redux 同步
* [redux-router](https://github.com/acdlite/redux-router) — 由 React Router 繫結到 Redux 的庫

### 元件

* [redux-form](https://github.com/erikras/redux-form) — 在 Redux 中時時持有 React 表格的 state
* [react-redux-form](https://github.com/davidkpiano/react-redux-form) — 在 React 中使用 Redux 生成表格

### 增強器（Enhancer）

* [redux-batched-subscribe](https://github.com/tappleby/redux-batched-subscribe) — 針對 store subscribers 的自定義批處理與防跳請求
* [redux-history-transitions](https://github.com/johanneslumpe/redux-history-transitions) — 基於獨斷的 action 的 history 庫轉換
* [redux-optimist](https://github.com/ForbesLindesay/redux-optimist) — 使 action 可稍後提交或撤銷
* [redux-optimistic-ui](https://github.com/mattkrick/redux-optimistic-ui) — A reducer enhancer to enable type-agnostic optimistic updates 允許對未知類型進行更新的 reducer 增強器
* [redux-undo](https://github.com/omnidan/redux-undo) — 使 reducer 便捷的重做/撤銷，以及 action 記錄功能
* [redux-ignore](https://github.com/omnidan/redux-ignore) — 通過陣列或過濾功能忽略 redux action
* [redux-recycle](https://github.com/omnidan/redux-recycle) — 在確定的 action 上重置 redux 的 state
* [redux-batched-actions](https://github.com/tshelburne/redux-batched-actions) — 單使用者通知去 dispatch 多個 action
* [redux-search](https://github.com/treasure-data/redux-search) — 自動 index 站點資源並實現即時搜尋
* [redux-electron-store](https://github.com/samiskin/redux-electron-store) — Store 增強器， 可同步不同 Electron 程序上的多個 Redux store
* [redux-loop](https://github.com/raisemarketplace/redux-loop) — Sequence effects purely and naturally by returning them from your reducers
* [redux-side-effects](https://github.com/salsita/redux-side-effects) — Utilize Generators for declarative yielding of side effects from your pure reducers

### 工具集

* [reselect](https://github.com/faassen/reselect) — 受 NuclearJS 啟發，有效派生資料的選擇器
* [normalizr](https://github.com/gaearon/normalizr) — 為了讓 reducers 更好的消化資料，將API返回的巢狀資料正規化化
* [redux-actions](https://github.com/acdlite/redux-actions) — 在初始化 reducer 和 action 構造器時減少樣板程式碼 (boilerplate)
* [redux-act](https://github.com/pauldijou/redux-act) — 生成 reducer 和 action 建立函數的庫
* [redux-transducers](https://github.com/acdlite/redux-transducers) — Redux 的編譯器工具
* [redux-immutablejs](https://github.com/indexiatech/redux-immutablejs) — 將Redux 和 [Immutable](https://github.com/facebook/immutable-js/) 整合到一起的工具
* [redux-tcomb](https://github.com/gcanti/redux-tcomb) — 在 Redux 中使用具有不可變特性、並經過類型檢查的 state 和 action
* [redux-mock-store](https://github.com/arnaudbenard/redux-mock-store) - 模擬 redux 來測試應用
* [redux-actions-assertions](https://github.com/dmitry-zaets/redux-actions-assertions) — Redux actions 測試斷言

### 開發者工具

* [redux-devtools](http://github.com/gaearon/redux-devtools) — 一個使用時間旅行 UI 、熱載入和 reducer 錯誤處理器的 action 日誌工具，[最早演示於 React Europe 會議](https://www.youtube.com/watch?v=xsSnOQynTHs)
* [Redux DevTools Extension](https://github.com/zalmoxisus/redux-devtools-extension) — 打包了 Redux DevTools 及附加功能的 Chrome 外掛

### 開發者工具監聽器

* [Log Monitor](https://github.com/gaearon/redux-devtools-log-monitor) — Redux DevTools 預設監聽器，提供樹狀檢視
* [Dock Monitor](https://github.com/gaearon/redux-devtools-dock-monitor) — A resizable and movable dock for Redux DevTools monitors
* [Slider Monitor](https://github.com/calesce/redux-slider-monitor) — Redux DevTools 自定義監聽器，可回放被記錄的 Redux action
* [Inspector](https://github.com/alexkuz/redux-devtools-inspector) — Redux DevTools 自定義監聽器，可篩選、區分 action，深入 state 並監測變化
* [Diff Monitor](https://github.com/whetstone/redux-devtools-diff-monitor) — 區分不同 action 的 store 變動的 Redux Devtools 監聽器
* [Filterable Log Monitor](https://github.com/bvaughn/redux-devtools-filterable-log-monitor/) — 樹狀可篩選檢視的 Redux DevTools 監聽器
* [Chart Monitor](https://github.com/romseguy/redux-devtools-chart-monitor) — Redux DevTools 圖表監聽器
* [Filter Actions](https://github.com/zalmoxisus/redux-devtools-filter-actions) — 可篩選 action 、可組合使用的 Redux DevTools 監聽器


### 社羣公約

* [Flux Standard Action](https://github.com/acdlite/flux-standard-action) —  Flux 中 action object 的人性化標準
* [Canonical Reducer Composition](https://github.com/gajus/canonical-reducer-composition) — 巢狀 reducer 組成的武斷標準
* [Ducks: Redux Reducer Bundles](https://github.com/erikras/ducks-modular-redux) — 關於捆綁多個 reducer, action 類型 和 action 的提案

### 翻譯

* [中文文件](http://camsong.github.io/redux-in-chinese/) — 簡體中文
* [繁體中文檔案](https://github.com/chentsulin/redux) — 繁體中文
* [Redux in Russian](https://github.com/rajdee/redux-in-russian) — 俄語
* [Redux en Español](http://es.redux.js.org/) - 西班牙語

## 更多

* [Awesome Redux](https://github.com/xgrommx/awesome-redux) 是一個包含大量與 Redux 相關的庫列表。
* [React-Redux Links](https://github.com/markerikson/react-redux-links) React、Redux、ES6 的高質量文章、教程、及相關內容列表。
* [Redux Ecosystem Links](https://github.com/markerikson/redux-ecosystem-links) Redux 相關庫、外掛、工具集的分類資源。
