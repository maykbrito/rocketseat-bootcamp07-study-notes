# Config Store

Para usar dados do usuário durante todo o app, verificar permissões, pegar nome, id, é interessante usar o redux.

<p align="center">*</p>

# Dependencies

```sh
yarn add redux redux-saga react-redux reactotron-redux reactotron-redux-saga immer
``` 

<p align="center">*</p>

# Folder Structure
  - src/
    - store/
      - index.js
      - createStore.js
      - modules/
        - auth/
          - reducer.js
          - actions.js
          - sagas.js
        - rootReducer.js
        - rootSaga.js
    - config/
      - ReactotronConfig.js
    - App.js

<p align="center">*</p>

**src/store/modules/auth/reducer.js**

```js
const INITIAL_STATE = {}
export default function auth(state = INITIAL_STATE, action) {
  switch(action.type) {
    default:
      return state;   
  }
}
```

<p align="center">*</p>

**src/store/modules/auth/sagas.js**

```js
import { all } from 'redux-saga/effects'

export default all([])
```

<p align="center">*</p>
 
**src/store/modules/rootReducer.js**

```js
import { combineReducers } from 'redux';

import auth from './auth/reducer';

export default combineReducers({})
```

<p align="center">*</p>

**src/store/modules/rootSaga.js**

```js
import { all } from 'redux-saga/effects'

import auth from './auth/sagas';

export default function* rootSaga() {
  return yield all([auth]);
}
```

<p align="center">*</p>

**src/store/modules/index.js**

```js
import createSagaMiddleware from 'redux-saga'
import createStore from './createStore'

import rootReducer from './modules/rootReducer'
import rootSaga from './modules/rootSaga'

const sagaMonitor = process.env.NODE_ENV === 'development'
  ? console.tron.createSagaMonitor() 
  : null

const sagaMiddleware = createSagaMiddleware({ sagaMonitor })

const middlewares = [sagaMiddleware]

const store = createStore(rootReducer, middlewares)

sagaMiddleware.run(rootSaga)

export default store
```

<p align="center">*</p>

**src/store/modules/createStore.js**

```js
import { createStore, compose, applyMiddleware } from 'redux'

export default (reducers, middlewares) => {
  const enhancer = 
    process.env.NODE_ENV === 'development'
      ? compose(
        console.tron.createEnhancer(),
        applyMiddleware(...middlewares)
        )
      : applyMiddleware(...middlewares)
  
  return createStore(reducers, enhancer)
}
```

<p align="center">*</p>

**src/config/ReactotronConfig.js**

```js
import Reactotron from 'reactotron-react-js';
import { reactotronRedux } from 'reactotron-redux';
import reactotronSaga from 'reactotron-redux-saga';

if (process.env.NODE_ENV === 'development') {
  const tron = Reactotron.configure()
    .use(reactotronRedux())
    .use(reactotronSaga())
    .connect();

  tron.clear();

  console.tron = tron;
}
```

<p align="center">*</p>

**src/App.js**

```js
import React from 'react';
import { Provider } from 'react-redux';

import './config/ReactotronConfig';

import store from './store';

export default function App() {
  return (
    <Provider store={store}>
    </Provider>
  );
}
```