# 編寫測試

因為你寫的大部分 Redux 程式碼都是些函數，而且大部分是純函數，所以很好測，不需要模擬。

### 設定

我們建議用 [Jest](http://facebook.github.io/jest/) 作為測試引擎。
注意因為是在 node 環境下執行，所以你不能訪問 DOM。

```
npm install --save-dev jest
```

若想結合 [Babel](http://babeljs.io) 使用，你需要安裝 `babel-jest` ：

```
npm install --save-dev babel-jest
```
並且在 `.babelrc` 中啟用 ES2015 的功能 ：

```js
{
  "presets": ["es2015"]
}
```
然後，在 `package.json` 的 `scripts` 里加入這一段：

```js
{
  ...
  "scripts": {
    ...
    "test": "jest",
    "test:watch": "npm test -- --watch"
  },
  ...
}
```

然後執行 `npm test` 就能單次執行了，或者也可以使用 `npm run test:watch` 在每次有檔案改變時自動執行測試。

### Action 建立函數 (Action Creators)

Redux 裡的 action 建立函數是會返回普通物件的函數。在測試 action 建立函數的時候我們想要測試是否呼叫了正確的 action 建立函數，還有是否返回了正確的 action。

#### 示例

```js
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}
```
可以這樣測試：

```js
import * as actions from '../../actions/TodoActions'
import * as types from '../../constants/ActionTypes'

describe('actions', () => {
  it('should create an action to add a todo', () => {
    const text = 'Finish docs'
    const expectedAction = {
      type: types.ADD_TODO,
      text
    }
    expect(actions.addTodo(text)).toEqual(expectedAction)
  })
})
```

### 非同步 Action 建立函數

對於使用 [Redux Thunk](https://github.com/gaearon/redux-thunk) 或其它中介軟體的非同步 action 建立函數，最好完全模擬 Redux store 來測試。 你可以使用 [redux-mock-store](https://github.com/arnaudbenard/redux-mock-store) 把 middleware 應用到模擬的 store。也可以使用 [nock](https://github.com/pgte/nock) 來模擬 HTTP 請求。

#### 示例

```js
import fetch from 'isomorphic-fetch';

function fetchTodosRequest() {
  return {
    type: FETCH_TODOS_REQUEST
  }
}

function fetchTodosSuccess(body) {
  return {
    type: FETCH_TODOS_SUCCESS,
    body
  }
}

function fetchTodosFailure(ex) {
  return {
    type: FETCH_TODOS_FAILURE,
    ex
  }
}

export function fetchTodos() {
  return dispatch => {
    dispatch(fetchTodosRequest())
    return fetch('http://example.com/todos')
      .then(res => res.json())
      .then(json => dispatch(fetchTodosSuccess(json.body)))
      .catch(ex => dispatch(fetchTodosFailure(ex)))
  }
}
```

可以這樣測試:

```js
import configureMockStore from 'redux-mock-store'
import thunk from 'redux-thunk'
import * as actions from '../../actions/TodoActions'
import * as types from '../../constants/ActionTypes'
import nock from 'nock'
import expect from 'expect' // 你可以使用任何測試庫

const middlewares = [ thunk ]
const mockStore = configureMockStore(middlewares)

describe('async actions', () => {
  afterEach(() => {
    nock.cleanAll()
  })

  it('creates FETCH_TODOS_SUCCESS when fetching todos has been done', () => {
    nock('http://example.com/')
      .get('/todos')
      .reply(200, { body: { todos: ['do something'] }})

    const expectedActions = [
      { type: types.FETCH_TODOS_REQUEST },
      { type: types.FETCH_TODOS_SUCCESS, body: { todos: ['do something']  } }
    ]
    const store = mockStore({ todos: [] })

    return store.dispatch(actions.fetchTodos())
      .then(() => { // 非同步 actions 的返回
        expect(store.getActions()).toEqual(expectedActions)
      })
  })
})
```

### Reducers

Reducer 把 action 應用到之前的 state，並返回新的 state。測試如下。

#### 示例

```js
import { ADD_TODO } from '../constants/ActionTypes'

const initialState = [
  {
    text: 'Use Redux',
    completed: false,
    id: 0
  }
]

export default function todos(state = initialState, action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        {
          id: state.reduce((maxId, todo) => Math.max(todo.id, maxId), -1) + 1,
          completed: false,
          text: action.text
        },
        ...state
      ]

    default:
      return state
  }
}
```
可以這樣測試:

```js
import reducer from '../../reducers/todos'
import * as types from '../../constants/ActionTypes'

describe('todos reducer', () => {
  it('should return the initial state', () => {
    expect(
      reducer(undefined, {})
    ).toEqual([
      {
        text: 'Use Redux',
        completed: false,
        id: 0
      }
    ])
  })

  it('should handle ADD_TODO', () => {
    expect(
      reducer([], {
        type: types.ADD_TODO,
        text: 'Run the tests'
      })
    ).toEqual(
      [
        {
          text: 'Run the tests',
          completed: false,
          id: 0
        }
      ]
    )

    expect(
      reducer(
        [
          {
            text: 'Use Redux',
            completed: false,
            id: 0
          }
        ],
        {
          type: types.ADD_TODO,
          text: 'Run the tests'
        }
      )
    ).toEqual(
      [
        {
          text: 'Run the tests',
          completed: false,
          id: 1
        },
        {
          text: 'Use Redux',
          completed: false,
          id: 0
        }
      ]
    )
  })
})
```

### Components

React components 的優點是，一般都很小且依賴於 props 。因此測試起來很簡便。

首先，安裝 [Enzyme](http://airbnb.io/enzyme/) 。 Enzyme 底層使用了 [React Test Utilities](https://facebook.github.io/react/docs/test-utils.html) ，但是更方便、更易讀，而且更強大。

```
npm install --save-dev enzyme
```

要測 components ，我們要建立一個叫 `setup()` 的輔助方法，用來把模擬過的（stubbed）回撥函數當作 props 傳入，然後使用 [React 淺渲染](https://facebook.github.io/react/docs/test-utils.html#shallow-rendering) 來渲染元件。這樣就可以依據 “是否呼叫了回撥函數” 的斷言來寫獨立的測試。

#### 示例

```js
import React, { PropTypes, Component } from 'react'
import TodoTextInput from './TodoTextInput'

class Header extends Component {
  handleSave(text) {
    if (text.length !== 0) {
      this.props.addTodo(text)
    }
  }

  render() {
    return (
      <header className='header'>
          <h1>todos</h1>
          <TodoTextInput newTodo={true}
                         onSave={this.handleSave.bind(this)}
                         placeholder='What needs to be done?' />
      </header>
    )
  }
}

Header.propTypes = {
  addTodo: PropTypes.func.isRequired
}

export default Header
```

可以這樣測試:

```js
import React from 'react'
import { shallow } from 'enzyme'
import Header from '../../components/Header'

function setup() {
  const props = {
    addTodo: jest.fn()
  }

  const enzymeWrapper = shallow(<Header {...props} />)

  return {
    props,
    enzymeWrapper
  }
}

describe('components', () => {
  describe('Header', () => {
    it('should render self and subcomponents', () => {
      const { enzymeWrapper } = setup()

      expect(enzymeWrapper.find('header').hasClass('header')).toBe(true)

      expect(enzymeWrapper.find('h1').text()).toBe('todos')

      const todoInputProps = enzymeWrapper.find('TodoTextInput').props()
      expect(todoInputProps.newTodo).toBe(true)
      expect(todoInputProps.placeholder).toEqual('What needs to be done?')
    })

    it('should call addTodo if length of text is greater than 0', () => {
      const { enzymeWrapper, props } = setup()
      const input = enzymeWrapper.find('TodoTextInput')
      input.props().onSave('')
      expect(props.addTodo.mock.calls.length).toBe(0)
      input.props().onSave('Use Redux')
      expect(props.addTodo.mock.calls.length).toBe(1)
    })
  })
})
```

### 連線元件

如果你使用了 [React Redux](https://github.com/rackt/react-redux), 可能你也同時在使用類似 [`connect()`](https://github.com/rackt/react-redux#connectmapstatetoprops-mapdispatchtoprops-mergeprops) 的 [higher-order components](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750) ，將 Redux state 注入到常見的 React 元件中。

請看這個 `App` 元件:

```js
import { connect } from 'react-redux'

class App extends Component { /* ... */ }

export default connect(mapStateToProps)(App)
```

在單元測試中，一般會這樣匯入 `App` 元件

```js
import App from './App'
```

但是，當這樣匯入時，實際上持有的是 `connect()` 返回的包裝過元件，而不是 `App` 元件本身。如果想測試它和 Redux 間的互動，好訊息是可以使用一個專為單元測試建立的 store， 將它包裝在[`<Provider>`](https://github.com/rackt/react-redux#provider-store) 中。但有時我們僅僅是想測試元件的渲染，並不想要這麼一個 Redux store。

想要不和裝飾件打交道而測試 App 元件本身，我們建議你同時匯出未包裝的元件：

```js
import { connect } from 'react-redux'

// 命名匯出未連線的元件 (測試用)
export class App extends Component { /* ... */ }

// 預設匯出已連線的元件 (app 用)
export default connect(mapDispatchToProps)(App)
```

鑑於預設匯出的依舊是包裝過的元件，上面的匯入語句會和之前一樣工作，不需要更改應用中的程式碼。不過，可以這樣在測試檔案中匯入沒有包裝的 `App` 元件：

```js
// 注意花括號：抓取命名匯出，而不是預設匯出
import { App } from './App'
```

如果兩者都需要:

```js
import ConnectedApp, { App } from './App'
```

在 app 中，仍然正常地匯入：

```js
import App from './App'
```

只在測試中使用命名匯出。

>##### 混用 ES6 模組和 CommonJS 的注意事項

>如果在應用程式碼中使用 ES6，但在測試中使用 ES5，Babel 會通過其 [interop](http://babeljs.io/docs/usage/modules/#interop) 的機制處理 ES6 的 `import` 和 CommonJS 的 `require` 的轉換，使這兩個模組的格式各自運作，但其行為依舊有[細微的區別](https://github.com/babel/babel/issues/2047)。 如果在預設匯出的附近增加另一個匯出，將導致無法預設匯出 `require('./App')`。此時，應代以 `require('./App').default` 。

### 中介軟體

中介軟體函數會對 Redux 中 `dispatch` 的呼叫行為進行封裝。因此，需要通過模擬 `dispatch` 的呼叫行為來測試。

#### 示例

```js
import * as types from '../../constants/ActionTypes'
import singleDispatch from '../../middleware/singleDispatch'

const createFakeStore = fakeData => ({
  getState() {
    return fakeData
  }
})

const dispatchWithStoreOf = (storeData, action) => {
  let dispatched = null
  const dispatch = singleDispatch(createFakeStore(storeData))(actionAttempt => dispatched = actionAttempt)
  dispatch(action)
  return dispatched
}

describe('middleware', () => {
  it('should dispatch if store is empty', () => {
    const action = {
      type: types.ADD_TODO
    }

    expect(
      dispatchWithStoreOf({}, action)
    ).toEqual(action)
  })

  it('should not dispatch if store already has type', () => {
    const action = {
      type: types.ADD_TODO
    }

    expect(
      dispatchWithStoreOf({
        [types.ADD_TODO]: 'dispatched'
      }, action)
    ).toNotExist()
  })
})
```

### 詞彙表

- [Enzyme](http://airbnb.io/enzyme/)：Enzyme 是一個 React 的 JavaScript 測試工具，能夠讓斷言、操作以及遍歷你的 React 元件的輸出變得更簡單。

- [React Test Utils](http://facebook.github.io/react/docs/test-utils.html)： React 測試工具。被 Enzyme 所使用。

- [淺渲染（shallow renderer）](http://facebook.github.io/react/docs/test-utils.html#shallow-rendering)： 淺渲染的中心思想是，初始化一個元件然後得到它的 `渲染` 方法作為結果，渲染深度僅一層，而非遞迴渲染整個 DOM 。淺渲染對單元測試很有用， 你只要測試某個特定的元件，而不包括它的子元件。這也意味著，更改一個子元件不會影響到其父元件的測試。如果要測試一個元件和它所有的子元件，可以用 [Enzyme's mount() method](http://airbnb.io/enzyme/docs/api/mount.html) ，也就是完全 DOM 渲染來實現。
