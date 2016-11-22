# 組織 Reducer

作為核心概念， Redux 真的是一種十分簡單的設計模式：所有你“寫”的邏輯都集中在一個單獨的函數中，並且執行這些邏輯的唯一方式就是傳給 Redux 一個能夠描述當時情景的普通物件（plain object）。Redux store 呼叫這些邏輯函數，並傳入當前的 state tree 以及這些描述物件，返回新的 state tree，接著 Redux store 便開始通知這些訂閱者（subscriber）state tree 已經改變了。

Redux 設定了一些基本的限制來保證這些邏輯函數的正常工作，就像 [Reducers](../basics/Reducers.md) 裡面描述的一樣，它必須有類似 `(previousState, action) => newState` 這樣的結構，它們通常被稱作 **reducer 函數**，並且必須是**純**函數和可預測的。

在這之後，只要遵循這些基本的規則，Redux 就不會關心你在這些 reducer 函數中是如何組織邏輯的。這既能帶來會多的自由，也會導致很多的困惑。不過在寫這些 reducer 的時候，也會有很多的常見的模式以及很多需要注意的相關資訊與概念。而隨著應用規模逐漸變大，這些模式在管理這些錯綜複雜的 reducer 時，處理真實世界的資料時，以及優化 UI 效能時都起著至關重要的作用。


### 寫 Reducer 時必要的概念

這些概念中的一部分，可能已經在別的 Redux 文件中描述過了。其他的概念也都是些比較普通的或者可以適用於 Redux 外的，這裡有許多文章來詳細的解釋這些概念。這些概念和技巧是能寫出符合 Solid 原則的 Redux reducer 邏輯的基礎。

**深入的理解**這些概念是你要學習更高階的 Redux 技術之前必不可少的事情。這裡有一個推薦的閱讀列表。

#### [必要的概念](./reducers/PrerequisiteConcepts.md)

另外還值得注意的是，在特定的應用特定的架構下，這些建議可能也不是非常的適合。舉個例子，如果一個應用使用了 Immutable.js Map 來儲存資料，那麼它組織 reducer 邏輯的時候就可能和用普通物件儲存資料的情況不一樣。 這些文件主要假設我們使用的都是 Javascript 普通物件，但即使你使用一些其他的工具，這裡的很多規則其實依然適用。



Reducer 概念和技巧

- [基本 Reducer 結構](./reducers/BasicReducerStructure.md)
- [拆分 Reducer 邏輯](./reducers/SplittingReducerLogic.md)
- [重構 Reducer 的例子](./reducers/RefactoringReducersExample.md)
- [使用 `combineReducers`](./reducers/UsingCombineReducers.md)
- [超越 `combineReducers`](./reducers/BeyondCombineReducers.md)
- [正規化化 State 結構](./reducers/NormalizingStateShape.md)
- [更新正規化化資料](./reducers/UpdatingNormalizedData.md)
- [重用 Reducer 邏輯](./reducers/ReusingReducerLogic.md)
- [Immutable 的更新模式](./reducers/ImmutableUpdatePatterns.md)
- [初始化 State](./reducers/InitializingState.md)
