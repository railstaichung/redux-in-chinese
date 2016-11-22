# 先前技術

Redux 是一個混合產物。它和一些設計模式及技術相似，但也有不同之處。讓我們來探索一下這些相似與不同。

### Flux

Redux 可以被看作 [Flux](https://facebook.github.io/flux/) 的一種實現嗎？
[是](https://twitter.com/fisherwebdev/status/616278911886884864)，也可以說 [不是](https://twitter.com/andrestaltz/status/616270755605708800)。

（別擔心，它得到了[Flux 作者](https://twitter.com/jingc/status/616608251463909376)[的認可](https://twitter.com/fisherwebdev/status/616286955693682688)，如果你想確認。）

Redux 的靈感來源於 Flux 的幾個重要特性。和 Flux 一樣，Redux 規定，將模型的更新邏輯全部集中於一個特定的層（Flux 裡的 store，Redux 裡的 reducer）。Flux 和 Redux 都不允許程式直接修改資料，而是用一個叫作  “action” 的普通物件來對更改進行描述。

而不同於 Flux ，**Redux 並沒有 dispatcher 的概念**。原因是它依賴純函數來替代事件處理器。純函數構建簡單，也不需額外的實體來管理它們。你可以將這點看作這兩個框架的差異或細節實現，取決於你怎麼看 Flux。Flux 常常[被表述為 `(state, action) => state`](https://speakerdeck.com/jmorrell/jsconf-uy-flux-those-who-forget-the-past-dot-dot-dot)。從這個意義上說，Redux 無疑是 Flux 架構的實現，且得益於純函數而更為簡單。

和 Flux 的另一個重要區別，是 **Redux 設想你永遠不會變動你的資料**。你可以很好地使用普通物件和陣列來管理 state ，而不是在多個 reducer 裡變動資料。正確且簡便的方式是，你應該在 reducer 中返回一個新物件來更新 state， 同時配合 [object spread 運算符提案](../recipes/UsingObjectSpreadOperator.md) 或一些庫，如 [Immutable](https://facebook.github.io/immutable-js)。

雖然出於效能方面的考慮，[寫不純的 reducer](https://github.com/gaearon/redux/issues/328#issuecomment-125035516) 來變動資料在技術上是**可行**的，但我們並不鼓勵這麼做。不純的 reducer 會使一些開發特性，如時間旅行、記錄/回放或熱載入不可實現。此外，在大部分實際應用中，這種資料不可變動的特性並不會帶來效能問題，就像 [Om](https://github.com/omcljs/om) 所表現的，即使物件分配失敗，仍可以防止昂貴的重渲染和重計算。而得益於 reducer 的純度，應用內的變化更是一目瞭然。

### Elm

[Elm](http://elm-lang.org/) 是一種函數語言程式設計語言，由 [Evan Czaplicki](https://twitter.com/czaplic) 受 Haskell 語言的啟發開發。它執行一種 [“model view update” 的架構](http://elm-lang.org/guide/architecture) ，更新遵循 `(state, action) => state` 的規則。 Elm 的 “updater” 與 Redux 裡的 reducer 服務於相同的目的。

不同於 Redux，Elm 是一門語言，因此它在執行純度，靜態類型，不可變動性，action 和模式匹配等方面更具優勢。即使你不打算使用 Elm，也可以讀一讀 Elm 的架構，嘗試一把。基於此，有一個有趣的[使用 JavaScript 庫實現類似想法](https://github.com/paldepind/noname-functional-frontend-framework) 的項目。Redux 應該能從中獲得更多的啟發！ 為了更接近 Elm 的靜態類型，[Redux 可以使用一個類似 Flow 的漸進類型解決方案](https://github.com/gaearon/redux/issues/290) 。

### Immutable

[Immutable](https://facebook.github.io/immutable-js) 是一個可實現持久資料結構的 JavaScript 庫。它效能很好，並且命名符合 JavaScript API 的語言習慣 。

Immutable 及類似的庫都可以與 Redux 對接良好。儘可隨意捆綁使用！

**Redux 並不在意你如何儲存 state，state 可以是普通物件，不可變物件，或者其它類型。** 為了從 server 端寫同構應用或融合它們的 state ，你可能要用到序列化或反序列化的機制。但除此以外，你可以使用任何資料儲存的庫，**只要它支援資料的不可變動性**。舉例說明，對於 Redux state ，Backbone 並無意義，因為 Backbone model 是可變的。

注意，即便你使用支援 cursor 的不可變庫，也不應在 Redux 的應用中使用。整個 state tree 應被視為只讀，並需通過 Redux 來更新 state 和訂閱更新。因此，通過 cursor 來改寫，對 Redux 來說沒有意義。**而如果只是想用 cursor 把 state tree 從 UI tree 解耦並逐步細化 cursor，應使用 selector 來替代。** Selector 是可組合的 getter 函陣列。具體可參考 [reselect](http://github.com/faassen/reselect)，這是一個優秀、簡潔的可組合 selector 的實現。

### Baobab

[Baobab](https://github.com/Yomguithereal/baobab) 是另一個流行的庫，實現了資料不可變特性的 API，用以更新純 JavaScript 物件。你當然可以在 Redux 中使用它，但兩者一起使用並沒有什麼優勢。

Baobab 所提供的大部分功能都與使用 cursors 更新資料相關，而 Redux 更新資料的唯一方法是分發一個 action 。可見，兩者用不同方法，解決的卻是同樣的問題，相互並無增益。

不同於 Immutable ，Baobab 在引擎下還不能現實任何特別有效的資料結構，同時使用 Baobab 和 Redux 並無裨益。這種情形下，使用普通物件會更簡便。

### Rx

[Reactive Extensions](https://github.com/Reactive-Extensions/RxJS) (和它們正在進行的 [現代化重寫](https://github.com/ReactiveX/RxJS)) 是管理複雜非同步應用非常優秀的方案。[以外，還有致力於構建將人機互動作模擬為相互依賴的可觀測變數的庫](http://cycle.js.org)。

同時使用它和 Redux 有意義麼？ 當然！ 它們配合得很好。將 Redux store 視作可觀察變數非常簡便，例如：

```js
function toObservable(store) {
  return {
    subscribe({ onNext }) {
      let dispose = store.subscribe(() => onNext(store.getState()))
      onNext(store.getState())
      return { dispose }
    }
  }
}
```

使用類似方法，你可以組合不同的非同步流，將其轉化為 action ，再提交到 `store.dispatch()` 。

問題在於: 在已經使用了 Rx 的情況下，你真的需要 Redux 嗎？ 不一定。[通過 Rx 重新實現 Redux](https://github.com/jas-chen/rx-redux) 並不難。有人說僅需使用一兩句的 `.scan()` 方法即可。這種做法說不定不錯！

如果你仍有疑慮，可以去檢視 Redux 的原始碼 (並不多) 以及生態系統 (例如[開發者工具](https://github.com/gaearon/redux-devtools))。如果你無意於此，仍堅持使用互動資料流，可以去探索一下 [Cycle](http://cycle.js.org) 這樣的庫，或把它合併到 Redux 中。記得告訴我們它運作得如何！
