# 動機

隨著 JavaScript 單頁應用開發日趨複雜，**JavaScript 需要管理比任何時候都要多的 state （狀態）**。 這些 state 可能包括伺服器響應、快取資料、本地生成尚未持久化到伺服器的資料，也包括 UI 狀態，如啟用的路由，被選中的標籤，是否顯示載入動效或者分頁器等等。

管理不斷變化的 state 非常困難。如果一個 model 的變化會引起另一個 model 變化，那麼當 view 變化時，就可能引起對應 model 以及另一個 model 的變化，依次地，可能會引起另一個 view 的變化。直至你搞不清楚到底發生了什麼。**state 在什麼時候，由於什麼原因，如何變化已然不受控制。** 當系統變得錯綜複雜的時候，想重現問題或者新增新功能就會變得舉步維艱。

如果這還不夠糟糕，考慮一些**來自前端開發領域的新需求**，如更新調優、服務端渲染、路由跳轉前請求資料等等。前端開發者正在經受前所未有的複雜性，[難道就這麼放棄了嗎？](http://www.quirksmode.org/blog/archives/2015/07/stop_pushing_th.html)當然不是。

這裡的複雜性很大程度上來自於：**我們總是將兩個難以釐清的概念混淆在一起：變化和非同步**。 我稱它們為[曼妥思和可樂](https://en.wikipedia.org/wiki/Diet_Coke_and_Mentos_eruption)。如果把二者分開，能做的很好，但混到一起，就變得一團糟。一些庫如 [React](http://facebook.github.io/react) 試圖在檢視層禁止非同步和直接操作 DOM 來解決這個問題。美中不足的是，React 依舊把處理 state 中資料的問題留給了你。Redux就是為了幫你解決這個問題。

跟隨 [Flux](http://facebook.github.io/flux)、[CQRS](http://martinfowler.com/bliki/CQRS.html) 和 [Event Sourcing](http://martinfowler.com/eaaDev/EventSourcing.html) 的腳步，通過限制更新發生的時間和方式，**Redux 試圖讓 state 的變化變得可預測**。這些限制條件反映在 Redux 的 [三大原則](ThreePrinciples.md)中。
