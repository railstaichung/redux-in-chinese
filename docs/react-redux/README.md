React Redux
=========================

> 譯者注：本庫並不是 Redux 內建，需要單獨安裝。因為一般會和 Redux 一起使用，所以放到一起翻譯

[Redux](https://github.com/rackt/redux) 官方提供的 React 繫結庫。
具有高效且靈活的特性。

## 安裝

React Redux 依賴 **React 0.14 或更新版本。**

```
npm install --save react-redux
```

你需要使用 [npm](http://npmjs.com/) 作為包管理工具，配合 [Webpack](http://webpack.github.io) 或 [Browserify](http://browserify.org/) 作為模組打包工具來載入 [CommonJS 模組](http://webpack.github.io/docs/commonjs.html)。

如果你不想使用 [npm](http://npmjs.com/) 和模組打包工具，只想打包一個 [UMD](https://github.com/umdjs/umd) 檔案來提供 `ReactRedux` 全局變數，那麼可以使用 [cdnjs](https://cdnjs.com/libraries/react-redux) 上打包好的版本。但對於非常正式的項目並不建議這麼做，因為和 Redux 一起工作的大部分庫都只有 [npm](http://npmjs.com/) 才能提供。

## React Native

在 [React Native 支援 React 0.14](https://github.com/facebook/react-native/issues/2985) 前，你需要使用 [React Redux 3.x 分支和對應文件](https://github.com/rackt/react-redux/tree/v3.1.0).

## 文件

- [快速入門](quick-start.md#quick-start)
- [API](api.md#api)
  - [`<Provider store>`](api.md#provider-store)
  - [`connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])`](api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options)
- [排錯](troubleshooting.md#troubleshooting)
