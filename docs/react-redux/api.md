## API

### `<Provider store>`

`<Provider store>` 使元件層級中的 `connect()` 方法都能夠獲得 Redux store。正常情況下，你的根元件應該巢狀在 `<Provider>` 中才能使用 `connect()` 方法。

如果你**真的**不想把根元件巢狀在 `<Provider>` 中，你可以把 `store` 作為 props 傳遞到每一個被 `connet()` 包裝的元件，但是我們只推薦您在單元測試中對 `store` 進行偽造 (stub) 或者在非完全基於 React 的程式碼中才這樣做。正常情況下，你應該使用 `<Provider>`。

#### 屬性

* `store` (*[Redux Store](http://rackt.github.io/redux/docs/api/Store.html)*): 應用程式中唯一的 Redux store 物件
* `children` (*ReactElement*) 元件層級的根元件。

#### 例子

##### React

```js
ReactDOM.render(
  <Provider store={store}>
    <MyRootComponent />
  </Provider>,
  rootEl
);
```

##### React Router 0.13

```js
Router.run(routes, Router.HistoryLocation, (Handler, routerState) => { // 注意這裡的 "routerState"
  ReactDOM.render(
    <Provider store={store}>
      {/* 注意這裡的 "routerState": 該變數應該傳遞到子元件 */}
      <Handler routerState={routerState} />
    </Provider>,
    document.getElementById('root')
  );
});
```

##### React Router 1.0

```js
ReactDOM.render(
  <Provider store={store}>
    <Router history={history}>...</Router>
  </Provider>,
  targetEl
);
```

### `connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])`

連線 React 元件與 Redux store。

連線操作不會改變原來的元件類，反而**返回**一個新的已與 Redux store 連線的元件類。


#### 參數

* [`mapStateToProps(state, [ownProps]): stateProps`] \(*Function*): 如果定義該參數，元件將會監聽 Redux store 的變化。任何時候，只要 Redux store 發生改變，`mapStateToProps` 函數就會被呼叫。該回撥函數必須返回一個純物件，這個物件會與元件的 props 合併。如果你省略了這個參數，你的元件將不會監聽 Redux store。如果指定了該回撥函數中的第二個參數 `ownProps`，則該參數的值為傳遞到元件的 props，而且只要元件接收到新的 props，`mapStateToProps` 也會被呼叫。

* [`mapDispatchToProps(dispatch, [ownProps]): dispatchProps`] \(*Object* or *Function*): 如果傳遞的是一個物件，那麼每個定義在該物件的函數都將被當作 Redux action creator，而且這個物件會與 Redux store 繫結在一起，其中所定義的方法名將作為屬性名，合併到元件的 props 中。如果傳遞的是一個函數，該函數將接收一個 `dispatch` 函數，然後由你來決定如何返回一個物件，這個物件通過 `dispatch` 函數與 action creator 以某種方式繫結在一起（提示：你也許會用到 Redux 的輔助函數 [`bindActionCreators()`](http://rackt.github.io/redux/docs/api/bindActionCreators.html)）。如果你省略這個 `mapDispatchToProps` 參數，預設情況下，`dispatch` 會注入到你的元件 props 中。如果指定了該回撥函數中第二個參數 `ownProps`，該參數的值為傳遞到元件的 props，而且只要元件接收到新 props，`mapDispatchToProps` 也會被呼叫。

* [`mergeProps(stateProps, dispatchProps, ownProps): props`] \(*Function*):  如果指定了這個參數，`mapStateToProps()` 與 `mapDispatchToProps()` 的執行結果和元件自身的 `props` 將傳入到這個回撥函數中。該回撥函數返回的物件將作為 props 傳遞到被包裝的元件中。你也許可以用這個回撥函數，根據元件的 props 來篩選部分的 state 資料，或者把 props 中的某個特定變數與 action creator 繫結在一起。如果你省略這個參數，預設情況下返回 `Object.assign({}, ownProps, stateProps, dispatchProps)` 的結果。

* [`options`] *(Object)* 如果指定這個參數，可以定製 connector 的行為。
  * [`pure = true`] *(Boolean)*: 如果為 true，connector 將執行 `shouldComponentUpdate` 並且淺對比 `mergeProps` 的結果，避免不必要的更新，前提是當前元件是一個“純”元件，它不依賴於任何的輸入或 state 而只依賴於 props 和 Redux store 的 state。*預設值為 `true`。*
  * [`withRef = false`] *(Boolean)*: 如果為 true，connector 會儲存一個對被包裝元件例項的引用，該引用通過 `getWrappedInstance()` 方法獲得。*預設值為 `false`*

#### 返回值

根據配置資訊，返回一個注入了 state 和 action creator 的 React 元件。

##### 靜態屬性

* `WrappedComponent` *(Component)*: 傳遞到 `connect()` 函數的原始元件類。

##### 靜態方法

元件原來的靜態方法都被提升到被包裝的 React 元件。

##### 例項方法

###### `getWrappedInstance(): ReactComponent`

僅當 `connect()` 函數的第四個參數 `options` 設定了 `{ withRef: true }` 才返回被包裝的元件例項。

#### 備註

* 函數將被呼叫兩次。第一次是設定參數，第二次是元件與 Redux store 連線：`connect(mapStateToProps, mapDispatchToProps, mergeProps)(MyComponent)`。

* connect 函數不會修改傳入的 React 元件，返回的是一個新的已與 Redux store 連線的元件，而且你應該使用這個新元件。

* `mapStateToProps` 函數接收整個 Redux store 的 state 作為 props，然後返回一個傳入到元件 props 的物件。該函數被稱之為 **selector**。參考使用 [reselect](https://github.com/rackt/reselect) 高效地組合多個 **selector** ，並對 [收集到的資料進行處理](http://rackt.github.io/redux/docs/recipes/ComputingDerivedData.html)。

#### Examples 例子

##### 只注入 `dispatch`，不監聽 store

```js
export default connect()(TodoApp);
```

##### 注入 `dispatch` 和全局 state

>不要這樣做！這會導致每次 action 都觸發整個 `TodoApp` 重新渲染，你做的所有效能優化都將付之東流。
>
>最好在多個元件上使用 `connect()`，每個元件只監聽它所關聯的部分 state。

```js
export default connect(state => state)(TodoApp);
```

##### 注入 `dispatch` 和 `todos`

```js
function mapStateToProps(state) {
  return { todos: state.todos };
}

export default connect(mapStateToProps)(TodoApp);
```

##### 注入 `todos` 和所有 action creator  (`addTodo`, `completeTodo`, ...)

```js
import * as actionCreators from './actionCreators';

function mapStateToProps(state) {
  return { todos: state.todos };
}

export default connect(mapStateToProps, actionCreators)(TodoApp);
```

##### 注入 `todos` 並把所有 action creator 作為 `actions` 屬性也注入元件中

```js
import * as actionCreators from './actionCreators';
import { bindActionCreators } from 'redux';

function mapStateToProps(state) {
  return { todos: state.todos };
}

function mapDispatchToProps(dispatch) {
  return { actions: bindActionCreators(actionCreators, dispatch) };
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp);
```

#####  注入 `todos` 和指定的 action creator (`addTodo`)

```js
import { addTodo } from './actionCreators';
import { bindActionCreators } from 'redux';

function mapStateToProps(state) {
  return { todos: state.todos };
}

function mapDispatchToProps(dispatch) {
  return bindActionCreators({ addTodo }, dispatch);
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp);
```

##### 注入 `todos` 並把 todoActionCreators 作為 `todoActions` 屬性、counterActionCreators 作為 `counterActions` 屬性注入到元件中

```js
import * as todoActionCreators from './todoActionCreators';
import * as counterActionCreators from './counterActionCreators';
import { bindActionCreators } from 'redux';

function mapStateToProps(state) {
  return { todos: state.todos };
}

function mapDispatchToProps(dispatch) {
  return {
    todoActions: bindActionCreators(todoActionCreators, dispatch),
    counterActions: bindActionCreators(counterActionCreators, dispatch)
  };
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp);
```

##### 注入 `todos` 並把 todoActionCreators 與 counterActionCreators 一同作為 `actions` 屬性注入到元件中

```js
import * as todoActionCreators from './todoActionCreators';
import * as counterActionCreators from './counterActionCreators';
import { bindActionCreators } from 'redux';

function mapStateToProps(state) {
  return { todos: state.todos };
}

function mapDispatchToProps(dispatch) {
  return {
    actions: bindActionCreators(Object.assign({}, todoActionCreators, counterActionCreators), dispatch)
  };
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp);
```

##### 注入 `todos` 並把所有的 todoActionCreators 和 counterActionCreators 作為 props 注入到元件中

```js
import * as todoActionCreators from './todoActionCreators';
import * as counterActionCreators from './counterActionCreators';
import { bindActionCreators } from 'redux';

function mapStateToProps(state) {
  return { todos: state.todos };
}

function mapDispatchToProps(dispatch) {
  return bindActionCreators(Object.assign({}, todoActionCreators, counterActionCreators), dispatch);
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp);
```

##### 根據元件的 props 注入特定使用者的 `todos`

```js
import * as actionCreators from './actionCreators';

function mapStateToProps(state, ownProps) {
  return { todos: state.todos[ownProps.userId] };
}

export default connect(mapStateToProps)(TodoApp);
```

##### 根據元件的 props 注入特定使用者的 `todos` 並把 `props.userId` 傳入到 action 中

```js
import * as actionCreators from './actionCreators';

function mapStateToProps(state) {
  return { todos: state.todos };
}

function mergeProps(stateProps, dispatchProps, ownProps) {
  return Object.assign({}, ownProps, {
    todos: stateProps.todos[ownProps.userId],
    addTodo: (text) => dispatchProps.addTodo(ownProps.userId, text)
  });
}

export default connect(mapStateToProps, actionCreators, mergeProps)(TodoApp);
```
