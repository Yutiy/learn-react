## 为什么需要combineReducers

先写一个正常点的代码:
```javascript
const initialState = {
  counter: 0,
  todos: []
}

export function reducer(state = initialState, action) {
  switch(action.type) {
    case 'ADD_TODO':
      state.todos.push(action.payload)
      break
    case 'INCREMENT':
      state.counter += 1
      break
    default:

  }
  return state
}
```

如果说还有其他操作，都需要在这个`reducer`中进行操作，而且不同模块的都要写在一起，reducer也会变得很臃肿，并且取数据和修改逻辑也会变得很麻烦。
因此，就需要一种方案，帮助我们取到局部数据以及拆分`reducer`，这个时候就需要**combineReducers**

## combineReducers
```javascript
export default function combineReducers(reducers) {
  const reducerKeys = Object.keys(reducers)
  const finalReducers = {}

  // ...省略错误处理

  for(let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i]

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }

  const finalReducerKeys = Object.keys(finalReducers)

  // dispatch({ type: ActionTypes.INIT })，默认会初始化执行一遍，在对应的 reducer 上挂载默认的对应state
  return function combination(state = {}, action) {
    let hasChanged = false
    const nextState = {}

    for(let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]

      // 获取当前的子state
      const previousStateForKey = state[key]
      // 执行各子reducer，获取子 nextState
      const nextStateForKey = reducer(previousStateForKey, action)
      // 将子 nextState 挂载到对应键名
      nextState[key] = nextStateForKey

      // 引用对比，看是否发生改变，所以值改变后 reducer 要返回一个新对象的原因
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    return hasChanged ? nextState : state
  }
}
```

返回一个`combination`函数，就相当于一个总的reducer，每次`action`都会走到`combination`中，它会遍历每一个`reducer`，将`action`
放到每个`reducer`中去执行一下，计算出返回结果就是`nextState`，然后与前一次的 `state` 进行对比，看看是否发生改变


## 拆分reducer
这样，我们就可以对原先比较臃肿的`reducer`进行一个拆分，拆分方案如下:

### 建立一个index.js，导入相关reducer
```javascript
import import { combineReducers } from 'redux'
import counter from './counter'
import todos from './todos'

export default combineReducers({
  counter,    // 键名就是reducer 对应管理的state
  todos
})
```

### 简化原先的reducer
```javascript
// counter.js
export default function counter(counter = 0, action) {
  switch(action.type) {
    case 'INCREMENT':
      return counter + 1    // 值传递，可以直接返回
    default:
      return counter
  }
}

// todos.js
export default function todos(todos = [], action) {
  switch(action.type) {
    case 'ADD_TODOS':
      return [ ...todos, action.payload ]
    default:
      return todos
  }
}
```

这样，无论你的应用状态树多么复杂，都可以分而治之，非常便于管理
