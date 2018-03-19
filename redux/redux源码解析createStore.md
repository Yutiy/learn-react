createStore是redux提供的一个非常重要的方法，也是其核心部分，用来创建一个全局唯一的store对象

## ActionTypes
这里的ActionTypes 对象主要是声明了一个默认的`action`，用作`reducer` 的初始化，我们可以在chrome的插件 **Redux DevTools** 中清晰的看到其使用
```javascript
export const ActionTypes = {
  INIT: '@@redux/INIT'
}
```

## createStore
createStore 方法主要利用一个闭包，用来维护自己的私有变量:
```javascript
/**
 * @param {Function} reducer A function that returns the next state tree, given the current state tree and the action to handle
 * @param {any} [preloadedState] 初始化的state
 * @param {Function} [enhancer] store扩展，必须通过`applyMiddleware`方法包装
 * @returns {Store}
 */
export default function createStore(reducer, preloadedState, enhancer) {
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    preloadedState = undefined
    enhancer = preloadedState
  }

  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }

    return enhancer(createStore)(reducer, preloadedState)   // 高阶函数，如果存在enhancer的话
  }

  if (typeof reducer !== 'function') {
    throw new Error('Expected the reducer to be a function.')
  }

  let currentReducer = reducer
  let currentState = preloadedState
  let currentListeners = []
  let nextListeners = currentListeners
  let isDispatching = false

  // 判断引用是否相同，保存一份currentListeners 的快照
  // 由于可以在 listeners 中嵌套使用subscribe 和 unsubscribe，为了不影响正在执行的 listeners 顺序，就需要保存快照
  function ensureCanMutateNextListeners() {
    if (nextListeners === currentListeners) {
      nextListeners = currentListeners.slice()
    }
  }

  // 获取state 的方法
  function getState () {
    return currentState
  }

  function subscribe(){...}

  function dispatch(){...}

  function replaceReducer(){...}

  function observable(){...}

  dispatch({ type: ActionTypes.INIT })

  return {
    dispatch,         // 用来分发action
    subscribe,        // 用来订阅,每当dispatch action的时候就会执行
    getState,         // 获取state
    replaceReducer,
    [$$observable]: observable
  }
}
```

### 订阅方法subscribe
```javascript
// 订阅方法
function subscribe(listener) {
  if (typeof listener !== 'function') {
    throw new Error('Expected listener to be a function.')
  }

  let isSubscribed = true   // 标记是否有订阅

  ensureCanMutateNextListeners()
  nextListeners.push(listener)

  // 返回一个unsubscribe函数
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
```

`ensureCanMutateNextListeners`将 currentListeners 复制给 nextListeners，这样的话，dispatch后，在逐个执行回调函数的过程中，
如果有新增或取消订阅，都会在 nextListeners 中进行，可以确保 currentListeners 中的函数执行完毕

显然，在使用react 的时候，只要把 `View` 的更新函数(就是组件的render方法或者setState方法)放入listen，就会实现 `View` 的自动渲染。
但是我们好像没有手动调用 `subscribe` 方法，实际上，react redux库中的 `connect` 方法隐式的帮我们完成了这个工作

### dispatch
dispatch 就是将当前的`state` 和`action` 传入`reducer`，然后依次执行当前的监听函数
```javascript
function dispatch(action) {
  if (!isPlainObject(action)) {
    throw new Error(
      'Actions must be plain objects. ' +
      'Use custom middleware for async actions.'
    )
  }

  if (typeof action.type === 'undefined') {
    throw new Error(
      'Actions may not have an undefined "type" property. ' +
      'Have you misspelled a constant?'
    )
  }

  if (isDispatching) {
    throw new Error('Reducers may not dispatch actions.')
  }

  try {
    isDispatching = true
    currentState = currentReducer(currentState, action) // 使用currentReducer来合并currentState与action中的操作，并返回一个新的currentState
  } finally {
    isDispatching = false
  }

  // 保存一份最新的快照，表示正在逐个执行的 subscribe 中的回调函数
  const listeners = currentListeners = nextListeners
  for(let i = 0; i < listeners.length; i++) {
    const listener = listeners[i]
    listener()
  }

  return action   // 方便链式调用
}
```

- 可见reducer内部不允许 dispatch action，容易造成死循环
- 循环体中执行回调函数为什么直接 `listeners[i]()`，主要原因是为了避免订阅函数中this 指向发生变化

### replaceReducer
更换当前的`reducer`，不常用，主要目的有两个: 本地开发的代码热替换；代码分割后，可能出现动态更新reducer的情况。
```javascript
function replaceReducer(nextReducer) {
  if (typeof nextReducer !== 'function') {
    throw new Error('Expected the nextReducer to be a function.')
  }

  currentReducer = nextReducer
  dispatch({ type: ActionTypes.INIT })    // 重新初始化state
}
```

### observable
留给 `可观察(subscribe)/响应式库([$$observable]())` 的接口，需要提供`next`方法，用于处理订阅关系，内部并没有使用这个API，简单看一下吧:
```javascript
function observable() {
  const outerSubscribe = subscribe    // 将当前subscribe函数赋值在外部的 subscribe
  return {
    subscribe(observer) {
      if (typeof observer !== 'object') {
        throw new TypeError('Expected the observer to be an object.')
      }

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
```