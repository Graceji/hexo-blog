---
title: Redux
date: 2019-11-21 20:04:03
categories: '前端'
tags: 'React'
---

### 主要涉及的Api
  - createStore
  - combineReducers
  - bindActionCreators
  - applyMiddleware
  - compose
***以下所有源码均为去除错误判断后的代码***
#### 1. createStore
```
import isPlainObject from 'lodash/isPlainObject'
import $$observable from 'symbol-observable'

export const ActionTypes = {
  INIT: '@@redux/INIT'
}

export default function createStore(reducer, preloadedState, enhancer) {
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }

  if (typeof enhancer !== 'undefined') {
    return enhancer(createStore)(reducer, preloadedState)
  }

  let currentReducer = reducer
  let currentState = preloadedState
  let currentListeners = []
  let nextListeners = currentListeners
  let isDispatching = false

  function ensureCanMutateNextListeners() {
    if (nextListeners === currentListeners) {
      nextListeners = currentListeners.slice()
    }
  }

  function getState() {
    return currentState
  }

  function subscribe(listener) {
    let isSubscribed = true

    ensureCanMutateNextListeners()
    nextListeners.push(listener)

    return function unsubscribe() {
      if (!isSubscribed) {
        return
      }

      isSubscribed = false

      ensureCanMutateNextListeners()
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
    }
  }

  function dispatch(action) {
    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.')
    }

    try {
      isDispatching = true
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false
    }

    const listeners = currentListeners = nextListeners
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    return action
  }

  function replaceReducer(nextReducer) {
    currentReducer = nextReducer
    dispatch({ type: ActionTypes.INIT })
  }

  function observable() {
    const outerSubscribe = subscribe
    return {
      subscribe(observer) {
        function observeState() {
          if (observer.next) {
            observer.next(getState())
          }
        }

        observeState()
        const unsubscribe = outerSubscribe(observeState)
        return { unsubscribe }
      },

      [$$observable]() {
        return this
      }
    }
  }

  dispatch({ type: ActionTypes.INIT })

  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  }
}
```
#### 2. combineReducers
```
export default function combineReducers(reducers) {
  const reducerKeys = Object.keys(reducers)
  const finalReducers = {}
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i]

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  const finalReducerKeys = Object.keys(finalReducers)

  return function combination(state = {}, action) {
    let hasChanged = false
    const nextState = {}
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      const previousStateForKey = state[key]
      const nextStateForKey = reducer(previousStateForKey, action)
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    return hasChanged ? nextState : state
  }
}
```
combineReducers返回一个函数，接收state和action。然后将state和action传入每个reducer进行处理得到nextState。
#### 3. bindActionCreators
```
function bindActionCreator(actionCreator, dispatch) {
  return (...args) => dispatch(actionCreator(...args))
}

export default function bindActionCreators(actionCreators, dispatch) {
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }

  const keys = Object.keys(actionCreators)
  const boundActionCreators = {}
  for (let i = 0; i < keys.length; i++) {
    const key = keys[i]
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  return boundActionCreators
}
```
#### 4. applyMiddleware
作用： 扩展dispatch方法
```
export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    const store = createStore(reducer, preloadedState, enhancer)
    let dispatch = store.dispatch
    let chain = []

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    }
    chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```
#### 5. compose
返回值(Function): 从右到左把接收到的函数合成后的最终函数。
```
/*
from right to left. For example, compose(f, g, h) is identical to doing (...args) => f(g(h(...args))).
*/
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```
### 流程
1. 当store被创建时，会首先dispatch一个初始action({ type: '@@redux/INIT' })，以获取初始state(`currentState = currentReducer(currentState, action)`)。
2. 每次dispatch action的时候，store都会将当前state和action传递给reducer处理函数以得到最新的state(`currentState = currentReducer(currentState, action)`)。
3. createStore创建store时若传入enhancer函数，则store.dispatch方法会被扩展。当dispatch(action)时，action会依次经过每个中间件函数。
4. middleware 的函数签名是 ({ getState, dispatch }) => next => action。
