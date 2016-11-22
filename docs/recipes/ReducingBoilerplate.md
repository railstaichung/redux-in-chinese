# 減少樣板程式碼

Redux 很大部分 [受到 Flux 的啟發](../introduction/PriorArt.md)，而最常見的關於 Flux 的抱怨是必須寫一大堆的模板。而在這章技巧中，根據個人樣式，團隊選項，長期可維護等等因素，Redux 可以讓我們自由決定程式碼的繁複程度。

## Actions

Actions 是用來描述在 app 中發生了什麼的普通物件，是描述物件變異意圖的唯一途徑。很重要的一點是 **必須分發的 action 物件並非模板，而是 Redux 的一個[基本設計選項](../introduction/ThreePrinciples.md)**.

不少框架聲稱自己和 Flux 很像，只不過缺少了 action 物件的概念。但可預測的是，這是從 Flux 或 Redux 的倒退。如果沒有可序列化的普通物件 action，便無法記錄或重演使用者會話，也無法實現 [帶有時間旅行的熱過載](https://www.youtube.com/watch?v=xsSnOQynTHs)。如果你更喜歡直接修改資料，那你並不需要使用 Redux 。

Action 一般長這樣:

```js
{ type: 'ADD_TODO', text: 'Use Redux' }
{ type: 'REMOVE_TODO', id: 42 }
{ type: 'LOAD_ARTICLE', response: { ... } }
```

一個約定俗成的做法是，actions 擁有一個常量 type 幫助 reducer (或 Flux 中的 Stores ) 識別它們。我們建議你使用 string 而不是 [Symbols](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol) 作為 action type ，因為 string 是可序列化的，而使用 Symbols 會毫無必要地使記錄和重演變得困難。

在 Flux 中，傳統的想法是將每個 action type 定義為 string 常量：

```js
const ADD_TODO = 'ADD_TODO';
const REMOVE_TODO = 'REMOVE_TODO';
const LOAD_ARTICLE = 'LOAD_ARTICLE';
```

這麼做的優勢？**人們通常聲稱常量不是必要的。對於小項目也許正確。** 對於大的項目，將 action types 定義為常量有如下好處：

* 幫助維護命名一致性，因為所有的 action type 彙總在同一位置。
* 有時，在開發一個新功能之前你想看到所有現存的 actions 。而你的團隊裡可能已經有人新增了你所需要的action，而你並不知道。
* Action types 列表在 Pull Request 中能查到所有新增，刪除，修改的記錄。這能幫助團隊中的所有人及時追蹤新功能的範圍與實現。
* 如果你在匯入一個 Action 常量的時候拼寫錯誤，你會得到 `undefined` 。當你納悶 action 被分發出去而什麼也沒發生的時候，一個拼寫錯誤更容易被發現。

你的項目的約定取決與你自己。開始時，可能用的是 inline string，之後轉為常量，也許之後將他們歸為一個獨立檔案。Redux 不會給予任何建議，選擇你自己最喜歡的。

## Action Creators

另一個約定俗成的做法，通過建立函數生成 action 物件，而不是在你分發的時候內聯生成它們。

例如，比起使用物件文字呼叫 `dispatch` ：

```js
// somewhere in an event handler
dispatch({
  type: 'ADD_TODO',
  text: 'Use Redux'
});
```

你其實可以在單獨的檔案中寫一個 action creator ，然後從 component 裡匯入：

#### `actionCreators.js`

```js
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  };
}
```

#### `AddTodo.js`

```js
import { addTodo } from './actionCreators';

// event handler 裡的某處
dispatch(addTodo('Use Redux'))
```

Action creators 總被當作模板受到批評。好吧，其實你並不非得把他們寫出來！**如果你覺得更適合你的項目，你可以選用物件文字** 然而，你應該知道寫 action creators 是存在某種優勢的。

假設有個設計師看完我們的原型之後回來說，我們最多隻允許三個 todo 。我們可以使用 [redux-thunk](https://github.com/gaearon/redux-thunk) 中介軟體，並新增一個提前退出，把我們的 action creator 重寫成回撥形式：

```js
function addTodoWithoutCheck(text) {
  return {
    type: 'ADD_TODO',
    text
  };
}

export function addTodo(text) {
  // Redux Thunk 中介軟體允許這種形式
  // 在下面的 “非同步 Action Creators” 段落中有寫
  return function (dispatch, getState) {
    if (getState().todos.length === 3) {
      // 提前退出
      return;
    }

    dispatch(addTodoWithoutCheck(text));
  }
}
```

我們剛修改了 `addTodo` action creator 的行為，使得它對呼叫它的程式碼完全不可見。**我們不用擔心去每個新增 todo 的地方看一看，以確認他們有了這個檢查** Action creator 讓你可以解耦額外的分發 action 邏輯與實際傳送這些 action 的 components 。當你有大量開發工作且需求經常變更的時候，這種方法十分簡便易用。

### 生成 Action Creators

某些框架如 [Flummox](https://github.com/acdlite/flummox) 自動從 action creator 函數定義生成 action type 常量。這個想法是說你不需要同時定義 `ADD_TODO` 常量和 `addTodo()` action creator 。這樣的方法在底層也生成了 action type 常量，但他們是隱式生成的、間接級，會造成混亂。因此我們建議直接清晰地建立 action type 常量。

寫簡單的 action creator 很容易讓人厭煩，且往往最終生成多餘的樣板程式碼：

```js
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}

export function editTodo(id, text) {
  return {
    type: 'EDIT_TODO',
    id,
    text
  }
}

export function removeTodo(id) {
  return {
    type: 'REMOVE_TODO',
    id
  }
}
```

你可以寫一個用於生成 action creator 的函數：

```js
function makeActionCreator(type, ...argNames) {
  return function(...args) {
    let action = { type }
    argNames.forEach((arg, index) => {
      action[argNames[index]] = args[index]
    })
    return action
  }
}

const ADD_TODO = 'ADD_TODO'
const EDIT_TODO = 'EDIT_TODO'
const REMOVE_TODO = 'REMOVE_TODO'

export const addTodo = makeActionCreator(ADD_TODO, 'todo')
export const editTodo = makeActionCreator(EDIT_TODO, 'id', 'todo')
export const removeTodo = makeActionCreator(REMOVE_TODO, 'id')
```
一些工具庫也可以幫助生成 action creator ，例如 [redux-act](https://github.com/pauldijou/redux-act) 和 [redux-actions](https://github.com/acdlite/redux-actions) 。這些庫可以有效減少你的樣板程式碼，並緊守例如 [Flux Standard Action (FSA)](https://github.com/acdlite/flux-standard-action) 一類的標準。

## 非同步 Action Creators

[中介軟體](../Glossary.html#middleware) 讓你在每個 action 物件分發出去之前，注入一個自定義的邏輯來解釋你的 action 物件。非同步 action 是中介軟體的最常見用例。

如果沒有中介軟體，[`dispatch`](../api/Store.md#dispatch) 只能接收一個普通物件。因此我們必須在 components 裡面進行 AJAX 呼叫：

#### `actionCreators.js`

```js
export function loadPostsSuccess(userId, response) {
  return {
    type: 'LOAD_POSTS_SUCCESS',
    userId,
    response
  };
}

export function loadPostsFailure(userId, error) {
  return {
    type: 'LOAD_POSTS_FAILURE',
    userId,
    error
  };
}

export function loadPostsRequest(userId) {
  return {
    type: 'LOAD_POSTS_REQUEST',
    userId
  };
}
```

#### `UserInfo.js`

```js
import { Component } from 'react';
import { connect } from 'react-redux';
import { loadPostsRequest, loadPostsSuccess, loadPostsFailure } from './actionCreators';

class Posts extends Component {
  loadData(userId) {
    // 呼叫 React Redux `connect()` 注入 props ：
    let { dispatch, posts } = this.props;

    if (posts[userId]) {
      // 這裡是被快取的資料！啥也不做。
      return;
    }

    // Reducer 可以通過設定 `isFetching` 響應這個 action
    // 因此讓我們顯示一個 Spinner 控制項。
    dispatch(loadPostsRequest(userId));

    // Reducer 可以通過填寫 `users` 響應這些 actions
    fetch(`http://myapi.com/users/${userId}/posts`).then(
      response => dispatch(loadPostsSuccess(userId, response)),
      error => dispatch(loadPostsFailure(userId, error))
    );
  }

  componentDidMount() {
    this.loadData(this.props.userId);
  }

  componentWillReceiveProps(nextProps) {
    if (nextProps.userId !== this.props.userId) {
      this.loadData(nextProps.userId);
    }
  }

  render() {
    if (this.props.isLoading) {
      return <p>Loading...</p>;
    }

    let posts = this.props.posts.map(post =>
      <Post post={post} key={post.id} />
    );

    return <div>{posts}</div>;
  }
}

export default connect(state => ({
  posts: state.posts
}))(Posts);
```

然而，不久就需要再來一遍，因為不同的 components 從同樣的 API 端點請求資料。而且我們想要在多個components 中重用一些邏輯（比如，當快取資料有效的時候提前退出）。

**中介軟體讓我們能寫表達更清晰的、潛在的非同步 action creators。** 它允許我們分發普通物件之外的東西，並且解釋它們的值。比如，中介軟體能 “捕捉” 到已經分發的 Promises 並把他們變為一對請求和成功/失敗的 action.

中介軟體最簡單的例子是 [redux-thunk](https://github.com/gaearon/redux-thunk). **“Thunk” 中介軟體讓你可以把 action creators 寫成 “thunks”，也就是返回函數的函數。** 這使得控制被反轉了： 你會像一個參數一樣取得 `dispatch` ，所以你也能寫一個多次分發的 action creator 。

>##### 注意

>Thunk 只是一箇中介軟體的例子。中介軟體不僅僅是關於 “分發函數” 的：而是關於，你可以使用特定的中介軟體來分發任何該中介軟體可以處理的東西。例子中的 Thunk 中介軟體新增了一個特定的行為用來分發函數，但這實際取決於你用的中介軟體。

用 [redux-thunk](https://github.com/gaearon/redux-thunk) 上面的程式碼：

#### `actionCreators.js`

```js
export function loadPosts(userId) {
  // 用 thunk 中介軟體解釋：
  return function (dispatch, getState) {
    let { posts } = getState();
    if (posts[userId]) {
      // 這裡是資料快取！啥也不做。
      return;
    }

    dispatch({
      type: 'LOAD_POSTS_REQUEST',
      userId
    });

    // 非同步分發原味 action
    fetch(`http://myapi.com/users/${userId}/posts`).then(
      response => dispatch({
        type: 'LOAD_POSTS_SUCCESS',
        userId,
        response
      }),
      error => dispatch({
        type: 'LOAD_POSTS_FAILURE',
        userId,
        error
      })
    );
  }
}
```

#### `UserInfo.js`

```js
import { Component } from 'react';
import { connect } from 'react-redux';
import { loadPosts } from './actionCreators';

class Posts extends Component {
  componentDidMount() {
    this.props.dispatch(loadPosts(this.props.userId));
  }

  componentWillReceiveProps(nextProps) {
    if (nextProps.userId !== this.props.userId) {
      this.props.dispatch(loadPosts(nextProps.userId));
    }
  }

  render() {
    if (this.props.isLoading) {
      return <p>Loading...</p>;
    }

    let posts = this.props.posts.map(post =>
      <Post post={post} key={post.id} />
    );

    return <div>{posts}</div>;
  }
}

export default connect(state => ({
  posts: state.posts
}))(Posts);
```

這樣打得字少多了！如果你喜歡，你還是可以保留 “原味” action creators 比如從一個容器 `loadPosts` action creator 裡用到的 `loadPostsSuccess` 。

**最後，你可以編寫你自己的中介軟體** 你可以把上面的模式泛化，然後代之以這樣的非同步 action creators ：

```js
export function loadPosts(userId) {
  return {
    // 要在之前和之後傳送的 action types
    types: ['LOAD_POSTS_REQUEST', 'LOAD_POSTS_SUCCESS', 'LOAD_POSTS_FAILURE'],
    // 檢查快取 (可選):
    shouldCallAPI: (state) => !state.users[userId],
    // 進行取：
    callAPI: () => fetch(`http://myapi.com/users/${userId}/posts`),
    // 在 actions 的開始和結束注入的參數
    payload: { userId }
  };
}
```

解釋這個 actions 的中介軟體可以像這樣：

```js
function callAPIMiddleware({ dispatch, getState }) {
  return function (next) {
    return function (action) {
      const {
        types,
        callAPI,
        shouldCallAPI = () => true,
        payload = {}
      } = action;

      if (!types) {
        // 普通 action：傳遞
        return next(action);
      }

      if (
        !Array.isArray(types) ||
        types.length !== 3 ||
        !types.every(type => typeof type === 'string')
      ) {
        throw new Error('Expected an array of three string types.');
      }

      if (typeof callAPI !== 'function') {
        throw new Error('Expected fetch to be a function.');
      }

      if (!shouldCallAPI(getState())) {
        return;
      }

      const [requestType, successType, failureType] = types;

      dispatch(Object.assign({}, payload, {
        type: requestType
      }));

      return callAPI().then(
        response => dispatch(Object.assign({}, payload, {
          response: response,
          type: successType
        })),
        error => dispatch(Object.assign({}, payload, {
          error: error,
          type: failureType
        }))
      );
    };
  };
}
```

在傳給 [`applyMiddleware(...middlewares)`](../api/applyMiddleware.md) 一次以後，你能用相同方式寫你的 API呼叫 action creators ：

```js
export function loadPosts(userId) {
  return {
    types: ['LOAD_POSTS_REQUEST', 'LOAD_POSTS_SUCCESS', 'LOAD_POSTS_FAILURE'],
    shouldCallAPI: (state) => !state.users[userId],
    callAPI: () => fetch(`http://myapi.com/users/${userId}/posts`),
    payload: { userId }
  };
}

export function loadComments(postId) {
  return {
    types: ['LOAD_COMMENTS_REQUEST', 'LOAD_COMMENTS_SUCCESS', 'LOAD_COMMENTS_FAILURE'],
    shouldCallAPI: (state) => !state.posts[postId],
    callAPI: () => fetch(`http://myapi.com/posts/${postId}/comments`),
    payload: { postId }
  };
}

export function addComment(postId, message) {
  return {
    types: ['ADD_COMMENT_REQUEST', 'ADD_COMMENT_SUCCESS', 'ADD_COMMENT_FAILURE'],
    callAPI: () => fetch(`http://myapi.com/posts/${postId}/comments`, {
      method: 'post',
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ message })
    }),
    payload: { postId, message }
  };
}
```

## Reducers

Redux 用函數描述邏輯更新減少了模版裡大量的 Flux stores 。函數比物件簡單，比類更簡單得多。

這個 Flux store:

```js
let _todos = []

const TodoStore = Object.assign({}, EventEmitter.prototype, {
  getAll() {
    return _todos
  }
})

AppDispatcher.register(function (action) {
  switch (action.type) {
    case ActionTypes.ADD_TODO:
      let text = action.text.trim()
      _todos.push(text)
      TodoStore.emitChange()
  }
})

export default TodoStore
```

用了 Redux 之後，同樣的邏輯更新可以被寫成 reducing function：

```js
export function todos(state = [], action) {
  switch (action.type) {
  case ActionTypes.ADD_TODO:
    let text = action.text.trim()
    return [ ...state, text ]
  default:
    return state
  }
}
```

`switch` 語句 *不是* 真正的模版。真正的 Flux 模版是概念性的：傳送更新的需求，用 Dispatcher 註冊 Store 的需求，Store 是物件的需求 (當你想要一個哪都能跑的 App 的時候複雜度會提升)。

不幸的是很多人仍然靠文件裡用沒用 `switch` 來選擇 Flux 框架。如果你不愛用 `switch` 你可以用一個單獨的函數來解決，下面會演示。

### 生成 Reducers

寫一個函數將 reducers 表達為 action types 到 handlers 的對映物件。例如，如果想在 `todos` reducer 裡這樣定義：

```js
export const todos = createReducer([], {
  [ActionTypes.ADD_TODO](state, action) {
    let text = action.text.trim();
    return [...state, text];
  }
})
```

我們可以編寫下面的輔助函數來完成：

```js
function createReducer(initialState, handlers) {
  return function reducer(state = initialState, action) {
    if (handlers.function hasOwnProperty() { [native code] }(action.type)) {
      return handlers[action.type](state, action);
    } else {
      return state;
    }
  }
}
```

不難對吧？鑑於寫法多種多樣，Redux 沒有預設提供這樣的輔助函數。可能你想要自動地將普通 JS 物件變成 Immutable 物件，以填滿伺服器狀態的物件資料。可能你想合併返回狀態和當前狀態。有多種多樣的方法來 “獲取所有” handler，具體怎麼做則取決於項目中你和你的團隊的約定。

Redux reducer 的 API 是 `(state, action) => state`，但是怎麼建立這些 reducers 由你來定。
