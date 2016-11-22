# Redux 常見問題

## 目錄

- **綜合**
  - [何時使用 Redux ？](#general-when-to-use)
  - [Redux 只能搭配 React 使用？](#general-only-react)
  - [Redux 需要特殊的編譯工具支援嗎？](#general-build-tools)
- **Reducer**
  - [如何在 reducer 之間共享 state ？ combineReducers 是必須的嗎？](#reducers-share-state)
  - [處理 action 必須用 switch 語句嗎？](#reducers-use-switch)
- **組織 State**
  - [必須將所有 state 都維護在 Redux 中嗎？ 可以用 React 的 setState() 方法嗎？](#organizing-state-only-redux-state)
  - [可以將 store 的 state 設定為函數、promise或者其它非序列化值嗎？](#organizing-state-non-serializable)
  - [如何在 state 中組織巢狀及重複資料？](#organizing-state-nested-data)
- **建立 Store**
  - [可以建立多個 store 嗎，應該這麼做嗎？能在元件中直接引用 store 並使用嗎？](#store-setup-multiple-stores)
  - [在 store enhancer 中可以存在多個 middleware 鏈嗎？ 在 middleware 方法中，next 和 dispatch 之間區別是什麼？](#store-setup-middleware-chains)
  - [怎樣只訂閱 state 的一部分變更？如何將分發的 action 作為訂閱的一部分？](#store-setup-subscriptions)
- **Action**
  - [為何 type 必須是字元串，或者至少可以被序列化？ 為什麼 action 類型應該作為常量？](#actions-string-constants)
  - [是否存在 reducer 和 action 之間的一對一對映？](#actions-reducer-mappings)
  - [怎樣表示類似 AJAX 請求的 “副作用”？為何需要 “action 建立函數”、“thunks” 以及 “middleware” 類似的東西去處理非同步行為？](#actions-side-effects)
  - [是否應該在 action 建立函數中連續分發多個 action？](#actions-multiple-actions)
- **程式碼結構**
  - [檔案結構應該是什麼樣？項目中該如何對 action 建立函數和 reducer 分組？ selector 又該放在哪裡？](#structure-file-structure)
  - [如何將邏輯在 reducer 和 action 建立函數之間劃分？ “業務邏輯” 應該放在哪裡？](#structure-business-logic)
- **效能**
  - [考慮到效能和架構， Redux “可擴充套件性” 如何？](#performance-scaling)
  - [每個 action 都呼叫 “所有的 reducer” 會不會很慢？](#performance-all-reducers)
  - [在 reducer 中必須對 state 進行深拷貝嗎？拷貝 state 不會很慢嗎？](#performance-clone-state)
  - [怎樣減少 store 更新事件的數量？](#performance-update-events)
  - [僅有 “一個 state 樹” 會引發記憶體問題嗎？分發多個 action 會佔用記憶體空間嗎？](#performance-state-memory)
- **React Redux**
  - [為何元件沒有被重新渲染、或者 mapStateToProps 沒有執行？](#react-not-rerendering)
  - [為何元件頻繁的重新渲染？](#react-rendering-too-often)
  - [怎樣使 mapStateToProps 執行更快？](#react-mapstate-speed)
  - [為何不在被連線的元件中使用 this.props.dispatch ？](#react-props-dispatch)
  - [應該只連線到頂層元件嗎，或者可以在元件樹中連線到不同元件嗎？](#react-multiple-components)
- **其它**
  - [有 “真實存在” 且很龐大的 Redux 項目嗎？](#miscellaneous-real-projects)
  - [如何在 Redux 中實現鑑權？](#miscellaneous-authentication)


## 綜合

<a id="general-when-to-use"></a>
### 何時使用 Redux？

React 早起貢獻者之一 Pete Hunt 說：

> 你應當清楚何時需要 Flux。如果你不確定是否需要它，那麼其實你並不需要它。

Redux 的建立者之一 Dan Abramov 也曾表達過類似的意思:

> 我想修正一個觀點：當你在使用 React 遇到問題時，才使用 Redux。

一般而言，Redux 的適應場景有以下幾點特徵：

- 隨著時間的推移，資料處於合理的變動之中；
- 單一資料來源
- 在 React 頂層元件 state 中維護所有內容的辦法已經無法滿足需求

#### 補充資料

**文件**
- [Introduction: Motivation](introduction/Motivation.md)

**討論**
- [React How-To](https://github.com/petehunt/react-howto)
- [Twitter: Don't use Redux until...](https://twitter.com/dan_abramov/status/699241546248536064)
- [The Case for Flux](https://medium.com/swlh/the-case-for-flux-379b7d1982c6)
- [Stack Overflow: Why use Redux over Facebook Flux？](http://stackoverflow.com/questions/32461229/why-use-redux-over-facebook-flux)
- [Stack Overflow: Why should I use Redux in this example？](http://stackoverflow.com/questions/35675339/why-should-i-use-redux-in-this-example)
- [Stack Overflow: What could be the downsides of using Redux instead of Flux？](http://stackoverflow.com/questions/32021763/what-could-be-the-downsides-of-using-redux-instead-of-flux)

<a id="general-only-react"></a>
### Redux 只能搭配 React 使用？

Redux 能作為任何 UI 層的 store。通常是與 React 或 React Native 搭配使用，但是也可以繫結 Angular、 Angular 2、 Vue、 Mithril 等框架使用。 Redux 提供的訂閱機制，可以與任何程式碼整合。即便如此，只有在結合聲明式，即 UI 隨 state 變化的檢視層實現時，才能發揮它的最大作用。

<a id="general-build-tools"></a>
### Redux 需要特殊的編譯工具支援嗎？

Redux 寫法遵循 ES6 語法，但在釋出時被 Webpack 和 Babel 編譯成了 ES5，所以在使用時可以忽略 JavaScript 的編譯過程。 Redux 也提供了 UMD 版本，可以直接使用而不需要任何編譯過程。[counter-vanilla](https://github.com/reactjs/redux/tree/master/examples/counter-vanilla) 示例展示了 Redux 基本的 ES5 用法。正如相關 pull 請求中的說法：

> Counter Vanilla 例子意圖是消除 Redux 需要 Webpack、 React、 熱過載、 sagas、 action 建立函數、 constants、 Babel、 npm、 CSS 模組化、 decorators、 fluent Latin、 Egghead subscription、 博士學位或者需要達到 Exceeds Expectations O.W.L. 這一級別的荒謬觀點。

> 僅僅是 HTML， 一些 `<script>`  標籤，和簡單的 DOM 操作而已。


## Reducers

<a id="reducers-share-state"></a>
### 如何在 reducer 之間共享 state ？ combineReducers 是必須的嗎？

Redux store 推薦的結構是將 state 物件按鍵值切分成 “層” 或者 “域”，並提供獨立的 reducer 方法管理各自的資料層。就像 Flux 模式中的多個獨立 store 一樣， Redux 為此還提供了 [`combineReducers`](api/combineReducers.md) 工具來簡化該模型。應當注意的是， `combineReducers` **不是** 必須的，它僅僅是通過簡單的 JavaScript 物件作為資料，讓 state 層能與 reducer 一一關聯的函數而已。

許多使用者想在 reducer 之間共享資料，但是 `combineReducers` 不允許此種行為。有許多可用的辦法：

* 如果一個 reducer 想獲取其它 state 層的資料，往往意味著 state 樹需要重構，需要讓單獨的 reducer 處理更多的資料。
* 你可能需要自定義方法去處理這些 action，用自定義的頂層 reducer 方法替換 `combineReducers`。你可以使用類似於 [reduce-reducers](https://github.com/acdlite/reduce-reducers) 的工具執行 `combineReducers` 去處理儘可能多的 action，同時還要為存在 state 交叉部分的若干 action 執行更專用的 reducer。
* 類似於 `redux-thunk` 的 [非同步 action 建立函數](advanced/AsyncActions.md) 能通過 `getState()` 方法獲取所有的 state。 action 建立函數能從 state 中檢索到額外的資料並傳入 action，所以 reducer 有足夠的資訊去更新所維護的 state 層。

只需牢記 reducer 僅僅是函數，可以隨心所欲的進行劃分和組合，而且也推薦將其分解成更小、可複用的函數 (“reducer 合成”)。按照這種做法，如果子 reducer 需要一些參數時，可以從父 reducer 傳入。你只需要確保他們遵循 reducer 的基本準則： `(state, action) => newState`，並且以不可變的方式更新 state，而不是直接修改 state。

#### 補充資料

**文件**
- [API: combineReducers](api/combineReducers.md)

**討論**
- [#601: A concern on combineReducers, when an action is related to multiple reducers](https://github.com/reactjs/redux/issues/601)
- [#1400: Is passing top-level state object to branch reducer an anti-pattern？](https://github.com/reactjs/redux/issues/1400)
- [Stack Overflow: Accessing other parts of the state when using combined reducers？](http://stackoverflow.com/questions/34333979/accessing-other-parts-of-the-state-when-using-combined-reducers)
- [Stack Overflow: Reducing an entire subtree with redux combineReducers](http://stackoverflow.com/questions/34427851/reducing-an-entire-subtree-with-redux-combinereducers)
- [Sharing State Between Redux Reducers](https://invalidpatent.wordpress.com/2016/02/18/sharing-state-between-redux-reducers/)

<a id="reducers-use-switch"></a>
### 處理 action 必須用 `switch` 語句嗎？？

不是。在 reducer 裡面你可以使用任何方法響應 action。 `switch` 語句是最常用的方式，當然你也可以用 `if`、功能查詢表、建立抽象函數等。

#### 補充資料

**文件**
- [Recipes: Reducing Boilerplate](recipes/ReducingBoilerplate.md)

**討論**
- [#883: take away the huge switch block](https://github.com/reactjs/redux/issues/883)
- [#1167: Reducer without switch](https://github.com/reactjs/redux/issues/1167)


## 組織 State

<a id="organizing-state-only-redux-state"></a>
### 必須將所有 state 都維護在 Redux 中嗎？ 可以用 React 的 `setState()` 方法嗎？

沒有 “標準”。有些使用者選擇將所有資料都在 Redux 中維護，那麼在任何時刻，應用都是完全有序及可控的。也有人將類似於“下拉選單是否開啟”的非關鍵或者 UI 狀態，在元件內部維護。適合自己的才是最好的。

有許多開源元件實現了各式各樣在 Redux store 儲存獨立元件狀態的替代方法，比如 [redux-ui](https://github.com/tonyhb/redux-ui)、 [redux-component](https://github.com/tomchentw/redux-component)、 [redux-react-local](https://github.com/threepointone/redux-react-local)等等。

#### 補充資料

**討論**

- [#159: Investigate using Redux for pseudo-local component state](https://github.com/reactjs/redux/issues/159)
- [#1098: Using Redux in reusable React component](https://github.com/reactjs/redux/issues/1098)
- [#1287: How to choose between Redux's store and React's state？](https://github.com/reactjs/redux/issues/1287)
- [#1385: What are the disadvantages of storing all your state in a single immutable atom？](https://github.com/reactjs/redux/issues/1385)
- [Stack Overflow: Why is state all in one place, even state that isn't global？](http://stackoverflow.com/questions/35664594/redux-why-is-state-all-in-one-place-even-state-that-isnt-global)
- [Stack Overflow: Should all component state be kept in Redux store？](http://stackoverflow.com/questions/35328056/react-redux-should-all-component-states-be-kept-in-redux-store)

<a id="organizing-state-non-serializable"></a>
### 可以將 store 的 state 設定為函數、promise或者其它非序列化值嗎？

強烈推薦只在 store 中維護普通的可序列化物件、陣列以及基本資料類型。雖然從 *技術* 層面上將非序列化項儲存在 store 中是可行的，但這樣會破壞 store 內容持久化和深加工的能力。

#### 補充資料

**討論**
- [#1248: Is it ok and possible to store a react component in a reducer？](https://github.com/reactjs/redux/issues/1248)
- [#1279: Have any suggestions for where to put a Map Component in Flux？](https://github.com/reactjs/redux/issues/1279)
- [#1390: Component Loading](https://github.com/reactjs/redux/issues/1390)
- [#1407: Just sharing a great base class](https://github.com/reactjs/redux/issues/1407)

<a id="organizing-state-nested-data"></a>
### 如何在 state 中組織巢狀及重複資料？

當資料存在 ID、巢狀或者關聯關係時，應當以 “正規化化” 形式儲存：物件只能儲存一次，ID 作為鍵值，物件間通過 ID 相互引用。將 store 類比於資料庫，每一項都是獨立的 “表”。[normalizr](https://github.com/gaearon/normalizr) 、 [redux-orm](https://github.com/tommikaikkonen/redux-orm) 此類的庫能在管理規範化資料時提供參考和抽象。

#### 補充資料

**文件**
- [Advanced: Async Actions](advanced/AsyncActions.md)
- [Examples: Real World example](introduction/Examples.html#real-world)

**討論**
- [#316: How to create nested reducers？](https://github.com/reactjs/redux/issues/316)
- [#815: Working with Data Structures](https://github.com/reactjs/redux/issues/815)
- [#946: Best way to update related state fields with split reducers？](https://github.com/reactjs/redux/issues/946)
- [#994: How to cut the boilerplate when updating nested entities？](https://github.com/reactjs/redux/issues/994)
- [#1255: Normalizr usage with nested objects in React/Redux](https://github.com/reactjs/redux/issues/1255)
- [Twitter: state shape should be normalized](https://twitter.com/dan_abramov/status/715507260244496384)

## 建立 Store

<a id="store-setup-multiple-stores"></a>
### 可以建立多個 store 嗎，應該這麼做嗎？能在元件中直接引用 store 並使用嗎？

Flux 原始模型中一個應用有多個 “store”，每個都維護了不同維度的資料。這樣導致了類似於一個 store “等待” 另一 store 操作的問題。Redux 中將 reducer 分解成多個小而美的 reducer，進而切分資料域，避免了這種情況的發生。

正如上述問題所述，“可能” 在一個頁面中建立多個獨立的 Redux store，但是預設模式中只會有一個 store。僅維持單個 store 不僅可以使用 Redux DevTools，還能簡化資料的持久化及深加工、精簡訂閱的邏輯處理。

在 Redux 中使用多個 store 的理由可能包括：

* 對應用進行效能分析時，解決由於過於頻繁更新部分 state 引起的效能問題。
* 在更大的應用中 Redux 只是作為一個元件，這種情況下，你也許更傾向於為每個根元件建立單獨的 store。

然而，建立新的 store 不應成為你的第一反應，特別是當你從 Flux 背景遷移而來。首先嚐試組合 reducer，只有當它無法解決你的問題時才使用多個 store。

類似的，雖然你 *能* 直接匯入並獲取 store 例項，但這並非 Redux 的推薦方式。當你建立 store 例項並從元件匯出，它將變成一個單例。這意味著將很難把 Redux 應用封裝成一個應用的子元件，除非這是必要的，或者為了實現服務端渲染，因為在服務端你需要為每一個請求建立單獨的 store 例項。

藉助 [React Redux](https://github.com/rackt/react-redux)，由 `connect()` 生成的包裝類實際上會檢索存在的 `props.store`，但還是推薦將根元件包裝在 `<Provider store={store}>` 中，這樣傳遞 store 的任務都交由 React Redux 處理。這種方式，我們不用考慮 store 模組的匯入、 Redux 應用的封裝，後期支援伺服器渲染也將變得更為簡便。

#### 補充資料

**文件**
- [API: Store](api/Store.md)

**討論**
- [#1346: Is it bad practice to just have a 'stores' directory？](https://github.com/reactjs/redux/issues/1436)
- [Stack Overflow: Redux multiple stores, why not？](http://stackoverflow.com/questions/33619775/redux-multiple-stores-why-not)
- [Stack Overflow: Accessing Redux state in an action creator](http://stackoverflow.com/questions/35667249/accessing-redux-state-in-an-action-creator)
- [Gist: Breaking out of Redux paradigm to isolate apps](https://gist.github.com/gaearon/eeee2f619620ab7b55673a4ee2bf8400)

<a id="store-setup-middleware-chains"></a>
### 在 store enhancer 中可以存在多個 middleware 鏈嗎？ 在 middleware 方法中，`next` 和 `dispatch` 之間區別是什麼？

Redux middleware 就像一個連結串列。每個 middleware 方法既能呼叫 `next(action)` 傳遞 action 到下一個 middleware，也可以呼叫 `dispatch(action)` 重新開始處理，或者什麼都不做而僅僅終止 action 的處理程序。

建立 store 時， `applyMiddleware` 方法的入參定義了 middleware 鏈。定義多個鏈將無法正常執行，因為它們的 `dispatch` 引用顯然是不一樣的，而且不同的鏈也無法有效連線到一起。

#### 補充資料

**文件**
- [Advanced: Middleware](advanced/Middleware.md)
- [API: applyMiddleware](api/applyMiddleware.md)

**討論**
- [#1051: Shortcomings of the current applyMiddleware and composing createStore](https://github.com/reactjs/redux/issues/1051)
- [Understanding Redux Middleware](https://medium.com/@meagle/understanding-87566abcfb7a)
- [Exploring Redux Middleware](http://blog.krawaller.se/posts/exploring-redux-middleware/)

<a id="store-setup-subscriptions"></a>
### 怎樣只訂閱 state 的一部分變更？如何將分發的 action 作為訂閱的一部分？

Redux 提供了獨立的 `store.subscribe` 方法用於通知監聽器 store 的變更資訊。監聽器的回撥方法並沒有把當前的 state 作為入參，它僅僅代表了 *有些資料* 被更新。訂閱者的邏輯中呼叫 `getState()` 獲取當前的 state 值。

這個 API 是沒有依賴及副作用的底層介面，可以用於建立訂閱者邏輯。類似 React Redux 的 UI 繫結能為所有連線的元件都建立訂閱。也可以用於編寫智慧的新舊 state 比對方法，從而在某些內容變化時執行額外的邏輯處理。示例 [redux-watch](https://github.com/jprichardson/redux-watch) 和 [redux-subscribe](https://github.com/ashaffer/redux-subscribe) 提供不同的方式用於指定訂閱及處理變更。

新的 state 沒有傳遞給監聽者，目的是簡化 store enhancer 的實現，比如 Redux DevTools。此外，訂閱者旨在響應 state 值本身，而非 action。當 action 很重要且需要特殊處理時，使用 middleware 。

#### 補充資料

**文件**
- [Basics: Store](basics/Store.md)
- [API: Store](api/Store.md)

**討論**
- [#303: subscribe API with state as an argument](https://github.com/reactjs/redux/issues/303)
- [#580: Is it possible to get action and state in store.subscribe？](https://github.com/reactjs/redux/issues/580)
- [#922: Proposal: add subscribe to middleware API](https://github.com/reactjs/redux/issues/922)
- [#1057: subscribe listener can get action param？](https://github.com/reactjs/redux/issues/1057)
- [#1300: Redux is great but major feature is missing](https://github.com/reactjs/redux/issues/1300)

## Actions

<a id="actions-string-constants"></a>
### 為何 `type` 必須是字元串，或者至少可以被序列化？ 為什麼 action 類型應該作為常量？

和 state 一樣，可序列化的 action 使得若干 Redux 的經典特性變得可能，比如時間旅行偵錯程式、錄製和重放 action。若使用 `Symbol` 等去定義 `type` 值，或者用 `instanceof` 對 action 做自檢查都會破壞這些特性。字元串是可序列化的、自解釋型，所以是更好的選擇。注意，如果 action 目的是在 middleware 中處理，那麼使用 Symbols、 Promises 或者其它非可序列化值也是 *可以* 的。 action 只有當它們正真到達 store 且被傳遞給 reducer 時才需要被序列化。

因為效能原因，我們無法強制序列化 action，所以 Redux 只會校驗 action 是否是普通物件，以及 `type` 是否定義。其它的都交由你決定，但是確保資料是可序列化將對偵錯以及問題的重現有很大幫助。

封裝並集聚公共程式碼是程式規劃時的核心概念。雖然可以在任何地方手動建立 action 物件、手動指定 `type` 值，定義常量的方式使得程式碼的維護更為方便。如果將常量維護在單獨的檔案中，[在 `import` 時校驗](https://www.npmjs.com/package/eslint-plugin-import)，能避免偶然的拼寫錯誤。

#### 補充資料

**文件**
- [Reducing Boilerplate](http://rackt.github.io/redux/docs/recipes/ReducingBoilerplate.html#actions)

**Discussion**
- [#384: Recommend that Action constants be named in the past tense](https://github.com/reactjs/redux/issues/384)
- [#628: Solution for simple action creation with less boilerplate](https://github.com/reactjs/redux/issues/628)
- [#1024: Proposal: Declarative reducers](https://github.com/reactjs/redux/issues/1024)
- [#1167: Reducer without switch](https://github.com/reactjs/redux/issues/1167)
- [Stack Overflow: Why do you need 'Actions' as data in Redux？](http://stackoverflow.com/q/34759047/62937)
- [Stack Overflow: What is the point of the constants in Redux？](http://stackoverflow.com/q/34965856/62937)

<a id="actions-reducer-mappings"></a>
### 是否存在 reducer 和 action 之間的一對一對映？

沒有。建議的方式是編寫獨立且很小的 reducer 方法去更新指定的 state 部分，這種模式被稱為 “reducer 合成”。一個指定的 action 也許被它們中的全部、部分、甚至沒有一個處理到。這種方式把元件從實際的資料變更中解耦，一個 action 可能影響到 state 樹的不同部分，對元件而言再也不必知道這些了。有些使用者選擇將它們緊密繫結在一起，就像 “ducks” 檔案結構，顯然是沒有預設的一對一對映。所以當你想在多個 reducer 中處理同一個 action 時，應當避免此類結構。

#### 補充資料

**文件**
- [Basics: Reducers](basics/Reducers.md)

**討論**
- [Twitter: most common Redux misconception](https://twitter.com/dan_abramov/status/682923564006248448)
- [#1167: Reducer without switch](https://github.com/reactjs/redux/issues/1167)
- [Reduxible #8: Reducers and action creators aren't a one-to-one mapping](https://github.com/reduxible/reduxible/issues/8)
- [Stack Overflow: Can I dispatch multiple actions without Redux Thunk middleware？](http://stackoverflow.com/questions/35493352/can-i-dispatch-multiple-actions-without-redux-thunk-middleware/35642783)

<a id="actions-side-effects"></a>
### 怎樣表示類似 AJAX 請求的 “副作用”？為何需要 “action 建立函數”、“thunks” 以及 “middleware” 類似的東西去處理非同步行為？

這是一個持久且複雜的話題，針對如何組織程式碼以及採用何種方式有很多的觀點。

任何有價值的 web 應用都必然要執行復雜的邏輯，通常包括 AJAX 請求等非同步工作。這類程式碼不再是針對輸入的純函數，與第三方的互動被認為是 [“副作用”](https://en.wikipedia.org/wiki/Side_effect_%28computer_science%29)。

Redux 深受函數語言程式設計的影響，創造性的不支援副作用的執行。尤其是 reducer， *必須* 是符合  `(state, action) => newState` 的純函數。然而，Redux 的 middleware 能攔截分發的 action 並新增額外的複雜行為，有副作用時也是如此。

Redux 建議將帶副作用的程式碼作為 action 建立過程的一部分。因為該邏輯 *能* 在 UI 元件內執行，那麼通常抽取此類邏輯作為可重用的方法都是有意義的，因此同樣的邏輯能被多個地方呼叫，也就是所謂的 action 建立函數。

最簡單也是最常用的方法就是新增 [Redux Thunk](https://github.com/gaearon/redux-thunk) middleware，這樣就能用更為複雜或者非同步的邏輯書寫 action 建立函數。另一個被廣泛使用的方法是 [Redux Saga](https://github.com/yelouafi/redux-saga)，你可以用 generator 書寫類同步程式碼，就像在 Redux 應用中使用 “後臺執行緒” 或者 “守護程序”。還有一個方法是 [Redux Loop](https://github.com/raisemarketplace/redux-loop)，它允許 reducer 以聲明副作用的方式去響應 state 變化，並讓它們分別執行，從而反轉了程序。除此之外，還有 *許多* 其它開源的庫和理念，都有各自針對副作用的管理方法。

#### 補充資料
**文件**
- [Advanced: Async Actions](advanced/AsyncActions.md)
- [Advanced: Async Flow](advanced/AsyncFlow.md)
- [Advanced: Middleware](advanced/Middleware.md)

**討論**
- [#291: Trying to put API calls in the right place](https://github.com/reactjs/redux/issues/291)
- [#455: Modeling side effects](https://github.com/reactjs/redux/issues/455)
- [#533: Simpler introduction to async action creators](https://github.com/reactjs/redux/issues/533)
- [#569: Proposal: API for explicit side effects](https://github.com/reactjs/redux/pull/569)
- [#1139: An alternative side effect model based on generators and sagas](https://github.com/reactjs/redux/issues/1139)
- [Stack Overflow: Why do we need middleware for async flow in Redux？](http://stackoverflow.com/questions/34570758/why-do-we-need-middleware-for-async-flow-in-redux)
- [Stack Overflow: How to dispatch a Redux action with a timeout？](http://stackoverflow.com/questions/35411423/how-to-dispatch-a-redux-action-with-a-timeout/35415559)
- [Stack Overflow: Where should I put synchronous side effects linked to actions in redux？](http://stackoverflow.com/questions/32982237/where-should-i-put-synchronous-side-effects-linked-to-actions-in-redux/33036344)
- [Stack Overflow: How to handle complex side-effects in Redux？](http://stackoverflow.com/questions/32925837/how-to-handle-complex-side-effects-in-redux/33036594)
- [Stack Overflow: How to unit test async Redux actions to mock ajax response](http://stackoverflow.com/questions/33011729/how-to-unit-test-async-redux-actions-to-mock-ajax-response/33053465)
- [Stack Overflow: How to fire AJAX calls in response to the state changes with Redux？](http://stackoverflow.com/questions/35262692/how-to-fire-ajax-calls-in-response-to-the-state-changes-with-redux/35675447)
- [Reddit: Help performing Async API calls with Redux-Promise Middleware.](https://www.reddit.com/r/reactjs/comments/469iyc/help_performing_async_api_calls_with_reduxpromise/)
- [Twitter: possible comparison between sagas, loops, and other approaches](https://twitter.com/dan_abramov/status/689639582120415232)
- [Redux Side-Effects and You](https://medium.com/@fward/redux-side-effects-and-you-66f2e0842fc3)
- [Pure functionality and side effects in Redux](http://blog.hivejs.org/building-the-ui-2/)

<a id="actions-multiple-actions"></a>
### 是否應該在 action 建立函數中連續分發多個 action？

關於如何構建 action 並沒有統一的規範。使用類似 Redux Thunk 的非同步 middleware 支援了更多的場景，比如分發連續多個獨立且相關聯的 action、 分發 action 指示 AJAX 請求的階段、 根據 state 有條件的分發 action、甚至分發 action 並隨後校驗更新的 state。

通常，明確這些 action 是關聯還是獨立，是否應當作為一個 action。評判當前場景影響因素的同時，還需根據 action 日誌權衡 reducer 的可讀性。例如，一個包含新 state 樹的 action 會使你的 reducer 只有一行，副作用是沒有任何歷史表明 *為什麼* 發生了變更，進而導致偵錯異常困難。另一方面，如果為了維持它們的粒狀結構（granular），在迴圈中分發 action，這表明也許需要引入新的 acton 類型並以不同的方式去處理它。

避免在同一地方連續多次以同步的方式進行分發，其效能問題是值得擔憂的。如果使用 React，將多個同步分發包裝在 `ReactDOM.unstable_batchedUpdates()` 方法中能改善效能，但是這個 API 是實驗性質的，也許在某個 React 版本中就被移除了，所以不要過度依賴它。可以參考 [redux-batched-actions](https://github.com/tshelburne/redux-batched-actions)，分發多個 action 就如分發一個這麼簡單，並且會在 reducer 中將它們 “解包”。[redux-batched-subscribe](https://github.com/tappleby/redux-batched-subscribe) 讓你為多次分發分別呼叫訂閱者。

#### 補充資料

**討論**
- [#597: Valid to dispatch multiple actions from an event handler？](https://github.com/reactjs/redux/issues/597)
- [#959: Multiple actions one dispatch？](https://github.com/reactjs/redux/issues/959)
- [Stack Overflow: Should I use one or several action types to represent this async action？](http://stackoverflow.com/questions/33637740/should-i-use-one-or-several-action-types-to-represent-this-async-action/33816695)
- [Stack Overflow: Do events and actions have a 1:1 relationship in Redux？](http://stackoverflow.com/questions/35406707/do-events-and-actions-have-a-11-relationship-in-redux/35410524)
- [Stack Overflow: Should actions be handled by reducers to related actions or generated by action creators themselves？](http://stackoverflow.com/questions/33220776/should-actions-like-showing-hiding-loading-screens-be-handled-by-reducers-to-rel/33226443#33226443)


## 程式碼結構

<a id="structure-file-structure"></a>
### 檔案結構應該是什麼樣？項目中該如何對 action 建立函數和 reducer 分組？ selector 又該放在哪裡？

因為 Redux 只是資料儲存的庫，它沒有關於工程應該被如何組織的直接主張。然後，有一些被大多數 Redux 開發者所推薦的模式：

- Rails-style：“actions”、“constants”、“reducers”、“containers” 以及 “components” 分屬不同的資料夾
- Domain-style：為每個功能或者域建立單獨的資料夾，可能會為某些檔案類型建立子資料夾
- “Ducks”：類似於 Domain-style，但是明確地將 action、 reducer 繫結在一起，通常將它們定義在同一檔案內。

推薦做法是將 selector 與 reducer 定義在一起並輸出，並在 reducer 檔案中與知道 state 樹真實形狀的程式碼一起被重用（例如在 `mapStateToProps` 方法、非同步 action 建立函數，或者 sagas）。

#### 補充資料

**討論**
- [#839: Emphasize defining selectors alongside reducers](https://github.com/reactjs/redux/issues/839)
- [#943: Reducer querying](https://github.com/reactjs/redux/issues/943)
- [React Boilerplate #27: Application Structure](https://github.com/mxstbr/react-boilerplate/issues/27)
- [Stack Overflow: How to structure Redux components/containers](http://stackoverflow.com/questions/32634320/how-to-structure-redux-components-containers/32921576)
- [Redux Best Practices](https://medium.com/lexical-labs-engineering/redux-best-practices-64d59775802e)
- [Rules For Structuring (Redux) Applications ](http://jaysoo.ca/2016/02/28/organizing-redux-application/)
- [A Better File Structure for React/Redux Applications](http://marmelab.com/blog/2015/12/17/react-directory-structure.html)
- [Organizing Large React Applications](http://engineering.kapost.com/2016/01/organizing-large-react-applications/)
- [Four Strategies for Organizing Code](https://medium.com/@msandin/strategies-for-organizing-code-2c9d690b6f33)

<a id="structure-business-logic"></a>
### 如何將邏輯在 reducer 和 action 建立函數之間劃分？ “業務邏輯” 應該放在哪裡？

關於邏輯的哪個部分應該放在 reducer 或者 action 建立函數中，沒有清晰的答案。一些開發者喜歡 “fat” action 建立函數，“thin” reducer 僅僅從 action 拿到資料並繫結到 state 樹。其他人的則強調 action 越簡單越好，儘量減少在 action 建立函數中使用 `getState()` 方法。

下面的評論恰如其分的概括了這兩種分歧：

> 問題是什麼在 action 建立函數中、什麼在 reducer 中，就是關於 fat 和 thin action 建立函數的選擇。如果你將邏輯都放在 action 建立函數中，最終用於更新 state 的 action 物件就會變得 fat，相應的 reducer 就變得純淨、簡潔。因為只涉及很少的業務邏輯，將非常有利於組合。
> 如果你將大部門邏輯置於 reducer 之中，action 將變得精簡、美觀，大部分資料邏輯都在一個地方維護，但是 reducer 由於引用了其它分支的資訊，將很難組合。最終的 reducer 會很龐大，而且需要從更高層的 state 獲取額外資訊。

當你從這兩種極端情況中找到一個平衡時，就意味著你已經掌握了 Redux。

#### 補充資料

**討論**
- [#1165: Where to put business logic / validation？](https://github.com/reactjs/redux/issues/1165)
- [#1171: Recommendations for best practices regarding action-creators, reducers, and selectors](https://github.com/reactjs/redux/issues/1171 )
- [Stack Overflow: Accessing Redux state in an action creator？](http://stackoverflow.com/questions/35667249/accessing-redux-state-in-an-action-creator/35674575)


## Performance

<a id="performance-scaling"></a>
### 考慮到效能和架構， Redux “可擴充套件性” 如何？

因為沒有一個明確的答案，所以在大多數情況下都不需要考慮該問題。

Redux 所做的工作可以分為以下幾部分：在 middleware 和 reducer 中處理 action （包括物件複製及不可變更新）、 action 分發之後通知訂閱者、根據 state 變化更新 UI 元件。雖然在一些複雜場景下，這些都 *可能* 變成一個效能問題，Redux 本質上並沒有任何慢或者低效的實現。實際上，React Redux 已經做了大量的優化工作減少不必要的重複渲染。

考慮到架構方面，事實證據表明在各種項目及團隊規模下，Redux 都表現出色。Redux 目前正被成百上千的公司以及更多的開發者使用著，NPM 上每月都有幾十萬的安裝量。有一位開發者這樣說：

> 規模方面，我們大約有500個 action 類型、400個 reducer、150個元件、5個 middleware、200個 action、2300個測試案例。

#### 補充資料

**討論**
- [#310: Who uses Redux？](https://github.com/reactjs/redux/issues/310)
- [Reddit: What's the best place to keep the initial state？](https://www.reddit.com/r/reactjs/comments/47m9h5/whats_the_best_place_to_keep_the_initial_state/)
- [Reddit: Help designing Redux state for a single page app](https://www.reddit.com/r/reactjs/comments/48k852/help_designing_redux_state_for_a_single_page/)
- [Reddit: Redux performance issues with a large state object？](https://www.reddit.com/r/reactjs/comments/41wdqn/redux_performance_issues_with_a_large_state_object/)
- [Reddit: React/Redux for Ultra Large Scale apps](https://www.reddit.com/r/javascript/comments/49box8/reactredux_for_ultra_large_scale_apps/)
- [Twitter: Redux scaling](https://twitter.com/NickPresta/status/684058236828266496)

<a id="performance-all-reducers"></a>
### 每個 action 都呼叫 “所有的 reducer” 會不會很慢？

我們應當清楚的認識到 Redux store 只有一個 reducer 方法。 store 將當前的 state 和分發的 action 傳遞給這個 reducer 方法，剩下的就讓 reducer 去處理。

顯然，在單獨的方法裡處理所有的 action 僅從方法大小及可讀性方面考慮，就已經很不利於擴充套件了，所以將實際工作分割成獨立的方法並在頂層的 reducer 中呼叫就變得很有意義。尤其是目前的建議模式中推薦讓單獨的子 reducer 只負責更新特定的 state 部分。 `combineReducers()` 和 Redux 搭配的方案只是許多實現方式中的一種。強烈建議儘可能保持 store 中 state 的扁平化和正規化化，至少你可以隨心所欲的組織你的 reducer 邏輯。

即使你在不經意間已經維護了許多獨立的子 reducer，甚至 state 也是深度巢狀，reducer 的速度也並不構成任何問題。JavaScript 引擎有足夠的能力在每秒執行大量的函數呼叫，而且大部門的子 reducer 只是使用 `switch` 語句，並且針對大部分 action 返回的都是預設的 state。

如果你仍然關心 reducer 的效能，可以使用類似 [redux-ignore](https://github.com/omnidan/redux-ignore) 和 [reduxr-scoped-reducer](https://github.com/chrisdavies/reduxr-scoped-reducer) 的工具，確保只有某幾個 reducer 響應特定的 action。你還可以使用 [redux-log-slow-reducers](https://github.com/michaelcontento/redux-log-slow-reducers) 進行效能測試。

#### 補充資料

**討論**
- [#912: Proposal: action filter utility](https://github.com/reactjs/redux/issues/912)
- [#1303: Redux Performance with Large Store and frequent updates](https://github.com/reactjs/redux/issues/1303)
- [Stack Overflow: State in Redux app has the name of the reducer](http://stackoverflow.com/questions/35667775/state-in-redux-react-app-has-a-property-with-the-name-of-the-reducer/35674297)
- [Stack Overflow: How does Redux deal with deeply nested models？](http://stackoverflow.com/questions/34494866/how-does-redux-deals-with-deeply-nested-models/34495397)

<a id="performance-clone-state"></a>
### 在 reducer 中必須對 state 進行深拷貝嗎？拷貝 state 不會很慢嗎？

以不可變的方式更新 state 意味著淺拷貝，而非深拷貝。

相比於深拷貝，淺拷貝更快，因為只需複製很少的欄位和物件，實際的底層實現中也只是移動了若干指針而已。

因此，你需要建立一個副本，並且更新受影響的各個巢狀的物件層級即可。儘管上述動作代價不會很大，但這也是為什麼需要維護正規化化及扁平化 state 的又一充分理由。

> Redux 常見的誤解： 需要深拷貝 state。實際情況是：如果內部的某些資料沒有改變，繼續保持統一引用即可。

#### 補充資料

**討論**
- [#454: Handling big states in reducer](https://github.com/reactjs/redux/issues/454)
- [#758: Why can't state be mutated？](https://github.com/reactjs/redux/issues/758)
- [#994: How to cut the boilerplate when updating nested entities？](https://github.com/reactjs/redux/issues/994)
- [Twitter: common misconception - deep cloning](https://twitter.com/dan_abramov/status/688087202312491008)
- [Cloning Objects in JavaScript](http://www.zsoltnagy.eu/cloning-objects-in-javascript/)

<a id="performance-update-events"></a>
### 怎樣減少 store 更新事件的數量？

Redux 在 action 分發成功（例如，action 到達 store 被 reducer 處理）後通知訂閱者。在有些情況下，減少訂閱者被呼叫的次數會很有用，特別在當 action 建立函數分發了一系列不同的 action 時。有很多開源的元件提供了在多個 action 分發時，批量訂閱通知的擴充套件，比如 [redux-batched-subscribe](https://github.com/tappleby/redux-batched-subscribe) 和 [redux-batched-actions](https://github.com/tshelburne/redux-batched-actions)。

#### 補充資料

**討論**
- [#125: Strategy for avoiding cascading renders](https://github.com/reactjs/redux/issues/125)
- [#542: Idea: batching actions](https://github.com/reactjs/redux/issues/542)
- [#911: Batching actions](https://github.com/reactjs/redux/issues/911)
- [React Redux #263: Huge performance issue when dispatching hundreds of actions](https://github.com/reactjs/react-redux/issues/263)

<a id="performance-state-memory"></a>
### 僅有 “一個 state 樹” 會引發記憶體問題嗎？分發多個 action 會佔用記憶體空間嗎？

首先，在原始記憶體使用方面，Redux 和其它的 JavaScript 庫並沒有什麼不同。唯一的區別就是所有的物件引用都巢狀在同一棵樹中，而不是像類似於 Backbone 那樣儲存在不同的模型例項中。第二，與同樣的 Backbone 應用相比，典型的 Redux 應用可能使用 *更少* 的記憶體，因為 Redux 推薦使用普通的 JavaScript 物件和陣列，而不是建立模型和集合例項。最後，Redux 僅維護一棵 state 樹。不再被引用的 state 樹通常都會被垃圾回收。

Redux 本身不儲存 action 的歷史。然而，Redux DevTools 會記錄這些 action 以便支援重放，而且也僅在開發環境被允許，生產環境則不會使用。

#### 補充資料

**文件**
- [Docs: Async Actions](advanced/AsyncActions.md])

**討論**
- [Stack Overflow: Is there any way to "commit" the state in Redux to free memory？](http://stackoverflow.com/questions/35627553/is-there-any-way-to-commit-the-state-in-redux-to-free-memory/35634004)
- [Reddit: What's the best place to keep initial state？](https://www.reddit.com/r/reactjs/comments/47m9h5/whats_the_best_place_to_keep_the_initial_state/)

## React Redux

<a id="react-not-rerendering"></a>
### 為何元件沒有被重新渲染、或者 mapStateToProps 沒有執行？

目前來看，導致元件在 action 分發後卻沒有被重新渲染，最常見的原因是對 state 進行了直接修改。Redux 期望 reducer 以 “不可變的方式” 更新 state，實際使用中則意味著複製資料，然後更新資料副本。如果直接返回同一物件，即使你改變了資料內容，Redux 也會認為沒有變化。類似的，React Redux 會在 `shouldComponentUpdate` 中對新的 props 進行淺層的判等檢查，以期提升效能。如果所有的引用都是相同的，則返回 `false` 從而跳過此次對元件的更新。

需要注意的是，不管何時更新了一個巢狀的值，都必須同時返回上層的任何資料副本給 state 樹。如果資料是 `state.a.b.c.d`，你想更新 `d`，你也必須返回 `c`、`b`、`a` 以及 `state` 的拷貝。[state 樹變化圖](http://arqex.com/wp-content/uploads/2015/02/trees.png) 展示了樹的深層變化為何需要改變途經的結點。

“以不可變的方式更新資料” 並 *不* 代表你必須使用 [Immutable.js](https://facebook.github.io/immutable-js/), 雖然是很好的選擇。你可以使用多種方法，達到對普通 JS 物件進行不可變更新的目的：

- 使用類似於 `Object.assign()` 或者 `_.extend()` 的方法複製物件， `slice()` 和 `concat()` 方法複製陣列。
- ES6 陣列的 spread sperator（展開運算符），JavaScript 新版本提案中類似的物件展開運算符。
- 將不可變更新邏輯包裝成簡單方法的工具庫。

#### 補充資料

**文件**
- [Troubleshooting](Troubleshooting.md)
- [React Redux: Troubleshooting](https://github.com/reactjs/react-redux/blob/master/docs/troubleshooting.md)
- [Recipes: Using the Object Spread Operator](recipes/UsingObjectSpreadOperator.md)

**討論**
- [#1262: Immutable data + bad performance](https://github.com/reactjs/redux/issues/1262)
- [React Redux #235: Predicate function for updating component](https://github.com/reactjs/react-redux/issues/235)
- [React Redux #291: Should mapStateToProps be called every time an action is dispatched？](https://github.com/reactjs/react-redux/issues/291)
- [Stack Overflow: Cleaner/shorter way to update nested state in Redux？](http://stackoverflow.com/questions/35592078/cleaner-shorter-way-to-update-nested-state-in-redux)
- [Gist: state mutations](https://gist.github.com/amcdnl/7d93c0c67a9a44fe5761#gistcomment-1706579)
- [Pros and Cons of Using Immutability with React](http://reactkungfu.com/2015/08/pros-and-cons-of-using-immutability-with-react-js/)

<a id="react-rendering-too-often"></a>
### 為何元件頻繁的重新渲染？

React Redux 採取了很多的優化手段，保證元件直到必要時才執行重新渲染。一種是對 `mapStateToProps` 和 `mapDispatchToProps` 生成後傳入 `connect` 的 props 物件進行淺層的判等檢查。遺憾的是，如果當 `mapStateToProps` 呼叫時都生成新的陣列或物件例項的話，此種情況下的淺層判等不會起任何作用。一個典型的示例就是通過 ID 陣列返回對映的物件引用，如下所示：

```js
const mapStateToProps = (state) => {
  return {
    objects: state.objectIds.map(id => state.objects[id])
  }
}
```

儘管每次陣列內都包含了同樣的物件引用，陣列本身卻指向不同的引用，所以淺層判等的檢查結果會導致 React Redux 重新渲染包裝的元件。

這種額外的重新渲染也可以避免，使用 reducer 將物件陣列儲存到 state，利用 [Reselect](https://github.com/reactjs/reselect) 快取對映的陣列，或者在元件的 `shouldComponentUpdate` 方法中，採用 `_.isEqual` 等對 props 進行更深層次的比較。注意在自定義的 `shouldComponentUpdate()` 方法中不要採用了比重新渲染本身更為昂貴的實現。可以使用分析器評估方案的效能。

對於獨立的元件，也許你想檢查傳入的 props。一個普遍存在的問題就是在 render 方法中繫結父元件的回撥，比如 `<Child onClick={this.handleClick.bind(this)} />`。這樣就會在每次父元件重新渲染時重新生成一個函數的引用。所以只在父元件的建構函式中繫結一次回撥是更好的做法。

#### 補充資料

**討論**
- [Stack Overflow: Can a React Redux app scale as well as Backbone？](http://stackoverflow.com/questions/34782249/can-a-react-redux-app-really-scale-as-well-as-say-backbone-even-with-reselect)
- [React.js pure render performance anti-pattern](https://medium.com/@esamatti/react-js-pure-render-performance-anti-pattern-fb88c101332f)
- [A Deep Dive into React Perf Debugging](http://benchling.engineering/deep-dive-react-perf-debugging/)

<a id="react-mapstate-speed"></a>
### 怎樣使 `mapStateToProps` 執行更快？

儘管 React Redux 已經優化並儘量減少對 `mapStateToProps` 的呼叫次數，加快 `mapStateToProps` 執行並減少其執行次數仍然是非常有價值的。普遍的推薦方式是利用 [Reselect](https://github.com/reactjs/reselect) 建立可記憶（memoized）的 “selector” 方法。這樣，selector 就能被組合在一起，並且同一管道（pipeline）後面的 selector 只有當輸入變化時才會執行。意味著你可以像篩選器或過濾器那樣建立 selector，並確保任務的執行時機。

#### 補充資料

**文件**
- [Recipes: Computed Derived Data](recipes/ComputingDerivedData.md)

**討論**
- [#815: Working with Data Structures](https://github.com/reactjs/redux/issues/815)
- [Reselect #47: Memoizing Hierarchical Selectors](https://github.com/reactjs/reselect/issues/47)

<a id="react-props-dispatch"></a>
### 為何不在被連線的元件中使用 `this.props.dispatch`？

`connect()` 方法有兩個主要的參數，而且都是可選的。第一個參數 `mapStateToProps` 是個函數，讓你在資料變化時從 store 獲取資料，並作為 props 傳到元件中。第二個參數 `mapDispatchToProps` 依然是函數，讓你可以使用 store 的 `dispatch` 方法，通常都是建立 action 建立函數並預先繫結，那麼在呼叫時就能直接分發 action。

如果在執行 `connect()` 時沒有指定 `mapDispatchToProps` 方法，React Redux 預設將 `dispatch` 作為 prop 傳入。所以當你指定方法時， `dispatch` 將 *不* 會自動注入。如果你還想讓其作為 prop，需要在 `mapDispatchToProps` 實現的返回值中明確指出。

#### 補充資料

**文件**
- [React Redux API: connect()](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options)

**討論**
- [React Redux #89: can i wrap multi actionCreators into one props with name？](https://github.com/reactjs/react-redux/issues/89)
- [React Redux #145: consider always passing down dispatch regardless of what mapDispatchToProps does](https://github.com/reactjs/react-redux/issues/145)
- [React Redux #255: this.props.dispatch is undefined if using mapDispatchToProps](https://github.com/reactjs/react-redux/issues/255)
- [Stack Overflow: How to get simple dispatch from this.props using connect w/ Redux？](http://stackoverflow.com/questions/34458261/how-to-get-simple-dispatch-from-this-props-using-connect-w-redux/34458710])

<a id="react-multiple-components"></a>
### 應該只連線到頂層元件嗎，或者可以在元件樹中連線到不同元件嗎？

早期的 Redux 文件中建議只在元件樹頂層附近連線若干元件。然而，時間和經驗都表明，這需要讓這些元件非常瞭解它們子孫元件的資料需求，還導致它們會向下傳遞一些令人困惑的 props。

目前的最佳實踐是將元件按照 “展現層（presentational）” 或者 “容器（container）” 分類，並在合理的地方抽象出一個連線的容器元件：

> Redux 示例中強調的 “在頂層保持一個容器元件” 是錯誤的。不要把這個當做準則。讓你的展現層元件保持獨立。然後建立容器元件並在合適時進行連線。當你感覺到你是在父元件裡通過複製程式碼為某些子元件提供資料時，就是時候抽取出一個容器了。只要你認為父元件過多瞭解子元件的資料或者 action，就可以抽取容器。

總之，試著在資料流和元件職責間找到平衡。

#### 補充資料

**文件**
- [Basics: Usage with React](basics/UsageWithReact.md)

**討論**
- [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)
- [Twitter: emphasizing “one container” was a mistake](https://twitter.com/dan_abramov/status/668585589609005056)
- [#419: Recommended usage of connect](https://github.com/reactjs/redux/issues/419)
- [#756: container vs component？](https://github.com/reactjs/redux/issues/756)
- [#1176: Redux+React with only stateless components](https://github.com/reactjs/redux/issues/1176)
- [Stack Overflow: can a dumb component use a Redux container？](http://stackoverflow.com/questions/34992247/can-a-dumb-component-use-render-redux-container-component)

## 其它

<a id="miscellaneous-real-projects"></a>
### 有 “真實存在” 且很龐大的 Redux 項目嗎？

Redux 的 “examples” 目錄下有幾個複雜度不一的工程，包括一個 “real-world” 示例。雖然有很多公司在使用 Redux，大部分的應用都有版權，無法獲得。依然可以在 Github 上找到大量的 Redux 相關項目，比如 [SoundRedux](https://github.com/andrewngu/sound-redux)。

#### 補充資料

**文件**
- [Introduction: Examples](introduction/Examples.md)

**討論**
- [Reddit: Large open source react/redux projects？](https://www.reddit.com/r/reactjs/comments/496db2/large_open_source_reactredux_projects/)
- [HN: Is there any huge web application built using Redux？](https://news.ycombinator.com/item？id=10710240)

<a id="miscellaneous-authentication"></a>
### 如何在 Redux 中實現鑑權？

在任何真正的應用中，鑑權都必不可少。當考慮鑑權時須謹記：不管你怎樣組織應用，都並不會改變什麼，你應當像實現其它功能一樣實現鑑權。這實際上很簡單：

1. 為 `LOGIN_SUCCESS`、`LOGIN_FAILURE` 等定義 action 常量。

2. 建立接受憑證的 action 建立函數，憑證是指示身份驗證成功與否的標誌、一個令牌、或者作為負載的錯誤資訊。

3. 使用 Redux Thunk middleware 或者其它適合於觸發網路請求（請求 API，如果是合法鑑權則返回令牌）的 middleware 建立一個非同步的 action 建立函數。之後在本地儲存中儲存令牌或者給使用者一個非法提示。可以通過執行上一步的 action 建立函數達到此效果。

4. 為每個可能出現的鑑權場景（`LOGIN_SUCCESS`、`LOGIN_FAILURE`等）編寫獨立的 reducer。

#### 補充資料

**討論**
- [Authentication with JWT by Auth0](https://auth0.com/blog/2016/01/04/secure-your-react-and-redux-app-with-jwt-authentication/)
- [Tips to Handle Authentication in Redux](https://medium.com/@MattiaManzati/tips-to-handle-authentication-in-redux-2-introducing-redux-saga-130d6872fbe7)
- [react-redux-jwt-auth-example](https://github.com/joshgeller/react-redux-jwt-auth-example)
- [redux-auth](https://github.com/lynndylanhurley/redux-auth)
