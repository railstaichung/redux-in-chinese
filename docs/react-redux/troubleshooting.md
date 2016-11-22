## 排錯

開始之前，一定你已經學習 [Redux 排錯](http://redux.js.org/docs/Troubleshooting.html)。

### View 不更新的問題

閱讀上面的連結。
簡而言之，

* Reducer 永遠不應該更改原有 state，應該始終返回新的物件，否則，React Redux 覺察不到資料變化。
* 確保你使用了 `connect()` 的 `mapDispatchToProps` 參數或者 `bindActionCreators` 來繫結 action creator 函數，你也可以手動呼叫 `dispatch()` 進行繫結。直接呼叫 `MyActionCreators.addTodo()` 並不會起任何作用，因為它只會**返回**一個 action 物件，並不會 *dispatch* 它。

### React Router 0.13 的 route 變化中，view 不更新

如果你正在使用 React Router 0.13，你可能會[碰到這樣的問題](https://github.com/rackt/react-redux/issues/43)。解決方法很簡單：當使用 `<RouteHandler>` 或者 `Router.run` 提供的 `Handler` 時，不要忘記傳遞 router state。

根 View：

```js
Router.run(routes, Router.HistoryLocation, (Handler, routerState) => { // 注意這裡的 "routerState"
  ReactDOM.render(
    <Provider store={store}>
      {/* 注意這裡的 "routerState" */}
      <Handler routerState={routerState} />
    </Provider>,
    document.getElementById('root')
  );
});
```

巢狀 view：

```js
render() {
  // 保持這樣傳遞下去
  return <RouteHandler routerState={this.props.routerState} />;
}
```

很方便地，這樣你的元件就能訪問 router 的 state 了！
當然，你可以將 React Router 升級到 1.0，這樣就不會有此問題了。（如果還有問題，聯絡我們！）

### Redux 外部的一些東西更新時，view 不更新

如果 view 依賴全局的 state 或是 [React “context”](http://facebook.github.io/react/docs/context.html)，你可能發現那些使用 `connect()` 進行修飾的 view 無法更新。

>這是因為，預設情況下 `connect()` 實現了 [shouldComponentUpdate](https://facebook.github.io/react/docs/component-specs.html#updating-shouldcomponentupdate)，它假定在 props 和 state 一樣的情況下，元件會渲染出同樣的結果。這與 React 中 [PureRenderMixin](https://facebook.github.io/react/docs/pure-render-mixin.html) 的概念很類似。

這個問題的**最好**的解決方案是保持元件的純淨，並且所有外部的 state 都應通過 props 傳遞給它們。這將確保元件只在需要重新渲染時才會重新渲染，這將大大地提高了應用的速度。

當不可抗力導致上述解法無法實現時（比如，你使用了嚴重依賴 React context 的外部庫），你可以設定 `connect()` 的 `pure: false` 選項：

```
function mapStateToProps(state) {
  return { todos: state.todos };
}

export default connect(mapStateToProps, null, null, {
  pure: false
})(TodoApp);
```

這樣就表示你的 `TodoApp` 不是純淨的，只要父元件渲染，自身都會重新渲染。注意，這會降低應用的效能，所以只有在別無他法的情況下才使用它。

### 在 context 或 props 中都找不到 “store”

如果你有 context 的問題，

1. [確保你沒有引入多個 React 例項](https://medium.com/@dan_abramov/two-weird-tricks-that-fix-react-7cf9bbdef375) 到頁面上。
2. 確保你沒有忘記將根元件包裝進 [`<Provider>`](#provider-store)。
3. 確保你執行的 React 和 React Redux 是最新版本。

### Invariant Violation：addComponentAsRefTo(...)：只有 ReactOwner 才有 refs。這通常意味著你在一個沒有 owner 的元件中新增了 ref

如果你在 web 中使用 React，就通常意味著你[引用了兩遍 React](https://medium.com/@dan_abramov/two-weird-tricks-that-fix-react-7cf9bbdef375)。按照這個連結解決即可。
