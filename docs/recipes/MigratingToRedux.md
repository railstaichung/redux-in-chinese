# 遷移到 Redux

Redux 不是一個單一的框架，而是一系列的約定和[一些讓他們協同工作的函數](../api/README.md)。你的 Redux 項目的主體程式碼甚至不需要使用 Redux 的 API，大部分時間你其實是在編寫函數。

這讓到 Redux 的雙向遷移都非常的容易。
我們可不想把你限制得死死的！

## 從 Flux 項目遷移

[Reducer](../Glossary.md#reducer) 抓住了 Flux Store 的本質，因此，將一個 Flux 項目逐步到 Redux 是可行的，無論你使用了 [Flummox](http://github.com/acdlite/flummox)、[Alt](http://github.com/goatslacker/alt)、[traditional Flux](https://github.com/facebook/flux) 還是其他 Flux 庫。

同樣你也可以將 Redux 的項目通過相同的步驟遷移回上述的這些 Flux 框架。

你的遷移過程大致包含幾個步驟：

* 建立一個叫做 `createFluxStore(reducer)` 的函數，通過 reducer 函數適配你當前項目的 Flux Store。從程式碼來看，這個函數很像 Redux 中 [`createStore`](../api/createStore.md) ([來源](https://github.com/rackt/redux/blob/master/src/createStore.js))的實現。它的 dispatch 處理器應該根據不同的 action 來呼叫不同的 `reducer`，儲存新的 state 並拋出更新事件。

* 通過建立 `createFluxStore(reducer)` 的方法來將每個 Flux Store 逐步重寫為 Reducer，這個過程中你的應用中其他部分程式碼感知不到任何變化，仍可以和原來一樣使用 Flux Store 。

* 當重寫你的 Store 時，你會發現你應該避免一些明顯違反 Flux 模式的使用方法，例如在 Store 中請求 API、或者在 Store 中觸發 action。一旦基於 reducer 來構建你的 Flux 程式碼，它會變得更易於理解。

* 當你所有的 Flux Store 全部基於 reducer 來實現時，你就可以利用 [`combineReducers(reducers)`](../api/combineReducers.md) 將多個 reducer 合併到一起，然後在應用裡使用這個唯一的 Redux Store。

* 現在，剩下的就只是[使用 react-redux](../basics/UsageWithReact.md) 或者類似的庫來處理你的UI部分。

* 最後，你可以使用一些 Redux 的特性，例如利用 middleware 來進一步簡化非同步的程式碼。


## 從 Backbone 項目遷移

Backbone 的 Model 層與 Redux 有著巨大的差別，因此，我們不建議從 Backbone
 項目遷移到 Redux 。如果可以的話，最好的方法是徹底重寫 app 的 Model 層。不過，如果重寫不可行，也可以試試使用 [backbone-redux](https://github.com/redbooth/backbone-redux) 來逐步遷移，並使 Redux 的 store 和 Backbone 的 model 層及 collection 保持同步。
