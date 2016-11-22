# 使用 Object Spread Operator

從不直接修改 state 是 Redux 的核心理念之一, 所以你會發現自己總是在使用 [`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 建立物件拷貝, 而拷貝中會包含新建立或更新過的屬性值。在下面的 `todoApp` 示例中, `Object.assign()` 將會返回一個新的
`state` 物件, 而其中的 `visibilityFilter` 屬性被更新了:

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    default:
      return state
  }
}
```

儘管這樣可行, 但 `Object.assign()` 冗長的寫法會迅速降低 reducer 的可讀性。

一個可行的替代方案是使用ES7的[物件展開語法](https://github.com/sebmarkbage/ecmascript-rest-spread)提案。該提案讓你可以通過展開運算符 (`...`) , 以更加簡潔的形式將一個物件的可列舉屬性拷貝至另一個物件。物件展開運算符在概念上與ES6的[陣列展開運算符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator)相似。 我們試著用這種方式簡化 `todoApp` :

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return { ...state, visibilityFilter: action.filter }
    default:
      return state
  }
}
```

當你在組合複雜物件時, 使用物件展開運算符帶來的好處將更加突出。例如下面的 `getAddedIds` 將一個 `id` 陣列轉換為一個物件陣列, 而這些物件的內容是由 `getProduct` 和 `getQuantity` 的結果組合而成。

```js
return getAddedIds(state.cart).map(id => Object.assign(
  {},
  getProduct(state.products, id),
  {
    quantity: getQuantity(state.cart, id)
  }
))
```

運用物件擴充套件運算符簡化上面的 `map` 呼叫:

```js
return getAddedIds(state.cart).map(id => ({
  ...getProduct(state.products, id),
  quantity: getQuantity(state.cart, id)
}))
```

目前物件展開運算符提案還處於 ES7 Stage 2 草案階段, 若你想在產品中使用它得依靠轉換編譯器, 如 [Babel](http://babeljs.io/)。 你可以使用 `es2015` 預設值, 安裝 [`babel-plugin-transform-object-rest-spread`](http://babeljs.io/docs/plugins/transform-object-rest-spread/) 並將其單獨新增到位於 `.babelrc` 的 `plugins` 陣列中。

```js
{
  "presets": ["es2015"],
  "plugins": ["transform-object-rest-spread"]
}
```

注意這仍然是一個試驗性的語言特性, 在將來可能發生改變。不過一些大型項目如
[React Native](https://github.com/facebook/react-native) 已經在廣泛使用它。所以我們大可放心, 即使真的發生改變, 也應該會有自動化的遷移方案。
